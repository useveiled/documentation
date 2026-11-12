# Agent Wallets

On Veiled, an AI agent is an account holder in its own right. You don't hand it a copy of your API key and hope for the best. It gets a wallet, an identity, and a ledger of its own, and it answers to a policy you write. The platform was designed around agents from the start rather than adapted for them later.

---

## What every agent receives

| Component | What it is |
|---|---|
| Smart account | A program-derived address (PDA) on Solana, owned by the Veiled program and tied to your profile |
| Vault | A token account controlled by that PDA, holding the funds the agent is allowed to spend |
| Signing key | The agent's own `agent_signer` key, separate from the key that controls your account |
| Identity | Its own `.veiled` handle, shown in the app as something like `research-bot.veiled` and referenced in API events under its parent, for example `@yourname/research-bot` |
| History | Its own transaction record, which also streams into your account's activity feed as it happens |

The agent's key can only move what is already sitting in its vault, and only within the policy you configured. The agent cannot pull money from your account, and it has no authority to change its own limits. Only the owning profile can sign a policy update.

---

## From creation to settlement

```
You create the agent account
        ↓
You set its spend policy (limits, approved recipients, active hours, approval threshold)
        ↓
You fund its vault
        ↓
The agent works on a task and decides a payment is needed
        ↓
The policy is checked before anything is signed
        ↓
Inside the policy: the agent signs and submits the payment itself
        ↓
Over the approval threshold: the payment is queued and you get a push notification to approve or reject
        ↓
The payment settles on Solana (usually within a slot or two), a webhook fires, and your feed updates
```

The check that matters most happens on-chain. Each spend instruction from an agent is validated by the Veiled program against the policy stored in the agent's account before any tokens move. A payment that breaks the per-transaction limit, the rolling daily limit, the recipient allowlist, or the approval threshold fails at the program level. It is not flagged after the fact; it simply cannot land. [Spending Guardrails](spending-guardrails.md) covers each rule in detail.

---

## Keeping control of a running agent

You keep several levers on an agent after it goes live. Each maps to an instruction in the Veiled program:

| Action | Program instruction | Effect |
|---|---|---|
| Top up | `fund_agent` | Deposits the agent's settlement token into its vault |
| Draw down | `withdraw_from_agent` | Returns part of the vault balance to you while the agent keeps running |
| Pause or resume | `set_agent_status` | Stops (or restarts) all spending without touching the vault or policy |
| Replace its key | `rotate_agent_signer` | Swaps the agent's signing key, keeping its vault, policy, and history |
| Shut it down | `revoke_agent` | Permanently revokes the agent and sweeps whatever is left in the vault back to you |

If an agent's key leaks, pausing or revoking the agent neutralises it. Revocation and withdrawals stay available even while the protocol is paused, so you always have a way out.

---

## Where agents earn their keep

- **Coding assistants** that pay for model API calls as they make them, with no prepaid credit pool to manage.
- **Trading agents** that run swaps inside a fixed risk budget.
- **Agentic SaaS products** that bill each sub-task to the customer almost as soon as it finishes.
- **Developer tools** that meter API usage straight to the agent consuming it, with no separate billing system in between.

---

## Agents and privacy

An agent's spending pattern is sensitive. Published openly, it would reveal how much a business uses a service, what that usage costs, and what it is working on.

Be aware of how this plays out on-chain today. To enforce a limit, the program has to compare the amount against it, so spends that settle from an agent's vault carry the amount in the instruction and in the program's `AgentPaymentExecuted` event. The agent's public address and the fact that it transacted are also visible, as they are for every Solana account. Confidential, amount-hiding transfers are described in [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).

---

## Keep reading

- [Spending Guardrails](spending-guardrails.md)
- [Paying per Request with x402](paying-per-request.md)
- [Reacting to Events](reacting-to-events.md)
- [Managing Agents and Policies](../developer-api/managing-agents.md)
