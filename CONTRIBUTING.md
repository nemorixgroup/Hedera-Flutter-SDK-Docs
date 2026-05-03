# Contributing to hedera_flutter_sdk
 
Thank you for your interest in contributing to `hedera_flutter_sdk` - the first native Flutter/Dart SDK for the Hedera network. We welcome contributions of all kinds: bug reports, feature requests, documentation improvements, and code contributions.
 
This project is developed by [Nemorix Group](https://nemorixpay.com) and will be contributed to the [Hiero](https://github.com/hiero-ledger) project under the Linux Foundation Decentralized Trust. All contributions are licensed under [Apache 2.0](LICENSE).
 
---
 
## Table of Contents
 
- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Coding Standards](#coding-standards)
- [Testing](#testing)
- [Submitting a Pull Request](#submitting-a-pull-request)
- [Commit Convention](#commit-convention)
- [Reporting Bugs](#reporting-bugs)
- [Requesting Features](#requesting-features)
- [Questions & Discussions](#questions--discussions)
---
 
## Code of Conduct
 
This project follows our [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold these standards. Please report unacceptable behavior to **hedera@nemorixpay.com**.
 
## Getting Started
 
> ⚠️ The SDK is currently in the **Research & Documentation** phase (April 2026).  
> No implementation code exists yet. The best ways to contribute right now are:
 
- ⭐ **Star the repository** to show support
- 💬 **Open a Discussion** with ideas, questions, or feedback on the architecture
- 🐛 **Open an Issue** if you find problems in the documentation or roadmap
- 📖 **Improve documentation** - README, ROADMAP, or inline comments
When development begins (Stage 1), this guide will be updated with full setup instructions.
 
## How to Contribute
 
### 1. Bug Reports
Found something wrong? [Open a bug report](../../issues/new?template=bug_report.md) with as much detail as possible.
 
### 2. Feature Requests
Have an idea? [Open a feature request](../../issues/new?template=feature_request.md) describing the use case and expected behavior.
 
### 3. Documentation
Spotted a typo, unclear explanation, or missing example? Open a PR directly, documentation improvements are always welcome.
 
### 4. Code Contributions
Once development begins, follow the process below.
 
## Development Setup
 
### Prerequisites
 
- [Flutter SDK](https://flutter.dev/docs/get-started/install) ≥ 3.10.0
- [Dart SDK](https://dart.dev/get-dart) ≥ 3.0.0
- [protoc](https://grpc.io/docs/protoc-installation/) + [protoc-gen-dart](https://pub.dev/packages/protoc_plugin)
- A Hedera testnet account - get one free at [portal.hedera.com](https://portal.hedera.com)
### Setup steps
 
```bash
# 1. Fork and clone the repository
git clone https://github.com/YOUR_USERNAME/hedera_flutter_sdk.git
cd hedera_flutter_sdk
 
# 2. Install dependencies
flutter pub get
 
# 3. Generate Protobuf code (when available)
./scripts/generate_proto.sh
 
# 4. Run static analysis
dart analyze
 
# 5. Run tests
flutter test
 
# 6. Run integration tests (requires testnet credentials)
flutter test integration_test/ \
  --dart-define=TESTNET_ACCOUNT_ID=0.0.XXXXXX \
  --dart-define=TESTNET_PRIVATE_KEY=your-private-key
```
 
## Coding Standards
 
This project enforces strict Dart quality standards via [`very_good_analysis`](https://pub.dev/packages/very_good_analysis).
 
### Rules
 
- **Analysis must pass** - `dart analyze` with zero warnings or errors (CI will fail otherwise)
- **Formatting is enforced** - run `dart format .` before committing
- **Null safety** - all code must be null-safe (Dart 3.x)
- **No `dynamic`** - use strong typing throughout; avoid `dynamic` except in generated Protobuf code
- **No platform channels** - the SDK must remain pure Dart for cross-platform compatibility
- **Private keys are sacred** - never log, print, or expose private keys in any code path, including error messages and stack traces
### Code style
 
```dart
// ✅ Good - fluent builder pattern, strong typing
final receipt = await AccountCreateTransaction()
    .setKey(privateKey.publicKey)
    .setInitialBalance(Hbar(1))
    .setMaxAutomaticTokenAssociations(10)
    .execute(client);
 
// ✅ Good - typed error handling
try {
  await transaction.execute(client);
} on HederaStatusException catch (e) {
  if (e.status == Status.insufficientAccountBalance) {
    // handle specifically
  }
}
 
// ❌ Avoid - untyped catch
try {
  await transaction.execute(client);
} catch (e) {
  print(e); // too broad
}
```
 
### Documentation
 
Every public class, method, and property **must** have dartdoc comments:
 
```dart
/// Creates a new account on the Hedera network.
///
/// Requires an initial HBAR balance to cover the account creation fee.
/// The recommended minimum is [Hbar(1)].
///
/// Example:
/// ```dart
/// final receipt = await AccountCreateTransaction()
///     .setKey(privateKey.publicKey)
///     .setInitialBalance(Hbar(1))
///     .execute(client);
/// ```
///
/// Throws [HederaStatusException] with [Status.insufficientPayerBalance]
/// if the operator does not have enough HBAR to cover fees.
class AccountCreateTransaction extends Transaction { ... }
```
 
## Testing
 
### Test requirements
 
All contributions that include code **must** include tests. The project enforces:
 
| Metric | Minimum |
|:-------|:-------:|
| Line coverage | 85% |
| Branch coverage | 85% |
 
### Test structure
 
```
test/
  unit/
    crypto/         ← key generation, signing, BIP-39
    transactions/   ← serialization, construction, validation
    models/         ← AccountId, TokenId, Hbar parsing
    mirror/         ← JSON model parsing (no network)
  integration/
    accounts/       ← against Hedera testnet
    hts/            ← token operations on testnet
    mirror/         ← Mirror Node queries
    hcs/            ← Consensus Service
```
 
### Running tests
 
```bash
# Unit tests only (fast, no network)
flutter test test/unit/
 
# All tests with coverage
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
 
# Integration tests (requires testnet credentials)
flutter test integration_test/
```
 
### Writing tests
 
```dart
// Unit test example - no network required
test('ED25519 key roundtrip from string', () {
  final original = PrivateKey.generateED25519();
  final restored = PrivateKey.fromString(original.toString());
  expect(restored.publicKey.toString(), equals(original.publicKey.toString()));
});
 
// Integration test example - runs against testnet
test('creates account on testnet', () async {
  final client = HederaClient.forTestnet()
      .setOperator(testnetAccountId, testnetPrivateKey);
 
  final receipt = await AccountCreateTransaction()
      .setKey(PrivateKey.generateED25519().publicKey)
      .setInitialBalance(Hbar(1))
      .execute(client);
 
  expect(receipt.accountId, isNotNull);
});
```
 
## Submitting a Pull Request
 
1. **Fork** the repository and create a branch from `develop`
   ```bash
   git checkout -b feat/your-feature-name
   ```
 
2. **Make your changes** following the coding standards above
3. **Add tests** for all new functionality
4. **Run the full suite** locally before pushing
   ```bash
   dart analyze
   dart format --set-exit-if-changed .
   flutter test --coverage
   ```
 
5. **Commit** following the [commit convention](#commit-convention) below
6. **Push** your branch and open a Pull Request against `develop`
7. **Fill in the PR template** - describe what changed and why
8. **Wait for review** - a maintainer will review within 5 business days
### PR checklist
 
- [ ] `dart analyze` passes with zero warnings
- [ ] `dart format` applied
- [ ] Tests added for all new functionality
- [ ] Coverage remains ≥ 85%
- [ ] dartdoc added for all public API changes
- [ ] `CHANGELOG.md` updated
 
## Commit Convention
 
This project uses [Conventional Commits](https://www.conventionalcommits.org/) for automatic changelog generation.
 
```
<type>(<scope>): <short description>
 
[optional body]
 
[optional footer]
```
 
### Types
 
| Type | Description |
|:-----|:------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `test` | Adding or updating tests |
| `refactor` | Code change without feature or fix |
| `perf` | Performance improvement |
| `chore` | Build process or tooling changes |
| `ci` | CI/CD configuration changes |
 
### Examples
 
```bash
feat(hts): add TokenPauseTransaction support
fix(crypto): fix ED25519 key derivation for legacy mnemonics
docs(readme): update Quick Start with Spanish mnemonic example
test(mirror): add WebSocket reconnection integration test
chore(deps): update grpc to 3.2.4
```
 
### Breaking changes
 
Add `BREAKING CHANGE:` in the commit footer:
 
```
feat(accounts)!: rename AccountId.fromString to AccountId.parse
 
BREAKING CHANGE: AccountId.fromString has been renamed to
AccountId.parse for consistency with other model types.
```
 
## Reporting Bugs
 
Please use the [bug report template](../../issues/new?template=bug_report.md) and include:
 
- **SDK version** - from `pubspec.yaml`
- **Flutter version** - output of `flutter --version`
- **Platform** - iOS, Android, macOS, Windows, Linux
- **Hedera network** - mainnet, testnet, or previewnet
- **Minimal reproduction** - the smallest code that reproduces the issue
- **Expected behavior** vs **actual behavior**
- **Error message / stack trace** - with private keys redacted
 
## Requesting Features
 
Please use the [feature request template](../../issues/new?template=feature_request.md) and include:
 
- **Use case** - what problem does this solve?
- **Proposed API** - what would the code look like?
- **Hedera service** - which HAPI service does this relate to?
- **Priority** - is this blocking your project?
 
## Questions & Discussions
 
- 💬 **GitHub Discussions** - for questions, ideas, and general conversation → [Open a Discussion](../../discussions)
- 🤝 **Hedera Discord** - `#dev-tools` channel → [discord.gg/hedera](https://discord.gg/hedera)
- 📧 **Email** - hedera@nemorixpay.com
 
## Recognition
 
All contributors will be recognized in the `CHANGELOG.md` and in the pub.dev package credits. Significant contributors may be invited to join the project as maintainers.
 
<p align="center">
  <sub>Thank you for helping build the Flutter ecosystem on Hedera. 🙏</sub><br>
  <sub>Licensed under Apache 2.0 · <a href="https://nemorixpay.com">Nemorix Group</a> · Contributed to <a href="https://github.com/hiero-ledger">Hiero</a> / Linux Foundation</sub>
</p>
