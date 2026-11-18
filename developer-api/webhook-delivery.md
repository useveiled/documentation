# Webhook Delivery and Signatures

Webhooks push events to your server as they happen, so you learn about a settled payment without polling for it. This page covers the endpoints that manage subscriptions and explains how to check that a delivery really came from Veiled. For what each event means and when it fires, see [Reacting to Events](../autonomous-agents/reacting-to-events.md).

| Method | Path | SDK method |
|---|---|---|
| `POST` | `/v1/webhooks` | `webhooks.create()` |
| `GET` | `/v1/webhooks` | `webhooks.list()` |
| `DELETE` | `/v1/webhooks/{webhook_id}` | `webhooks.delete()` |
| `POST` | `/v1/webhooks/{webhook_id}/replay/{event_id}` | `webhooks.replay()` |
| `GET` | `/v1/webhooks/{webhook_id}/deliveries` | `webhooks.deliveries()` |

---

## Subscribe

```
POST /v1/webhooks
```

| Field | Type | Required | Meaning |
|---|---|---|---|
| `url` | `string` | Yes | The URL of the endpoint that will receive deliveries |
| `events` | `string[]` | Yes | Which events to send there |

These are the event names you can subscribe to:

| Event | Fires when |
|---|---|
| `transfer.initiated` | A transfer has been submitted |
| `transfer.confirmed` | A transfer has settled on-chain |
| `transfer.failed` | Sent when the payment ended without moving funds |
| `agent.transaction.pending_approval` | An agent transaction is being held for a human |
| `agent.transaction.approved` | A held agent transaction was approved |
| `agent.transaction.rejected` | A held agent transaction was rejected |
| `agent.policy.updated` | An agent's spend policy was changed |

```bash
curl -X POST https://api.useveiled.com/v1/webhooks \
  -H "Authorization: Bearer vl_live_Qm7tY2pXc4Rk9WdA0eLs" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://ops.harbourlabs.dev/webhooks/veiled",
    "events": ["transfer.confirmed", "transfer.failed", "agent.transaction.pending_approval"]
  }'
```

```json
{
  "webhook_id": "wh_7d40be",
  "url": "https://ops.harbourlabs.dev/webhooks/veiled",
  "events": ["transfer.confirmed", "transfer.failed", "agent.transaction.pending_approval"],
  "secret": "<signing secret, returned only here>",
  "created_at": "2026-11-19T11:05:27Z"
}
```

The `secret` appears in this response and nowhere else, so save it now in your secrets manager. You will need it to check the `X-Veiled-Signature` header on every delivery.

---

## List subscriptions

```
GET /v1/webhooks
```

Returns your subscriptions in a `webhooks` array. Each entry has `webhook_id`, `url`, `events` and `created_at`. The signing secret is never included.

```json
{
  "webhooks": [
    {
      "webhook_id": "wh_7d40be",
      "url": "https://ops.harbourlabs.dev/webhooks/veiled",
      "events": ["transfer.confirmed", "transfer.failed", "agent.transaction.pending_approval"],
      "created_at": "2026-11-19T11:05:27Z"
    }
  ]
}
```

---

## Unsubscribe

```
DELETE /v1/webhooks/{webhook_id}
```

The subscription is removed the moment the request succeeds, and Veiled will not attempt any more deliveries to it.

---

## What a delivery contains

Each delivery is a `POST` to your URL with a JSON body and an `X-Veiled-Signature` header.

```json
{
  "event": "transfer.confirmed",
  "account": "@harbourlabs",
  "direction": "received",
  "counterparty": "@quillprint",
  "asset": "USDG",
  "confidential": true,
  "tx_signature": "4mTq8WbLk3RvXc9ZnYp2HdJs7GaE6oUf1KyTn5WrVq2N",
  "timestamp": "2026-11-20T15:31:08Z"
}
```

| Field | Type | Present | What it holds |
|---|---|---|---|
| `event` | `string` | Always | The event name |
| `account` | `string` | Always | The account the event is about, for example `@harbourlabs/invoice-runner` for an agent |
| `direction` | `string` | Sometimes | `sent` or `received` |
| `counterparty` | `string` | Sometimes | The other side of the payment |
| `asset` | `string` | Sometimes | Asset involved |
| `confidential` | `boolean` | Sometimes | Whether the amount is encrypted |
| `tx_signature` | `string` | Sometimes | Base58 Solana transaction signature |
| `timestamp` | `string` | Always | When the event happened, ISO 8601 |

Payloads never contain an amount. Amounts are encrypted on-chain and Veiled cannot read them, so if your handler needs the number it has to decrypt it locally with your own key.

---

## Verify the signature

`X-Veiled-Signature` holds an HMAC-SHA256 of the raw request body, keyed with your subscription's secret and written as a hex string. Check it before you act on anything in the payload, and turn away any delivery that fails.

Two details trip people up:

- **Hash the body exactly as it arrived.** If your framework has already parsed the JSON and you serialise it again, the bytes change and the HMAC will not match.
- **Compare in constant time**, so the check does not leak the expected value through timing.

### With the SDK

`webhooks.constructEvent()` verifies the header and parses the body in a single call. It throws `VeiledWebhookVerificationError` when the signature is wrong or the body is not valid JSON, and otherwise returns the payload with camelCase field names.

```ts
import express from "express";
import { Veiled, VeiledWebhookVerificationError } from "@useveiled/sdk";

const veiled = new Veiled({ apiKey: process.env.VEILED_API_KEY! });
const app = express();

app.post(
  "/webhooks/veiled",
  express.raw({ type: "application/json" }), // keep the body as raw bytes
  async (req, res) => {
    try {
      const event = await veiled.webhooks.constructEvent({
        payload: new Uint8Array(req.body),
        signature: req.header("x-veiled-signature") ?? "",
        secret: process.env.VEILED_WEBHOOK_SECRET!,
      });

      if (event.event === "transfer.failed") {
        await flagForFollowUp(event.txSignature);
      }
      res.sendStatus(200);
    } catch (err) {
      if (err instanceof VeiledWebhookVerificationError) {
        res.sendStatus(400); // not from Veiled; ignore it
        return;
      }
      throw err;
    }
  },
);
```

If you only want a yes or no, `webhooks.verifySignature({ payload, signature, secret })` returns a boolean and does not parse anything. The same function is exported on its own as `verifyWebhookSignature`. Both need the Web Crypto API, which Node 18+, current browsers and edge runtimes provide.

### Without the SDK

The check is short in any language. The SDK trims whitespace from the header and lowercases it before comparing, and this Python version does the same:

```python
import hashlib
import hmac


def is_from_veiled(raw_body: bytes, header_value: str, signing_secret: str) -> bool:
    digest = hmac.new(signing_secret.encode(), raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(digest, header_value.strip().lower())
```

---

## Recover a missed event

```
POST /v1/webhooks/{webhook_id}/replay/{event_id}
```

Sends one past event to the subscription's endpoint again. This is handy when your server was down for a while and you want a particular event now instead of waiting for automatic retries. Look up the `event_id` in the delivery log.

```ts
await veiled.webhooks.replay("wh_7d40be", "evt_3b91c6");
```

---

## Inspecting past deliveries

```
GET /v1/webhooks/{webhook_id}/deliveries
```

Shows recent delivery attempts for one subscription, which is usually the first place to look when an integration stops receiving events.

```json
{
  "deliveries": [
    {
      "event_id": "evt_3b91c6",
      "event": "transfer.confirmed",
      "response_status": 0,
      "attempts": 4,
      "last_attempt_at": "2026-11-20T15:47:12Z"
    },
    {
      "event_id": "evt_3b90a2",
      "event": "agent.transaction.pending_approval",
      "response_status": 200,
      "attempts": 1,
      "last_attempt_at": "2026-11-20T14:02:55Z"
    }
  ]
}
```

| Field | Type | What it holds |
|---|---|---|
| `event_id` | `string` | The event's ID, which you can pass to the replay endpoint |
| `event` | `string` | The event name |
| `response_status` | `integer` | The HTTP status your endpoint returned, or `0` if it could not be reached |
| `attempts` | `integer` | How many times delivery has been tried, counting the first |
| `last_attempt_at` | `string` | Time of the latest attempt, ISO 8601 |

```ts
const { deliveries } = await veiled.webhooks.deliveries("wh_7d40be");
const unreachable = deliveries.filter((d) => d.responseStatus === 0);
for (const d of unreachable) {
  await veiled.webhooks.replay("wh_7d40be", d.eventId);
}
```
