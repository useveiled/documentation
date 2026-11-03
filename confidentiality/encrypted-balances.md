# Encrypted Balances and Private Payments

Veiled aims for confidentiality, not anonymity. Your address is visible on Solana like everyone else's; the amounts you move and the balances you hold are not. The network can confirm that your activity is valid without anyone outside the transaction learning the figures.

This page explains the cryptography behind that.

---

## Two transfers side by side

A plain SPL token transfer exposes everything:

```
Sender address [PUBLIC] → Amount [PUBLIC] → Recipient address [PUBLIC]
```

Anyone connected to an RPC node can read that amount, and it stays readable permanently. Veiled exists to remove that default.

A confidential Veiled transfer looks like this:

```
Sender address [PUBLIC] → Amount [ENCRYPTED] → Recipient address [PUBLIC]

Attached: a zero-knowledge proof that the hidden amount is valid,
          which reveals nothing about the amount
```

---

## The primitive: Token-2022 confidential balances

Solana's token program, Token-2022, includes a confidential transfer extension. No other major L1 has an encrypted-amount feature built into the protocol, so Veiled does not need to bolt privacy onto the application. It inherits it from the chain. Two pieces do the work:

| Technique | Job |
|---|---|
| Twisted ElGamal encryption | Encrypts balances and transfer amounts. The encryption is homomorphic, so the token program can add to and subtract from an encrypted balance without decrypting it. |
| Zero-knowledge proofs | Verified on-chain by the ZK proof program built into Solana. They show the sender can cover the transfer and that the amount is non-negative and within range, without disclosing the amount. |

Put together, the chain confirms the payment is legitimate and updates both encrypted balances correctly, yet never learns how much moved. The Veiled program (Rust, Anchor) layers handles, agent accounts, spend policies and disclosure receipts over this base.

---

## A confidential payment, start to finish

| Step | Where | What happens |
|---|---|---|
| 1 | Veiled app | You start a payment. |
| 2 | Your device | Using the decryption key for your account, the Privacy Engine turns the amount into a twisted ElGamal ciphertext and builds the zero-knowledge proofs that go with it. The plaintext amount stays on the device. |
| 3 | Solana | One transaction carries the confidential transfer instruction for your token account and the proof verification by the native ZK proof program. |
| 4 | Solana | Validators check the proofs and update both encrypted balances, usually within one or two slots of about 400ms. Addresses are public; the amount remains ciphertext. |
| 5 | Both parties | The sender and the recipient decrypt the amount with their own private keys. |
| 6 | Optional | If compliance requires it, you can prove a single transaction's amount to a chosen party such as an auditor, without exposing any other transaction. See [Sharing Records with Auditors](auditor-access.md). |

From the API, confidentiality is the default as well. The `confidential` flag on a transfer is `true` unless you turn it off, which you should only do when paying a wallet that cannot handle Veiled confidential tokens:

```ts
import { Veiled } from "@useveiled/sdk";

const veiled = new Veiled({ apiKey: process.env.VEILED_API_KEY! });

// Confidential by default: only the two parties can read 75.50
const payment = await veiled.transfers.create({
  to: "@studio_rent",
  amount: "75.50",
  memo: "March desk rental",
});
```

The same rule shapes balance reads. `veiled.accounts.balances()` without arguments only tells you which assets are non-zero; to see amounts you pass a `decryptionProof` generated on your side with your own key, because Veiled's servers cannot decrypt your balances.

---

## What is not amount-confidential today

Some instructions in the Veiled program move tokens with a standard checked token transfer instead of a confidential one, and record the amount in their events:

- agent spends from a program-controlled vault (`pay_invoice`, `approve_pending`)
- funding and drawing down an agent vault (`fund_agent`, `withdraw_from_agent`, `revoke_agent`)
- paying a request link (`fulfill_request`)

For those, the amount can be read on-chain. A confidential request link still keeps the amount out of the request record itself, storing only a commitment until the payer settles it.

---

## Going further: shielded state

Every account starts with encrypted balances. On Shadow Pro and above, the state of an account can also be shielded through Veiled's commitment-tree shielded pool. In that pool the chain holds only Merkle roots that commit to balances and history. Solana's alt_bn128 syscalls check the Groth16 proofs, and stealth meta-addresses stop anyone linking a payee to their public identity. [The Solana Program](../under-the-hood/solana-program.md) covers how the pool fits into the on-chain design.

---

## Why bother

Public chains achieved verifiability by publishing everything. Analysts and auditors like that, but it also lets anyone track your salary, your rent or your company's runway as it happens. Veiled keeps the part that matters: the network still validates every payment and keeps a permanent record of it. What disappears is the ability of strangers to watch. A payment rail for an open economy where agents transact alongside people should work this way.

---

## Speed and cost

Your own device builds each proof, running WASM inside the browser or the mobile app. On current hardware this adds a short delay of under a second before the transaction is sent. Low-power devices can use a server-assisted proving path instead; it works only from proof inputs your device has already derived, and the plaintext amount is still never transmitted. Once submitted, the payment confirms at Solana speed, typically within a slot or two, and the network fee is a fraction of a cent in SOL, paid for you automatically.

---

## Related pages

- [Sharing Records with Auditors](auditor-access.md)
- [Keys, Signers, and Custody](keys-and-custody.md)
