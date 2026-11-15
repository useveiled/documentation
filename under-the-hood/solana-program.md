# The Solana Program

Two on-chain components carry Veiled. The first is Token-2022's confidential transfer extension, which Solana provides and which does the encryption. The second is Veiled's own Anchor program, `veiled`, which turns an encrypted balance into something that works like a bank account: unique `.veiled` names, agent accounts whose limits the chain enforces, an approval queue, request-to-pay links, disclosure receipts, and $VEILED staking with fee discounts. The program is open source.

| | |
|---|---|
| Program name | `veiled` |
| Program ID | `BgJQoxfRfpQYhJLFxnq7f8GcxPoAce47MGPnq1qJiV7V` |
| Framework | Anchor, using `anchor_spl::token_interface` (works with SPL Token and Token-2022 mints) |
| Clusters declared in `Anchor.toml` | localnet, devnet |

{% hint style="info" %}
The `veiled` program keeps sender and recipient addresses public at all times. Confidentiality of amounts comes from Token-2022, described in the first section below.
{% endhint %}

---

## Part 1: Token-2022 confidential balances

### Inside an encrypted token account

A user's USDG account (USDC, also native to Solana, works the same way) is a Token-2022 token account with the confidential transfer extension turned on. Next to the usual public token fields, the extension keeps encrypted state:

```rust
pub struct ConfidentialTransferAccount {
    pub approved: bool,                        // confidential transfers switched on for this account
    pub elgamal_pubkey: ElGamalPubkey,         // key that balances are encrypted to
    pub pending_balance_lo: ElGamalCiphertext, // incoming funds not yet applied, lower bits
    pub pending_balance_hi: ElGamalCiphertext, // incoming funds not yet applied, upper bits
    pub available_balance: ElGamalCiphertext,  // spendable balance, encrypted
    pub decryptable_available_balance: AeCiphertext, // copy the owner can decrypt quickly on device
    pub allow_confidential_credits: bool,
    pub allow_non_confidential_credits: bool,
    pub pending_balance_credit_counter: u64,
    pub maximum_pending_balance_credit_counter: u64,
    pub expected_pending_balance_credit_counter: u64,
    pub actual_pending_balance_credit_counter: u64,
}
```

The pending and available balances are twisted ElGamal ciphertexts. Decrypting them takes the ElGamal private key, which is derived from the account holder's Veiled key and never leaves their device. Veiled cannot read them, and neither can any validator.

### What a confidential transfer carries

The sender's device produces the encrypted amount and its proofs, and they travel in the same Solana transaction as the transfer:

```rust
pub struct TransferProofData {
    pub new_source_ciphertext: ElGamalCiphertext,         // sender's balance after the transfer, encrypted
    pub transfer_amount_ciphertext_lo: ElGamalCiphertext, // amount, lower bits, encrypted
    pub transfer_amount_ciphertext_hi: ElGamalCiphertext, // amount, upper bits, encrypted
    pub equality_proof: Proof,  // the ciphertexts all describe the same value
    pub validity_proof: Proof,  // the amount is correctly formed
    pub range_proof: Proof,     // the amount is in range and not negative
}
```

Solana's native ZK proof program verifies these before any encrypted balance changes. Because that check runs as part of consensus, a transfer with a bad proof fails and cannot settle. The proofs establish validity without exposing the amount.

---

## Part 2: The `veiled` program

### Accounts and PDA seeds

Every program account is a PDA. Indexes are `u64` values encoded little-endian.

| Account | Seeds | Purpose |
|---|---|---|
| `ProtocolConfig` | `"config"` | Admin authority, compliance authority, pause switch, fee routing |
| `FeeConfig` | `"fees"` | Fee schedule, discount tiers, total staked |
| Stake vault (token account) | `"stake_vault"` | Holds staked $VEILED; its token authority is `FeeConfig` |
| `StakePosition` | `"stake"`, staker | One wallet's staked $VEILED |
| `Profile` | `"profile"`, owner | One identity per wallet |
| `HandleClaim` | `"handle"`, handle bytes | Marks a handle as taken and points it at the owner |
| `Agent` | `"agent"`, owner, agent index | An agent and its spend policy |
| Agent vault (token account) | `"vault"`, agent PDA | The agent's funds; its token authority is the `Agent` PDA |
| `PendingApproval` | `"pending"`, agent PDA, pending index | A spend waiting for the owner |
| `PaymentRequest` | `"request"`, requester, request index | A request-to-pay link or QR code |
| `DisclosureReceipt` | `"disclosure"`, owner, disclosure index | Record that a disclosure was produced |

The agent, request and disclosure indexes come from counters on the parent account (`Profile.agent_count`, `Profile.request_count`, `Profile.disclosure_count`, `Agent.pending_count`), so each new record gets the next free address.

### Limits and constants

| Constant | Value |
|---|---|
| `MAX_HANDLE_LEN` | 32 |
| `MAX_LABEL_LEN` | 40 |
| `MAX_TX_REFERENCE_LEN` | 88 |
| `MAX_ALLOWED_RECIPIENTS` | 10 |
| `MAX_FEE_TIERS` | 8 |
| `BPS` | 10,000 |
| `WINDOW_SECONDS` | 86,400 (24 hours) |

### Enums

| Enum | Variants |
|---|---|
| `KycTier` | `Unverified`, `Basic`, `Enhanced` |
| `AccountKind` | `Personal`, `Business`, `AgentOperator` |
| `AutonomyTier` | `Supervised`, `SemiAutonomous`, `FullyAutonomous` |
| `AgentStatus` | `Active`, `Paused`, `Revoked` |
| `RequestStatus` | `Open`, `Fulfilled`, `Cancelled` |

### Account fields

**`ProtocolConfig`**

| Field | Type | Meaning |
|---|---|---|
| `authority` | `Pubkey` | Admin key: rotates authorities, flips the pause, sets fee routing. Planned to move to a Squads multisig after beta. |
| `pending_authority` | `Option<Pubkey>` | Nominee in a two-step handoff |
| `compliance_authority` | `Pubkey` | The only key that can write a profile's KYC tier |
| `paused` | `bool` | Circuit breaker for fund-moving instructions |
| `treasury` | `Pubkey` | Wallet that collects protocol fees |
| `fees_enabled` | `bool` | Whether fee-charging instructions charge anything; starts `false` |
| `bump` | `u8` | |

**`FeeConfig`**

| Field | Type | Meaning |
|---|---|---|
| `authority` | `Pubkey` | Key that can retune fees |
| `pending_authority` | `Option<Pubkey>` | Nominee in a two-step handoff |
| `veiled_mint` | `Pubkey` | The $VEILED mint used for loyalty weight |
| `base_fee_bps` | `u16` | Rate before discounts |
| `fee_cap` | `u64` | Maximum fee in settlement-token base units; `0` means no cap |
| `held_weight_bps` | `u16` | How much held (unstaked) $VEILED counts, in basis points of full weight |
| `tiers` | `Vec<FeeTier>` | Up to 8 tiers, ascending by `min_weight`; each `FeeTier` has `min_weight: u64` and `discount_bps: u16` |
| `total_staked` | `u64` | $VEILED currently in the stake vault |
| `bump` | `u8` | |

**`Profile`**

| Field | Type | Meaning |
|---|---|---|
| `owner` | `Pubkey` | Wallet the profile belongs to |
| `handle` | `String` (max 32) | Lowercase name without the `.veiled` suffix |
| `kyc_tier` | `KycTier` | Starts at `Unverified` |
| `account_kind` | `AccountKind` | |
| `agent_count` | `u64` | Agents created; also the next agent's index |
| `request_count` | `u64` | Payment requests created |
| `disclosure_count` | `u64` | Disclosure receipts filed |
| `created_at`, `updated_at` | `i64` | Unix timestamps |
| `bump` | `u8` | |

**`HandleClaim`**: `owner: Pubkey`, `bump: u8`. Because the address is derived from the handle itself, checking whether `ada.veiled` is free and resolving it to a wallet are both one account fetch.

**`Agent`**

| Field | Type | Meaning |
|---|---|---|
| `owner` | `Pubkey` | Owning profile's wallet |
| `agent_signer` | `Pubkey` | Key the running agent signs spends with |
| `label` | `String` (max 40) | Short name, for example `research-bot` |
| `autonomy_tier` | `AutonomyTier` | How much the agent can settle without a human |
| `status` | `AgentStatus` | |
| `mint` | `Pubkey` | The single mint this agent may move (USDG by default) |
| `per_tx_limit` | `u64` | Largest single settled spend |
| `daily_limit` | `u64` | Largest total per 24 hour window |
| `hitl_threshold` | `u64` | For `SemiAutonomous`, spends above this go to the queue |
| `window_start` | `i64` | When the current 24 hour window opened |
| `window_spent` | `u64` | Settled so far in that window |
| `allowlist_enabled` | `bool` | When true, only listed wallets can be paid |
| `allowed_recipients` | `Vec<Pubkey>` (max 10) | Permitted recipient wallets |
| `pending_count` | `u64` | Next pending-approval index |
| `index` | `u64` | Position under the owner, used in the PDA seeds |
| `created_at`, `updated_at` | `i64` | |
| `bump`, `vault_bump` | `u8` | |

**`PendingApproval`**: `agent`, `owner`, `recipient`, `mint` (all `Pubkey`), `amount: u64`, `invoice_id: [u8; 32]` (x402 invoice reference, zero for a plain spend), `memo_hash: [u8; 32]` (hash of an encrypted off-chain memo shown to the approver), `index: u64`, `created_at: i64`, `bump: u8`.

**`PaymentRequest`**: `requester`, `receiver`, `mint` (all `Pubkey`), `is_confidential: bool`, `amount: u64` (zero when confidential), `amount_commitment: [u8; 32]` (equal to `keccak256(amount_le, blinding)`), `memo_hash: [u8; 32]`, `status: RequestStatus`, `index: u64`, `created_at: i64`, `expires_at: i64`, `bump: u8`. The `receiver` can differ from the requester, which lets a business send receipts straight to a treasury wallet.

**`DisclosureReceipt`**: `profile: Pubkey`, `tx_reference: String` (max 88, usually a base58 transaction signature), `viewer_hash: [u8; 32]` (identifies the viewer without naming them on-chain), `disclosure_commitment: [u8; 32]` (hash of the proof payload), `index: u64`, `filed_at: i64`, `bump: u8`.

**`StakePosition`**: `owner: Pubkey`, `amount: u64`, `staked_since: i64` (set when a position opens, kept on top-ups, cleared on a full unstake), `bump: u8`.

---

### Instructions

"Pause" marks instructions that fail with `ProtocolPaused` while `ProtocolConfig.paused` is true. Everything else keeps working during a pause, so an owner can always reconfigure, pause, revoke or withdraw from an agent mid incident.

#### Protocol administration

| Instruction | Arguments | Signer | Notes |
|---|---|---|---|
| `init_config` | `compliance_authority` | Becomes `authority` | Fees start disabled, treasury unset |
| `update_config` | `new_authority`, `new_compliance_authority` | `authority` | Direct rotation; clears any pending handoff |
| `begin_authority_transfer` | `new_authority: Option<Pubkey>` | `authority` | Nominates a successor; `None` cancels |
| `accept_authority` | none | The nominee | Fails with `NotPendingAuthority` for anyone else |
| `set_pause` | `paused` | `authority` | |
| `set_fee_routing` | `treasury`, `fees_enabled` | `authority` | Enabling fees requires a non-default treasury |

#### Fees and staking

| Instruction | Arguments | Signer | Notes |
|---|---|---|---|
| `init_fees` | none | Config `authority` | Creates `FeeConfig` and the stake vault with the defaults below |
| `set_fee_schedule` | `base_fee_bps`, `fee_cap`, `held_weight_bps` | Fee `authority` | Both bps values must be at most 10,000 |
| `set_fee_tiers` | `tiers: Vec<FeeTier>` | Fee `authority` | At most 8, strictly ascending `min_weight`, each discount at most 10,000 bps |
| `begin_fee_authority_transfer` | `new_authority: Option<Pubkey>` | Fee `authority` | Same two-step pattern as the config |
| `accept_fee_authority` | none | The nominee | |
| `stake` | `amount` | Staker | Creates the `StakePosition` if needed; moves $VEILED into the vault |
| `unstake` | `amount` | Staker | No lockup; vault releases tokens with `FeeConfig` signing |

#### Identity

| Instruction | Arguments | Signer | Notes |
|---|---|---|---|
| `create_profile` | `handle`, `account_kind` | Owner | Creates `Profile` and `HandleClaim`; a taken handle fails because its claim account already exists |
| `change_handle` | `new_handle` | Owner | Closes the old claim (rent back to owner) so anyone can take that name, opens a new claim, leaves everything else on the profile as it was |
| `set_kyc_tier` | `kyc_tier` | `compliance_authority` | Records an off-chain verification result; moves no funds |

Handles must be 1 to 32 characters drawn from lowercase letters, digits and underscores, which keeps them safe inside links and QR payloads.

#### Agents

| Instruction | Arguments | Signer | Pause | Notes |
|---|---|---|---|---|
| `create_agent` | `agent_signer`, `label`, `autonomy_tier`, `per_tx_limit`, `daily_limit`, `hitl_threshold`, `allowlist_enabled`, `allowed_recipients` | Owner (needs a profile) | | Creates the `Agent` and its vault for the given mint; status starts `Active` |
| `update_spend_policy` | `per_tx_limit`, `daily_limit`, `hitl_threshold`, `allowlist_enabled`, `allowed_recipients` | Owner | | The agent key cannot loosen its own policy |
| `set_agent_status` | `status` | Owner | | Toggles `Active` and `Paused`; cannot set or leave `Revoked` |
| `rotate_agent_signer` | `new_signer` | Owner | | Swaps the agent key, keeping vault, policy and history |
| `revoke_agent` | none | Owner | | Permanent; sweeps the whole vault to the owner's token account |
| `fund_agent` | `amount` | Anyone | | Deposits into the vault; rejected once revoked |
| `withdraw_from_agent` | `amount` | Owner | | Pulls part of the vault back; works in any status |
| `pay_invoice` | `amount`, `invoice_id` | `agent_signer` | Yes | Settles immediately if the policy allows |
| `queue_invoice` | `amount`, `invoice_id`, `memo_hash` | `agent_signer` | Yes | Opens a `PendingApproval`; the agent key pays its rent |
| `approve_pending` | none | Owner | Yes | Re-checks the policy, settles, closes the pending record |
| `reject_pending` | none | Owner | | Closes the pending record; nothing had moved |

Every policy update is validated the same way: `hitl_threshold`, `per_tx_limit` and `daily_limit` must all be above zero, `per_tx_limit` cannot exceed `daily_limit`, the allowlist holds at most 10 wallets, and an enabled allowlist cannot be empty (pausing the agent is how you stop all spending). A default (all zero) public key passed as `agent_signer` or `new_signer` is rejected.

#### Payment requests

| Instruction | Arguments | Signer | Pause | Notes |
|---|---|---|---|---|
| `create_request` | `is_confidential`, `amount`, `amount_commitment`, `memo_hash`, `expires_at` | Requester (needs a profile) | | Expiry must be in the future; a plain request needs `amount > 0` |
| `fulfill_request` | `amount`, `blinding` | Payer | Yes | Checks the amount or commitment, pays the receiver, charges any fee to the payer on top |
| `cancel_request` | none | Requester | | Only while `Open` |
| `close_request` | none | Requester | | Returns rent; allowed when fulfilled or cancelled, or when open but expired |

For a confidential request, the requester stores `amount = 0` and a commitment. The payer learns the amount and the blinding factor off-chain (normally from the encrypted memo), and `fulfill_request` recomputes `keccak256(amount_le_bytes, blinding)` and compares. A payer who was given the wrong figure is caught without the real amount ever being written on-chain. An amount of zero is always refused, so a request cannot be marked paid without tokens moving.

#### Selective disclosure

| Instruction | Arguments | Signer | Notes |
|---|---|---|---|
| `file_disclosure` | `tx_reference`, `viewer_hash`, `disclosure_commitment` | Owner (needs a profile) | Writes a timestamped `DisclosureReceipt`. The decrypted-amount proof is made on the client against the view key and stays off-chain. |

See [Sharing Records with Auditors](../confidentiality/auditor-access.md) for how disclosures are produced.

---

### How an agent spend is decided

The autonomy tier decides which path a spend takes. The policy itself binds every tier.

| Tier | `pay_invoice` (settle now) | `queue_invoice` (ask a human) |
|---|---|---|
| `Supervised` | Always refused with `TierRequiresApproval` | Any amount |
| `SemiAutonomous` | Only if `amount <= hitl_threshold` (else `AmountExceedsHitlThreshold`) | Only if `amount > hitl_threshold` (else `AmountWithinHitlThreshold`) |
| `FullyAutonomous` | Any amount the policy allows | Any amount, if the agent wants a human to look |

Checks on each path:

| Check | `pay_invoice` | `queue_invoice` | `approve_pending` |
|---|---|---|---|
| Protocol not paused | Yes | Yes | Yes |
| Agent status is `Active` | Yes | Yes | Yes |
| `amount <= per_tx_limit` | Yes | Yes | Yes, against the current policy |
| Recipient on allowlist (if enabled) | Yes | Yes | Yes, against the current policy |
| 24 hour total within `daily_limit` | Yes | No | Yes |
| Vault holds `amount + fee` | Yes | No | Yes |

The 24 hour window is reset lazily. When a spend arrives 86,400 seconds or more after `window_start`, the program sets `window_start` to now and `window_spent` to zero before checking, so no separate reset call is ever needed.

The vault is a token account whose authority is the `Agent` PDA, so only this program can move its contents. A leaked agent key can spend no faster and no further than the policy allows, and the owner can neutralise it immediately with `set_agent_status`, `rotate_agent_signer` or `revoke_agent`. Vault movements use the token interface's `transfer_checked`, with the amount passed as a plain instruction argument.

---

### Fee logic

All fee math lives in one function, `quote_fee`, which the app and SDK mirror so the displayed rate always matches the charged rate.

```
weight   = staked + held * held_weight_bps / 10_000
discount = discount_bps of the highest tier with min_weight <= weight
gross    = amount * base_fee_bps / 10_000
net      = gross * (10_000 - discount) / 10_000
fee      = min(net, fee_cap)          // fee_cap of 0 means no cap
```

- `staked` is the payer's `StakePosition.amount`; `held` is the balance of the payer's $VEILED token account. Both accounts are optional; leaving one out simply counts it as zero.
- Integer division rounds every step down.
- If `fees_enabled` is false, no fee is charged and the fee accounts are not needed. If it is true, a missing `FeeConfig` fails with `MissingFeeAccounts`, and a treasury token account not owned by `ProtocolConfig.treasury` fails with `InvalidFeeAccount`.
- The fee is always added on top of the amount, so the recipient receives the full amount.

**Where fees apply**

| Instruction | Whose weight counts | Who pays |
|---|---|---|
| `pay_invoice` | The agent's owner | The agent vault |
| `approve_pending` | The agent's owner | The agent vault |
| `fulfill_request` | The payer | The payer's token account |

Every other instruction, including `fund_agent`, `withdraw_from_agent` and `revoke_agent`, is fee free. Each charge emits `ProtocolFeeCharged`.

**Defaults written by `init_fees`**

| Setting | Value |
|---|---|
| `base_fee_bps` | 10 (0.10%) |
| `fee_cap` | 5,000,000 (5 USDG at 6 decimals) |
| `held_weight_bps` | 5,000 (held $VEILED counts at half weight) |

| Tier | `min_weight` (whole $VEILED) | `discount_bps` | Effective rate |
|---|---|---|---|
| 1 | 1,000 | 1,000 (10% off) | 0.090% |
| 2 | 10,000 | 2,500 (25% off) | 0.075% |
| 3 | 100,000 | 5,000 (50% off) | 0.050% |
| 4 | 1,000,000 | 7,500 (75% off) | 0.025% |

Tier weights are scaled by the $VEILED mint's decimals when written.

**Worked example.** An agent pays a 2,400 USDG invoice. Its owner has 7,000 $VEILED staked and 8,000 more sitting in their wallet. Weight is 7,000 + 8,000 x 0.5 = 11,000, which reaches tier 2. Gross fee is 2,400 x 0.10% = 2.40 USDG, and 25% off brings it to 1.80 USDG, under the cap. The vault must hold at least 2,401.80 USDG; the recipient gets 2,400 and the treasury 1.80.

For an 8,000 USDG payment with no $VEILED weight, the gross fee is 8 USDG, so the 5 USDG cap applies.

---

### Events

The program emits Anchor events, which Veiled's indexer turns into the activity feed and webhooks.

| Area | Events |
|---|---|
| Configuration | `ConfigAuthorityUpdated`, `AuthorityTransferStarted`, `AuthorityTransferAccepted`, `ProtocolPauseToggled`, `FeeRoutingUpdated`, `FeeScheduleUpdated`, `TiersUpdated` |
| Identity | `ProfileCreated`, `HandleChanged`, `KycTierUpdated` |
| Agents | `AgentCreated`, `SpendPolicyUpdated`, `AgentStatusChanged`, `AgentSignerRotated`, `AgentFunded`, `AgentDefunded`, `AgentPaymentExecuted`, `ApprovalRequired`, `PendingPaymentApproved`, `PendingPaymentRejected` |
| Fees and staking | `ProtocolFeeCharged`, `Staked`, `Unstaked` |
| Requests | `PaymentRequestCreated`, `PaymentRequestFulfilled`, `PaymentRequestCancelled`, `PaymentRequestClosed` |
| Disclosure | `DisclosureFiled` |

---

### Errors

Anchor numbers custom errors from 6000 in declaration order.

| Code | Variant | Raised when |
|---|---|---|
| 6000 | `InvalidHandleLength` | Handle is empty or longer than 32 characters |
| 6001 | `InvalidHandleCharacters` | Handle has anything other than lowercase letters, digits or underscores |
| 6002 | `UnauthorizedComplianceAuthority` | `set_kyc_tier` signed by the wrong key |
| 6003 | `ProtocolPaused` | A fund-moving instruction runs during a pause |
| 6004 | `UnauthorizedAgentOwner` | Owner-only agent instruction signed by someone else |
| 6005 | `UnauthorizedAgentSigner` | Spend signed by a key other than `agent_signer` |
| 6006 | `InvalidLabelLength` | Label is empty or longer than 40 characters |
| 6007 | `InvalidHitlThreshold` | `hitl_threshold` is zero |
| 6008 | `InvalidSpendAmount` | An amount or limit is zero where it must be positive |
| 6009 | `PerTxLimitExceedsDailyLimit` | `per_tx_limit` is above `daily_limit` |
| 6010 | `TooManyAllowedRecipients` | More than 10 allowlist entries |
| 6011 | `AgentNotActive` | Spend or approval while the agent is not `Active` |
| 6012 | `AgentAlreadyRevoked` | Acting on a revoked agent, or passing `Revoked` to `set_agent_status` |
| 6013 | `SameSigner` | `rotate_agent_signer` given the current key |
| 6014 | `TierRequiresApproval` | `pay_invoice` from a `Supervised` agent |
| 6015 | `AmountExceedsHitlThreshold` | `SemiAutonomous` direct spend above the threshold |
| 6016 | `AmountWithinHitlThreshold` | `SemiAutonomous` queued spend at or below the threshold |
| 6017 | `PerTransactionLimitExceeded` | Amount above `per_tx_limit` |
| 6018 | `DailyLimitExceeded` | Amount would push the window total past `daily_limit` |
| 6019 | `RecipientNotAllowed` | Recipient missing from an enabled allowlist |
| 6020 | `InsufficientVaultBalance` | Vault cannot cover amount plus fee, or a withdrawal is too large |
| 6021 | `InvalidExpiry` | Request expiry is not in the future |
| 6022 | `RequestNotOpen` | Fulfilling or cancelling a request that is not `Open` |
| 6023 | `RequestExpired` | Fulfilling after `expires_at` |
| 6024 | `RequestCommitmentMismatch` | Amount and blinding do not hash to the stored commitment |
| 6025 | `RequestAmountMismatch` | Plain request paid with a different amount |
| 6026 | `Unauthorized` | Generic signer or account ownership mismatch |
| 6027 | `NotPendingAuthority` | Handoff accepted by a key that was not nominated |
| 6028 | `InvalidFeeConfig` | Bad fee parameters or tiers, enabling fees with no treasury, or a default public key where a real one is required |
| 6029 | `InvalidTxReferenceLength` | `tx_reference` is empty or longer than 88 characters |
| 6030 | `InsufficientStakedBalance` | Unstaking more than is staked |
| 6031 | `MissingFeeAccounts` | Fees are on but a needed fee account was not supplied |
| 6032 | `InvalidFeeAccount` | A fee or staking account does not match the fee configuration or the payer |
| 6033 | `InvalidMint` | A token account or mint differs from the agent's or request's mint |
| 6034 | `AmountOverflow` | Arithmetic overflow while adding amounts |
| 6035 | `EmptyAllowlist` | Allowlist enabled with no entries |
| 6036 | `RequestStillOpen` | Closing an open request before it expires |

---

## Shielded pool

The deepest privacy layer goes beyond encrypted amounts and is not part of the `veiled` program. Account state is committed as leaves in an on-chain Merkle commitment tree, while the leaf data stays with the account owner off-chain. A spend proves in zero knowledge that the spender owns some unspent leaf, without saying which. The proofs are Groth16, so they are small and cheap to check; they are generated on the client from Circom and Noir circuits and verified on-chain by a Rust verifier using Solana's alt_bn128 syscalls.

To receive without being linked, a recipient publishes a stealth meta-address once. Each payer derives a new one-time address from it for every payment, and chain analysis cannot tie those addresses back to the recipient.

---

## The off-chain index

Veiled populates a PostgreSQL index from program event streams so the app can search and render quickly. It covers address to handle mappings, the public parts of each transaction (sender, receiver, timestamp, signature) and webhook subscriptions and delivery state. It is a cache of what the chain already says, not a ledger of record. More on the surrounding services in [System Design](system-design.md).

---

## Related pages

- [System Design](system-design.md)
- [Security Assumptions](security-model.md)
- [Spending Guardrails](../autonomous-agents/spending-guardrails.md)
