# Deadline prediction API

The deadline prediction API is a paid add-on for builders who want to time submissions closer to each Ultra Sound auction cutoff. It provides a separate stream for each of our auction geographies: rbx (Roubaix, France), vin (Virginia, USA), and tyo (Tokyo, Japan). Each geo runs an independent auction.

## What it provides

At slot start, the stream publishes:

- Predicted p50 and p99 deadline estimates for the upcoming proposer;
- Predicted p50 and p99 proposer request counts.

When a proposer request arrives, the stream sends a live update with the exact scheduled reply time for that request and refined p50 and p99 estimates for the slot-level deadline. If multiple requests arrive, each produces another update; the last request determines the effective auction cutoff.

## Historical performance

In a historical backtest covering January 1 through April 14, 2026:

- The overall median absolute error for the pre-slot p50 prediction was 66 ms;
- A pre-slot prediction was available for 99.7% of auctions;
- Per-request signals provided approximately 900 ms of median lead time before the scheduled reply.

These results describe the evaluation window, not a service-level guarantee.

## Caveats

- The service predicts Ultra Sound relay cutoffs, not network-wide builder deadlines;
- Each geo has its own independent deadline stream;
- The scheduled reply time marks when we expect to select the top bid, with possible low-millisecond task-scheduling jitter;
- Predictions do not include network latency from the builder to the relay or the low-millisecond processing time before a received bid becomes eligible in the auction;
- Some slots receive no proposer request, so no live request update follows the pre-slot prediction.

## Connection limits

Up to 8 simultaneous connections per source IP, per geo. Connection attempts beyond that are rejected with HTTP 429.

## Access

Access is available as a paid add-on. Pricing has not settled; after a free trial, the current price is $1,400/month. Stream URLs, authentication, and integration details are shared during onboarding. If you're interested, contact [@ultrasoundrelay](https://t.me/ultrasoundrelay) or [@smilingalex](https://t.me/smilingalex) directly on Telegram.
