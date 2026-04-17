# top bid websocket

Provides a stream of updates of the auction's current top bid.

Endpoints (also available on the respective direct auction hosts):
* `ws://relay-builders-eu.ultrasound.money/ws/v1/top_bid`
* `ws://relay-builders-us.ultrasound.money/ws/v1/top_bid`* `ws://relay-builders-jp.ultrasound.money/ws/v1/top_bid`

It sends ping frames, clients should respond with pong.

It emits SSZ encoded data of the following (rust) types:

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

Note that each (slot, parent\_hash) combination is a separate auction with its own top bid.

Use persistent connections if possible. When closing connections please make sure to close the socket properly.

You can find an example implementation here: [https://github.com/ultrasoundmoney/top-bid-websocket-client](https://github.com/ultrasoundmoney/top-bid-websocket-client)

If <1ms latency matters to you, the websocket is available directly from the auction server as well — reach out on [@ultrasoundrelay](https://t.me/ultrasoundrelay) for direct connection details.
