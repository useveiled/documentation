# Personal and Agent Accounts

Veiled has two kinds of holder: people and AI agents. Both sit on the same base, a self-custodial account on Solana with confidential transfers switched on from the start. What changes from one tier to the next is how deep the privacy goes, how much verification is needed, and what agents are allowed to do. Custody never changes. You hold the keys in every case.

---

## Account kinds at a glance

The protocol and the API each label accounts slightly differently:

| Where | Values |
|---|---|
| On-chain profile (`AccountKind`) | `Personal`, `Business`, `AgentOperator` |
| API and SDK (`AccountType`) | `personal`, `business`, `agent` |

Verification depth is tracked separately. On-chain, every profile starts as `Unverified`, and only the protocol's compliance authority can raise it to `Basic` or `Enhanced` (via `set_kyc_tier`) once an off-chain check clears. The API reports the same idea as `kycStatus`: `unverified`, `pending`, `verified` or `enhanced`. The tier records how well an account is verified; it never reveals amounts.

---

## Accounts for people

People and teams use this kind of account.

**What you can do with it**

- Balances in USDG, USDC and SOL, with every payment in or out kept confidential
- Reserve a `.veiled` handle so people can pay `maya.veiled` instead of a raw address
- Create payment request links and QR codes, with the option of keeping the requested amount confidential
- Export your decrypted transaction history for tax and bookkeeping
- Use a Squads multisig for social recovery and for rotating hardware keys

**How handles work on-chain.** `create_profile` registers one profile per wallet. The profile's address is a PDA derived from your wallet address, and the handle is reserved in a separate claim account derived from the handle text, which is how a handle resolves to its owner in a single lookup. Handles are 1 to 32 characters of lowercase letters, digits and underscores. The program also offers `change_handle`, which moves a profile to a new handle and frees the old one.

**Tiers**

| Tier | What it adds | Requirements |
|---|---|---|
| Shadow Lite (default) | Confidential amounts on every transfer, using Token-2022 confidential balances | Verified email plus a wallet attestation |
| Shadow Pro | Layer 3 shielded state: balances and history committed into a commitment-tree shielded pool, plus stealth meta-addresses so on-chain analysis cannot link payments to the recipient. Multisig is included. | Identity verification |

**Privacy at every tier**

- Every payment hides its amount unless you choose otherwise.
- Disclosure is entirely yours to control. Third parties learn only what a proof you created reveals to them.

---

## Accounts for AI agents (the Shadow Agent tier)

An agent account is a wallet built for software to hold instead of a person.

**How it is put together**

- `create_agent` creates the agent under your profile along with its spend policy and a program-controlled vault. The vault is a token account only the Veiled program can debit.
- The running agent signs with its own key (`agent_signer`). That key can only spend what is already in the vault, and only inside the policy. If it leaks, you pause the agent (`set_agent_status`), swap the key (`rotate_agent_signer`), or shut it down for good (`revoke_agent`, which returns the remaining vault balance to you).
- Each agent carries a DID-compatible Veiled Agent ID and belongs to the operator's account.
- In the API, an agent is namespaced under its owner, for example `@maya/research-bot`.

**The spend policy**

The policy lives in the agent account and the program checks it on every spend, so it is not a rule a server could quietly skip. Only the owner can change it (`update_spend_policy`); the agent has no way to loosen its own limits.

| Rule | Enforced on-chain |
|---|---|
| Per-transaction limit | `per_tx_limit` |
| Daily limit over a rolling 24 hours | `daily_limit` |
| Human approval threshold | `hitl_threshold` |
| Recipient allowlist, up to 10 addresses | `allowlist_enabled`, `allowed_recipients` |
| Single settlement token (USDG by default) | `mint` |

The API's `SpendPolicy` adds controls on top of those: `allowedRecipients` accepts domains and handles with wildcards, `assets` lists which assets may be spent, and `activeHours` sets the time window in which the agent may transact.

**Autonomy**

| `AutonomyTier` | Behaviour |
|---|---|
| `Supervised` | Every spend waits for your approval |
| `SemiAutonomous` | Spends up to the approval threshold settle at once; larger ones wait for you |
| `FullyAutonomous` | Any spend the policy allows settles at once |

Limits and the allowlist apply at every tier. Approval simply puts a human gate in front of spends above the threshold you choose.

**Other capabilities**

- Native x402 payments, with MPP available for split-path settlement
- Agent hierarchies, where a parent agent hands budgets to its child agents
- Every agent transaction appears in the operator's feed in real time

**Privacy.** Confidential transfers are the default for agents too, the same as for any other account. One exception in the current program: a spend an agent makes from its program-controlled vault settles through the Veiled program with a standard token transfer, so that amount is visible on-chain. See [The Veiled Model](the-veiled-model.md).

The full lifecycle is covered in [Agent Wallets](../autonomous-agents/agent-wallets.md), and the policy controls in [Spending Guardrails](../autonomous-agents/spending-guardrails.md).

---

## Shadow Enterprise

Built for organizations that run people and agent fleets side by side.

- Everything in Shadow Pro and Shadow Agent
- Deploying agents in bulk through the API
- Team approvals through Squads multisig, where each signer gets permissions matched to their role
- The top privacy layer (Layer 4), whose shielded pools that hide both sender and recipient, with association sets keeping illicit funds out
- Exports for auditors, shaped to load into popular accounting and tax products

Access requires KYB verification and a compliance review specific to your jurisdiction.

---

## Which one fits

| You want to... | Choose |
|---|---|
| Personal payments in USDG with hidden amounts | Shadow Lite |
| Hidden account state plus one-time receiving addresses | Shadow Pro |
| Hand payments to software that acts for you under your caps | Shadow Agent |
| Large agent deployments with bulk tools and maximum privacy | Shadow Enterprise |

An agent account cannot stand on its own. It always belongs to the operator account that created it, and only that operator can set or change its spend policy on-chain. The agent gets to act on its own; you keep the final say.
