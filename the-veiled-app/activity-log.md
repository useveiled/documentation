# Activity Log

The **Activity** page lists every payment into or out of any account you own, including your primary account and each of your agents. You can read it at a glance without looking up transaction hashes.

---

## Opening it

**Activity** is in the **Agents** group of the sidebar. A summary line at the top gives the number of transactions, the total settled volume, and a reminder that amounts are confidential by default and verified on Solana.

---

## Anatomy of a row

Each payment takes up one row. Here are a few example rows:

```
Settled   Coffee beans, March order     → beanroasters        Confidential   ••••• USDG
          3f9a1c07 · Mar 14, 2026 09:12
Settled   Initial agent funding          → research-bot        Confidential   ••••• USDG
          a81d0e44 · Mar 13, 2026 17:40 · internal transfer
Pending   Weather API call               → api.weatherfeed.io  Public         $0.04
          c02b9f31 · Mar 13, 2026 17:41
```

From left to right, a row shows:

| Part | What it tells you |
|---|---|
| Status badge | `Settled`, `Pending` or `Failed` |
| Memo | The note attached to the payment, or `n/a` if there wasn't one |
| Reference line | The first eight characters of the transaction ID, then the date and time |
| `internal transfer` | Added when both sides of the payment are accounts you own, such as funding one of your agents |
| Counterparty | `→` followed by the recipient for money going out, `←` followed by the sender for money coming in |
| Privacy layer | `Confidential`, `Shielded` or `Public`, each shown in its own colour |
| Amount | Hidden as `••••• USDG` for confidential and shielded payments until you click it. Public amounts are always shown. |

Clicking a hidden amount shows it on your screen only. Nothing changes on-chain, and nothing is disclosed to anyone else.

---

## Status meanings

| Status | What it means | What you'll see in the detail panel |
|---|---|---|
| `Settled` | Finalised on Solana. It can't be reversed. | The full receipt, plus the compliance disclosure option |
| `Pending` | Submitted, but the validator hasn't finalised the block yet. This usually clears in one or two slots, well under a second. | A notice that the ZK range proof is waiting for confirmation and that the balance will update once the block is finalised |
| `Failed` | Never completed | A note that no balance was moved |

---

## Narrowing the list

The tabs above the list filter by status: **all**, **settled**, **pending** or **failed**. If nothing matches, the page shows **No transactions found.**

---

## The receipt panel

Clicking a row opens its receipt next to the list:

- **Transaction ID**: Veiled's full identifier for the payment.
- **Counterparty**: "To" or "From" followed by the other party, with "(internal transfer)" added when both accounts are yours.
- **Timestamp** and **Finality**, where finality is the time in milliseconds the payment took to settle.
- **Amount** in USDG.
- **Memo**.
- **ZK proof hash (Solana)**: the hash of the zero-knowledge proof that came with the transfer.
- **Transaction**: the Solana transaction signature.

### Checking it yourself

You don't have to rely on Veiled's word. Copy the transaction signature into a Solana block explorer such as Solscan or Solana Explorer and you'll see that the transaction took place between the two addresses involved. For a confidential payment, the explorer can't show the amount.

### Compliance disclosure

A settled payment has a **Generate compliance disclosure** button. Pressing it creates a view key that covers that one transaction's full proof and settlement details, so it can be shared for compliance review. The payment is then marked as disclosed with a timestamp, the button changes to **View compliance disclosure**, and a **Security** alert titled **View key access granted** is added to your feed. For when and how to use this, see [Sharing Records with Auditors](../confidentiality/auditor-access.md).

---

## What you see compared with what's public

You see the real amounts because the records belong to your accounts. Someone looking at the same transactions from outside, for example on a block explorer, sees the sending and receiving addresses and an encrypted amount they can't decrypt. For the mechanics, read [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).

---

## Taking your history with you

For bookkeeping or tax work you have two options:

- **Export all data** in [Profile and Preferences](profile-and-preferences.md) downloads a JSON file with your accounts, transactions, alerts, API keys and webhooks.
- For a decrypted record of selected transactions to hand to an accountant or regulator, use the disclosure tools described in [Sharing Records with Auditors](../confidentiality/auditor-access.md). Decryption happens with your own key, so your records leave your device only when you send them somewhere.

---

## Related pages

- [Funding Your Balance](funding-your-balance.md)
- [Spending Insights](spending-insights.md)
