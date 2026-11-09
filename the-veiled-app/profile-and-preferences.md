# Profile and Preferences

**Settings**, at the bottom of the sidebar, holds your identity details, linked wallet, personal spending limits, a few behaviour switches, and the controls for exporting or deleting your data. The sections below follow the order of the page.

---

## Your name and email

| Field | Editable? | Notes |
|---|---|---|
| **Handle** | No | Shown as `@yourname.veiled`. It's your permanent identity in the app. See [Your .veiled Name](your-veiled-name.md). |
| **Display name** | Yes | Appears on your public creator profile. Change it and press **Save changes**. |
| **Email** | No | Always the address you signed up with. Receipts and alerts go there. |

---

## Linking a payout wallet

Enter the Solana address Veiled should use for payouts and on-chain activity, then press **Save wallet**. Addresses from mainstream Solana wallets such as Backpack, Solflare or Phantom are all accepted.

Before saving, Veiled checks that the address looks like a real one: 32 to 44 base58 characters, which excludes `0`, `O`, `I` and `l`. If it doesn't pass, you see **Invalid wallet address** and nothing is saved, so a mistyped address doesn't quietly become your payout destination. To unlink the wallet, clear the field and save.

---

## Caps on what you spend

These caps apply to payments from your personal accounts:

| Setting | Effect |
|---|---|
| **Daily limit** | A send is refused if it would take today's settled outgoing total over this amount. `0` means unlimited. |
| **Monthly limit** | The same check against the calendar month. `0` means unlimited. |
| **Require confirmation** | Asks you to approve every payment larger than $0.50 before it goes out. On by default. |
| **Auto top-up** | Refills your main account automatically whenever its funds run short. Off by default. |

Press **Save limits** after editing either amount. Negative or non-numeric values are rejected with **Enter a valid non-negative amount**. When a payment would go over a limit, it fails with a message naming the limit from this page, for example "Amount would exceed your daily spending limit set in Settings".

Agent accounts aren't governed by these settings. Each agent has its own spend policy with a per-request maximum and a daily maximum, and the Veiled agent program enforces that policy on-chain. A setting here is a preference you can change. An agent's policy is a hard limit the agent can't get past. See [Spending Guardrails](../autonomous-agents/spending-guardrails.md).

---

## How runs are handled

| Setting | Default | Effect |
|---|---|---|
| **Stream results** | On | Shows output as it's produced instead of waiting until the run finishes |
| **Save execution history** | On | Stores the results and receipts from each run so you can look back at them |
| **Share anonymised usage data** | Off | Sends Veiled aggregate, anonymised statistics about how you use the app so the team can improve it. Your input content is never included. |

Each switch saves as soon as you click it. If saving fails, the page says "Could not save that change. Try again."

---

## Danger zone

### Export all data

Press **Export** to download one JSON file named `veiled-export-<handle>-<date>.json`. It contains your accounts, transactions, alerts, API keys and webhooks, which together make up your whole account. If you only need a decrypted transaction record for bookkeeping or a regulator, use the per-transaction disclosure flow in [Sharing Records with Auditors](../confidentiality/auditor-access.md) instead.

### Delete account

Deletion needs two clicks: **Delete account**, then **Click again to confirm**. At that point you are logged out straight away, and your account plus everything in it is scheduled to be erased for good inside 30 days. Until then, Settings shows the date you made the request and asks you to contact support if you want to cancel. Once those 30 days pass, nothing can be recovered.

Because Veiled is non-custodial, deletion removes your profile and everything stored on the platform. It can't touch transactions that have already settled on Solana. Those remain on-chain, encrypted, as they were.

---

## Related pages

- [Funding Your Balance](funding-your-balance.md)
- [Sharing Records with Auditors](../confidentiality/auditor-access.md)
