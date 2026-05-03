# Changelog
 
All notable changes to `hedera_flutter_sdk` will be documented in this file.
 
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
 
## [Unreleased]
 
### Status
Research & Documentation phase - April 2026.  
No implementation code yet. Active investigation of the Hedera ecosystem,
HAPI protocol, and SDK architecture before development begins.
 
### Added
- `README.md` - project overview, architecture, planned API examples, and team info
- `ROADMAP.md` - 6-phase development plan with milestones and verification criteria
- `CONTRIBUTING.md` - coding standards, PR process, and commit conventions
- `CODE_OF_CONDUCT.md` - Contributor Covenant 2.1
- `LICENSE` - Apache 2.0
- `pubspec.yaml` - base package structure with all planned dependencies
- `CHANGELOG.md` - this file
### Planned - Phase 1 (Architecture + Protobuf)
- Dart code generation from Hedera HAPI `.proto` files
- `HederaClient` with gRPC connection to consensus nodes
- SDK layer architecture: Client, Crypto, Transactions, Queries, Models
- GitHub Actions CI/CD pipeline
- pub.dev `v0.0.1-dev` publication
### Planned - Phase 2 (Crypto + Accounts)
- `PrivateKey` / `PublicKey` - ED25519 and ECDSA generation and signing
- `Mnemonic` - BIP-39 in English and Spanish · HD key derivation
- `AccountCreateTransaction` / `AccountUpdateTransaction` / `AccountDeleteTransaction`
- `CryptoTransferTransaction` - HBAR transfers
- `AccountInfoQuery` / `AccountBalanceQuery`
- `KeyList` - M-of-N multi-signature support
### Planned - Phase 3 (Hedera Token Service)
- `TokenCreateTransaction` - fungible tokens and NFTs
- `TokenMintTransaction` / `TokenBurnTransaction` / `TokenWipeTransaction`
- `TokenAssociateTransaction` / `TokenDissociateTransaction`
- `TransferTransaction` - atomic HBAR + multi-token transfers
- `TokenFreezeTransaction` / `TokenGrantKycTransaction` - native compliance
- Custom fees - `FixedFee`, `FractionalFee`, `RoyaltyFee`
- NemorixPay demo v0.1 - USDC transfer on testnet
### Planned - Phase 4 (Mirror Node + HCS)
- `MirrorNodeClient` - full REST API with automatic pagination
- `Stream<TransactionEvent>` - real-time WebSocket subscriptions
- `TopicCreateTransaction` / `TopicMessageSubmitTransaction` - HCS
- `ScheduleCreateTransaction` - deferred multi-signature transactions
- NemorixPay demo v0.2 - live transaction history and payment notifications
### Planned - Phase 5 (Docs + pub.dev)
- 100% dartdoc on public API
- GitHub Pages documentation site
- `v1.0.0` publication on pub.dev (target score: 130+/140)
- PR to [hiero-ledger](https://github.com/hiero-ledger) for official adoption
### Planned - Phase 6 (Launch + HIP)
- HIP submission to Hiero community
- Technical article on Medium / Dev.to
- 500+ pub.dev downloads · 3+ external projects adopting SDK
 
## How this file is maintained
 
Entries are added manually for each release following this structure:
 
```
## [x.y.z] - YYYY-MM-DD
 
### Added
- New features or files
 
### Changed
- Changes to existing functionality
 
### Deprecated
- Features that will be removed in a future release
 
### Removed
- Features removed in this release
 
### Fixed
- Bug fixes
 
### Security
- Security-related fixes
```
 
Unreleased changes accumulate under `[Unreleased]` until a version is tagged.  
On release, `[Unreleased]` is renamed to the version and a new empty `[Unreleased]` section is added.
 
## Version history
 
| Version | Date | Description |
|:--------|:-----|:------------|
| Unreleased | April 2026 | Research & Documentation phase |
 
---
 
[Unreleased]: https://github.com/nemorixgroup/hedera_flutter_sdk/commits/main

<p align="center">
  <sub>Built by <a href="https://nemorixpay.com">Nemorix Group</a> · Licensed Apache 2.0 · Contributed to <a href="https://github.com/hiero-ledger">Hiero</a> / Linux Foundation</sub>
</p>
