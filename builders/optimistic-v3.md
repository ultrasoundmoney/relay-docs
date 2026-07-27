# optimistic v3

NOTE: this API is still being tested and is subject to change.

This document describes the ultra sound relay V3 submission endpoint, which includes adjustments. For a description of vanilla V3 submissions, see [here](https://ethresear.ch/t/introduction-to-optimistic-v3-relays/22066). Like vanilla V3 submissions, these are only available for optimistic/collateralized builders. Unlike for our V1 submissions, submission adjustability is not optional. This is done to cover the extra operational expense of a builder dependency in the critical path, and to enable value-only bid updates in the future.

Note that with V3 the relay holds only your header — the full payload is fetched from your local gateway at `getPayload` time. If a winning V3 header's payload cannot be retrieved in time, the slot is missed; such incidents fall under [missed slot reimbursement](../proposers/missed-slot-reimbursement.md).

Submissions are otherwise identical to vanilla V3, except they include adjustment data:

```rust
// Electra submission
pub struct HeaderSubmission {
    pub url: Vec<u8>,
    pub tx_count: u32,
    pub submission: SignedAdjustableHeaderSubmission,
}

pub struct SignedAdjustableHeaderSubmission {
    pub message: AdjustableHeaderSubmission,
    pub signature: BlsSignature,
}

pub struct AdjustableHeaderSubmission {
    pub bid_trace: BidTrace,
    pub execution_payload_header: ExecutionPayloadHeader,
    pub execution_requests: ExecutionRequests,
    pub commitments: Vec<Bytes48>,
    // Additional field, either AdjustmentDataV2 or AdjustmentDataV3
    pub adjustment_data: AdjustmentDataV2,
}

// EL: execution layer, CL: consensus layer
pub struct AdjustmentDataV2 {
    pub el_transactions_root: B256,
    pub el_withdrawals_root: B256,
    pub builder_address: Address,
    pub builder_proof: Vec<Bytes>,
    pub fee_recipient_address: Address,
    pub fee_recipient_proof: Vec<Bytes>,
    pub fee_payer_address: Address,
    pub fee_payer_proof: Vec<Bytes>,
    pub el_placeholder_transaction_proof: Vec<Bytes>,
    // New in V2: SSZ merkle proof for last transaction
    pub cl_placeholder_transaction_proof: Vec<B256>,
    pub placeholder_receipt_proof: Vec<Bytes>,
    // New in V2: Logs bloom accrued until but not including the last (payment) transaction.
    pub pre_payment_logs_bloom: Bloom,
}

// Updated version, otherwise same as V2 but adds gas_used for the placeholder transaction.
// With V3 we relax the gas_limit == gas_used assumption and use the provided value instead.
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

~~A working POC builder implementation can be found~~ [~~here~~](https://github.com/blombern/rbuilder/tree/optimistic-v3-adjustments)~~.~~

A production implementation can be found in the rbuilder repo: [https://github.com/flashbots/rbuilder](https://github.com/flashbots/rbuilder)

The endpoint is `/relay/v3/builder/headers` and it's available on our Hoodi relay: `relay-builders-eu-hoodi.ultrasound.money`/`relay-builders-us-hoodi.ultrasound.money`.

## Error responses

V3 Submissions are available only for whitelisted builders who have shown that they can reliably deliver the corresponding payload in time. \
If you see the below error it means that your builder is currently not whitelisted for v3. You'll need to contact the ultrasound team to change that:

```json
{
  "code": 400,
  "message": "received v3 submission from builder without header_submissions enabled"
}
```

\
If the payload retrieval from the builder supplied URL fails, the builder is demoted. V3 submissions from pessimistic builders will be rejected and receive the following HTTP 400 response:

```json
{
  "code": 400,
  "message": "received v3 submission from pessimistic builder"
}
```

This can be used as a trigger to fall back to V1 submissions until re-promotion.

In some cases, we may be temporarily unable to verify a builder's optimistic/v3 status, i.e. we don't know if the builder is optimistic or not and wether v3 is enabled for it. This could happen from time to time, especially early in the bidding window. We distinguish this case from a true pessimistic status by returning the following HTTP 503 response:

```json
{
  "code": 503,
  "message": "temporarily unable to verify builder v3 eligibility"
}
```

Another rejection case is if the value of the submission exceeds builder collateral. In these cases the response will be:

```json
{
  "code": 400,
  "message": "received v3 submission with value above optimistic collateral"
}
```
