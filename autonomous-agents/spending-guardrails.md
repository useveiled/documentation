# Spending Guardrails

Every agent runs under a spend policy: a set of rules its parent account writes and only its parent account can change. The policy is what lets you give an agent real money to work with while you decide exactly how far that money can go.

---

## The rules you can set

| Rule | API field | What it controls | Enforced by the Veiled program? |
|---|---|---|---|
| Daily limit | `daily_limit_usdg` | Total the agent may spend across a rolling 24-hour window | Yes |
| Per-transaction limit | `per_transaction_limit_usdg` | Ceiling on any single payment | Yes |
| Allowed recipients | `allowed_recipients` | Who the agent may pay: addresses, handles, or domains for x402 services, with wildcards such as `*.anthropic.com` | Yes, for Solana addresses (up to 10 per agent) |
| Assets | `assets` | Which assets the agent may spend. `USDG` is the default; `USDC` and `SOL` can be switched on | Yes: each agent's vault is bound to a single settlement mint |
| Active hours | `active_hours` | The time window in which the agent may transact at all, such as `"00:00-23:59"` | No: checked by the agent's signing flow |
| Approval threshold | `hitl_threshold_usdg` | The amount above which a person has to sign off before the payment executes | Yes |

The SDK exposes the same fields in camelCase (`dailyLimitUsdg`, `perTransactionLimitUsdg`, `allowedRecipients`, `assets`, `activeHours`, `hitlThresholdUsdg`) and converts them for you.

The program also refuses a policy that doesn't make sense. Both limits must be above zero, the per-transaction limit can't exceed the daily limit, the approval threshold must be above zero, the allowlist holds at most 10 addresses, and an allowlist that is switched on can't be empty. (To stop an agent spending anywhere, pause it instead.)

---

## Two layers of enforcement

A proposed payment is checked twice.

1. **In the agent's signing flow.** Before a transaction is even built, the agent compares it with its current policy. Anything that fails is never constructed or sent.
2. **On Solana.** The Veiled program validates each spend instruction against the policy stored in the agent's account before any tokens leave the vault.

The second layer is the guarantee. Limits there are program constraints, not server-side settings, so a compromised client or a bug in the agent's own reasoning still cannot land a payment that breaks the per-transaction limit, the rolling daily limit, the allowlist, or the approval threshold. A rejected spend fails with a program error such as `PerTransactionLimitExceeded`, `DailyLimitExceeded`, `RecipientNotAllowed`, or `AmountExceedsHitlThreshold`.

The daily window rolls forward on its own: the first spend that lands 24 hours or more after the current window began starts a fresh one.

---

## Changing a policy

You can edit a policy whenever you like:

- **In the app:** open **Agents → Agent Accounts**, pick the agent, and edit its policy.
- **Through the API:** send a partial update with `PATCH /v1/agents/{account_id}/spend-policy` (`agents.updateSpendPolicy` in the SDK). Only the fields you include change. Details are in [Managing Agents and Policies](../developer-api/managing-agents.md).

Updates are written to the agent's account on-chain and govern the next transaction it signs. Payments that already settled are never affected. The agent itself has no authority to loosen its own policy; only the owning profile can sign the change.

---

## Approving payments by hand

When a payment is over the approval threshold, the agent doesn't send it. It queues it, and you get a push notification showing the amount, the recipient, and any context the agent attached. You decide with a single tap.

| Outcome | What happens |
|---|---|
| Approved | The payment is signed and submitted straight away. The program re-checks it against the policy as it stands at that moment, including the daily total, in case anything changed while it waited. |
| Rejected | The queued payment is thrown away. Nothing was ever submitted, so no funds move and there is nothing to refund. |
| No response | The request expires after a configurable window (24 hours by default) and counts as rejected. |

From code, list the queue with `agents.listPendingTransactions()` (`GET /v1/agents/transactions/pending`) and resolve each item with `agents.approveTransaction()` or `agents.rejectTransaction()`.

### Choosing how much autonomy to give

On-chain, each agent also carries an autonomy tier that sets how the threshold applies:

| Tier | Behaviour |
|---|---|
| `Supervised` | Every payment is queued for approval, whatever the amount |
| `SemiAutonomous` | Payments at or below the threshold settle directly; anything strictly above it is queued |
| `FullyAutonomous` | Any payment the policy allows settles directly; the agent can still choose to queue one for a human |

The per-transaction limit, daily limit, and allowlist bind all three tiers. Pick `Supervised` for an agent whose every payment you want to see, or a semi-autonomous agent with a generous threshold for one you trust to handle routine, low-value spending on its own.

---

## Worked example

```json
{
  "daily_limit_usdg": 200.00,
  "per_transaction_limit_usdg": 40.00,
  "allowed_recipients": ["inference.example.ai", "*.anthropic.com"],
  "assets": ["USDG"],
  "active_hours": "06:00-22:00",
  "hitl_threshold_usdg": 10.00
}
```

With this policy, an agent can pay `inference.example.ai` or any `anthropic.com` domain up to $10 a time between 06:00 and 22:00 without asking. A $30 payment to one of those services goes into the approval queue. A $45 payment, a payment to a service that isn't on the list, or anything that would push the day's total past $200 is refused outright.

---

## Keep reading

- [Paying per Request with x402](paying-per-request.md)
- [Reacting to Events](reacting-to-events.md)
