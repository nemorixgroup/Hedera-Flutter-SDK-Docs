# Roadmap - hedera_flutter_sdk
 
> Development roadmap for the first native Flutter/Dart SDK for the Hedera network.  
> Built by [Nemorix Group](https://nemorixpay.com) · Licensed Apache 2.0

## Overview

### Stage 1 - Initial SDK Release (v1.0)
 
The SDK is developed in **6 phases across 6 months**, with dedicated testing weeks in iterations 2, 3, and 4. Each phase culminates in a clearly defined, independently verifiable milestone.
 
```
  Phase 1 ----- Phase 2 ----- Phase 3 ---- Phase 4 ----- Phase 5 ------ Phase 6
    M1            M2            M3            M4            M5            M6
Architecture   Crypto +     HTS Tokens   Mirror Node      Docs +        Launch +
+ Protobuf     Accounts       + NFTs       + HCS        pub.dev v1.0     HIP
 (3 weeks)     (5 weeks)     (5 weeks)    (5 weeks)     (3 weeks)      (3 weeks)
```
 
**Total duration:** ~6 months (24 weeks)
 
### Phase 1 - Architecture & Protobuf Setup
**Duration:** 3 weeks  
**Status:** ⏳ Pending
 
### Goals
Set up the foundational architecture, generate Dart code from Hedera's Protobuf definitions, establish the repository structure, and publish a dev version to pub.dev to reserve the package name.
 
### Tasks
- [ ] Map all HAPI `.proto` files - [hedera-protobufs](https://github.com/hashgraph/hedera-protobufs)
- [ ] Install and configure `protoc` + `protoc-gen-dart`
- [ ] Generate Dart classes from all Hedera Protobuf definitions
- [ ] Design SDK layer architecture: Client, Services, Models, Crypto
- [ ] Define public API conventions (builder pattern, fluent interface, async/await)
- [ ] Set up public GitHub repository with Apache 2.0 license
- [ ] Configure `pubspec.yaml` with all dependencies
- [ ] Set up GitHub Actions CI/CD: `dart analyze`, `dart format`, `flutter test`, Codecov
- [ ] Configure `very_good_analysis` linter
- [ ] Publish `v0.0.1-dev` to pub.dev (reserve package name)
### Milestone 1 ✅
> **Definition of done:** Public repository is live, CI/CD passes without errors, Protobuf-generated Dart code compiles, and a basic `HederaClient` instance can connect to Hedera testnet.
 
 
### Phase 2 - Cryptography & Account Management
**Duration:** 5 weeks (4 dev + 1 testing)  
**Status:** ⏳ Pending
 
### Goals
Implement the complete cryptography layer and account management service. By the end of this phase, a developer can generate a wallet, create a Hedera account, and transfer HBAR on testnet.
 
### Tasks
 
**Cryptography**
- [ ] ED25519 key generation, import (DER / PEM / hex) and signing
- [ ] ECDSA secp256k1 key generation (EVM wallet compatibility)
- [ ] BIP-39 mnemonic generation - 12 and 24 words
- [ ] BIP-39 language support: English `🇺🇸` and Spanish `🇪🇸`
- [ ] HD key derivation from mnemonic with optional passphrase
- [ ] Legacy Hedera mnemonic derivation (HashPack / Blade Wallet compatible)
- [ ] `KeyList` with M-of-N threshold for multi-signature accounts
**Account Management**
- [ ] `AccountCreateTransaction` - key, initial HBAR, auto-token-associations
- [ ] `AccountUpdateTransaction` - key, memo, auto-renewal
- [ ] `AccountDeleteTransaction` - delete and transfer remaining balance
- [ ] `CryptoTransferTransaction` - transfer HBAR between accounts
- [ ] `AccountInfoQuery` - full account info via consensus node
- [ ] `AccountBalanceQuery` - HBAR and token balances
- [ ] EVM address alias support
**Transaction Base**
- [ ] `Transaction` base class: nodeAccountId, fee, memo, validDuration, transactionId
- [ ] Protobuf serialization: `toBytes()` / `fromBytes()`
- [ ] Multi-signature: `addSignature()`, `signWith()`, `signWithOperator()`
- [ ] `TransactionResponse`: `getReceipt()`, `getRecord()` with auto-polling
- [ ] Typed error handling: `HederaStatusException` with all `Status` codes
**Testing week**
- [ ] Unit tests: key generation, signing, verification, serialization
- [ ] Compatibility tests: keys importable in HashPack and Blade Wallet
- [ ] Integration tests against testnet: create → query → transfer → verify receipt
- [ ] Multi-sig tests: M-of-N threshold transactions
- [ ] Edge cases: insufficient balance, expired TX, invalid keys
- [ ] Coverage: ≥ 85% lines and branches for `crypto` and `accounts` modules
### Milestone 2 ✅
> **Definition of done:** A developer can generate a 24-word Spanish mnemonic, derive a key, create a Hedera testnet account, query its balance, and send HBAR. The resulting wallet is importable in HashPack without errors.
 
 
### Phase 3 - Hedera Token Service (HTS)
**Duration:** 5 weeks (4 dev + 1 testing)  
**Status:** ⏳ Pending
 
### Goals
Implement the complete HTS surface: fungible tokens, NFTs, compliance features (KYC, freeze, pause), and custom fees. Deliver the first NemorixPay demo showing USDC transfers on testnet.
 
### Tasks
 
**Fungible Tokens**
- [ ] `TokenCreateTransaction` - name, symbol, decimals, supply, treasury, all keys
- [ ] `TokenMintTransaction` - mint additional supply
- [ ] `TokenBurnTransaction` - burn tokens from treasury
- [ ] `TokenDeleteTransaction` - delete token (admin key)
- [ ] `TokenUpdateTransaction` - update token metadata
**Associate & Transfer**
- [ ] `TokenAssociateTransaction` - link token to account
- [ ] `TokenDissociateTransaction` - unlink token (zero balance required)
- [ ] `TransferTransaction` (tokens) - atomic HBAR + multi-token transfers
- [ ] `AccountAllowanceApproveTransaction` - delegate spending (ERC-20 approve equivalent)
- [ ] `AccountAllowanceDeleteTransaction` - revoke allowances
**NFTs**
- [ ] `TokenCreateTransaction` (NFT type) - `NON_FUNGIBLE_UNIQUE`
- [ ] `TokenMintTransaction` (NFT) - mint with individual metadata
- [ ] NFT transfer via `TransferTransaction.addNftTransfer()`
**Compliance**
- [ ] `TokenFreezeTransaction` / `TokenUnfreezeTransaction`
- [ ] `TokenGrantKycTransaction` / `TokenRevokeKycTransaction`
- [ ] `TokenPauseTransaction` / `TokenUnpauseTransaction`
- [ ] `TokenWipeTransaction` - remove tokens from specific account
**Custom Fees**
- [ ] `FixedFee` - flat fee per transfer
- [ ] `FractionalFee` - percentage fee with min/max
- [ ] `RoyaltyFee` - NFT royalty on transfer
- [ ] `TokenFeeScheduleUpdateTransaction` - update fees post-creation
**Queries**
- [ ] `TokenInfoQuery` - full token metadata
- [ ] `TokenNftInfoQuery` - NFT info by serial number
**Testing week**
- [ ] Unit tests for every HTS transaction type
- [ ] Integration tests: full cycle create → associate → transfer → burn
- [ ] KYC compliance flow: grant → transfer → revoke → transfer blocked
- [ ] Custom fees: FixedFee, FractionalFee, RoyaltyFee verification
- [ ] NFT cycle: mint → transfer → wipe with metadata integrity
- [ ] Atomic transfers: HBAR + token in same TX
- [ ] Coverage: ≥ 85% for `hts` module
### Milestone 3 ✅
> **Definition of done:** An account can create a fungible token, associate it with another account, and transfer it with native KYC controls. **NemorixPay demo v0.1** shows a real USDC transfer between two accounts on Hedera testnet inside a Flutter app.
 
 
### Phase 4 - Mirror Node Client + HCS
**Duration:** 5 weeks (4 dev + 1 testing)  
**Status:** ⏳ Pending
 
### Goals
Implement the Mirror Node REST client with full pagination and real-time WebSocket subscriptions. Add Hedera Consensus Service support. Deliver NemorixPay demo v0.2 with live transaction history and push notifications.
 
### Tasks
 
**Mirror Node REST Client**
- [ ] `MirrorNodeClient` - configurable base URL (mainnet / testnet / previewnet / custom)
- [ ] `AccountMirrorQuery` - account info, tokens, allowances
- [ ] `TokenMirrorQuery` - metadata, holders, supply history
- [ ] `NftMirrorQuery` - NFT info and ownership history by serial
- [ ] `TransactionMirrorQuery` - history with filters (type, timestamp, result)
- [ ] `TokenTransferMirrorQuery` - transfer history per token
- [ ] `NetworkMirrorQuery` - current fee schedule, node list
- [ ] Automatic pagination - transparent `next_link` handling
- [ ] Retry with exponential backoff
**WebSocket Subscriptions**
- [ ] `MirrorNodeSubscription` - `Stream<TransactionEvent>` via `web_socket_channel`
- [ ] Subscribe to account transactions in real time
- [ ] Filter by token ID, transaction type
- [ ] Subscribe to HCS topic messages
- [ ] Auto-reconnect on disconnect
- [ ] Clean `cancel()` / `dispose()` - no memory leaks
**Hedera Consensus Service (HCS)**
- [ ] `TopicCreateTransaction` - public or private (with `submitKey`)
- [ ] `TopicMessageSubmitTransaction` - on-chain timestamp
- [ ] `TopicUpdateTransaction` / `TopicDeleteTransaction`
- [ ] `TopicInfoQuery`
- [ ] `Stream<TopicMessage>` - subscribe via Mirror Node WebSocket
**Scheduled Transactions**
- [ ] `ScheduleCreateTransaction` - deferred multi-sig
- [ ] `ScheduleSignTransaction` - add signatures
- [ ] `ScheduleInfoQuery` - status and pending signatures
**Testing week**
- [ ] Unit tests: JSON parsing for all Mirror Node models
- [ ] Integration tests: real queries against Mirror Node testnet
- [ ] Pagination test: traverse 100+ transactions without data loss
- [ ] WebSocket test: receive live event within 10 seconds of sending TX
- [ ] Reconnection test: simulate disconnect → auto-reconnect
- [ ] HCS test: create → submit → subscribe full cycle
- [ ] Memory test: WebSocket subscriptions release resources on `dispose()`
- [ ] Coverage: ≥ 85% for `mirror` and `hcs` modules
### Milestone 4 ✅
> **Definition of done:** SDK is feature-complete for Phase 1. **NemorixPay demo v0.2** shows transaction history loaded from Mirror Node and real-time payment notifications via WebSocket - fully functional in a Flutter app on Hedera testnet.
 
 
### Phase 5 - Testing, Documentation & pub.dev
**Duration:** 3 weeks  
**Status:** ⏳ Pending
 
### Goals
Comprehensive SDK-wide testing, complete dartdoc API documentation, open-source app demo, and official v1.0.0 publication on pub.dev with a score of 130+/140.
 
### Tasks
 
**Testing**
- [ ] Full SDK regression tests across all modules
- [ ] End-to-end test: complete NemorixPay flow (create → KYC → send → receive → history)
- [ ] Performance benchmarks: Mirror Node query latency, transaction signing time
- [ ] Security audit: private keys never exposed in logs, errors, or stack traces
- [ ] Flutter compatibility: iOS 14+, Android API 24+, multiple Flutter SDK versions
- [ ] Final coverage: ≥ 80% lines, ≥ 85% branches across entire SDK
**Documentation**
- [ ] 100% dartdoc on public API - descriptions, parameters, exceptions, code examples
- [ ] Auto-generated docs site published on GitHub Pages
- [ ] Quick Start guide: create wallet → send USDC in under 10 minutes
- [ ] Error handling guide: all `HederaStatusException` codes with solutions
- [ ] FAQ: common issues found during testing phases
**Open Source**
- [ ] `README.md` - complete with badges, examples, and architecture diagram
- [ ] `CHANGELOG.md` - semantic versioning history
- [ ] `CONTRIBUTING.md` - development setup, coding standards, PR process
- [ ] `CODE_OF_CONDUCT.md` - required by Hiero / Linux Foundation
- [ ] Example app: NemorixPay demo published in `example/` folder
**pub.dev Publication**
- [ ] `dart pub publish --dry-run` - zero warnings
- [ ] pub.dev score target: **130+ / 140**
- [ ] Publish `v1.0.0` to pub.dev
- [ ] Open PR to [hiero-ledger](https://github.com/hiero-ledger) to register SDK as community resource
- [ ] Notify Hedera Developer Relations (Discord `#dev-tools`)
### Milestone 5 ✅
> **Definition of done:** `hedera_flutter_sdk` is available on pub.dev at v1.0.0 with score ≥ 130/140. Any Flutter developer can run `flutter pub add hedera_flutter_sdk` and send their first transaction in under 10 minutes following the Quick Start guide.
 
 
### Phase 6 - Launch, Community & HIP
**Duration:** 3 weeks  
**Status:** ⏳ Pending
 
### Goals
Official public launch, community outreach, HIP submission to Hiero, adoption by external projects, and final grant report with verified impact metrics.
 
### Tasks
 
**Launch**
- [ ] Announcement in Hedera Discord (`#dev-tools`, `#announcements`)
- [ ] Post on Reddit `r/Hedera`
- [ ] Twitter/X announcement mentioning `@hedera` and `@hiero_ledger`
- [ ] Contact Hedera Developer Relations for official visibility
- [ ] Post in Flutter LATAM communities (Slack, Telegram, Discord)
**Content**
- [ ] Technical article on Medium: *"Building a Flutter app on Hedera - a complete guide"*
- [ ] Cross-post on Dev.to
- [ ] 5-minute video demo: install SDK → create account → send USDC on testnet
- [ ] Showcase NemorixPay v0.2 as production use case
**HIP - Hedera Improvement Proposal**
- [ ] Study existing HIPs format at [hashgraph/hedera-improvement-proposal](https://github.com/hashgraph/hedera-improvement-proposal)
- [ ] Draft HIP proposing Flutter/Dart as officially supported SDK in Hiero
- [ ] Open PR in HIP repository for community review
- [ ] Respond to community feedback during review period
- [ ] Present HIP in Hedera Developer Community Call (if available)
**Adoption**
- [ ] Contact 5-10 Flutter projects in the Hedera ecosystem
- [ ] Onboard first 3 external projects using the SDK
- [ ] Provide direct technical support to early adopters
- [ ] Open GitHub Discussions for community Q&A
**Final Report**
- [ ] Compile impact metrics for grant report
- [ ] Verify: 500+ pub.dev downloads in first 90 days
- [ ] Verify: 3+ external projects confirmed using SDK
- [ ] Verify: HIP opened and in community review
- [ ] Deliver sustainability plan: Phase 2 roadmap + NemorixPay production dependency
### Milestone 6 ✅
> **Definition of done:** The Flutter ecosystem has its first native Hedera SDK with 500+ downloads, 3+ external adopters, a HIP in review, and a published technical article. NemorixPay demonstrates the SDK in a real US-LATAM remittance use case.

 
## Progress Summary
 
| Phase | Description | Weeks | Status |
|:------|:-----------|:-----:|:------:|
| 1 | Architecture + Protobuf | 3 | ⏳ Pending |
| 2 | Crypto + Accounts | 5 | ⏳ Pending |
| 3 | HTS Tokens + NFTs | 5 | ⏳ Pending |
| 4 | Mirror Node + HCS | 5 | ⏳ Pending |
| 5 | Docs + pub.dev v1.0 | 3 | ⏳ Pending |
| 6 | Launch + HIP | 3 | ⏳ Pending |
| **Total** | | **24** | |
 
 
## Stage 2 - Future Enhancements
 
After successful delivery of Stage 1, we plan to extend the SDK with:
 
- **Flutter Web** - HTTP/2 transport layer for gRPC without `dart:io`
- **HSCS** - Hedera Smart Contract Service (Solidity / EVM)
- **HFS** - Hedera File Service
- **JSON-RPC Relay** - Full EVM / MetaMask compatibility
- **Extended mnemonics** - French 🇫🇷 · Portuguese 🇧🇷 · Japanese 🇯🇵 · Korean 🇰🇷
 

## Contributing
 
See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get involved.  
Questions? Open a [Discussion](../../discussions) or reach us at **hedera@nemorixpay.com**
 
---
 
<p align="center">
  <sub>Built by <a href="https://nemorixpay.com">Nemorix Group</a> · Licensed Apache 2.0 · Contributed to <a href="https://github.com/hiero-ledger">Hiero</a> / Linux Foundation</sub>
</p>
 
