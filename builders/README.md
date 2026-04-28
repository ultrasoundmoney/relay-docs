# builders

submit bids to the ultra sound relay.

## endpoints

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

## index

start here:

- [getting started](builder-getting-started.md) — colocation, submission format, tokens, the essentials.
- [rate limits](rate-limits.md) — bandwidth limits per slot per region; how to authenticate.

auctions:

- [bid forwarding](bid-forwarding.md) — how bids gossip between auction instances and what wins globally.
- [floor bid](floor-bid.md) — non-cancellable submissions set a per-auction floor; what `accepted bid below floor, skipped validation` means.
- [TYO auction](tyo-auction.md) — Tokyo instance, whitelist and pricing.
- [US auction](us-auction.md) — Vint Hill instance.

submission optimizations:

- [optimistic relaying](optimistic-relaying-builder-guide.md) — collateral-backed asynchronous simulation.
- [optimistic v3](optimistic-v3.md) — header-only V3 submissions (in testing).
- [bid adjustment](bid-adjustment.md) — capture latency-driven bid delta as kickback.
- [top bid websocket](top-bid-websocket.md) — stream of the current top bid.

data:

- [data api](data-api.md) — local and global views of submissions and adjustments.
- [recent registration](recent-registration.md) — non-standard endpoint to check when a proposer last registered.

## questions?

ping us on telegram: [@ultrasoundrelay](https://t.me/ultrasoundrelay) or email [contact@ultrasound.money](mailto:contact@ultrasound.money).
