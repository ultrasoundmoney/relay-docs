# top bid websocket

Provides a stream of updates of the auction's current top bid. We run multiple auctions globally and forward top bids between them (see [bid-forwarding.md](bid-forwarding.md)). Two versions are available, we recommend using v2.

Both versions send ping frames, clients should respond with pong. Use persistent connections if possible. When closing connections please make sure to close the socket properly.

Note: at times, multiple parent blocks are reasonable to build on due to a split chain view. For these slots auction  instances run multiple auctions. Each unique (slot, parent\_hash) combination on a top bid update then refers to an independent auction with its own top bid.

## v1

Streams out every new local top bid: the highest bid among bids submitted directly to an auction instance. Any higher bids forwarded from other auction instances - which may be returned at the reply deadline - do not show up in this stream.

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

## v2

Streams out the global top bid: the highest bid known to an auction instance among bids submitted directly and those forwarded from other auctions. The global top bid is the bid the relay will deliver to the proposer at the reply deadline, so it is the one to compete with. Note: this does not mean a builder ends up outbidding their own fresher, lower bids with more stale forwarded bids. see [bid-forwarding.md](https://github.com/ultrasoundmoney/relay-docs/blob/main/builders/bid-forwarding.md) for more detail.

v2 timestamps have nanosecond granularity, up from millisecond in v1.

Endpoints (also available on the respective direct auction hosts):
* `ws://relay-builders-eu.ultrasound.money/ws/v2/top_bid`
* `ws://relay-builders-us.ultrasound.money/ws/v2/top_bid`
* `ws://relay-builders-jp.ultrasound.money/ws/v2/top_bid`

It emits SSZ encoded data of the following (rust) type:

```rust
pub struct TopBidUpdateV2 {
    /// Nanosecond timestamp at which this became the top bid in an auction instance
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

If <1ms latency matters to you and you are co-located with us, both websocket versions are available directly from the auction server as well. Reach out on [@ultrasoundrelay](https://t.me/ultrasoundrelay) for direct connection details.
