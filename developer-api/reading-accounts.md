# Reading Accounts and Balances

Three read-only endpoints describe the account behind your API key: its profile, its balances, and the agent accounts it owns. Because Veiled cannot read encrypted balances, the balance endpoint only returns amounts when you supply a proof you generated yourself.

| Method | Path | SDK method |
|---|---|---|
| `GET` | `/v1/accounts/me` | `accounts.me()` |
| `GET` | `/v1/accounts/me/balances` | `accounts.balances()` |
| `GET` | `/v1/accounts/me/agents` | `accounts.listAgents()` |

---

## The authenticated account

```
GET /v1/accounts/me
```

Returns the account that owns the API key.

```bash
curl https://api.useveiled.com/v1/accounts/me \
  -H "Authorization: Bearer vl_live_Qm7tY2pXc4Rk9WdA0eLs"
```

```json
{
  "account_id": "acct_51c0e7",
  "handle": "@harbourlabs",
  "type": "business",
  "public_key": "5HqvTkR2mWb8yNc3LpXe9ZfJ4sGdA7oVu1KtBnQr6Ywz",
  "kyc_status": "enhanced",
  "created_at": "2026-03-08T16:42:15Z"
}
```

| Field | Type | What it holds |
|---|---|---|
| `account_id` | `string` | Veiled's internal ID for the account |
| `handle` | `string` | Public name of the account, written as `@handle` |
| `type` | `string` | Account kind: `personal`, `business` or `agent` |
| `public_key` | `string` | Base58 Solana address. For a Veiled smart account this is a program derived address computed from the handle, so it never changes. |
| `kyc_status` | `string` | Verification state: `unverified`, `pending`, `verified` or `enhanced` |
| `created_at` | `string` | Creation time, ISO 8601 |

With the SDK:

```ts
const account = await veiled.accounts.me();
if (account.kycStatus === "unverified") {
  console.warn(`${account.handle} still needs identity verification`);
}
```

---

## Balances

```
GET /v1/accounts/me/balances
```

Balances on Veiled are ciphertexts on-chain, and the servers have no key that opens them. That is guaranteed by the cryptography, not by a policy you are asked to trust. To see amounts, generate a decryption proof on your side with your account key and pass it in the `decryption_proof` query parameter.

| Query parameter | Type | Required | Meaning |
|---|---|---|---|
| `decryption_proof` | `string` | No | Proof generated client-side that lets the response reveal decrypted amounts |

```bash
curl -G https://api.useveiled.com/v1/accounts/me/balances \
  -H "Authorization: Bearer vl_live_Qm7tY2pXc4Rk9WdA0eLs" \
  --data-urlencode "decryption_proof=<proof generated on your machine>"
```

With a valid proof, each held asset comes back with its amount as a decimal string:

```json
{
  "balances": [
    { "asset": "USDG", "amount": "38210.75" },
    { "asset": "USDC", "amount": "1500.00" },
    { "asset": "SOL", "amount": "2.3150" }
  ]
}
```

| Field | Type | What it holds |
|---|---|---|
| `balances[].asset` | `string` | Asset symbol, such as `USDG`, `USDC` or `SOL` |
| `balances[].amount` | `string` | Decrypted balance as a decimal string |

Leave `decryption_proof` out and the response only tells you which assets hold a non-zero balance. No amounts are included, since Veiled has no way to read the encrypted values.

```ts
// With a proof: real amounts
const { balances } = await veiled.accounts.balances({ decryptionProof });

// Without one: only which assets are non-zero
const holdings = await veiled.accounts.balances();
```

---

## Agent accounts you own

```
GET /v1/accounts/me/agents
```

Returns all the agent accounts that sit under the authenticated account, so you can see the whole fleet with one request.

```json
{
  "agents": [
    {
      "account_id": "acct_a4f6d2",
      "handle": "@harbourlabs/invoice-runner",
      "public_key": "Gx3PdN8vLk2RtQw7ZbYc5HmE1sJaU9oKf4TnVq6WrBe2",
      "status": "active",
      "spend_policy_id": "policy_8820"
    },
    {
      "account_id": "acct_b07e19",
      "handle": "@harbourlabs/data-buyer",
      "public_key": "3TkWq9ZbLm5XcRv2NpYe8HsGd4JaF7oUt1KyBn6QrPw",
      "status": "paused",
      "spend_policy_id": "policy_8821"
    }
  ]
}
```

| Field | Type | What it holds |
|---|---|---|
| `account_id` | `string` | The agent account's ID |
| `handle` | `string` | The agent's handle, namespaced under yours (`@owner/agent-name`) |
| `public_key` | `string` | Base58 address of the agent on Solana |
| `status` | `string` | `active`, `paused` or `disabled` |
| `spend_policy_id` | `string` | ID of the spend policy attached to the agent |

```ts
const { agents } = await veiled.accounts.listAgents();
const running = agents.filter((agent) => agent.status === "active");
```

To create agents or change their limits, see [Managing Agents and Policies](managing-agents.md).

---

## Errors

| Status | Code | Cause |
|---|---|---|
| `401` | `unauthorized` | The API key is missing or not valid |
| `403` | `kyc_required` | The account has not finished the identity verification this request needs |
| `422` | `decryption_proof_invalid` | The proof you sent was not produced from this account's encrypted balance |

Error bodies and the SDK error classes are described in [Making Authenticated Requests](making-requests.md#errors).
