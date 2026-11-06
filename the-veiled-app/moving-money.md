# Moving Money

A Veiled payment takes about as long as sending a chat message. While you fill in a short form, the app encrypts the amount and builds a zero-knowledge proof, and Solana checks that proof before the transfer settles.

---

## Sending a payment

Open **Send Payment** from the **Banking** group in the sidebar. The form has four parts.

### Recipient

Type into the recipient box. As you type, Veiled searches active accounts and suggests up to six whose handles match. You can enter any of these:

| What you enter | How Veiled handles it |
|---|---|
| A Veiled handle, such as `@maria`, `maria` or `maria.veiled` | The `@` prefix and `.veiled` suffix are optional, and case doesn't matter. The payment goes to that account. |
| An external domain, such as `api.weatherfeed.io` | Recorded as an `x402` payment to an outside service. See [Paying per Request with x402](../autonomous-agents/paying-per-request.md). |
| Any other label or a raw Solana address | Sent as an external transfer. The app tells you: "No Veiled account found ... This will be sent as an external transfer." |

When the recipient is a Veiled account, a side panel shows its account type (human or agent), its privacy tier and its **Member since** date. It also shows your history with that account: payments sent, total sent and the date of your last payment. Search never reveals another person's balance. You can't send a payment from an account to itself.

### From account

Choose which of your active accounts will pay. Your primary account is labelled as such, and every agent account you own is listed under its handle. If the amount you enter is more than the selected account holds, the form shows **Insufficient balance in this account.** and won't let you continue.

### Amount, privacy layer and memo

- **Amount**: in USDG, with up to two decimal places. It must be greater than zero.
- **Privacy layer**: `Confidential` (the default), `Shielded` or `Public`. This setting determines what observers can see on-chain. See [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).
- **Memo** (optional): a short note on what the payment is for. Only you and the recipient can read it. It never appears on-chain in plaintext.

### Review and confirm

Press **Send payment**. After a moment a review card titled **Review confidential transfer** opens with a five-minute countdown. It shows the amount, the from and to handles, the memo and the privacy layer. You then have three options:

- **Confirm & send** submits the payment. Once it settles it is final.
- **Cancel** discards the payment and returns you to the form.
- If you don't act before the timer hits zero, the card changes to **Review expired**. Dismiss it and start over.

---

## What happens after you confirm

A progress log shows each stage as it runs. Your device encrypts the amount and generates a zero-knowledge range proof, the transaction goes to the current Solana leader, and the network verifies the proof using its native ZK proof program. Your keys and the proof stay on your hardware, and only the finished transaction leaves it.

Solana produces a new slot about every 400 milliseconds, and a transfer usually confirms within one or two slots, so both sides typically see the payment in under a second. When it's done the log shows the finality time, the transaction signature and the proof hash, and a **View receipt** link takes you to the [Activity Log](activity-log.md).

If the payment is rejected (insufficient funds, an inactive sender account, or a spending limit), the log shows the reason and no money moves.

---

## Limits that can stop a payment

| Sending from | Checks applied |
|---|---|
| Your personal account | The **Daily limit** and **Monthly limit** in [Profile and Preferences](profile-and-preferences.md). Either one set to `0` means no limit. |
| An agent account | That agent's per-request maximum and daily maximum. See [Spending Guardrails](../autonomous-agents/spending-guardrails.md). |

Only settled payments count toward these totals.

---

## Getting paid

There's nothing to set up before you can receive. Anything sent to your handle lands in your account and shows up in your feed. You'll also get a **Transfer received** alert if that rule is on (see [Alert Rules](alert-rules.md)).

### Payment requests

The Veiled program supports payment requests, which can be shared as a link or a QR code. A request can be:

- **Plain**: it states a fixed amount, and the payer has to pay exactly that amount.
- **Confidential**: the amount isn't stored on-chain. The request holds only a commitment to it. The payer receives the amount and a blinding factor privately, usually in the request's encrypted memo, and the program checks the payment against the commitment.

Every request has an expiry time and can be paid only once. The person who created it can cancel it while it's still open, and the funds can be sent to a different wallet from the creator's, for example a company treasury. Someone without a Veiled account can open a request link, and they're taken through sign-up before they pay, so the request also works as an invitation.

---

## Paying a wallet outside Veiled

You can send USDG to any Solana address. If the recipient's token account doesn't have the confidential transfer extension enabled, the transfer can't be confidential. In that case it settles as an ordinary, public SPL token transfer. Veiled tells you this before you confirm, so a payment is never made public without your knowledge.

---

## What it costs

The fee breakdown next to the form lists the payment amount, a network fee of $0.00 and the amount the recipient receives. Those two amounts are the same because Veiled doesn't take a cut. You pay only Solana's own network fee, usually a fraction of a cent, and it comes out of your SOL reserve automatically. The program can also charge a protocol fee, but that fee is off by default. See [The Solana Program](../under-the-hood/solana-program.md).

---

## Related pages

- [Your .veiled Name](your-veiled-name.md)
- [Activity Log](activity-log.md)
