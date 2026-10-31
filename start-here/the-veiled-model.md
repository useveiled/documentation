# The Veiled Model

A Veiled account pairs a self-custodial Solana wallet with the Veiled on-chain program, on top of the confidential transfer extension in Token-2022. You hold the keys, payments finalize on Solana in a fraction of a second, and confidential amounts travel as ciphertext instead of plain numbers.

The rule to remember: **addresses are public, confidential amounts are not.**

---

## Following one payment

Say Maya pays her designer 400 USDG from the Veiled app. Here is what happens between the tap and the receipt:

1. **On Maya's device.** The Veiled Privacy Engine turns the 400 into a twisted ElGamal ciphertext and builds a zero-knowledge proof that the payment is well formed. Both steps run locally.
2. **Submission.** The ciphertext and proof go to Solana as a confidential transfer instruction against Maya's token account.
3. **Verification.** In that same transaction, Solana's built-in ZK proof program checks the proof, and the token program updates both encrypted balances.
4. **On the ledger.** Maya's address and the designer's address are visible to anyone. The 400 is not; observers see only ciphertext.
5. **Reading it back.** Maya and the designer each hold a decryption key, so each of them can see the amount. Nobody else can.
6. **In the feed.** The payment shows up for both parties within one or two Solana slots, each roughly 400ms long.

The proof covers the two things a network needs to know: that Maya is not spending more than she has, and that the ciphertext matches what was actually sent. Because it is produced on her device, Veiled's servers never receive her plaintext balance or the plaintext amount, whether in transit or in storage.

The confidential transfer primitive only moves encrypted value. Everything that turns it into a bank account lives in the Veiled program, written in Rust with Anchor (program ID `BgJQoxfRfpQYhJLFxnq7f8GcxPoAce47MGPnq1qJiV7V`): `.veiled` handles, agent accounts and their spend policies, the approval queue, request-to-pay links, fee logic, and receipts for selective disclosure.

{% hint style="info" %}
Instructions that the Veiled program settles itself, such as an agent paying from its vault (`pay_invoice`, `approve_pending`), funding an agent (`fund_agent`), or paying a request link (`fulfill_request`), move tokens with a standard checked token transfer. The amount is an instruction argument and appears in the program's events, so those movements are not amount-confidential. Confidential requests store only a commitment to the amount (`keccak256(amount_le, blinding)`) until someone pays them.
{% endhint %}

---

## What a confidential transfer leaves on-chain

Every confidential payment produces an ordinary Solana transaction. It carries:

| Field | Form |
|---|---|
| Sender address | Base58, public |
| Recipient address | Base58, public |
| Amount | Stored encrypted (twisted ElGamal) inside Token-2022's confidential balance extension |
| Validity proof | Zero-knowledge proof, verified by Solana's native ZK proof program |
| Slot and signature | Slot number plus a base58 transaction signature |

You can look any of this up on a public explorer such as Solscan. What you will learn is that two addresses transacted. What you will not learn is the amount.

---

## Who sees what

| Information | Public chain | Veiled servers | You and your counterparty |
|---|---|---|---|
| Sender and recipient addresses | Yes | Yes | Yes |
| That a transfer happened | Yes | Yes | Yes |
| Which token moved | Yes | Yes | Yes |
| Confidential transfer amount | No (ciphertext) | No | Yes |
| Resulting balances | No (ciphertext) | No | Yes |
| Your decrypted history | No | Only if you generate a disclosure for it | Yes |

Veiled's backend does three jobs: it routes requests, renders app state, and indexes on-chain activity. It holds no keys and has no way to read balances in plaintext. Webhook payloads reflect the same limit and never contain amounts.

The network is in the same position. Solana has no central operator; blocks come from a validator set spread around the world. Since encryption happens on the client, validators only ever see and order ciphertext. Privacy is a consequence of how the system is built, not a promise in a policy document.

---

## Live updates

Veiled runs its own indexer over on-chain events, so the app does not have to poll the chain to know where a payment stands. A WebSocket connection pushes each status change, such as pending moving to confirmed, into your feed as soon as it happens.

---

## Keep reading

- [Personal and Agent Accounts](personal-and-agent-accounts.md): the kinds of account you can open and what each one can do
- [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md): the cryptography in more depth
