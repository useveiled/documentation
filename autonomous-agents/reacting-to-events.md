# Reacting to Events

Whenever money moves through your account or any agent under it, Veiled sends an event. Point those events at your own backend, dashboard, or monitoring stack and it knows about each payment as it happens, with no polling loop to maintain.

---

## Subscribing

You can add a webhook in the app under **Developer → API Keys**, or from code. Either way the setup is the same:

1. Give Veiled an HTTPS URL that accepts `POST` requests.
2. Choose the event types you want delivered.
3. Save the signing secret. It is displayed once, when the webhook is created, and never again.

Each delivery carries an `X-Veiled-Signature` header: a hex-encoded HMAC-SHA256 of the raw request body, keyed with that secret. Check it on every request before you trust the contents.

```ts
import { Veiled } from "@useveiled/sdk";

const veiled = new Veiled({ apiKey: process.env.VEILED_API_KEY! });

const hook = await veiled.webhooks.create({
  url: "https://ops.example.com/veiled/events",
  events: ["transfer.confirmed", "agent.transaction.pending_approval"],
});
// Store hook.secret somewhere safe now; the API will not return it again.

// Later, inside your HTTP handler:
const evt = await veiled.webhooks.constructEvent({
  payload: rawBody, // the body exactly as it arrived, not re-serialized JSON
  signature: req.headers["x-veiled-signature"],
  secret: process.env.VEILED_WEBHOOK_SECRET!,
});
```

`constructEvent` throws `VeiledWebhookVerificationError` if the signature doesn't match, so a forged request never reaches your logic. If you only want a yes or no, `veiled.webhooks.verifySignature()` returns a boolean instead. The full delivery and signature reference lives in [Webhook Delivery and Signatures](../developer-api/webhook-delivery.md).

---

## Events you can subscribe to

| Event | Sent when |
|---|---|
| `transfer.initiated` | Fires once Veiled has broadcast the payment and before Solana finalizes it |
| `transfer.confirmed` | Fires when the payment lands on Solana, normally within one or two slots |
| `transfer.failed` | Fires when the payment could not complete and no funds moved |
| `agent.transaction.pending_approval` | An agent payment went over its approval threshold and is waiting for you |
| `agent.transaction.approved` | Someone approved a queued agent payment |
| `agent.transaction.rejected` | Someone rejected a queued agent payment |
| `agent.policy.updated` | Someone edited the limits or rules on an agent you own |

---

## What a delivery looks like

```json
{
  "event": "transfer.confirmed",
  "account": "@yourname/billing-agent",
  "direction": "sent",
  "counterparty": "inference.example.ai",
  "asset": "USDC",
  "confidential": true,
  "tx_signature": "5hXq2mWcR8pLzTn4vKd9aBe...Yr3sQ",
  "timestamp": "2026-09-14T08:41:52Z"
}
```

| Field | Meaning |
|---|---|
| `event` | One of the event types above |
| `account` | The account the event is about, such as an agent under your profile |
| `direction` | `sent` or `received` |
| `counterparty` | The other side of the payment |
| `asset` | The asset that moved |
| `confidential` | Whether the transfer used confidential balances |
| `tx_signature` | The base58 Solana transaction signature, which you can paste straight into Solscan or Solana Explorer |
| `timestamp` | When the event happened, in ISO 8601 |

There is no amount field, and that is deliberate. On a confidential transfer the amount is encrypted on-chain and Veiled's backend does not hold your decryption key, so it has nothing to put there. If your integration needs the figure, decrypt it on your side with your own key, which is exactly how the app shows amounts in your feed.

---

## Failed deliveries

If your endpoint answers with anything outside the `2xx` range, Veiled tries again on a growing delay and keeps going for a full day. Every delivery attempt appears in the webhook log, and you can resend any event you missed, either from the dashboard or with `veiled.webhooks.replay(webhookId, eventId)`. A short outage on your side doesn't mean a lost event.

---

## Keep reading

- [Agent Wallets](agent-wallets.md)
- [Webhook Delivery and Signatures](../developer-api/webhook-delivery.md)
