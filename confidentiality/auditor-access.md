# Sharing Records with Auditors

Encrypted amounts do not make your records impossible to check. When a landlord, a tax office or an auditor needs to see something, you can prove exactly the transactions they need, to exactly the person who needs them, and keep everything else sealed.

---

## The foundation: keys that read but cannot spend

View keys are the foundation for any disclosure you make in Veiled. The key that decrypts your balance is not the key that signs for your funds, so you can let someone read encrypted data without giving them any power over your money. A view key decrypts. It cannot sign, and it cannot move a single token. That split is what lets you share records while custody stays with you. [Keys, Signers, and Custody](keys-and-custody.md) describes each key in detail.

---

## Proving one transaction

A selective disclosure reveals the amount of one transaction to one recipient, and is signed so the recipient knows you did not invent it.

**When you would use it**

- A landlord, employer or business partner asks for proof that you paid
- A tax authority or auditor asks about a particular transaction
- A counterparty needs Travel Rule screening information (see [Identity Checks and the Travel Rule](../regulation/identity-checks.md))

**Doing it in the app**

1. Go to **Activity** and open the transaction. Disclosure is available on settled transactions.
2. Select **Generate compliance disclosure**.
3. Once it is generated, the same button becomes **View compliance disclosure**, which shows when the disclosure was made. A view key covering that transaction's proof and settlement details is produced for the compliance review.

The proof that decrypts the amount is generated on your side against your view key and never goes on-chain. Because it can be checked against the ciphertext Solana already holds, the recipient verifies it against the chain itself rather than taking Veiled's word for it.

**The on-chain receipt**

You can also leave a timestamped public record that a disclosure took place. The `file_disclosure` instruction, signed by the profile owner, creates a `DisclosureReceipt` account containing:

| Field | Contents |
|---|---|
| `tx_reference` | The transfer being disclosed, usually its base58 transaction signature (up to 88 characters) |
| `viewer_hash` | A hash identifying who received the disclosure, so the receipt does not name them |
| `disclosure_commitment` | A hash of the proof payload, for matching against the off-chain proof |
| `filed_at` | When the receipt was written |

The receipt only proves that a disclosure happened. It contains no amount and no proof.

**Scope.** A disclosure covers the one transaction you picked and nothing else in your history. If someone needs to keep reading an account over time, such as an auditor on a year-long engagement, issue a scoped view key instead and revoke it when the work is done.

---

## Exporting your records

You can download your data in one go from **Settings**, under **Danger zone**, with **Export all data**. The result is a single JSON file covering your accounts, transactions, alerts, API keys and webhooks, named after your handle and the export date.

The file is assembled in your browser and saved straight to your device. Use it for tax season, bookkeeping, or simply keeping your own archive.

---

## Things Veiled will not do

- Decrypt your transactions for someone else. Every disclosure has to start with you and be signed by you.
- Hold an unencrypted record of your payments on its servers.
- Create a disclosure proof that verifies, unless your keys are involved. This is not a policy choice: without your keys it is technically impossible, so no request or order can make Veiled do it for you.

---

## How this fits regulation

This is what lets Veiled follow the "confidentiality, not anonymity" approach set out in [Our Legal Posture](../regulation/legal-posture.md). Your identity is checked through standard KYC when you sign up, wherever your tier calls for it. Anyone can see the addresses involved. The figures stay hidden unless you reveal them yourself or a lawful order served on you, the account holder, obliges you to.

---

## Related pages

- [Identity Checks and the Travel Rule](../regulation/identity-checks.md)
- [Keys, Signers, and Custody](keys-and-custody.md)
