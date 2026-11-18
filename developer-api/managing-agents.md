# Managing Agents and Policies

These endpoints create agent accounts, read and change their spend policies, and clear the payments that agents are not allowed to make without a human. Each agent gets its own PDA-based smart account, and the Veiled agent program enforces its policy on-chain, so the limits hold no matter what the agent or a server tries. For the concepts behind each policy field, read [Spending Guardrails](../autonomous-agents/spending-guardrails.md).

| Method | Path | SDK method |
|---|---|---|
| `POST` | `/v1/accounts/me/agents` | `agents.create()` |
| `GET` | `/v1/agents/{account_id}/spend-policy` | `agents.getSpendPolicy()` |
| `PATCH` | `/v1/agents/{account_id}/spend-policy` | `agents.updateSpendPolicy()` |
| `GET` | `/v1/agents/transactions/pending` | `agents.listPendingTransactions()` |
| `POST` | `/v1/agents/transactions/{transaction_id}/approve` | `agents.approveTransaction()` |
| `POST` | `/v1/agents/transactions/{transaction_id}/reject` | `agents.rejectTransaction()` |

To list the agents you already have, use `GET /v1/accounts/me/agents`, documented in [Reading Accounts and Balances](reading-accounts.md#agent-accounts-you-own).

---

## Spend policy fields

The same object is used when creating an agent, when reading its policy, and (partially) when updating it.

| Field | Type | Meaning |
|---|---|---|
| `daily_limit_usdg` | `number` | Most the agent may spend in any rolling 24 hours, in USDG |
| `per_transaction_limit_usdg` | `number` | Most the agent may spend in a single transaction, in USDG |
| `allowed_recipients` | `string[]` | Domains or handles the agent may pay. Wildcards work, for example `*.anthropic.com`. |
| `assets` | `string[]` | Assets the agent may spend: `USDG` (the default), `USDC`, `SOL` |
| `active_hours` | `string` | Time window when the agent may transact, written like `"08:00-20:00"` |
| `hitl_threshold_usdg` | `number` | Any transaction worth more than this (in USDG) waits for a human to approve it; amounts equal to it go through |

---

## Create an agent

```
POST /v1/accounts/me/agents
```

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | `string` | Yes | The agent's name. It becomes part of the agent's handle under yours. |
| `spend_policy` | `object` | Yes | A full spend policy, as described above |

```bash
curl -X POST https://api.useveiled.com/v1/accounts/me/agents \
  -H "Authorization: Bearer vl_live_Qm7tY2pXc4Rk9WdA0eLs" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "invoice-runner",
    "spend_policy": {
      "daily_limit_usdg": 1200.00,
      "per_transaction_limit_usdg": 150.00,
      "allowed_recipients": ["@northwind", "*.cloudvendor.io"],
      "assets": ["USDG", "USDC"],
      "active_hours": "08:00-20:00",
      "hitl_threshold_usdg": 100.00
    }
  }'
```

```json
{
  "account_id": "acct_a4f6d2",
  "handle": "@harbourlabs/invoice-runner",
  "public_key": "Gx3PdN8vLk2RtQw7ZbYc5HmE1sJaU9oKf4TnVq6WrBe2",
  "status": "active",
  "spend_policy_id": "policy_8820"
}
```

| Field | Type | What it holds |
|---|---|---|
| `account_id` | `string` | The new agent account's ID. Use it in the spend policy paths. |
| `handle` | `string` | The agent's handle, under the owner's |
| `public_key` | `string` | Base58 address of the agent on Solana. It is a program derived address computed from the agent handle, so the Veiled agent program holds authority over it |
| `status` | `string` | `active`, `paused` or `disabled` |
| `spend_policy_id` | `string` | ID of the policy now attached to the agent |

The SDK takes the same values in camelCase:

```ts
const agent = await veiled.agents.create({
  name: "invoice-runner",
  spendPolicy: {
    dailyLimitUsdg: 1200,
    perTransactionLimitUsdg: 150,
    allowedRecipients: ["@northwind", "*.cloudvendor.io"],
    assets: ["USDG", "USDC"],
    activeHours: "08:00-20:00",
    hitlThresholdUsdg: 100,
  },
});
```

---

## Read an agent's policy

```
GET /v1/agents/{account_id}/spend-policy
```

The response holds every spend policy field plus two more:

| Field | Type | What it holds |
|---|---|---|
| `policy_id` | `string` | The policy's ID |
| `updated_at` | `string` | When the policy last changed, ISO 8601 |

```json
{
  "policy_id": "policy_8820",
  "daily_limit_usdg": 1200.00,
  "per_transaction_limit_usdg": 150.00,
  "allowed_recipients": ["@northwind", "*.cloudvendor.io"],
  "assets": ["USDG", "USDC"],
  "active_hours": "08:00-20:00",
  "hitl_threshold_usdg": 100.00,
  "updated_at": "2026-11-18T13:27:05Z"
}
```

---

## Change an agent's policy

```
PATCH /v1/agents/{account_id}/spend-policy
```

Send just the fields you want to change. Anything you leave out keeps its current value. The response is the complete, updated policy in the same shape as the read endpoint.

```json
{
  "daily_limit_usdg": 2000.00,
  "active_hours": "00:00-23:59"
}
```

```ts
const policy = await veiled.agents.updateSpendPolicy(agent.accountId, {
  dailyLimitUsdg: 2000,
  activeHours: "00:00-23:59",
});
```

The change is written to the agent's policy account on-chain, which usually takes a slot or two. From then on it governs the next transaction the agent signs. Payments that already settled are not affected. You can loosen or tighten an agent's limits while it is running, without stopping it.

---

## Human approval

A transaction whose value is greater than the agent's `hitl_threshold_usdg` is not signed automatically. It is held until someone approves or rejects it, so the payments that matter always pass through a person.

### See what is waiting

```
GET /v1/agents/transactions/pending
```

Returns held transactions across all of your agents.

```json
{
  "pending": [
    {
      "transaction_id": "tx_e27b40",
      "agent": "@harbourlabs/invoice-runner",
      "to": "billing.cloudvendor.io",
      "requested_at": "2026-11-20T22:48:30Z",
      "expires_at": "2026-11-21T22:48:30Z"
    }
  ]
}
```

| Field | Type | What it holds |
|---|---|---|
| `transaction_id` | `string` | ID to pass to approve or reject |
| `agent` | `string` | Handle of the agent that asked to pay |
| `to` | `string` | The intended recipient |
| `requested_at` | `string` | When the agent made the request, ISO 8601 |
| `expires_at` | `string` | When the approval window closes, ISO 8601 |

There is no amount in this list. Like everywhere else in the API, amounts are encrypted and only your own client-side decryption can reveal them. The Veiled dashboard decrypts them locally when you open a pending approval for review.

### Approve or reject

```
POST /v1/agents/transactions/{transaction_id}/approve
POST /v1/agents/transactions/{transaction_id}/reject
```

Both calls take no body and return the transaction object in the same shape as the pending list entries. Approving signs and submits the transaction straight away. Rejecting throws it away, and nothing moves.

```ts
const { pending } = await veiled.agents.listPendingTransactions();

for (const held of pending) {
  if (held.to.endsWith(".cloudvendor.io")) {
    await veiled.agents.approveTransaction(held.transactionId);
  } else {
    await veiled.agents.rejectTransaction(held.transactionId);
  }
}
```

---

## Errors

| Status | Code | Cause |
|---|---|---|
| `400` | `invalid_policy` | The policy you sent contains an invalid value (a limit below zero, for instance) |
| `403` | `not_agent_owner` | The agent named in the path does not belong to the authenticated account |
| `404` | `transaction_not_found` | There is no pending transaction with that ID |
| `409` | `transaction_expired` | The transaction's approval window has already closed |

For the error body format and SDK error classes, see [Making Authenticated Requests](making-requests.md#errors).
