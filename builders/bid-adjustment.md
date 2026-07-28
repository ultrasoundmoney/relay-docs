# bid adjustment

Bid adjustment is feature of the ultra sound relay. The idea is that we try to adjust bids, ideally to `secondBestBid + 1 WEI`, capturing the delta (if any) from our latency advantage compared the the next best relay. Our goal is to make operating and developing (e.g. geo distribution, proper cancellations) the relay sustainable by having an incentive tied to its performance.

To achieve this, we need block submissions to include some additional data. As an incentive for builders to integrate we offer a percentage of the bid delta as a "kickback".

## Accounting

To use the feature, ultra sound sets up a wallet it controls (the `fee_payer_address` described below) and the builder deposits ETH into it, topping it up continually as it is spent.

With the wallet funded, the relay can adjust a submission: the last transaction of the block, which pays the bid, is replaced with a slightly lower payment from the fee payer wallet. The builder's own payout transaction is dropped, so the builder keeps the full original bid amount while the adjusted bid is paid out of the fee payer wallet.

Because the adjusted bid is paid from the fee payer wallet, an adjustment is only possible when the post-adjustment bid value fits within the wallet balance. The higher the balance, the more valuable the adjustments we can make. The largest adjustment we've landed to date took a 35.01 ETH bid down to 20.64 ETH, a delta of 14.37 ETH, which required the builder's balance to cover the 20.64 ETH payout.

A delta is only earned when the adjusted bid goes on to win the slot with the proposer through ultra sound. Adjusted submissions that don't land earn nothing.

Each earned delta is split 50/50 between the builder and the relay. Because the builder keeps the original bid amount while the wallet pays only the adjusted bid, the builder initially holds the entire delta. At the end of the month ultra sound invoices the builder for its 50%, which settles the balance: the builder pays ultra sound the half it hasn't received yet.

For example: a builder submits a 1.00 ETH bid which we adjust to 0.90 ETH, and the block wins the slot. The proposer receives 0.90 ETH from the fee payer wallet, and the builder keeps the 1.00 ETH it would have paid, netting the full 0.10 ETH delta up front. The monthly invoice transfers 0.05 ETH to ultra sound, leaving each side with half.

Every adjustment and its delta is available through the [Data API](#data-api) below, so builders can independently verify invoices.

## Technical implementation

Enabled by `?adjustments=1` and by including an `adjustment_data` object in the normal block submission:

```json
// Electra
{
 "message": { ... },
 "execution_payload": { ... },
 "blobs_bundle": { ... },
 "execution_requests": { ... },
 "signature": "0xa2def54237bfeb1d9269365e853b5469f68b7f4ad51ca7877e406ca94bc8a94bba54c14024b2f9ed37d8690bb9fac52600b7ff52b96b843cd8529e9ecc2497a0ecd5db8372e2049156e0fa9334d5c1b0ef642f192675b586ecbe6fc381178f88",
 // Either AdjustmentDataV1 or AdjustmentDataV3
 "adjustment_data": {
    "state_root": "0x74f74d15dcb00ba194901136f2019dd6be2d4c88c822786df90561a550193899",
    "transactions_root": "0x74f74d15dcb00ba194901136f2019dd6be2d4c88c822786df90561a550193899",
    "receipts_root": "0x74f74d15dcb00ba194901136f2019dd6be2d4c88c822786df90561a550193899",
    "builder_address": "0xb64a30399f7f6b0c154c2e7af0a3ec7b0a5b131a",
    "builder_proof": ["0xabc", "0x123", ...]
    "fee_recipient_address": "0xb64a30399f7f6b0c154c2e7af0a3ec7b0a5b131a",
    "fee_recipient_proof": ["0xabc", "0x123", ...],
    "fee_payer_address": "0xb64a30399f7f6b0c154c2e7af0a3ec7b0a5b131a",
    "fee_payer_proof": ["0xabc", "0x123", ...],
    "placeholder_transaction_proof": ["0xabc", "0x123", ...],
    "placeholder_receipt_proof": ["0xabc", "0x123", ...]
 }
}
```

* `builder_address` is the usual builder address that pays the proposer in the last transaction of the block. When we adjust a bid, this transaction is overwritten by a transaction from the collateral account `fee_payer_address`. If we don't adjust the bid, `builder_address` pays the proposer as per usual.
* `fee_payer_address` is an account which holds the ETH used by the relay to pay the fee recipient. Builders fund this account to use the feature. All adjusted bids are paid from this address.
* `fee_recipient_address` is the proposer's fee recipient.
* `placeholder_transaction_proof` is the merkle proof for the last transaction in the block, which will be overwritten with a payment from `fee_payer` to `fee_recipient` if we adjust the bid.
* `placeholder_receipt_proof` is the merkle proof for the receipt of the placeholder transaction. It's required for adjusting payments to contract addresses.

Note that we rely on the `gas_limit` of the payout transaction being strictly equal to `gas_used`, i.e. 21000 for EOA recipient, and equal to `gas_used` during execution for contract recipients.

### Computing the adjustment data

See example of our testnet builder with adjustments: https://github.com/blombern/rbuilder/tree/adjustments

### SSZ encoding

We strongly suggest SSZ (+ gzip) encoding all submissions for the best performance. The `AdjustmentData` is expected as the last field in `SubmitBlockRequest` (see JSON example above). Here's how to encode `AdjustmentData` as SSZ:

```rust
pub struct AdjustmentDataV1 {
    pub state_root: B256,
    pub transactions_root: B256,
    pub receipts_root: B256,
    pub builder_address: Address,
    pub builder_proof: Vec<Bytes>,
    pub fee_recipient_address: Address,
    pub fee_recipient_proof: Vec<Bytes>,
    pub fee_payer_address: Address,
    pub fee_payer_proof: Vec<Bytes>,
    pub placeholder_transaction_proof: Vec<Bytes>,
    pub placeholder_receipt_proof: Vec<Bytes>,
}

// New version. Includes `placeholder_gas_used` which allows us to relax the 
// gas_limit == gas_used requirement.
pub struct AdjustmentDataV3 {
    pub el_transactions_root: B256,
    pub el_withdrawals_root: B256,
    pub builder_address: Address,
    pub builder_proof: Vec<Bytes>,
    pub fee_recipient_address: Address,
    pub fee_recipient_proof: Vec<Bytes>,
    pub fee_payer_address: Address,
    pub fee_payer_proof: Vec<Bytes>,
    pub el_placeholder_transaction_proof: Vec<Bytes>,
    pub cl_placeholder_transaction_proof: Vec<B256>,
    pub el_placeholder_receipt_proof: Vec<Bytes>,
    pub pre_payment_logs_bloom: Bloom,
    pub placeholder_gas_used: u64,
}

```

An example of the SSZ codec in go: https://github.com/blombern/adjustable-bid-encoding

## Testing

Testing can be done on either Hoodi, or Mainnet (with optimistic relaying disabled).

## Performance considerations

The additional size to an uncompressed json payload (worst case) is 22KB. We recommend SSZ encoding payloads for significantly faster decoding.

On the relay side, the adjustment computation cost is negligible at \~300μs

## Data API

To see what adjustments were made, we offer an API.

https://relay-analytics.ultrasound.money/ultrasound/v1/data/adjustments?slot=7964405

The API expects a single query parameter, slot, indicating the slot you'd like to see the adjustments for. The earliest slot available in our production environment is `7869470`.

For a successful call the API returns a body with a field `data` with zero or more adjustments. Amounts are in Wei. Example:

```json
{
  "data": [
    {
      "adjusted_block_hash": "0x396fc66c421240af8a0b95598e7a2d1fc0e6ceafd18e4d128b96f918de7928e8",
      "adjusted_value": "137531619981380959",
      "block_number": 18771162,
      "builder_pubkey": "0xa32aadb23e45595fe4981114a8230128443fd5407d557dc0c158ab93bc2b88939b5a87a84b6863b0d04a4b5a2447f847",
      "delta": "26254846002041",
      "submitted_block_hash": "0xfd8c11c274dc1d631a1a5233f5f79e33cbc9f7abbb114ba630f21de5b8fb10c2",
      "submitted_received_at": "2023-12-12T16:01:23.154418Z",
      "submitted_value": "137557874827383000"
    },
    {
      "adjusted_block_hash": "0xd0ed51aa188e24836c03dd9c9b1a5c2bb96e29d32ab7b92055f073b8e423ef8b",
      "adjusted_value": "137558036898280751",
      "block_number": 18771162,
      "builder_pubkey": "0xa32aadb23e45595fe4981114a8230128443fd5407d557dc0c158ab93bc2b88939b5a87a84b6863b0d04a4b5a2447f847",
      "delta": "267665489777549",
      "submitted_block_hash": "0x9be20ae9fe1e35638050b22da5eaf17042e58065802e12774cebe89b2779f3c4",
      "submitted_received_at": "2023-12-12T16:01:23.245729Z",
      "submitted_value": "137825702388058300"
    }
  ]
}
```

In the event adjustments are requested for a slot which started less than 12 seconds ago, a 403 + JSON body with a single field `error` is returned, explaining the relay refuses to share data for slots that young.

Most bad requests will similarly have a JSON body with an `error` field. In rare error cases you may see no body, or a string body explaining the problem.
