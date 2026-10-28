# Welcome to Veiled

**Veiled is a self-custodial crypto bank on Solana for people and the AI agents that work for them, with payment amounts encrypted by default.**

---

## The short version

Money on Veiled settles on Solana in well under a second, the keys stay with you, and the size of every confidential payment is hidden from the public. The chain still records who paid whom, so anyone can verify that a payment took place. Only the two parties to it can read the number.

That combination is hard to find elsewhere. A bank keeps your history away from the public but sees every line of it itself, and passes data on to processors and partners. A public blockchain removes the middleman but publishes every balance and transfer for anyone to inspect. Veiled sits between the two: settlement is open and verifiable, while amounts are readable only by sender and recipient.

The same account works whether you are a freelancer collecting USDG from a client or the owner of an AI agent that pays for API calls overnight. Agents get wallets of their own, spend inside limits you set, and show up in your activity feed next to your own payments.

---

## How Veiled compares

| | Veiled | Ordinary self-custody wallet | High street bank |
|---|---|---|---|
| Who can see amounts | Encrypted by default, readable by sender and recipient | Anyone with a block explorer | The bank, but not the public |
| Who holds the keys | You (non-custodial) | You (non-custodial) | The bank (custodial) |
| Time to settle | Under a second, on-chain | Under a second, on-chain | 1 to 3 business days |
| AI agents | First-class agent accounts with spend policies | An add-on API at best | Not supported |
| Compliance | You disclose chosen transactions to auditors | Not applicable | The institution sees everything |
| Getting started | Sign up with email, no seed phrase to write down | Write down a seed phrase | Application and manual review |

---

## Why Solana

The privacy layer comes from Solana itself. Token-2022, Solana's token program, ships a confidential transfer extension that stores balances and amounts as twisted ElGamal ciphertexts and has zero-knowledge proofs checked on-chain to show each transfer is valid. Sender and recipient addresses remain in the clear. Among the major chains, Solana is the one with this primitive built into the protocol, and its fast slots and tiny fees are what make private, self-custodial payments practical at everyday scale.

Balances are held in USDG, and USDC is accepted as well. Both are issued natively on Solana. Veiled keeps a small SOL balance aside automatically to cover network fees, so you never have to manage gas yourself.

---

## Pick a starting point

| If you want to... | Read |
|---|---|
| Understand the overall design | [The Veiled Model](start-here/the-veiled-model.md) |
| Open an account and start paying people | [Signing Up](the-veiled-app/signing-up.md) |
| Give an AI agent its own wallet | [Agent Wallets](autonomous-agents/agent-wallets.md) |
| Build Veiled into your own product | [Making Authenticated Requests](developer-api/making-requests.md) |
| Dig into the cryptography | [Encrypted Balances and Private Payments](confidentiality/encrypted-balances.md) |
