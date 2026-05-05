# Phase 3 - Hedera Token Service (HTS)

**Duration:** 5 weeks (4 dev + 1 testing) &nbsp;|&nbsp; **Status:** ⏳ Pending &nbsp;|&nbsp; **Stage:** 3 of 6

← [Back to ROADMAP](../../ROADMAP.md)

---

## Overview

Phase 3 implements the complete Hedera Token Service (HTS) - the most critical service for NemorixPay and the broader DeFi ecosystem on Hedera. HTS allows creating and managing tokens natively at the protocol level, without smart contracts. This results in dramatically lower fees, faster finality, and built-in compliance features.

> **Key advantage over Stellar:** HTS native KYC eliminates the need for a costly anchor license. Compliance is managed on-chain by the token operator at $0.0001 per transfer - no third-party intermediary required.


## Week 1 - Fungible Token Creation

**Goal:** Implement token creation, supply management, and basic token lifecycle.

### Tasks

- [ ] `TokenCreateTransaction` - create fungible token with full configuration:
  - `setTokenName()`, `setTokenSymbol()`, `setDecimals()`, `setInitialSupply()`
  - `setMaxSupply()`, `setSupplyType()` (finite / infinite)
  - `setTreasuryAccountId()` - receives initial supply
  - Keys: `setAdminKey()`, `setSupplyKey()`, `setFreezeKey()`, `setWipeKey()`, `setKycKey()`, `setPauseKey()`, `setFeeScheduleKey()`
  - `setFreezeDefault()` - whether new accounts start frozen
  - `setTokenMemo()`
- [ ] `TokenMintTransaction` - mint additional supply (requires `supplyKey`)
- [ ] `TokenBurnTransaction` - burn tokens from treasury (requires `supplyKey`)
- [ ] `TokenDeleteTransaction` - delete token (requires `adminKey`)
- [ ] `TokenUpdateTransaction` - update name, symbol, memo, keys (requires `adminKey`)

### Code example

```dart
final tokenId = await TokenCreateTransaction()
    .setTokenName("NemorixUSD")
    .setTokenSymbol("NMUSD")
    .setDecimals(6)
    .setInitialSupply(1000000)
    .setTreasuryAccountId(operatorId)
    .setAdminKey(operatorKey.publicKey)
    .setSupplyKey(operatorKey.publicKey)
    .setFreezeDefault(false)
    .execute(client)
    .then((receipt) => receipt.tokenId!);

print('Token created: $tokenId');
```

## Week 2 - Associate, Transfer & Allowances

**Goal:** Implement token association and atomic transfers, the core remittance flow.

### Tasks

- [ ] `TokenAssociateTransaction` - link token to account (required before receiving)
- [ ] `TokenDissociateTransaction` - unlink token from account (balance must be zero)
- [ ] `TransferTransaction` (tokens) - atomic multi-asset transfer:
  - `addTokenTransfer(tokenId, accountId, amount)`
  - `addNftTransfer(tokenId, senderAccountId, receiverAccountId, serialNumber)`
  - Combined HBAR + token transfers in a single transaction
- [ ] `AccountAllowanceApproveTransaction` - delegate token spending to third party
- [ ] `AccountAllowanceDeleteTransaction` - revoke allowances

### Code example

```dart
// Atomic remittance: HBAR network fee + USDC amount - single transaction
await TransferTransaction()
    .addHbarTransfer(senderAccount, Hbar.fromTinybars(-100)) // fee
    .addHbarTransfer(feeCollector,  Hbar.fromTinybars(100))
    .addTokenTransfer(usdcTokenId, senderAccount,   -50000000) // 50 USDC
    .addTokenTransfer(usdcTokenId, receiverAccount,  50000000)
    .execute(client);
```

> This atomic transfer pattern is key for NemorixPay: sending the network fee and the remittance amount in a single transaction cuts costs and latency in half compared to two separate transactions.

## Week 3 - NFTs + Compliance Features

**Goal:** Implement NFT support and the full compliance toolkit (KYC, freeze, pause, wipe).

### Tasks

**NFTs**
- [ ] `TokenCreateTransaction` (NFT) - `TokenType.NON_FUNGIBLE_UNIQUE`
- [ ] `TokenMintTransaction` (NFT) - mint with individual metadata per serial
- [ ] NFT transfer via `TransferTransaction.addNftTransfer()`
- [ ] `TokenNftInfoQuery` - metadata and ownership by serial number

**Compliance**
- [ ] `TokenFreezeTransaction` - freeze specific account for a token
- [ ] `TokenUnfreezeTransaction` - unfreeze account
- [ ] `TokenGrantKycTransaction` - grant KYC to account (on-chain compliance)
- [ ] `TokenRevokeKycTransaction` - revoke KYC from account
- [ ] `TokenPauseTransaction` - pause all transfers of a token (emergency halt)
- [ ] `TokenUnpauseTransaction` - resume transfers
- [ ] `TokenWipeTransaction` - remove tokens from a specific account (wipe key)

### Code example - Native KYC flow

```dart
// Grant KYC to new user - no anchor license required
await TokenGrantKycTransaction()
    .setTokenId(usdcTokenId)
    .setAccountId(newUserAccountId)
    .execute(client);

// User can now receive USDC
await TransferTransaction()
    .addTokenTransfer(usdcTokenId, treasury, -100)
    .addTokenTransfer(usdcTokenId, newUserAccountId, 100)
    .execute(client);

// Revoke if user fails compliance check
await TokenRevokeKycTransaction()
    .setTokenId(usdcTokenId)
    .setAccountId(newUserAccountId)
    .execute(client);
```

## Week 4 - Custom Fees + Queries

**Goal:** Implement custom fee schedules and all HTS query types.

### Tasks

**Custom fees**
- [ ] `FixedFee` - flat fee per transfer (in HBAR or another token)
- [ ] `FractionalFee` - percentage fee with min/max caps
- [ ] `RoyaltyFee` - NFT royalty on ownership transfer
- [ ] `TokenFeeScheduleUpdateTransaction` - update fees post-creation (requires `feeScheduleKey`)

**Queries**
- [ ] `TokenInfoQuery` - full token metadata via consensus node
- [ ] `TokenNftInfoQuery` - NFT info by token ID + serial number

### Code example - Custom fee

```dart
// Collect 1% on every transfer, min 1 token, max 1000 tokens
final fractionalFee = FractionalFee()
    .setNumerator(1)
    .setDenominator(100)
    .setMin(1)
    .setMax(1000)
    .setFeeCollectorAccountId(feeCollectorId);

await TokenCreateTransaction()
    .setTokenName("RemessaToken")
    .setCustomFees([fractionalFee])
    // ...
    .execute(client);
```

> ⚠️ Custom fees are powerful but complex to test. A misconfigured fee can silently block transfers. Week 5 testing must verify all fee combinations exhaustively.

## Week 5 - Testing Week 🧪

**Goal:** Validate the complete HTS surface with unit and integration tests against Hedera testnet. Deliver NemorixPay demo v0.1.

### Unit tests

- [ ] `TokenCreateTransaction` - construction, serialization, all field combinations
- [ ] `TransferTransaction` - atomic HBAR + token + NFT in single TX
- [ ] `TokenFreezeTransaction` / `TokenGrantKycTransaction` - correct body construction
- [ ] Custom fees — `FixedFee`, `FractionalFee`, `RoyaltyFee` serialization

### Integration tests (testnet)

- [ ] Full fungible cycle: create → associate → transfer → burn
- [ ] KYC compliance flow: grant → transfer ✅ → revoke → transfer ❌ (blocked)
- [ ] Freeze flow: freeze → transfer ❌ → unfreeze → transfer ✅
- [ ] Custom fees: verify correct fee collection on each transfer type
- [ ] NFT cycle: create → mint → transfer → wipe → verify supply
- [ ] Atomic transfer: HBAR + USDC in single TX → verify both balances
- [ ] Edge cases:
  - Token not associated → `Status.tokenNotAssociatedToAccount`
  - Account frozen → `Status.accountFrozenForToken`
  - Token paused → `Status.tokenIsPaused`
  - Insufficient supply → `Status.insufficientTokenBalance`

### NemorixPay demo v0.1

- [ ] Flutter app on testnet demonstrating:
  - Create new Hedera account from Spanish mnemonic
  - Associate USDC token
  - Grant KYC to account
  - Send 50 USDC from sender to receiver
  - Display updated balances

### Coverage target

| Module | Lines | Branches |
|:-------|:-----:|:--------:|
| `hts/` | ≥ 85% | ≥ 85% |
| Overall Phase 3 | ≥ 85% | ≥ 85% |

---

## 🎯 Milestone 3 - Definition of Done

> Phase 3 is complete when:
> - Full HTS cycle works on testnet: create → associate → transfer → burn
> - Native KYC flow works end-to-end: grant → transfer → revoke → blocked
> - Custom fees are verified on testnet for all fee types
> - **NemorixPay demo v0.1** shows a real USDC transfer between two accounts in a Flutter app on Hedera testnet

## References

- [Hedera Token Service docs](https://docs.hedera.com/hedera/sdks-and-apis/sdks/token-service)
- [HTS Protobuf definitions](https://github.com/hashgraph/hedera-protobufs/blob/main/services/token_service.proto)
- [Circle USDC on Hedera](https://www.circle.com/en/usdc-multichain/hedera)
- [Hedera JS SDK - token service](https://github.com/hashgraph/hedera-sdk-js/tree/main/src/token)

---

*Previous: [Phase 2](Phase_2_Crypto_Accounts.md) &nbsp;|&nbsp; Next: [Phase 4 - Mirror Node + HCS](Phase_4_Mirror_Node_HCS.md)*
