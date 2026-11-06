# Funding Your Balance

The **Wallet** page covers your primary account's USDG. It shows the balance, the Solana wallet linked to the account, recent activity, and the steps to add more funds.

---

## The balance card

The large number at the top is your **Available balance**, which is the USDG your primary account can spend right now. Below it is the Solana wallet linked to your profile, shown in shortened form. If you haven't linked a wallet, the card says **No wallet linked, add one in Settings**. Link one in [Profile and Preferences](profile-and-preferences.md) before you expect external payouts.

---

## Adding funds

Press **Top up** on the balance card and follow these steps:

1. **Pick an amount.** Under **Add USDG**, choose $10, $25, $50 or $100, or type your own amount in **Custom**. Zero, negative or empty amounts are rejected with "Enter an amount greater than zero."
2. **Press Continue.** The panel shows **Send $X USDG to this address** along with a Solana deposit address. Use **Copy** to put the address on your clipboard.
3. **Make the transfer.** Send the USDG from any Solana wallet (Backpack, Phantom, Solflare or similar) or from an exchange. USDC is accepted too, since both stablecoins are issued natively on Solana, and either one confirms within a slot or two.
4. **Press Transferred.** This tells Veiled to look for your deposit. You can press **Back** if you want to change the amount first.

The panel then switches to **Verifying transaction** while Veiled checks for the deposit on Solana. This normally takes a few seconds. When it succeeds you'll see **Top up complete**, your balance updates without a page reload, and **Done** closes the panel. If verification fails, the panel returns to the amount step and shows the reason.

A deposit shows up in your history as a `Public` payment from **External funding source**, with the memo **Top up**.

---

## The three stats

Below the balance card is a row of three figures:

| Stat | How it's calculated |
|---|---|
| **Spent this month** | Settled payments out of your primary account since 00:00 on the 1st of the current month |
| **Total topped up** | Everything your primary account has received from external, non-Veiled sources |
| **Avg settlement time** | The average finality, in milliseconds, of the settled payments in the recent list below |

A figure shows `n/a` when there's no data for it yet.

---

## Recent transactions

The **Transaction history** list at the bottom shows your primary account's 50 most recent payments, newest first. Each entry shows:

- A **↓** for money in (with the amount in green and a `+` sign) or a **↑** for money out.
- The counterparty, plus the memo if there is one.
- The date and time, with the status added when it isn't `settled`.

This list covers only your primary account. To see every account you own, agents included, with status filters and full receipts, use the [Activity Log](activity-log.md).

---

## Costs of a top-up

You won't see a Veiled network charge on any transfer. Solana's own fee, a fraction of a cent paid in SOL, comes out of your account's SOL reserve without any action from you. As a result the recipient gets exactly the amount you send.

---

## Related pages

- [Profile and Preferences](profile-and-preferences.md)
- [Spending Insights](spending-insights.md)
