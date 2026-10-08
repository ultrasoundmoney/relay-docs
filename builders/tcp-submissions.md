# TCP submissions

The ultra sound relay accepts bid submissions over TCP for latency-sensitive builders. Keep one TCP connection open per builder pubkey and stream submissions over it.

TCP submissions support everything HTTP does except non-cancellable bids, plus compact v2 dehydrated submissions and merging data. We recommend v2: it is fewer bytes to send for your builder and fewer to process for our relay.

## Compared to HTTP

- Every TCP submission is cancellable, the same as HTTP with `cancellations=true`: a newer bid replaces your previous one even if it is lower. See [floor-bid.md](floor-bid.md).
- The sequence number in the submission header takes the place of `x-sequence`.
- Rate limits are the same, and shared with your HTTP submissions. See [rate-limits.md](rate-limits.md).

## Getting started

We expect to charge $3,000/month for dedicated TCP access, but you're welcome to try it for free. Contact us on telegram to get started.

1. Ask us on telegram for an API key and connection details. The API key is a UUID, separate from your HTTP `X-Api-Token`.
2. Send a registration frame with your API key and pubkey.
3. Stream submissions, and read the responses as they come back.

## Frames

Everything on the connection, in both directions, is a frame:

| bytes | field | encoding |
| --- | --- | --- |
| 4 | payload length | little-endian u32 |
| 8 | send time, nanoseconds since the unix epoch | little-endian u64 |
| n | payload | |

## Registration

The first frame must be a registration. The payload is SSZ, 64 bytes:

| bytes | field |
| --- | --- |
| 16 | API key, the raw bytes of the UUID |
| 48 | builder BLS pubkey |

The pubkey must be one of yours. If the API key or the pubkey does not check out, we close the connection. All submissions on the connection must be for the registered pubkey.

## Submissions

A submission payload is a 6-byte header followed by the SSZ body.

| bytes | field | encoding |
| --- | --- | --- |
| 4 | sequence number | big-endian u32 |
| 1 | merge type | `0` none, `1` mergeable, `2` append only |
| 1 | flags | bitmask, see below |

**Sequence number.** Increase it with every submission in a slot. A new bid replaces your current one only if its sequence number is higher.

**Merge type.** The same as the `x-merge-type` HTTP header, see [block merging for builders](https://docs.blockspace.forum/mpbc/builders).

**Flags.**

| bit | flag | meaning |
| --- | --- | --- |
| 0 | `IS_DEHYDRATED` | body is a v1 dehydrated submission, the same format as HTTP with `x-hydrate`, see [builder-getting-started.md](builder-getting-started.md) |
| 1 | `WITH_ADJUSTMENTS` | body carries adjustment data, see [bid-adjustment.md](bid-adjustment.md) |
| 2 | `ZSTD_COMPRESSED` | body is zstd compressed |
| 3 | `PESSIMISTIC` | ignored |
| 4 | `DEHYDRATED_V2` | body is a v2 dehydrated submission |
| 5 | `MERGING_V2` | merging data is v2, needs merge type `1` |

**Body.** SSZ:

- With `DEHYDRATED_V2` or `MERGING_V2`: a container of the submission, then the adjustment data if `WITH_ADJUSTMENTS` is set, then the merging data if the merge type is mergeable.
- Otherwise: the same body as over HTTP. The one exception is a full mergeable submission without adjustments, which is a container of the submission and then the merging data.

## Dehydrated submissions v2

Send each transaction, blob and withdrawals list once per slot, then refer to it by a 64-bit key.

```rust
// All types are assuming Fulu fork

struct DehydratedBidSubmissionV2 {
    message: BidTrace,
    // transactions holds only the transactions not sent earlier in this slot
    execution_payload: ExecutionPayload,
    blobs_bundle: DehydratedBlobsBundleV2,
    execution_requests: ExecutionRequests,
    signature: BlsSignature,
    tx_root: Option<B256>,
    tx_refs: Vec<u64>,
    withdrawals_ref: u64,
}

struct DehydratedBlobsBundleV2 {
    refs: Vec<u64>,
    new_items: Vec<BlobItem>,
}
```

- `tx_refs`: one entry per transaction in the block, in block order. Either the key of a transaction you sent earlier in the slot, or `0` to take the next transaction from `execution_payload.transactions`. For a block `A B C` where only `B` is new: `transactions = [B]`, `tx_refs = [key(A), 0, key(C)]`.
- `blobs_bundle.refs`: the same for blobs. Either the key of a blob sent earlier, or `0` for the next entry of `new_items`.
- `withdrawals_ref`: `0` when the withdrawals are in `execution_payload.withdrawals`, otherwise the key of a withdrawals list sent earlier in the slot.

Keys are 64-bit FxHash values:

- transaction: FxHash of its last 67 bytes, the same key as in v1 dehydrated submissions.
- blob: FxHash of its KZG commitment.
- withdrawals list: FxHash over each withdrawal's index, validator index, address and amount, in order.

We cache per slot and per builder pubkey, and the cache is shared with your HTTP dehydrated submissions. Frames on a connection are processed in the order they arrive, so a submission can refer to items sent in the frame right before it.

## Merging data v2

With `MERGING_V2` set, the merging data is:

```rust
struct BlockMergingDataV2 {
    allow_appending: bool,
    builder_address: Address,
    orders: Vec<OrderV2>,
    tx_codes: Vec<OrderTxCodes>,
}

struct OrderV2 {
    start: u16, // index of the order's first transaction in the block
    len: u8,    // number of consecutive transactions in the order
    flags: u8,  // bit 0: LATEST_ONLY, bit 1: ALL_REVERT
}

struct OrderTxCodes {
    order: u16,      // index into orders
    codes: Vec<u8>,  // 2 bits per transaction of that order
}
```

- Every order is a run of consecutive transactions in the block.
- An order of one transaction without `LATEST_ONLY` is a single transaction. `ALL_REVERT` marks it as allowed to revert.
- A longer order, or any order with `LATEST_ONLY`, is a bundle. `ALL_REVERT` lets every transaction in it revert.
- Only a bundle whose transactions behave differently needs an `OrderTxCodes` entry. Transaction `i` of the order sits at bits `2 * (i % 4)` of byte `i / 4`: `0` must succeed, `1` may revert, `2` may be dropped. An order with codes must not also set `ALL_REVERT`.
- A `LATEST_ONLY` order is only used while it is in your most recent submission.

Without `MERGING_V2` we expect the [v1 merging data](https://docs.blockspace.forum/mpbc/builders#contributing-transactions), the same as over HTTP.

## Responses

Every submission frame gets one response frame. The payload is SSZ:

| field | type |
| --- | --- |
| sequence number | u32 |
| request id | 16 bytes UUID |
| status | u8: `0` ok, `1` invalid request, `2` internal error |
| error message | UTF-8 bytes, empty when ok |

Responses can arrive out of order, so match them by sequence number. You can include the request id when you ask us about a submission.
