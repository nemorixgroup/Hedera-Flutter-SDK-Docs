# Phase 4 - Mirror Node Client + HCS

**Duration:** 5 weeks (4 dev + 1 testing) &nbsp;|&nbsp; **Status:** ⏳ Pending &nbsp;|&nbsp; **Stage:** 4 of 6

← [Back to ROADMAP](../../ROADMAP.md)  

---

## Overview

Phase 4 implements two major components: the Mirror Node client (REST + WebSocket) and the Hedera Consensus Service (HCS). Together, these enable real-time payment notifications, complete transaction history, and an immutable on-chain audit log. These are all critical for NemorixPay's production requirements.

> **Architecture note:** The Mirror Node is a read-only node that indexes all network activity and exposes it via REST and WebSocket APIs. It is the equivalent of Etherscan for Hedera. The SDK communicates with it via HTTP (not gRPC), a separate client from the consensus node connection.

## Week 1 - Mirror Node REST Client (Foundations)

**Goal:** Implement the base Mirror Node client with core account and token queries.

### Tasks

- [ ] `MirrorNodeClient` - configurable base URL:
  - Mainnet: `mainnet-public.mirrornode.hedera.com`
  - Testnet: `testnet.mirrornode.hedera.com`
  - Previewnet: `previewnet.mirrornode.hedera.com`
  - Custom: user-provided URL (for private Mirror Node deployments)
- [ ] Automatic pagination - transparent `next_link` handling
- [ ] Retry with exponential backoff - resilient to transient failures
- [ ] `AccountMirrorQuery` - account info, token balances, allowances, key info
- [ ] `TokenMirrorQuery` - token metadata, holders count, current supply
- [ ] `NftMirrorQuery` - NFT info and ownership history by serial number
- [ ] `NetworkMirrorQuery` - current fee schedule and active node list

### Code example

```dart
final mirror = MirrorNodeClient(network: HederaNetwork.testnet);

// Query account info
final account = await mirror.accounts.getById(accountId);
print(account.balance.hbars);   // balance in tinybars
print(account.tokens);          // list of associated tokens
print(account.transactions.count); // total transaction count

// Query token info
final token = await mirror.tokens.getById(usdcTokenId);
print(token.name);         // "USD Coin"
print(token.totalSupply);  // current supply
print(token.holders);      // number of holders
```

## Week 2 - Transaction History

**Goal:** Implement paginated transaction history - the "movements screen" of NemorixPay.

### Tasks

- [ ] `TransactionMirrorQuery` - query with filters:
  - By `accountId`
  - By `transactionType` (CryptoTransfer, TokenTransfer, etc.)
  - By timestamp range (`from` / `to`)
  - By result (`success` / `failure`)
- [ ] Automatic and manual pagination - `nextPage()` without data loss
- [ ] `TokenTransferMirrorQuery` - transfer history for a specific token per account
- [ ] `BalanceHistoryQuery` - account balance at a specific past timestamp
- [ ] Typed models for each transaction type:
  - `CryptoTransferRecord`
  - `TokenTransferRecord`
  - `NftTransferRecord`
  - `ScheduledTransactionRecord`

### Code example

```dart
// Load last 30 days of USDC transfers - NemorixPay movements screen
final history = await mirror.transactions
    .byAccount(accountId)
    .ofType(TransactionType.tokenTransfer)
    .between(
      from: DateTime.now().subtract(Duration(days: 30)),
      to:   DateTime.now(),
    )
    .limit(50)
    .get();

for (final tx in history.transactions) {
  print('${tx.consensusTimestamp}: ${tx.transfers}');
}

// Load next page seamlessly
final nextPage = await history.nextPage();
```

## Week 3 - WebSocket Real-Time Subscriptions

**Goal:** Implement real-time event streaming via WebSocket - enabling instant payment notifications.

### Tasks

- [ ] `MirrorNodeSubscription` using `package:web_socket_channel`
- [ ] Expose `Stream<TransactionEvent>` - native Dart streams for Flutter/BLoC integration
- [ ] Subscribe to transactions of a specific account in real time
- [ ] Filter subscriptions by:
  - Token ID (e.g., only USDC transfers)
  - Transaction type (e.g., only incoming transfers)
- [ ] Subscribe to HCS topic messages via Mirror Node WebSocket
- [ ] Auto-reconnect on disconnect with progressive backoff
- [ ] Clean `cancel()` / `dispose()` - no memory leaks on Flutter screen disposal

### Code example

```dart
// Real-time payment notification - NemorixPay incoming payment
final subscription = mirror.subscribe
    .toAccount(myAccountId)
    .onlyTokenTransfers(tokenId: usdcTokenId)
    .listen((event) {
      print('Payment received: ${event.amount} USDC');
      print('From: ${event.senderAccountId}');
      print('At: ${event.consensusTimestamp}');
      // → emit to BLoC → update Flutter UI instantly
    });

// Clean disposal when leaving the screen
@override
void dispose() {
  subscription.cancel();
  super.dispose();
}
```

> ⚠️ **Production note:** The public Mirror Node WebSocket endpoint (Hashio) has connection limits. For production, NemorixPay should use a dedicated Mirror Node or a provider with higher limits. Document this clearly in the SDK with configuration examples.

## Week 4 - Hedera Consensus Service (HCS) + Scheduled Transactions

**Goal:** Implement HCS for immutable audit logs and Scheduled Transactions for deferred multi-sig.

### Tasks

**Hedera Consensus Service (HCS)**
- [ ] `TopicCreateTransaction` - create topic (public or private with `submitKey`)
- [ ] `TopicMessageSubmitTransaction` - publish message with verified on-chain timestamp
- [ ] `TopicUpdateTransaction` - update topic memo or keys
- [ ] `TopicDeleteTransaction` - delete topic (requires `adminKey`)
- [ ] `TopicInfoQuery` - topic metadata via consensus node
- [ ] `Stream<TopicMessage>` - subscribe to topic messages via Mirror Node WebSocket

**Scheduled Transactions**
- [ ] `ScheduleCreateTransaction` - create deferred transaction awaiting signatures
- [ ] `ScheduleSignTransaction` - add signature to pending scheduled transaction
- [ ] `ScheduleDeleteTransaction` - cancel scheduled transaction
- [ ] `ScheduleInfoQuery` - check status and pending required signatures

### Code example - HCS audit log

```dart
// Create audit log topic - one time setup
final topicId = await TopicCreateTransaction()
    .setTopicMemo("NemorixPay, remittance audit log")
    .setSubmitKey(operatorKey.publicKey) // only operator can write
    .execute(client);

// Record every remittance as immutable on-chain event
await TopicMessageSubmitTransaction()
    .setTopicId(topicId)
    .setMessage(jsonEncode({
      'type':      'remittance',
      'sender':    senderAccountId.toString(),
      'receiver':  receiverAccountId.toString(),
      'amount':    '50.00',
      'currency':  'USDC',
      'timestamp': DateTime.now().toIso8601String(),
    }))
    .execute(client);

// Subscribe to audit log in real time
mirror.subscribe
    .toTopic(topicId)
    .listen((message) {
      final data = jsonDecode(message.contents);
      print('Remittance recorded: ${data['amount']} ${data['currency']}');
    });
```

> The HCS audit log provides regulatory-grade traceability for NemorixPay: every remittance is recorded with a verified consensus timestamp, without depending on a centralized database.

## Week 5 - Testing Week 🧪

**Goal:** Validate Mirror Node REST, WebSocket, and HCS with comprehensive tests. Deliver NemorixPay demo v0.2.

### Unit tests

- [ ] JSON parsing: correct model mapping for all Mirror Node response types
- [ ] Pagination: `next_link` correctly followed without data loss or duplication
- [ ] Model validation: all typed fields parse correctly from testnet data

### Integration tests (testnet)

- [ ] `AccountMirrorQuery` - verify balance and token list match testnet state
- [ ] `TransactionMirrorQuery` - verify history matches known transactions
- [ ] Pagination: traverse 100+ transactions across multiple pages without loss
- [ ] WebSocket: receive live `TransactionEvent` within 10 seconds of executing a TX
- [ ] Reconnection: simulate WebSocket disconnect → verify auto-reconnect
- [ ] HCS full cycle: create topic → submit message → subscribe → receive message
- [ ] `ScheduleCreateTransaction` → `ScheduleSignTransaction` → execution verification

### Edge case tests

- [ ] Mirror Node unavailable - retry with backoff, surface `MirrorNodeException`
- [ ] Rate limit exceeded - handle 429 response gracefully
- [ ] Empty history - return empty list, not error
- [ ] WebSocket `dispose()` - verify no memory leaks after cancellation

### NemorixPay demo v0.2

- [ ] Flutter app on testnet demonstrating:
  - Transaction history loaded from Mirror Node (paginated)
  - Real-time payment notification received via WebSocket
  - HCS audit log recording each remittance on-chain

### Coverage target

| Module | Lines | Branches |
|:-------|:-----:|:--------:|
| `mirror/` | ≥ 85% | ≥ 85% |
| `hcs/` | ≥ 85% | ≥ 85% |
| Overall Phase 4 | ≥ 85% | ≥ 85% |

## 🎯 Milestone 4 - Definition of Done

> Phase 4 is complete when:
> - SDK is feature-complete for Stage 1
> - **NemorixPay demo v0.2** shows: paginated transaction history from Mirror Node + real-time incoming payment notification via WebSocket + HCS audit log recording each remittance
> - All tests pass with ≥ 85% coverage for `mirror` and `hcs` modules

## References

- [Mirror Node REST API docs](https://mainnet-public.mirrornode.hedera.com/api/v1/docs)
- [Mirror Node WebSocket docs](https://docs.hedera.com/hedera/sdks-and-apis/rest-api)
- [HCS Protobuf definitions](https://github.com/hashgraph/hedera-protobufs/blob/main/services/consensus_service.proto)
- [Hashio - public Mirror Node](https://hashio.io)
- [web_socket_channel](https://pub.dev/packages/web_socket_channel)

---

*Previous: [Phase 3](Phase_3_HTS_Tokens.md) &nbsp;|&nbsp; Next: [Phase 5 - Docs + pub.dev](Phase_5_Docs_pubdev.md)*
