# System Design

Veiled has no private ledger of its own. Balances and transfers settle on Solana, the encryption comes from the Token-2022 confidential transfer extension that ships with Solana's token program, and Veiled adds an open-source Anchor program (`veiled`) for the parts a bank account needs on top: `.veiled` names, agent accounts with chain-enforced limits, an approval queue, payment requests, disclosure receipts, and the $VEILED fee model.

This page walks through how those pieces fit together. For the account-level detail of the program, see [The Solana Program](solana-program.md). For what each layer protects against, see [Security Assumptions](security-model.md).

---

## Two halves: what is enforced and what is convenient

The design draws one firm line.

| Enforced by Solana and the `veiled` program | Provided by Veiled's services as a convenience |
|---|---|
| Who can move funds (signatures) | Rendering the web and mobile apps |
| Whether a confidential transfer is valid (ZK proof checks) | Indexing on-chain activity into a searchable feed |
| Agent spending limits, approval rules and recipient allowlists | Delivering webhooks and alerts |
| Handle uniqueness and ownership | Keeping a small SOL reserve topped up for network fees |

Anything in the left column holds even if Veiled's servers are offline or misbehaving. Anything in the right column makes the product pleasant to use, but nothing about the safety of funds depends on it.

---

## The moving parts

### Keys and signing

Keys are generated on the client, and the private key never reaches Veiled's servers. Every fund movement is a Solana transaction the account holder (or the agent's own signing key) has signed. Veiled does not hold keys for users. Assets stay in token accounts owned by the user's own wallet, and recovery and key rotation are handled with Squads multisig; see [Keys, Signers, and Custody](../confidentiality/keys-and-custody.md).

### Encrypted amounts

Amounts in a confidential transfer are twisted ElGamal ciphertexts held in Token-2022 confidential balances. The sender's device builds the zero-knowledge proofs, and Solana's native ZK proof program checks them inside the same transaction. Validators order and settle ciphertext; they never learn the value. Addresses on both sides stay public. The full walkthrough is in [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).

### Shielded state

For the deepest privacy layer, Veiled describes a shielded pool: account state committed as leaves of an on-chain Merkle tree, spends proven with Groth16 proofs checked through Solana's alt_bn128 syscalls, and stealth meta-addresses so each payment lands at a fresh, unlinkable address. This layer is separate from the `veiled` program, which keeps addresses public throughout.

### Agent policy enforcement

An agent's limits live in its on-chain `Agent` account. The agent can only spend out of a program-controlled vault, and every spend instruction checks the per-transaction limit, the 24 hour total, the recipient allowlist and the autonomy tier before a single token moves. The agent's client runs the same checks before it signs, which saves a failed transaction but is not what makes the limit binding.

### Network fees

Each account keeps a small SOL balance for Solana network fees, which run to a fraction of a cent per transaction. When the reserve runs low, Veiled refills it with an internal swap, so people only ever deal in USDG.

### Indexing

Veiled reads program logs and event streams off-chain and writes them into PostgreSQL. That index drives the live activity feed, `.veiled` name lookup and the webhook system, so neither users nor integrators have to poll the chain. It holds data such as address to handle mappings, the public facts of each transaction (sender, receiver, time, signature) and webhook subscription and delivery state.

---

## Layer diagram

```
+-------------------------------------------------+
|                 Veiled clients                  |
|      web app . iOS and Android . TypeScript SDK |
+------------------------+------------------------+
                         |
+------------------------v------------------------+
|                   Veiled API                    |
|  REST at api.useveiled.com . WebSocket . hooks  |
+------------------------+------------------------+
                         |
+------------------------v------------------------+
|                Veiled services                  |
|                                                 |
|  Wallet engine     Privacy engine  Agent engine |
|  key handling,     client-side     policies,    |
|  tx assembly       proof building  x402 flows   |
|                                                 |
|  Indexer: program events -> PostgreSQL          |
+------------------------+------------------------+
                         |
+------------------------v------------------------+
|                     Solana                      |
|  Token-2022 confidential balances . ZK proof    |
|  program . shielded pool . veiled program . x402|
+-------------------------------------------------+
```

The arrows only ever point down. Nothing in a higher layer can change what a lower layer accepts.

---

## What each party has to trust

**Holding funds.** A person using Veiled does not hand their keys to anyone. The servers receive signed transactions and nothing that could produce a new signature.

**Seeing amounts.** Proofs are built and ciphertexts decrypted on the user's device. Veiled's backend and Solana's validator set both work with ciphertext, and the API only reveals a balance when the client supplies a decryption proof made with its own key.

**Letting agents spend.** An operator does not have to trust Veiled to police an agent after the fact. The spend policy is part of the on-chain program's checks, so a transaction outside it fails before it settles.

---

## Related pages

- [The Solana Program](solana-program.md): accounts, instructions, fees and errors in the `veiled` program
- [Security Assumptions](security-model.md): threat by threat, how each guarantee is backed
