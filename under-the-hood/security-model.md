# Security Assumptions

Veiled is built on the assumption that the company itself could fail, be breached or be pressured. Custody of funds, validity of transfers and confidentiality of amounts are each backed by keys the user holds, cryptography, or Solana consensus, so none of them rests on trusting Veiled. This page lists the threats Veiled plans for and what answers each one.

---

## Threats and the mechanism that answers them

| Threat | What stops it |
|---|---|
| **Veiled is hacked or forced to move someone's money** | Keys are created and kept on the user's device. Veiled holds no private keys, and the `veiled` program has no instruction that lets an admin move a user's or agent's funds. |
| **Veiled or an outsider tries to read amounts** | Proofs are generated and amounts decrypted on the client. Veiled's backend receives ciphertext only. |
| **Someone submits a malformed or overdrawn transfer** | Solana's native ZK proof program checks the range and validity proofs on confidential balances, and the Groth16 verifier (via alt_bn128 syscalls) checks shielded pool proofs, before any encrypted state changes. These checks are part of consensus, so a bad proof never settles. |
| **An agent tries to spend past its limit** | The policy lives in the agent's on-chain account and the program checks it on every spend; a violating spend fails. The agent's client applies the same rules earlier and refuses to sign. |
| **A user loses their device and passkey** | Passkeys that sync across devices, keys that can be exported, Squads-based key rotation, and (after beta) social recovery. See [Keys, Signers, and Custody](../confidentiality/keys-and-custody.md). |
| **A signed transaction is replayed or forged** | Normal Solana Ed25519 signatures apply. Each transaction is tied to a recent blockhash and deduplicated by signature, exactly as for any other Solana transaction. |
| **A block producer stalls or drops a transaction** | Solana has no single sequencer. Leaders rotate across a worldwide validator set, so a transaction one leader ignores is picked up by the next, and no one party can shut out Veiled traffic. |
| **Veiled's backend goes down** | User assets sit in token accounts owned by the user's own wallet, reachable from any Solana wallet or tool (Phantom, Solflare, Backpack) with an exported key. Agent vaults can be drained back to the owner with `withdraw_from_agent` or `revoke_agent`, which any client can build and which stay available during a protocol pause. |

---

## Powers Veiled has, and powers it lacks

**Veiled's services do:**

- route API calls, serve the apps, and index on-chain events into a live feed
- send webhooks and notifications
- check an agent's spend policy when the agent's client is about to sign
- run identity checks and selective disclosure flows that the user starts

**Veiled's on-chain admin keys can:**

- pause the four fund-moving instructions (`pay_invoice`, `queue_invoice`, `approve_pending`, `fulfill_request`) with `set_pause`
- set the fee treasury, switch fees on or off, and retune the fee schedule and discount tiers
- record a profile's KYC tier, through the separate `compliance_authority` key
- hand those roles to new keys, using a two-step nominate and accept flow so a typo cannot strand control; moving the admin authority onto a Squads multisig is planned for after beta

**Veiled cannot:**

- move funds without the owner's signature, because it does not have the key
- read amounts, because it does not have the decryption key
- change a transaction after it is signed, since Solana rejects anything without a matching signature and the ZK proof program rejects anything without a valid proof
- override an agent's limits; only the owning profile can sign `update_spend_policy`, and the program enforces the result on every spend
- stop an owner from pausing, revoking or withdrawing from their agents, since none of those instructions honours the protocol pause

The last list is the one that matters. Those limits come from cryptography and consensus, not from a company policy that could be rewritten.

---

## Proofs are built on the device

Zero-knowledge proofs for confidential transfers are produced on the user's device in WASM, not on Veiled's servers. That is a property of the architecture rather than a promise: the plaintext balance and amount never leave the device in a form Veiled's infrastructure could capture, whatever Veiled wanted. Validators see the same thing Veiled sees, which is ciphertext.

---

## RPC is not a gatekeeper

Veiled sends transactions through dedicated RPC infrastructure instead of public Solana endpoints so the app stays responsive under load. This is a performance choice only. Every Veiled transaction is an ordinary signed Solana transaction that any RPC provider would accept, so there is no privileged route to the chain.

---

## Sending twice is not possible

A Solana transaction is bound to a recent blockhash and identified by its signature, and the network refuses to include the same signature twice. If a retry happens because of a network hiccup, the transfer still lands at most once. If the blockhash expires before inclusion, that transaction can never land and has to be rebuilt and signed again. Veiled's submission logic is idempotent on top of those rules, so a double send cannot happen by construction.

---

## Known risks and how they are handled

| Risk | Handling |
|---|---|
| Device lost and the passkey was never synced | Export your key ahead of time and store it safely. Pro and higher plans add Squads-based rotation, and social recovery is planned after beta. |
| A slow device struggles to produce a ZK proof | A server-assisted proving fallback exists. It works only from inputs the client derives and never receives plaintext amounts. |
| The account that owns an agent is compromised | The agent's on-chain limits cap the damage even if the owner's session is hijacked. Revoke or rotate the agent's key from the dashboard straight away. |
| A regulator acts against Veiled the company | Funds are in wallets the users own, not in accounts Veiled controls, so users keep direct on-chain access whatever happens to the company. |
| Solana is congested or Veiled's services are down | Multiple RPC providers, priority fee handling and a public status page. An outage affects access to the app, not the safety of funds. |
| A fee setting is changed | Fee parameters are adjustable by the fee authority within program bounds (no rate or discount above 10,000 bps). Every change emits an on-chain event (`FeeScheduleUpdated`, `TiersUpdated`, `FeeRoutingUpdated`), and the fee charged is always added on top of the amount rather than taken out of it. |

---

## Related pages

- [System Design](system-design.md)
- [The Solana Program](solana-program.md)
