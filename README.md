# Hedera Flutter SDK Docs
The first native Flutter/Dart SDK for the Hedera network.

# hedera_flutter_sdk
 
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-teal.svg)](https://opensource.org/licenses/Apache-2.0)
[![Dart](https://img.shields.io/badge/Dart-3.x-teal.svg)](https://dart.dev)
[![Flutter](https://img.shields.io/badge/Flutter-3.x-blue.svg)](https://flutter.dev)
[![Status](https://img.shields.io/badge/Status-Research%20%26%20Docs-blue.svg)]()
[![Platform](https://img.shields.io/badge/Platform-iOS%20%7C%20Android%20%7C%20macOS%20%7C%20Windows%20%7C%20Linux-lightgrey.svg)]()
 
> **The first native Flutter/Dart SDK for the [Hedera](https://hedera.com) network.**  
> Pure Dart · No platform channels · Published soon on [pub.dev](https://pub.dev)
 
 
## ⚠️ Status: Research & Documentation - April 2026
 
This SDK is currently in the **research and documentation phase** by [Nemorix Group](https://nemorixpay.com).  
No code has been written yet, we are studying the Hedera ecosystem, defining the architecture, and preparing the technical foundation before development begins.
 
Expected first release: **pub.dev v0.1.0-dev** → See [Roadmap](ROADMAP.md)
 
## Overview
 
`hedera_flutter_sdk` is a pure Dart implementation of the [Hedera API (HAPI)](https://github.com/hashgraph/hedera-protobufs), enabling Flutter developers to build mobile and desktop applications on the Hedera network without workarounds or platform channels.
 
### Why this SDK?
 
Hedera maintains official SDKs for Java, JavaScript, Go, Swift, C++, and Rust, but **no native Dart/Flutter SDK exists**. Flutter developers who want to build on Hedera currently have no direct path to do so.
 
This project closes that gap.
 
| Language     | Status        |
|:-------------|:--------------|
| Java         | ✅ Official   |
| JavaScript   | ✅ Official   |
| Go           | ✅ Official   |
| Swift        | ✅ Official   |
| C++ / Rust   | ✅ Official   |
| .NET         | ✅ Community  |
| **Flutter/Dart** | 🔨 **This project** |
 
## Features
 
### Phase 1 (Planned)
 
- **Account Management** - Create, update, delete accounts · Transfer HBAR
- **Cryptography** - ED25519 + ECDSA keys · BIP-39 mnemonics in `🇺🇸` English and `🇪🇸` Spanish · HD key derivation
- **Hedera Token Service (HTS)** - Fungible tokens · NFTs · Native KYC · Custom fees · Freeze / Wipe / Pause
- **Mirror Node Client** - Full REST API · Paginated transaction history · Balance queries
- **Real-time Subscriptions** - WebSocket via Dart `Stream<T>` · Auto-reconnect
- **Hedera Consensus Service (HCS)** - Create topics · Submit messages · Subscribe to events
- **Scheduled Transactions** - Multi-signature deferred transactions

### Phase 2 (Future)
 
- Flutter Web support
- Hedera Smart Contract Service (HSCS / EVM)
- Hedera File Service (HFS)
- JSON-RPC Relay integration
- Extended language support for mnemonics
 
## Installation
 
> **Not yet published.** Coming soon to pub.dev.
 
```yaml
# pubspec.yaml — future installation
dependencies:
  hedera_flutter_sdk: ^1.0.0
```
 
## Quick Start
 
> Code examples below reflect the planned public API. *Implementation in progress*.
 
### 1. Initialize the client
 
```dart
import 'package:hedera_flutter_sdk/hedera_flutter_sdk.dart';
 
final client = HederaClient.forTestnet()
    .setOperator(
      AccountId.fromString('0.0.XXXXXX'),
      PrivateKey.fromString('your-private-key'),
    );
```
 
### 2. Generate a wallet with a Spanish mnemonic
 
```dart
// BIP-39 mnemonic in Spanish - built for LATAM users
final mnemonic = await Mnemonic.generate24(language: Language.spanish);
final privateKey = await mnemonic.toPrivateKey();
final publicKey  = privateKey.publicKey;
 
print(mnemonic.words); // ['agua', 'casa', 'luna', ...]
```
 
### 3. Create an account
 
```dart
final receipt = await AccountCreateTransaction()
    .setKey(privateKey.publicKey)
    .setInitialBalance(Hbar(1))
    .setMaxAutomaticTokenAssociations(10)
    .execute(client);
 
final newAccountId = receipt.accountId!;
print('Account created: $newAccountId'); // 0.0.XXXXXXX
```
 
### 4. Transfer HBAR
 
```dart
await TransferTransaction()
    .addHbarTransfer(senderAccountId, Hbar(-1))
    .addHbarTransfer(receiverAccountId, Hbar(1))
    .execute(client);
```
 
### 5. Send USDC (HTS token)
 
```dart
// Atomic transfer: network fee in HBAR + amount in USDC - single transaction
await TransferTransaction()
    .addHbarTransfer(senderAccount, Hbar.fromTinybars(-100))
    .addHbarTransfer(feeCollector,  Hbar.fromTinybars(100))
    .addTokenTransfer(usdcTokenId, senderAccount,   -50000000) // 50 USDC
    .addTokenTransfer(usdcTokenId, receiverAccount,  50000000)
    .execute(client);
```
 
### 6. Query transaction history
 
```dart
final mirror = MirrorNodeClient(network: HederaNetwork.testnet);
 
final history = await mirror.transactions
    .byAccount(accountId)
    .ofType(TransactionType.tokenTransfer)
    .between(
      from: DateTime.now().subtract(Duration(days: 30)),
      to:   DateTime.now(),
    )
    .limit(50)
    .get();
```
 
### 7. Real-time payment notifications
 
```dart
final subscription = mirror.subscribe
    .toAccount(myAccountId)
    .onlyTokenTransfers(tokenId: usdcTokenId)
    .listen((event) {
      print('Payment received: ${event.amount} USDC from ${event.senderAccountId}');
    });
 
// Clean up
@override
void dispose() {
  subscription.cancel();
  super.dispose();
}
```
 
## Architecture
 
```
hedera_flutter_sdk/
  lib/
    src/
      client/       ← HederaClient, Network, Operator
      crypto/       ← PrivateKey, PublicKey, Mnemonic (BIP-39)
      transactions/ ← TransactionBuilder + all TX types
      queries/      ← QueryBuilder + all Query types
      hts/          ← Hedera Token Service
      hcs/          ← Hedera Consensus Service
      mirror/       ← Mirror Node REST + WebSocket
      models/       ← AccountId, TokenId, Hbar, TransactionId
      proto/        ← Generated Protobuf code (DO NOT EDIT)
    hedera_flutter_sdk.dart  ← Public API barrel file
  test/
    unit/           ← Pure unit tests (no network)
    integration/    ← Tests against Hedera testnet
  example/          ← Demo Flutter app
```
 
### Communication layers
 
| Channel | Purpose | Protocol |
|:--------|:--------|:---------|
| Consensus Nodes | Submit transactions | gRPC + Protobuf |
| Mirror Node REST | Query historical data | HTTPS / JSON |
| Mirror Node WebSocket | Real-time subscriptions | WSS |
 
## Dependencies
 
```yaml
dependencies:
  grpc: ^3.2.4               # gRPC transport
  protobuf: ^3.1.0           # Protobuf runtime (HAPI)
  http: ^1.2.1               # Mirror Node REST
  web_socket_channel: ^2.4.0 # Real-time subscriptions
  pointycastle: ^3.9.1       # ED25519 / ECDSA cryptography
  bip39: ^1.0.6              # BIP-39 mnemonics (EN + ES)
  convert: ^3.1.1            # Hex encoding
  fixnum: ^1.1.0             # Int64 for Protobuf
```
 
## Real-World Use Case - NemorixPay
 
This SDK is being developed in parallel with **[NemorixPay](https://nemorixpay.com)** - a mobile remittance platform for US-to-LATAM payments (Mexico, Venezuela, and beyond). NemorixPay will use `hedera_flutter_sdk` in production, providing immediate real-world validation of the SDK.
 
NemorixPay uses Flutter + Clean Architecture + BLoC and integrates:
- **HTS** for USDC stablecoin remittances with native KYC compliance
- **Mirror Node WebSocket** for real-time payment notifications
- **HCS** as an immutable audit log for every remittance
 
## Contributing
 
We welcome contributions! This project is in early development - the best way to contribute right now is:
 
1. ⭐ **Star this repository** to show interest
2. 🐛 **Open an issue** to report bugs or suggest features
3. 💬 **Start a Discussion** with questions or ideas
4. 🔀 **Submit a PR** - see [CONTRIBUTING.md](CONTRIBUTING.md)
All contributions must follow our [Code of Conduct](CODE_OF_CONDUCT.md).  
This project is licensed under [Apache 2.0](LICENSE) and will be contributed to [Hiero](https://github.com/hiero-ledger) under the Linux Foundation Decentralized Trust.
 
## References
 
- [Hedera Developer Docs](https://docs.hedera.com)
- [Hedera HAPI Protobuf](https://github.com/hashgraph/hedera-protobufs)
- [Hiero SDK repositories](https://github.com/hiero-ledger)
- [Hedera JavaScript SDK](https://github.com/hashgraph/hedera-sdk-js) ← reference implementation
- [Mirror Node REST API](https://mainnet-public.mirrornode.hedera.com/api/v1/docs)
- [Hedera Improvement Proposals](https://github.com/hashgraph/hedera-improvement-proposal)
- [Soneso Stellar Flutter SDK](https://pub.dev/packages/stellar_flutter_sdk) ← quality benchmark
 
## Team
 
Built by [Nemorix Group, LLC](https://nemorixpay.com) - Ohio, United States.
 
| | |
|:--|:--|
| **Miguel Fagundez** | Lead SDK Developer · Flutter Senior - [LinkedIn](https://linkedin.com/in/miguelomarfagundez) & [GitHub](https://github.com/miguelfagundez) |
| **Juan Eliett** | Co-Lead · Full Stack Flutter Engineer - [LinkedIn](https://linkedin.com/in/juanrubeneliett) |
 
📧 hedera@nemorixpay.com
 
<p align="center">
  <sub>Licensed under Apache 2.0 - Built for the Hedera ecosystem - Contributed to Hiero / Linux Foundation</sub>
</p>
