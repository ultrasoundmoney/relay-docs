# deadline prediction api

the deadline prediction api is a paid add-on for builders who want to time submissions closer to each ultra sound auction cutoff. it provides a separate stream for each of our auction geographies: rbx (Roubaix, France), vin (Virginia, USA), and tyo (Tokyo, Japan). each geo runs an independent auction.

## what it provides

at slot start, the stream publishes:

- predicted p50 and p99 deadline estimates for the upcoming proposer;
- predicted p50 and p99 proposer request counts.

when a proposer request arrives, the stream sends a live update with the exact scheduled reply time for that request and refined p50 and p99 estimates for the slot-level deadline. if multiple requests arrive, each produces another update; the last request determines the effective auction cutoff.

## historical performance

in a historical backtest covering January 1 through April 14, 2026:

- the overall median absolute error for the pre-slot p50 prediction was 66 ms;
- a pre-slot prediction was available for 99.7% of auctions;
- per-request signals provided approximately 900 ms of median lead time before the scheduled reply.

these results describe the evaluation window, not a service-level guarantee.

## caveats

- the service predicts ultra sound relay cutoffs, not network-wide builder deadlines;
- each geo has its own independent deadline stream;
- the scheduled reply time marks when we expect to select the top bid, with possible low-millisecond task-scheduling jitter;
- predictions do not include network latency from the builder to the relay or the low-millisecond processing time before a received bid becomes eligible in the auction;
- some slots receive no proposer request, so no live request update follows the pre-slot prediction.

## connection limits

up to 8 simultaneous connections per source IP, per geo. connection attempts beyond that are rejected with HTTP 429.

## access

access is available as a paid add-on. pricing has not settled; after a free trial, the current price is $2,000/month. stream URLs, authentication, and integration details are shared during onboarding. if you're interested, contact [@ultrasoundrelay](https://t.me/ultrasoundrelay) or [@smilingalex](https://t.me/smilingalex) directly on telegram.
