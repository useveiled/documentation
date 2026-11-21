# Identity Checks and the Travel Rule

This page explains how Veiled verifies the people and businesses behind accounts, and how cross-border compliance works in practice.

---

## What each tier asks for

How much verification you go through depends on your tier. The deeper the privacy, the deeper the check.

### Shadow Lite and Shadow Agent

All this tier asks for is your email plus an attestation signed by your wallet on-chain. Veiled collects no identity documents at this tier. Agents don't verify separately; they rely on whoever operates them.

### Shadow Pro

Moving to Shadow Pro means verifying who you are, once, with Persona or another regulated KYC vendor Veiled works with:

1. Take a picture of your passport, driving licence or national ID card and upload it.
2. Pass a liveness check: a short selfie video that is matched to the photo on your ID.
3. Show where you live with a current bank statement or utility bill.

Most verifications finish within minutes. Some are sent for manual review.

### Shadow Enterprise

Shadow Enterprise needs complete KYB documentation, meaning incorporation records and beneficial ownership details. The compliance team reviews these directly before Layer 4 privacy features are switched on.

---

## Limits by tier

| Tier | Daily limit | What you provide |
|---|---|---|
| Shadow Lite and Shadow Agent | Up to $2,500 | Verified email plus a wallet attestation recorded on-chain |
| Shadow Pro | Up to $25,000 | Government ID, liveness check, and proof of address |
| Shadow Enterprise | Custom, by negotiation | KYB documentation and disclosure of beneficial ownership |

Anything above the Enterprise baseline is decided case by case and usually involves direct review by compliance.

---

## How the Travel Rule is handled

Under the Travel Rule, financial institutions have to share originator and beneficiary information on transfers above a threshold set by each jurisdiction. In most of Veiled's initial markets the FATF baseline of USD/EUR 1,000 applies. Where a regulated institution is involved in a transfer, Here is what Veiled provides for that case:

| Step | What happens |
|---|---|
| Screening the counterparty | Large transfers are held until the receiving address clears sanctions and risk screening, which runs on Chainalysis or TRM Labs data. |
| Disclosing to the institution | If the receiving institution is regulated and needs Travel Rule data, the required originator and beneficiary information goes directly to that institution's compliance system using view key-based disclosure. It does not go through the public chain. |
| Keeping the amount encrypted | The transfer amount stays encrypted on-chain the whole time. The Travel Rule is about identity information, not transaction amounts, and Veiled handles the two separately. Compliance data travels through compliance channels, and plaintext amounts never appear on the public ledger. |

Because Veiled never holds customer funds, obligations that apply to custodial institutions therefore stay with the regulated institutions on each side of a transfer, and with agent operators for whatever their agents do. What Veiled supplies is the disclosure tooling that lets those parties meet their obligations without exposing anything publicly. For how disclosure works, see [Sharing Records with Auditors](../confidentiality/auditor-access.md).

---

## When a transfer gets a closer look

A transfer can be flagged for additional review if:

- it is larger than your tier's limit,
- it goes to or comes from an address that sanctions or risk screening has flagged, or
- it fits a pattern that looks like structuring, such as several transfers sitting just below a reporting threshold.

A flagged transaction may be held while compliance reviews it manually. If that happens to a transfer you started, the app will notify you.

---

## Keep reading

- [Our Legal Posture](legal-posture.md)
- [Sharing Records with Auditors](../confidentiality/auditor-access.md)
