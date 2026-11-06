# Signing Up

You can go from the sign-up form to a working Veiled account in a few minutes. There is no seed phrase to copy down and no hardware wallet to buy, and you don't need any background in crypto.

---

## Before you start

Have these ready:

- An email address you can open right away, because activation happens through a link sent to it.
- A password of at least 8 characters.
- A government-issued ID, but only if you plan to move up to a verified tier later. You can skip this for a standard account.

---

## The sign-up form

The form at `/signup` asks for five things:

| Field | Notes |
|---|---|
| First name | Required. Combined with your last name to set your display name. |
| Last name | Required. |
| Email | Required. Your login, and where receipts and alerts are sent. |
| Password | Required, minimum 8 characters. |
| How you'll use Veiled | Pick **Personal account**, **Agent account**, or **Both**. **Both** is preselected. |

Press **Create account**. If something goes wrong (an address that's already registered, a password that's too short), the reason appears in a red banner above the form and nothing is created.

---

## Confirming your email

After you submit, the page changes to **Check your email**. Open the message from Veiled and follow the link. It brings you back to the site, signs you in, and lands you on a **You're all set** screen with an **Open the app** button. If the link has expired or can't be verified, you're sent to the login page with an error so you can try again.

After that, you sign in at `/login` with the same email and password. There is a **Forgot password?** link there if you need it.

---

## What your account starts with

Confirming your email triggers automatic setup. When you first open the app you'll find:

| Item | Starting state |
|---|---|
| Profile | Display name from the name you entered (or the part of your email before the `@` if the name was blank). |
| Handle | Built from your email's local part: lowercased, with anything other than `a-z`, `0-9` and `_` removed, trimmed to 28 characters. If the name is taken, a number is added to the end (`maria`, `maria1`, `maria2`). See [Your .veiled Name](your-veiled-name.md). |
| Primary Account | A personal account called **Primary Account**, set to the confidential privacy tier, with a balance of 0.00 USDG. |
| Alerts | One **Welcome to Veiled** system alert in your feed. |
| Alert rules | Four rules. Three are on (transfers received, agents nearing their limit, view key use) and platform updates is off. See [Alert Rules](alert-rules.md). |

Nothing has been encrypted at this point, because there are no balances or transfers yet. Your first payment is the first thing that gets sealed, so privacy is in place from that payment on and you never have to turn it on.

---

## On-chain identity

At the protocol level, each Veiled user is a `Profile` account on Solana. It's a program-derived address keyed to the owner's wallet and stores the handle, a KYC tier, and an account kind (`Personal`, `Business` or `AgentOperator`). Registering also creates a `HandleClaim` account keyed to the handle. That account reserves the name and lets anyone resolve it to its owner with one lookup. Confidential balances use Token-2022 confidential transfers, Solana's built-in extension for encrypted balances, and that's what allows a transfer amount to stay hidden. A new profile starts at the `Unverified` tier. Veiled's compliance authority raises it once off-chain checks pass.

For the full layout, see [The Solana Program](../under-the-hood/solana-program.md).

---

## Verification tiers

Accounts start on **Shadow Lite**, which hides the amount on every transfer from the start. Getting it requires nothing beyond a confirmed email and a wallet attestation, and the usual transaction limits apply.

To reach **Shadow Pro** or **Shadow Enterprise**, you go through a KYC check with Veiled's verification partner. Expect to provide:

1. An image of an official photo ID such as a passport.
2. A liveness check, which is a short selfie video.

The aim is to keep your activity hidden from the public while still meeting legal requirements. During beta, US persons can't open accounts. The first supported markets are the EU, the UK, Singapore and the UAE. See [Identity Checks and the Travel Rule](../regulation/identity-checks.md) for details.

Companies use the `Business` account kind, which adds team features and higher limits. To compare every option, including agent accounts, read [Personal and Agent Accounts](../start-here/personal-and-agent-accounts.md).

---

## Putting money in

Your balance starts at zero. To fund it, open **Wallet** and use **Top up**. You can send USDG or USDC from an exchange or from a Solana wallet like Backpack, Solflare or Phantom. Both stablecoins are issued natively on Solana. Where your jurisdiction supports it, a linked bank transfer is another option. Veiled keeps a small SOL reserve topped up automatically to pay network fees, so you never need to buy or manage SOL. Step by step instructions are in [Funding Your Balance](funding-your-balance.md).

---

## Where to go next

- [Moving Money](moving-money.md)
- [Your .veiled Name](your-veiled-name.md)
- [Funding Your Balance](funding-your-balance.md)
