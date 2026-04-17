# getting started

## First steps

1. colocation: Locate your builder close to relays. The ultra sound relay is at OVH Roubaix (RBX), France, OVH Vint Hill (VIN), US, and Latitude Tokyo (TYO).
2. submission format: Submit your bids using SSZ (not JSON). If at all possible, use block deltas.
3. submission frequency: Receive an API token and bypass our free-for-all-tier rate limits. Top builders make a few thousand submissions per block landed. See [rate-limits.md](rate-limits.md).
4. optimistic relaying: This is a massive relay-side optimization where bids are processed optimistically, skipping simulation and saving on simulation latency. Almost all builders (22 of them!) have enabled optimistic relaying with the ultra sound relay—it's hard to compete without. For more detail see: [optimistic-relaying-builder-guide.md](optimistic-relaying-builder-guide.md).
5. top bid websocket: To make competitive bids a builder needs to be aware of the auction's bid-to-beat, the top bid. We offer a websocket to efficiently keep track. Read more here: https://github.com/ultrasoundmoney/docs/blob/main/top-bid-websocket.md
6. use http/2 or `Connection: keep-alive` if using http/1

## URL

```bash
# hoodi - eu
https://relay-builders-eu-hoodi.ultrasound.money
# hoodi - us
https://relay-builders-us-hoodi.ultrasound.money
# hoodi - jp (whitelist only)
https://relay-builders-jp-hoodi.ultrasound.money

# mainnet - eu
https://relay-builders-eu.ultrasound.money
# mainnet - us
https://relay-builders-us.ultrasound.money
# mainnet - jp (whitelist only)
https://relay-builders-jp.ultrasound.money
```

When submitting to only a subset of these we still do our best to offer your bid to proposers everywhere. Bids submitted to auctions further from the proposer are at a natural latency based disadvantage. For more detail see: [bid-forwarding.md](bid-forwarding.md) .

## Bid Sequencing

By default we use `received_at`, i.e. the timestamp at which we received your bid but prior to downloading the full HTTP body, to sequence your bids. Meaning updates to your bid only happen if `received_at_new` > `received_at_current`. Due to unpredictability in network latency, you may wish to determine the sequencing yourself. To do this, include a HTTP header `x-sequence` with a monotonically increasing uint64 as the value. We then use this to perform the same check as with `recieved_at`, i.e. `sequence_new` > `sequence_current` before updating your bid.

## Transaction Filtering

The ultra sound relay supports both transaction filtering and non-filtering proposers. A proposer's filtering preference is communicated in the `/relay/v1/builder/validators` response. Possible values are `none` for no filtering, and `ofac` for filtering according to  [ofac.md](../proposers/ofac.md). By default, the endpoint only returns proposers with filtering set to `none`. To receive both filtering and non-filtering proposers in the lookahead, add the following query parameter: `?filtering=true`. A block containing filtered addresses in a filtered slot leads to a simulation failure. Below is an example response.

```json
 {
    "slot": "11411773",
    "validator_index": "1427827",
    "entry": {
      "message": {
        "fee_recipient": "0x4675c7e5baafbffbca748158becba61ef3b0a263",
        "gas_limit": "36000000",
        "timestamp": "1734571166",
        "pubkey": "0xb6e245c28445330cb858d52ad37190b41f8933bafa15a426d12864fac8af4b209920bc44374f15b2ccbab9fcd70cc9bb"
      },
      "signature": "0x8dc6efff70584da37d256459c39e457681574d615263c248af9c7cc46cfef104994762b9fc7d3a2ddf594359d2a258170e507fe09c9984405e11559b1b35ea6da4dc5a8b1938a6afd46ced846d0d4ffd3a8643b1ec794e371951aa790a07f4c8"
    },
    "preferences": {
      "filtering": "none"
    }
  },
```

## Block Deltas

This feature was first conceived and implemented by Titan relay.
To optimize bandwidth utilization we support sending _dehydrated_ payloads for V1 submissions. A builder may submit transactions and blobs only once per slot. Subsequent submissions within the same slot can reference those orders by their hash or proof, and the relay will rehydrate the payload using a local cache.
```rust
// All types are assuming Fulu fork

struct DehydratedBidSubmission {
    message: BidTrace,
    execution_payload: ExecutionPayload,
    blobs_bundle: DehydratedBlobsBundle,
    execution_requests: ExecutionRequests,
    signature: BlsSignature,
    tx_root: Option<B256>,
}

/// Identical to above, but with adjustment data added as last field.
/// See bid adjustment docs for more information.
struct AdjustableDehydratedBidSubmission {
    message: BidTrace,
    execution_payload: ExecutionPayload,
    blobs_bundle: DehydratedBlobsBundle,
    execution_requests: ExecutionRequests,
    signature: BlsSignature,
    tx_root: Option<B256>,
    adjustment_data: AdjustmentData,
}

struct DehydratedBlobsBundle {
    commitments: Vec<KzgCommitment>,
    new_items: Vec<BlobItem>,
}

struct BlobItem {
    proof: Vec<KzgProof>,
    commitment: KzgCommitment,
    blob: Blob,
}
```

The transaction hashes are directly reusing the `transaction` field in the `ExecutionPayload`. To enable block deltas make sure to add a `x-hydrate` header to the submission.

## Bid Adjustments

see: [bid-adjustment.md](bid-adjustment.md)

