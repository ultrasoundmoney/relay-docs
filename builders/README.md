# Builders

Submit bids to the Ultra Sound relay.

## Endpoints

```
# mainnet
https://relay-builders-eu.ultrasound.money
https://relay-builders-us.ultrasound.money
https://relay-builders-jp.ultrasound.money   # whitelist only, see tyo-auction.md

# hoodi
https://relay-builders-eu-hoodi.ultrasound.money
https://relay-builders-us-hoodi.ultrasound.money
https://relay-builders-jp-hoodi.ultrasound.money
```

## Index

Start here:

- [Getting started](builder-getting-started.md) — colocation, submission format, tokens, the essentials.
- [Rate limits](rate-limits.md) — bandwidth limits per slot per region; how to authenticate.

Auctions:

- [Bid forwarding](bid-forwarding.md) — how bids gossip between auction instances and what wins globally.
- [Floor bid](floor-bid.md) — non-cancellable submissions set a per-auction floor; what `accepted bid below floor, skipped validation` means.
- [TYO auction](tyo-auction.md) — Tokyo instance, whitelist and pricing.
- [US auction](us-auction.md) — Vint Hill instance.

Submission optimizations:

- [Optimistic relaying](optimistic-relaying-builder-guide.md) — collateral-backed asynchronous simulation.
- [Missed slot reimbursement](../proposers/missed-slot-reimbursement.md) — what an on-chain incident costs and who pays.
- [Optimistic v3](optimistic-v3.md) — header-only V3 submissions (in testing).
- [TCP submissions](tcp-submissions.md): the fastest way to submit, one persistent connection per pubkey, with v2 dehydrated submissions and merging data.
- [Bid adjustment](bid-adjustment.md) — capture latency-driven bid delta as kickback.
- [Top bid websocket](top-bid-websocket.md) — stream of the current top bid.
- [Deadline prediction API](deadline-prediction-api.md) — paid stream of predicted and live-refined auction deadlines.

Data:

- [Data API](data-api.md) — local and global views of submissions and adjustments.
- [Recent registration](recent-registration.md) — non-standard endpoint to check when a proposer last registered.

## Questions?

Ping us on Telegram: [@ultrasoundrelay](https://t.me/ultrasoundrelay) or email [contact@ultrasound.money](mailto:contact@ultrasound.money).
