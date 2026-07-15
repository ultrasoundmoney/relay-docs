# top bid websocket

Provides a stream of updates of the auction's current top bid. We run several auctions and forward top bids between them (see [bid-forwarding.md](bid-forwarding.md)). Two versions are available:

* **v1** streams the top bid among the bids submitted to this auction.
* **v2 (NEW)** streams the global top bid: the highest among the bids submitted to this auction and those forwarded from our other auctions.

We recommend v2. The global top bid is the bid the relay would deliver to the proposer at that moment, so it is the one to compete with. v2 timestamps also have nanosecond granularity, up from millisecond in v1.

Both versions send ping frames, clients should respond with pong. Use persistent connections if possible. When closing connections please make sure to close the socket properly.

Note that each (slot, parent\_hash) combination is a separate auction with its own top bid.

## v1

Emits an update whenever the top bid among the bids submitted to this auction changes. Bids forwarded from our other auctions are not considered, so a bid that is winning globally but was submitted to another auction will not show up in this stream.

Endpoints (also available on the respective direct auction hosts):
* `ws://relay-builders-eu.ultrasound.money/ws/v1/top_bid`
* `ws://relay-builders-us.ultrasound.money/ws/v1/top_bid`
* `ws://relay-builders-jp.ultrasound.money/ws/v1/top_bid`

It emits SSZ encoded data of the following (rust) type:

```rust
pub struct TopBidUpdate {
    /// Millisecond timestamp at which this became the top bid
    pub timestamp: u64,
    pub slot: u64,
    pub block_number: u64,
    pub block_hash: B256,
    pub parent_hash: B256,
    pub builder_pubkey: BlsPublicKey,
    pub fee_recipient: Address,
    pub value: U256,
}
```

You can find an example client here: [https://github.com/ultrasoundmoney/top-bid-websocket-client](https://github.com/ultrasoundmoney/top-bid-websocket-client)

## v2 (NEW)

Emits an update whenever the global top bid changes.

Endpoints (also available on the respective direct auction hosts):
* `ws://relay-builders-eu.ultrasound.money/ws/v2/top_bid`
* `ws://relay-builders-us.ultrasound.money/ws/v2/top_bid`
* `ws://relay-builders-jp.ultrasound.money/ws/v2/top_bid`

It emits SSZ encoded data of the following (rust) type:

```rust
pub struct TopBidUpdateV2 {
    /// Nanosecond timestamp at which this became the top bid in this auction instance
    pub timestamp: u64,
    pub slot: u64,
    pub block_number: u64,
    pub block_hash: B256,
    pub parent_hash: B256,
    pub builder_pubkey: BlsPublicKey,
    pub fee_recipient: Address,
    pub value: U256,
}
```

If <1ms latency matters to you, both websocket versions are available directly from the auction server as well. Reach out on [@ultrasoundrelay](https://t.me/ultrasoundrelay) for direct connection details.
