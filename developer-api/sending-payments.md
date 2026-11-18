# Sending Payments

The transfers endpoints create payments and let you look them up afterwards. A transfer made through the API is confidential unless you say otherwise, exactly like one sent from the app, so there is no privacy flag you need to remember to set.

| Method | Path | SDK method |
|---|---|---|
| `POST` | `/v1/transfers` | `transfers.create()` |
| `GET` | `/v1/transfers/{transfer_id}` | `transfers.get()`, `transfers.waitForConfirmation()` |
| `GET` | `/v1/transfers` | `transfers.list()`, `transfers.iterate()` |

---

## Sending a new transfer

```
POST /v1/transfers
```

### Body

| Field | Type | Required | Meaning |
|---|---|---|---|
| `to` | `string` | Yes | Who receives the funds: either an `@handle` or a raw base58 address |
| `amount` | `string` | Yes | How much to send, as a decimal string |
| `asset` | `string` | No | Asset to send. `USDG` if omitted; `USDC` and `SOL` are also accepted. |
| `confidential` | `boolean` | No | `true` if omitted. Only set `false` for a recipient wallet without Token-2022 confidential balance support. |
| `memo` | `string` | No | An encrypted note that only the sender and recipient can read |

### Headers

| Header | Required | Meaning |
|---|---|---|
| `Authorization` | Yes | `Bearer <api_key>` |
| `Content-Type` | Yes | `application/json` |
| `Idempotency-Key` | Recommended | A unique value per payment attempt. See [Idempotency](#idempotency). |

You never need SOL in your account to send a transfer. Network fees come out of a small SOL reserve that is managed for you automatically.

### Worked request

```bash
curl -X POST https://api.useveiled.com/v1/transfers \
  -H "Authorization: Bearer vl_live_Qm7tY2pXc4Rk9WdA0eLs" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: payout-2026-11-0193" \
  -d '{
    "to": "@northwind",
    "amount": "2340.50",
    "asset": "USDG",
    "memo": "November retainer"
  }'
```

```json
{
  "transfer_id": "tr_c84e09b1",
  "status": "confirmed",
  "to": "@northwind",
  "confidential": true,
  "tx_signature": "5yGk8RtWq2NvLb7XcPm3ZaJd9HsEf4UoK1TnYw6VrQe8",
  "created_at": "2026-11-20T09:15:42Z"
}
```

`tx_signature` is the base58 signature of the Solana transaction. You will notice the amount is missing from the response, even though your own request contained it. That is deliberate and has no exceptions: the API confirms that the payment settled, and if you want the figure back you decrypt it on your side, as with any confidential balance.

The same payment with the SDK:

```ts
const transfer = await veiled.transfers.create(
  {
    to: "@northwind",
    amount: "2340.50",
    asset: "USDG",
    memo: "November retainer",
  },
  { idempotencyKey: "payout-2026-11-0193" },
);

console.log(transfer.transferId, transfer.status);
```

### Idempotency

The `Idempotency-Key` header makes a payment request safe to resend. Requests that share a key produce one transfer at most, so retrying after a timeout or a dropped connection cannot send the money twice.

The SDK puts this header on every `transfers.create` call. If you pass no `idempotencyKey`, it generates a random one for that call, which protects its own automatic retries. Pass your own value (an invoice number, a job ID) when you also need protection across process restarts or redeploys, because a freshly generated key would not match the earlier attempt.

---

## Look up a transfer

```
GET /v1/transfers/{transfer_id}
```

```json
{
  "transfer_id": "tr_c84e09b1",
  "status": "confirmed",
  "from": "@harbourlabs",
  "to": "@northwind",
  "confidential": true,
  "tx_signature": "5yGk8RtWq2NvLb7XcPm3ZaJd9HsEf4UoK1TnYw6VrQe8",
  "created_at": "2026-11-20T09:15:42Z",
  "confirmed_at": "2026-11-20T09:15:43Z"
}
```

### Transfer fields

| Field | Type | What it holds |
|---|---|---|
| `transfer_id` | `string` | The transfer's ID |
| `status` | `string` | `pending`, `confirmed` or `failed` (see below) |
| `from` | `string` | The sender. May be absent, as in the create response. |
| `to` | `string` | The recipient |
| `confidential` | `boolean` | Whether the amount is encrypted |
| `tx_signature` | `string` | Base58 Solana transaction signature |
| `created_at` | `string` | When the transfer was created, ISO 8601 |
| `confirmed_at` | `string` | When it settled, ISO 8601. Present once settled. |

### Statuses

| Value | What it means |
|---|---|
| `pending` | Broadcast, with Solana confirmation still outstanding. With slots of about 400 ms, that usually takes one or two slots. |
| `confirmed` | Settled on-chain |
| `failed` | Never settled, and no funds moved |

### Waiting for settlement

`transfers.waitForConfirmation()` keeps calling `GET /v1/transfers/{transfer_id}` until the status is no longer `pending`, then hands back the transfer. Since settlement is normally sub-second, one poll is usually enough.

| Option | Default | Meaning |
|---|---|---|
| `pollIntervalMs` | `1000` | Gap between polls, in milliseconds |
| `maxWaitMs` | `60000` | Total time to wait before giving up, in milliseconds |
| `timeoutMs` | client setting | Timeout for each poll request |
| `signal` | (none) | `AbortSignal` that stops the wait |

The method resolves for `confirmed` and for `failed`, so check `status` on what it returns. It throws a `VeiledError` if the transfer is still `pending` after `maxWaitMs` (it may still settle later) or if the signal aborts.

```ts
const sent = await veiled.transfers.create({ to: "@northwind", amount: "75.00" });
const result = await veiled.transfers.waitForConfirmation(sent.transferId, {
  maxWaitMs: 15_000,
});

if (result.status === "failed") {
  // nothing moved; safe to try again
}
```

---

## Paging through past transfers

```
GET /v1/transfers
```

Transfers come back newest first, wrapped in a `transfers` array of the objects described above.

| Query parameter | Type | Default | Meaning |
|---|---|---|---|
| `limit` | `integer` | `20` | Page size, up to `100` |
| `offset` | `integer` | `0` | Number of results to skip |
| `status` | `string` | (none) | Only return transfers with this status |
| `from` | `string` | (none) | ISO 8601 timestamp: only transfers after it |
| `to` | `string` | (none) | ISO 8601 timestamp: only transfers before it |

```bash
curl -G https://api.useveiled.com/v1/transfers \
  -H "Authorization: Bearer vl_live_Qm7tY2pXc4Rk9WdA0eLs" \
  -d status=failed \
  -d from=2026-11-01T00:00:00Z \
  -d limit=50
```

```json
{
  "transfers": [
    {
      "transfer_id": "tr_9a1f33d0",
      "status": "failed",
      "from": "@harbourlabs",
      "to": "@quillprint",
      "confidential": true,
      "tx_signature": "2bNc7QwLk4RtXv9ZmYp3HdJs8GaE5oUf1KyTn6WrVq3P",
      "created_at": "2026-11-14T17:02:11Z"
    }
  ]
}
```

### Walking every page

`transfers.iterate()` takes the same filters except `offset` and fetches pages as you consume them. It requests `100` per page unless you set `limit`, stops when a page comes back short, and makes no further requests if you `break` out of the loop.

```ts
let failedCount = 0;
for await (const transfer of veiled.transfers.iterate({
  status: "failed",
  from: "2026-11-01T00:00:00Z",
})) {
  failedCount += 1;
}
```

---

## Errors

| Status | Code | Cause |
|---|---|---|
| `400` | `invalid_request` | The body is malformed or a required field is missing |
| `402` | `insufficient_balance` | Balance in that asset is below the amount you asked to send |
| `404` | `recipient_not_found` | That handle doesn't belong to any Veiled account |
| `422` | `recipient_not_confidential_ready` | Confidential transfers haven't been enabled on the payee's token account. To go ahead anyway as a standard SPL transfer, resend with `"confidential": false`. |

For the error body format and SDK error classes, see [Making Authenticated Requests](making-requests.md#errors).
