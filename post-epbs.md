# Ultra sound's view on the post-ePBS block building pipeline

_Early take. Ultra sound's views may update in light of new information._

This note is primarily for proposers — especially large staking operators — and secondarily for builders and searchers with an interest in the ethereum transaction pipeline post-ePBS. It lays out what we believe will protect proposer revenue, builder competition, and the long-term health of the auction.

## The pipeline will stay sophisticated

Building and delivering the highest value block to proposers is a challenging problem across various dimensions. Doing so efficiently requires the work and coordination of technically sophisticated actors.
ePBS does not change that; it changes who runs which piece, and on what terms.
We participate in the discussion on the future design of that pipeline, including through initiatives like the [Blockspace Forum](https://blockspace.forum/).
Everything below is our current view from that vantage point.

## Connecting to a relay is rational and altruistic

Post-ePBS proposers can choose among different channels to receive bids from builders:

1. The p2p network 
2. Direct builder connections
3. Relays

Proposers will also have to decide whether to keep accepting trusted bids or enforce trustless bids. 
Below we discuss why we think it is both in the interest of the individual proposer as well as the ecosystem as a whole to keep accepting trusted bids via relays.

## An open auction lowers the barrier to compete

An open, neutral auction is the most efficient way for many builders to compete for the same slot. Each builder posts to the same place, gets well-defined treatment, and doesn't need a direct integration with every proposer. That breadth of participation is what drives the winning bid up.

Once this auction is removed and builders bid directly to a proposer, the cost of bidding goes up: integration, identity, trust, and a blind first-price game where every builder has to guess what the others will do. In expectation, fewer builders compete per slot, and the proposer earns less on average.

## Collateral requirement caps bid value and centralizes the builder market

Ultra sound runs a trusted bid path with no ceiling on bid size and no collateral cap. We call it the no-cap bid lane, and it is where the long tail of proposer revenue lives. In one line: don't cap your payout.

Most slots are of relatively low value. The average proposer reward is dragged up by a long tail of rare, high-value opportunities — single slots worth tens or hundreds of ETH.
Over the past year on mainnet:

- mean slot value was about 2.6× the median 
- top 1% of slots accounted for roughly 37% of all proposer earnings
- a single slot paid out more than 560 ETH

If a proposer accepts only trustless bids, those long-tail opportunities are bounded by builder collateral.
Concretely it is bounded by the collateral balance of the second richest builder. Beyond that value there is at most one builder left who has no incentive to increase their bid.
For example: if the best block in a slot is worth 300 ETH but the second-richest builder holds only 100 ETH of collateral, the top builder need only bid just above 100 ETH and keeps the other ~200 ETH — value the proposer never sees, on a slot they may not get again for months.
Furthermore a trustless-only policy further centralizes the builder market: only the largest, best-collateralized builders can compete for those slots; the rest of the field is priced out by capital, not by skill.
Ultra sound strongly encourages proposers to keep the no-cap bid lane open: keep accepting trusted bids from us, with no ceiling. Those rare payouts are exactly what makes proposing on ethereum unusually profitable; capping them caps your average.

## A fully open auction is incompatible with private side-channels

As stated above we prefer an open auction: builders see the current top bid and can decide whether they have room to beat it. The price signal is useful, and it pushes the auction up.
When builders have access to a private side-channel in the form of direct proposer connections that same signal becomes exploitable.

Any such builder can:

1. Watch the current price on the open auction
2. Outbid it in a private bid delivered to the proposer directly, without revealing it to other builders

Eventually other builders are forced to do the same and the open auction dies forcing a blind first-price game on the proposer.
While the in protocol p2p auction is fully open by design and has no way to avoid this fate, relays are less constrained in the design of the auction.
Below are two potential solutions we are considering.

### Preferred solution: an authenticated open auction
Participating builders are trusted not to use the auction's information to route bids around it and excluded from the auction if they do. When it works, this is the highest-value format for proposers: real-time price discovery with all top builders pushing the bid up together. We also think it is fragile. The incentive to defect is real. One defector is enough to collapse the price signal for the rest of the field.

### Backup solution: a sealed second-price auction
Because the open format is fragile, we will have a sealed-bid second-price auction ready, and expect to experiment with it in the coming months. We will keep running an auction for as long as it works for the proposer — as long as at least two top builders engage with it, the proposer captures value they would not otherwise see.

## If relays disappear builders need to compete on proposer integration

Should every top builder forgo even the sealed second-price auction there is nothing left for a relay to do, and we stop running the auction.
In this situation builders will likely be forced to connect to proposers directly to stay competitive.

This involves doing a lot of the work that we have been focused on over the last years. In particular:

- Just-in-time bid delivery to proposers
- Estimating the second price in a blind auction

We'd much prefer to keep offering access to these capabilities to all builders via our relay.
However if this is no longer an option, we'd leverage them in the context of an **ultra sound builder** which we are currently considering.

## Let's talk

Post-ePBS market structure is uncertain enough that we don't want to write this doc and stop there. If you operate a large stake, or run searcher flow, your decisions are one of the main inputs to how this plays out, and we want to hear from you before the dust settles.

**Staking operators.** [Alex](https://t.me/smilingalex) would love to compare notes on where you agree, where you diverge, and especially:

- Would you expect to keep accepting trusted bids from ultra sound, with no cap?
- What do you expect your config to look like towards other relays and builders: trusted vs. not, and if trusted, what cap?
- Which builders, if any, are you planning to add directly?

We have opinions on the last one, but no urge to push hard; you know your situation better than we do.

**Searchers.** If you'd like more than one efficient, neutral, private, low-latency path into ethereum to keep existing, consider sending your high-fee-paying bundles to us. Our fees are very low, we expect many proposers will connect to us with no cap, and we have years of practice maintaining the liveness needed to keep high-cap trustless bids landing.

Contacts at the bottom of this page.

## Summary

- Keep a professional auctioneer in the loop. It broadens builder participation and lifts your average.
- Keep the no-cap bid lane open: trusted bids from ultra sound, with no ceiling. Don't cap your top bid; the long tail is where most of your edge lives.
- Expect us to try an authenticated open auction first, with a sealed second-price auction ready as a fallback.
- Ultra sound is seriously exploring running its own builder, regardless of how the auction shakes out. If the auction is bypassed entirely, our builder is the natural home for our JIT-delivery edge.
- Operators and searchers - do talk to us.

Reach us: Telegram [@ultrasoundrelay](https://t.me/ultrasoundrelay), [@ultrasoundbuilder](https://t.me/ultrasoundbuilder), [@smilingalex](https://t.me/smilingalex), or email [alex@ultrasound.money](mailto:alex@ultrasound.money).
