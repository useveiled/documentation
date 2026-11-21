# Common Questions

Short answers to the questions people ask most, with links to the page that covers each topic in depth.

---

## The basics

### In one sentence, what is Veiled?

A non-custodial crypto neobank on Solana where the amounts you hold and send are encrypted by default, payments settle within a slot or two (a slot is roughly 400ms), and people and AI agents hold accounts next to each other. It handles USDG, USDC and SOL, and it is designed as the private money layer for the agent economy.

### Will I need to understand crypto to use it?

No. All you need to sign up is an email address and a password. Keys, encryption and settlement all happen in the background without you managing them. People who already have a Solana wallet such as Phantom, Solflare or Backpack can connect that wallet instead. See [Signing Up](../the-veiled-app/signing-up.md).

### Which currencies do balances use?

USDG (the Global Dollar from Paxos) and USDC, both issued natively on Solana. Veiled also keeps a small amount of SOL in your account and refills it automatically, so you never have to think about network fees. See [Funding Your Balance](../the-veiled-app/funding-your-balance.md).

### Does Veiled make me anonymous?

Not on the standard tier, and that is on purpose. Your address is visible on-chain; what stays hidden is how much moves in and out. That is confidentiality rather than anonymity. Stronger privacy (stealth addresses on Shadow Pro, full privacy on Shadow Enterprise) only unlocks alongside stronger identity checks. [Our Legal Posture](../regulation/legal-posture.md) explains why the two go together.

---

## Using a personal account

### How are my amounts kept private?

Payments use Token-2022 confidential balances, the encrypted-balance extension built into Solana's token program. Each amount is stored as a twisted ElGamal ciphertext, your device builds the zero-knowledge proofs, and Solana's native ZK proof program checks them. Only you and the person you pay can decrypt the figure. See [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).

### Can someone look up my balance?

They can see that your address exists and holds a token balance, but the balance itself is ciphertext on-chain. Without your key, the number cannot be read.

### My accountant (or landlord) needs proof I paid. How do I give it?

Generate a disclosure scoped to just the payment in question. You decide which details it reveals and which person receives it, and nothing else is exposed. The program can also record a timestamped receipt that the disclosure happened, without putting the proof itself on-chain. See [Sharing Records with Auditors](../confidentiality/auditor-access.md).

### I lost my phone. Is my money gone?

No. If your passkey syncs through iCloud Keychain or Google Password Manager, sign in on another device and carry on. Shadow Pro accounts also have Squads-based multisig recovery, which means a broken or stolen phone can't cost you your balance. See [Keys, Signers, and Custody](../confidentiality/keys-and-custody.md).

### Can Veiled see my transaction history in readable form?

The amounts you see in the app are decrypted with your own key on your device. On-chain, every amount is ciphertext, and Veiled's API only returns a balance figure when your client supplies a decryption proof it generated with your key. See [Activity Log](../the-veiled-app/activity-log.md).

### Can I pick a name instead of sharing an address?

Yes. Each account reserves a unique name such as `ada.veiled`, made of lowercase letters, digits and underscores (up to 32 characters). You can move to a new name later, and the old one is released for others. See [Your .veiled Name](../the-veiled-app/your-veiled-name.md).

---

## Agents and developers

### What exactly is an agent account?

An account an AI agent uses to pay for things, owned by your profile. On-chain it is an `Agent` record plus a vault that only the Veiled program can move funds out of. The agent signs with its own key and gets a DID-compatible Veiled Agent ID. You set its rules: a per-transaction limit, a 24 hour limit, an approval threshold, and which recipients it may pay. See [Agent Wallets](../autonomous-agents/agent-wallets.md).

### Could my agent overspend?

Not past the on-chain rules. The per-transaction limit, the 24 hour limit, the recipient allowlist and the approval threshold are checked by the `veiled` program on every spend, so even a compromised server or a leaked agent key cannot push a payment through them. Extra rules such as allowed domains and active hours are checked by the agent's signing flow before it signs. If something looks wrong, you can pause the agent, swap its key or revoke it at any time, even while payments are paused protocol wide. See [Spending Guardrails](../autonomous-agents/spending-guardrails.md).

### Can a human sign off on bigger payments?

Yes. A `Supervised` agent sends every payment to you for approval. A `SemiAutonomous` agent pays on its own up to your threshold and queues anything larger. A `FullyAutonomous` agent pays anything within its limits but can still choose to ask you first. Nothing leaves the vault until you approve, and the limits are checked again at that moment.

### Is x402 supported?

Yes, natively, on every agent account. MPP is also available for split-path settlement, where several instructions are combined into one atomic Solana transaction. See [Paying per Request with x402](../autonomous-agents/paying-per-request.md).

### How do I find out when an agent pays or waits for approval?

Register a webhook endpoint and Veiled will push each event to it as it happens. [Reacting to Events](../autonomous-agents/reacting-to-events.md) describes the events, and [Webhook Delivery and Signatures](../developer-api/webhook-delivery.md) covers delivery and verification.

### Will the API tell me how much my agent spent?

Only if you decrypt it on your side. Veiled's servers do not hold your decryption key, so the API does not return plaintext balances on its own; your client generates a decryption proof with your key to reveal them. Privacy holds all the way through to the API. See [Reading Accounts and Balances](../developer-api/reading-accounts.md).

### Where do I start building?

Create an API key in [Developer Settings](../the-veiled-app/developer-settings.md), then follow [Making Authenticated Requests](../developer-api/making-requests.md).

---

## Rules and verification

### Why ask for ID on a privacy product?

Because hiding amounts and hiding identity are different things, and Veiled is upfront that more privacy costs more verification. Shadow Lite conceals amounts but not addresses, so it needs only an email and a wallet attestation. To unlock shielded state on Shadow Pro you complete KYC; for full privacy on Shadow Enterprise your business completes KYB. Each step up in privacy comes with a step up in accountability, which is what allows Veiled to respond to lawful legal process when it must. That accountability is what lets the privacy last. See [Identity Checks and the Travel Rule](../regulation/identity-checks.md).

### I'm in the US. Can I sign up?

Not during the beta. The beta is closed to US persons. It is opening first in the UAE, Singapore, the UK and the EU.

### What about large or international payments?

When a payment above a regulatory threshold involves a regulated institution, Travel Rule obligations apply, including screening of the counterparty. See [Identity Checks and the Travel Rule](../regulation/identity-checks.md).

### Can I read the code?

The on-chain layer is open source and auditable: the `veiled` Anchor program written in Rust, covering `.veiled` names, agent accounts and their spending limits, payment requests, disclosure receipts and fees. Confidential transfers themselves use Solana's Token-2022 extension. The app, backend and indexer are closed source for the beta. See [The Solana Program](../under-the-hood/solana-program.md).

---

## Pricing

### How much does a payment cost?

The protocol fee is 0.10% of the amount, never more than 5 USDG, added on top so the recipient gets the full sum. Holding or staking $VEILED lowers it by up to 75%. The Solana network fee, a fraction of a cent, comes out of your SOL reserve automatically. Agent accounts cost $10 a month while active, waived for any month under $500 in volume, and Layer 3 shielded state costs $15 a month. The full fee formula is in [The Solana Program](../under-the-hood/solana-program.md#fee-logic).

### Does API access cost extra?

There are no charges other than the account and payment fees already listed. Bulk agent deployments on Shadow Enterprise are priced individually and come with an SLA.
