# Making Authenticated Requests

Everything the Veiled app can do with accounts, payments, agents and webhooks is also available over a REST API at `https://api.useveiled.com`. Access is controlled by one thing: an API key belonging to a Veiled account, sent as a bearer token.

This page covers getting a key, attaching it to requests, choosing between live and test environments, and how errors and retries behave.

---

## Create a key in the dashboard

1. Sign in to Veiled.
2. In the dashboard, go to the **Developer** area and choose **API Keys**.
3. Choose **Generate**, give the key a name you will recognise later (say `billing-worker` or `staging`), and pick its environment.
4. Copy the key straight away. It is displayed this one time and cannot be retrieved afterwards.

A key acts on behalf of the account that generated it, and nothing else. Payments you make through the API get exactly the same confidentiality as payments made in the app: calling from code does not weaken any of the privacy guarantees.

The **API Keys** screen lists each key with its name, whether it is active or revoked, when it was created and when it was last used. You can revoke any key from there, and the revocation takes effect immediately. See [Developer Settings](../the-veiled-app/developer-settings.md) for a tour of the screen.

---

## Send the key as a bearer token

Each call authenticates through the `Authorization` header, set like this:

```
Authorization: Bearer <api_key>
```

Request and response bodies are JSON with snake_case field names. Here is a call that reads the account the key belongs to:

```bash
curl https://api.useveiled.com/v1/accounts/me \
  -H "Authorization: Bearer vl_test_5Rk2wQe8LmZt0aVb9cXy" \
  -H "Accept: application/json"
```

### Using the TypeScript SDK

The official SDK, `@useveiled/sdk`, attaches the header for you. It needs Node 18 or later (for global `fetch` and Web Crypto) and also runs in browsers and edge runtimes, though a live key should never be shipped to a browser.

```bash
npm install @useveiled/sdk
```

```ts
import { Veiled } from "@useveiled/sdk";

const veiled = new Veiled({ apiKey: process.env.VEILED_API_KEY! });

const me = await veiled.accounts.me();
console.log(me.handle, veiled.environment);
```

The SDK exposes camelCase names (`accountId`, `txSignature`) and converts them to and from the API's snake_case automatically. Each request it sends also carries `Accept: application/json` and a `User-Agent` of the form `veiled-sdk/<version>`.

---

## Live and test environments

The key's prefix decides which Solana cluster a request runs against. There is no separate mode switch to set.

| Prefix | Cluster | What happens |
|---|---|---|
| `vl_live_` | Solana mainnet-beta | Real on-chain transactions with real USDG. Use in production. |
| `vl_test_` | Solana devnet | Runs on devnet only. No real USDG is needed or moved. |

Build and test your integration with a `vl_test_` key; requests made with it never touch mainnet-beta. The SDK reads the prefix too and reports it on the client:

```ts
const sandbox = new Veiled({ apiKey: "vl_test_5Rk2wQe8LmZt0aVb9cXy" });
sandbox.environment; // "test"
```

You do not have to fund network fees on either cluster. They are paid in SOL out of a small reserve that Veiled tops up automatically.

---

## Client options

| Option | Default | Meaning |
|---|---|---|
| `apiKey` | (required) | Your `vl_live_` or `vl_test_` key. The constructor throws a `VeiledError` if it is missing. |
| `baseUrl` | `https://api.useveiled.com` | Point the client at a different host, such as a private gateway. |
| `timeoutMs` | `30000` | Timeout for each individual request, in milliseconds. |
| `maxRetries` | `2` | Extra attempts for transient failures. `0` turns retries off. |
| `fetch` | global `fetch` | Supply your own `fetch` implementation. |

Every SDK method also takes an optional final argument with a `signal` (an `AbortSignal`) and a `timeoutMs` that overrides the client setting for that call only. Aborting the signal cancels the request along with any retries still waiting.

```ts
const controller = new AbortController();
setTimeout(() => controller.abort(), 3_000);

const recent = await veiled.transfers.list(
  { limit: 10 },
  { signal: controller.signal, timeoutMs: 2_000 },
);
```

### Retries

The SDK retries a request after a network failure or a `408`, `429`, `500`, `502`, `503` or `504` response. It waits with exponential backoff and jitter (roughly 500 ms, then 1 s, then 2 s, never more than 8 s). If the response includes a `Retry-After` header, given in seconds or as an HTTP date, that delay is used instead, as long as it is 60 seconds or less.

Only requests that are safe to repeat get retried: `GET` and `DELETE` requests, plus `POST /v1/transfers`, which always carries an `Idempotency-Key`. See [Sending Payments](sending-payments.md#idempotency) for how that key works.

---

## Errors

A failed request returns a non-2xx status and a JSON body with a machine-readable `code` and a human-readable `message`:

```json
{
  "code": "unauthorized",
  "message": "API key is missing or invalid"
}
```

The response also carries an `x-request-id` header. Include it when you contact support.

### Error classes in the SDK

| Class | Thrown when | Useful properties |
|---|---|---|
| `VeiledAPIError` | The API answered with a non-2xx status | `status`, `code`, `body`, `requestId` |
| `VeiledConnectionError` | The request never got a response: network failure, timeout or abort | `cause` |
| `VeiledWebhookVerificationError` | `webhooks.constructEvent` rejected a delivery | (message only) |
| `VeiledError` | Base class of all of the above; catch it to handle any SDK failure | (message only) |

If an error body has no `code`, `VeiledAPIError.code` is set to `unknown_error`.

```ts
import { VeiledAPIError, VeiledConnectionError } from "@useveiled/sdk";

try {
  await veiled.transfers.create({ to: "@northwind", amount: "18000.00" });
} catch (err) {
  if (err instanceof VeiledAPIError && err.code === "insufficient_balance") {
    // err.status is 402; top up or lower the amount
  } else if (err instanceof VeiledConnectionError) {
    // nothing came back; check connectivity and try again
  } else {
    throw err;
  }
}
```

### Error codes

Each code is explained on the page for the endpoints that return it.

| Code | Status | Page |
|---|---|---|
| `unauthorized` | `401` | [Reading Accounts and Balances](reading-accounts.md) |
| `kyc_required` | `403` | [Reading Accounts and Balances](reading-accounts.md) |
| `decryption_proof_invalid` | `422` | [Reading Accounts and Balances](reading-accounts.md) |
| `invalid_request` | `400` | [Sending Payments](sending-payments.md) |
| `insufficient_balance` | `402` | [Sending Payments](sending-payments.md) |
| `recipient_not_found` | `404` | [Sending Payments](sending-payments.md) |
| `recipient_not_confidential_ready` | `422` | [Sending Payments](sending-payments.md) |
| `invalid_policy` | `400` | [Managing Agents and Policies](managing-agents.md) |
| `not_agent_owner` | `403` | [Managing Agents and Policies](managing-agents.md) |
| `transaction_not_found` | `404` | [Managing Agents and Policies](managing-agents.md) |
| `transaction_expired` | `409` | [Managing Agents and Policies](managing-agents.md) |

---

## Keeping keys safe

Anyone holding a Veiled API key can move money out of your account, so protect it as carefully as a private key.

- Read keys at runtime from a vault or from the process environment, and keep them out of version control.
- Give each environment (production, staging, development) its own key.
- If a key may have leaked, rotate it at once.
- Revoke keys you no longer use.

### The limit on what a key can reveal

No API key can get Veiled's servers to decrypt an amount for you. That is a property of how the system is built, not a rule someone enforces. Balances and transfer amounts are encrypted on-chain, and reading one means decrypting it on your side with your account's decryption key, which is exactly how the Veiled app does it. See [Keys, Signers, and Custody](../confidentiality/keys-and-custody.md) for more.
