# Our Legal Posture

One idea runs through Veiled's approach to compliance: the more privacy an account gets, the more verification it has to pass. Every account has confidential amounts by default. Going further, to shielded state at Layer 3 or full privacy at Layer 4, requires progressively stronger identity checks. The rest of this page follows from that rule.

---

## How Veiled is structured

Veiled never holds your money, and your keys remain yours. Veiled runs no signing infrastructure and never takes custody of funds. It acts as a protocol layer rather than as a custodian or a money transmitter, and how it approaches regulation in each market is shaped by that position.

Settlement uses regulated stablecoins, USDG (Paxos' Global Dollar) and USDC, both natively issued on Solana, together with SOL. Where on and off ramps to fiat are available, licensed third parties provide them.

---

## Where Veiled is available

| | During beta |
|---|---|
| United States | Not available to US persons |
| Initial markets | The EU (where MiCA applies), the UK, Singapore and the UAE, with each one kept under review against its local virtual asset rules |
| Other jurisdictions | No beta access where local law bans crypto payment services outright |

---

## Verification tiers

Each account tier asks for a level of verification that matches the privacy it unlocks:

| Tier | What you provide | What you get |
|---|---|---|
| Shadow Lite | Verified email and a wallet attestation on-chain | Hidden amounts, with the standard limits applied |
| Shadow Agent | An email address and a wallet attestation, supplied through the operator deploying the agent | Confidential amounts, plus x402 and MPP |
| Shadow Pro | Government ID, a liveness check, and proof of address | Layer 3 shielded state and higher limits |
| Shadow Enterprise | Full KYB, covering incorporation records and beneficial ownership | Layer 4 full privacy and the bulk agent API |

Stronger privacy always means a stronger identity requirement. The Layer 4 shielded pools are open only to KYB-verified enterprises, and they rely on association sets to keep illicit funds out of the pool. The detailed requirements and limits for each tier are in [Identity Checks and the Travel Rule](identity-checks.md).

---

## The Travel Rule

Travel Rule requirements apply to cross-border transfers, and to transfers above the threshold, that go to or come from regulated institutions. Veiled offers three tools for meeting them:

- **Selective disclosure proofs** built from view keys, which can be shared with a counterparty's compliance system when required.
- **Address screening** of counterparties, through an integration with Chainalysis or TRM Labs.
- **Identity data exchange** in the standard form between regulated originating and beneficiary institutions, where applicable.

The mechanics are explained in [Identity Checks and the Travel Rule](identity-checks.md).

---

## Who is responsible for agents

If you deploy agent accounts, your agents' compliance obligations under applicable law are your responsibility. Because the Veiled agent program enforces spend policies on-chain, you have a verifiable record showing that your agents stayed inside the limits you set. See [Spending Guardrails](../autonomous-agents/spending-guardrails.md).

---

## Where data is kept

Account profiles and any verification records (the user metadata) are stored in a region appropriate to the jurisdiction where the account was opened. On-chain data is a different matter: it lives on Solana mainnet, not on Veiled's infrastructure, so it is global by nature. Onboarding tells you this clearly.

---

## Privacy and compliance work together

What the confidential layer hides is the size of each payment. The fact that a payment happened stays public, and so do the addresses at the default tier. When lawful legal process arrives, an account holder can answer it with view key-based selective disclosure (see [Sharing Records with Auditors](../confidentiality/auditor-access.md)). That produces a cryptographic proof of correctness that can satisfy a regulator without publishing the underlying data and without the user's keys ever leaving their custody.

Regulated payment systems that preserve privacy have broadly settled on this approach, and Veiled is built on it: be fully answerable to the law when it asks, and keep everything else confidential.

---

## Keep reading

- [Identity Checks and the Travel Rule](identity-checks.md)
- [Sharing Records with Auditors](../confidentiality/auditor-access.md)
