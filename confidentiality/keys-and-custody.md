# Keys, Signers, and Custody

Veiled is non-custodial because of how it is built, not because it promises to behave. Keys are created and kept on your device. Veiled's servers do not store them and never see them unencrypted, so control of the account stays with you.

---

## The keys on every account

| Key | Type | What it does |
|---|---|---|
| Signing key | Solana Ed25519 keypair | Owns your account and signs your transactions, exactly as in any Solana wallet |
| Decryption key | ElGamal keypair | Lets you, and only you, see the real figures behind your encrypted balance and payments |
| View key | Read-only, derived from your decryption capability | Can be handed to an auditor or a tool. It decrypts, but it can never spend. |

All three come from one root secret that your device generates during onboarding. That secret is never sent to Veiled unencrypted.

Your signing key owns your on-chain Veiled profile, a program-derived address (PDA) seeded by your wallet address. On Pro and above, a Squads multisig can sit on top for social recovery and for rotating hardware keys.

---

## Keys derived from a passkey

Instead of asking you to write down a seed phrase, Veiled derives your keys from a device passkey (WebAuthn). You get self-custody without a slip of paper to lose.

| Stage | What happens | Where |
|---|---|---|
| Create | You register with your email and a passkey: Face ID, Touch ID, Windows Hello or a hardware security key | Your device |
| Derive | A key derivation function turns the passkey into your Veiled keypair | Your device |
| Store | The private key material is encrypted and kept in the secure enclave or its equivalent | Your device |
| Use | Veiled receives your public key and your signed transactions, never the private key | Veiled servers |

Lost seed phrases are the most common way people lose self-custodied funds. Passkeys remove that failure mode without taking anything from you, since you can still export the key.

{% hint style="info" %}
Signing in to the Veiled web app uses your email address and a password. That login is separate from the keys described here and gives no one access to them.
{% endhint %}

---

## Taking your key elsewhere

You can export your private key whenever you want. Your account is controlled by an ordinary Ed25519 key, so the exported key imports into any standard Solana wallet, for example Backpack, Solflare or Phantom. Your profile and token accounts live on Solana and any Solana tool can address them. Nothing about Veiled locks you in.

Keeping an exported key safe is your job. Whoever has it has complete control of the account.

---

## Agent signers

An agent account has its own key, `agent_signer`, which the running agent uses to submit spends. It is deliberately limited:

- It can only spend tokens already in the agent's program-controlled vault, and only within the spend policy.
- It cannot change the policy, withdraw to the owner, or reactivate itself. Those actions need the owner's signature.

If an agent key is exposed, there is no race to move funds first. The owner has three responses:

| Instruction | Effect |
|---|---|
| `set_agent_status` | Pause (or resume) the agent; its vault and policy stay as they are |
| `rotate_agent_signer` | Replace the signing key while keeping the vault, policy and spend history |
| `revoke_agent` | Shut the agent down permanently and sweep what is left in the vault back to the owner |

`revoke_agent` and `withdraw_from_agent` keep working even when the protocol is paused, so an owner can always get funds out during an incident.

---

## Getting back in after losing a device

| Route | Who can use it | How it works |
|---|---|---|
| Synced or multi-device passkeys | Anyone who registered a passkey on more than one device, or with a syncing provider such as iCloud Keychain or Google Password Manager | Sign in on another synced device and you are back immediately |
| Squads recovery | Pro and above | Your designated co-signers approve a key rotation together, replacing the lost signing key without moving any funds |
| Social recovery | All tiers, after beta | Trusted contacts you nominate approve a recovery request together |
| Exported key | Anyone who exported and stored their key beforehand | Import it on any device to regain access |

No recovery path skips your key material or your multisig quorum, and that is deliberate. Veiled cannot recover the account for you, because it never had the key in the first place.

---

## The protocol's own keys

The admin keys that govern the protocol are handled carefully too. The authority that controls the emergency pause, and the authority that controls the fee schedule, each change hands in two steps: the current holder nominates a successor, and the change only takes effect when that successor signs (`begin_authority_transfer` then `accept_authority`, and `begin_fee_authority_transfer` then `accept_fee_authority`). A typo can never hand control to an address nobody holds. The admin authority is intended to move onto a Squads multisig once Veiled leaves beta.

None of these protocol keys can move user funds. Profiles keep their assets in their own wallets, and each agent vault answers only to its own agent and owner.

---

## Custody and privacy come from the same place

Your decryption key is what lets you decrypt your history, export it, and prove a transaction to someone else without relying on any third party. Only you hold it. And because view keys are separate from signing keys, giving someone the ability to read never gives them the ability to spend. See [Sharing Records with Auditors](auditor-access.md) for how disclosure works in practice.
