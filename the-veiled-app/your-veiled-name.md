# Your .veiled Name

Every Veiled account has a readable name, such as `maria.veiled`. People use it to pay you, so nobody has to copy or check a long base58 address.

---

## How names are written

The app writes the same name in a few ways, and all of them point to the same account:

| You might see | Where |
|---|---|
| `@maria.veiled` | Your own handle on the Accounts page and in Settings |
| `@maria` | The from and to lines when you review a payment |
| `maria.veiled` | Agent accounts, which drop the `@` |

When you type a recipient on the **Send Payment** screen, the leading `@` and the `.veiled` suffix are optional, and capital letters are treated as lowercase. `@Maria`, `maria` and `maria.veiled` all resolve to the same account.

---

## Getting your name

You don't choose a name by hand. Veiled creates one when you confirm your email, using the part of your email address before the `@`:

1. Convert it to lowercase.
2. Remove every character except `a` to `z`, `0` to `9` and `_`.
3. Keep the first 28 characters. If nothing is left, use `user`.
4. If that name is already in use, add `1`, then `2`, and so on until one is free.

For example, `Maria.Lopez@example.com` becomes `marialopez`, and a second Maria Lopez would get `marialopez1`.

### Naming rules

| Rule | Value |
|---|---|
| Allowed characters | Lowercase letters, digits, underscore |
| Length | 1 to 32 characters |
| Uniqueness | One owner per name across the whole network |

The Solana program applies the same character set and the same 32 character limit. Restricting names this way means they can go into request links and QR codes without any escaping.

---

## Can I change it?

In the app, no. **Settings** labels the handle as **Your permanent Veiled identity, cannot be changed**, and the database refuses any edit to it. So think of your name as fixed for as long as you use the app.

The on-chain registry does include a `change_handle` instruction. It moves a profile to a new name, frees the old name for someone else to take, and leaves everything else on the profile as it was. Your profile address is derived from your wallet, not from your name, so a rename at that level wouldn't change your on-chain address or invalidate past transfers made to it. Any request link that used the old name would stop resolving.

---

## Agent names

When you create an agent account, its name comes from the agent's label. Any run of characters other than letters and digits becomes a hyphen, the result is lowercased and cut to 30 characters, and a `-1`, `-2` suffix is added if the name is already taken. An agent called "Research Bot" becomes `research-bot.veiled`. Agent names can contain hyphens, which personal names can't, and can be up to 40 characters long.

Agent names appear without the `@`. When someone searching for a recipient selects one, the recipient panel shows **Agent account** as its type. Once software can spend money on your behalf, you need to see straight away whether a person or a program made a payment. See [Agent Wallets](../autonomous-agents/agent-wallets.md).

---

## What happens when someone pays your name

Your name is only a lookup step. It doesn't add anything you have to trust.

1. The sender's app finds your name among active accounts. On-chain, a `HandleClaim` account keyed to the name records its owner, so one read is enough to resolve it.
2. The app builds and signs the transfer to your real account.
3. What settles on Solana is an ordinary transfer from one address to another. Your name never appears in the transaction.

The name search returns only public details (name, account type, privacy tier and creation date) and never a balance. The payment itself has the same confidentiality as any other, as described in [Encrypted Balances and Private Payments](../confidentiality/encrypted-balances.md).

---

## Related pages

- [Moving Money](moving-money.md)
- [Activity Log](activity-log.md)
