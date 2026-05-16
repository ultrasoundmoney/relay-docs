# ultra sound's view on the post-ePBS block building pipeline

_early take. ultra sound's views may update in light of new information._

this note is primarily for proposers — especially large staking operators — and secondarily for builders and searchers with an interest in the ethereum transaction pipeline post-ePBS. it lays out what we believe will protect proposer revenue, builder competition, and the long-term health of the auction.

## the pipeline will stay sophisticated

building the highest-value block fundamentally comes down to getting the most productive ordering of transactions into the block. that combines whatever methods turn out to matter, e.g. superior flow sharing in TEEs, better accommodation of private flow, low-latency communication and execution, and more. ePBS does not change that; it changes who runs which piece, and on what terms.

we therefore expect some component of the ethereum transaction pipeline to remain sophisticated and vertically integrated, from user to proposer. ultra sound is an active participant in the broader conversation about how that pipeline should look, including initiatives like the [Blockspace Forum](https://blockspace.forum/). everything below is our view from that vantage point.

## a professional auctioneer lowers the barrier to compete

an open, neutral auctioneer is the most efficient way for many builders to compete for the same slot. each builder posts to the same place, gets well-defined treatment, and doesn't need a direct integration with every proposer. that breadth of participation is what drives the winning bid up.

once the auctioneer is removed and builders bid directly to a proposer, the cost of bidding goes up: integration, identity, trust, and a blind first-price game where every builder has to guess what the others will do. in expectation, fewer builders compete per slot, and the proposer earns less on average.

## the no-cap bid lane: don't cap your payout

ultra sound runs a trusted bid path with no ceiling on bid size and no collateral cap on what we can deliver to you. we call it the no-cap bid lane, and it is where the long tail of proposer revenue lives.

most slots are unremarkable. the average proposer reward is dragged up by a long tail of rare, high-value opportunities — single slots worth tens or hundreds of ETH. over the past year on mainnet, the mean per-slot payout was about 2.6× the median, the top 1% of slots accounted for roughly 37% of all proposer earnings, and a single slot paid out more than 560 ETH. if a proposer accepts only trustless bids, those long-tail opportunities are bounded by builder collateral.

concretely: suppose the highest-value block in a given slot is worth 300 ETH, and the second-highest-collateral builder only has 100 ETH on-chain. the next builder cannot credibly bid above 100 ETH. the well-collateralized builder bids 100.001 ETH and pockets the remaining ~200 ETH. the proposer leaves 200 ETH on the table on a slot they will not see again for months.

a trustless-only policy turns collateral into a hard cap on your top bid. only the largest, best-collateralized builders can compete for those slots; the rest of the field is priced out by capital, not by skill. ultra sound strongly encourages proposers to keep the no-cap bid lane open: keep accepting trusted bids from us, with no ceiling. those rare payouts are exactly what makes proposing on ethereum unusually profitable; capping them caps your average.

## our auction plan post-ePBS

ultra sound has preferred an open auction so far: builders see the current top bid and can decide whether they have room to beat it. the price signal is useful, and it pushes the auction up.

the same signal is exploitable. a non-cooperative builder can watch the open auction, then route a bid direct to the proposer that is slightly higher in price but built from a lower-value block. by using the open price as a target, they undercut on real value while winning on bid. in the naive case where cooperative builders don't respond, the chain lands a lower-value block than the open auction would have produced. in practice, cooperative builders stop posting to the open auction to hide their bids from the freerider, and the price signal dies, which forces a blind first-price game on the proposer regardless.

### preferred: an authenticated open auction

our preference is still an open auction, run as an *authenticated* open auction, i.e. participating builders are trusted not to use the auction's information to route bids around it. when it works, this is the highest-value format for proposers: real-time price discovery with all top builders pushing the bid up together. we also think it is fragile. the incentive to defect is real, and one freerider is enough to collapse the price signal for the rest of the field.

### backup: a sealed second-price auction

because the open format is fragile, we will have a sealed-bid second-price auction ready, and expect to experiment with it in the coming months. we will keep running an auction for as long as it works for the proposer — as long as at least two top builders engage with it, the proposer captures value they would not otherwise see.

### if the auction is bypassed: where the JIT advantage goes

should every top builder forgo even the sealed second-price auction — each running their own price discovery by guessing the second-highest bid and submitting a sealed first-price bid direct to the proposer — there is nothing left for an auctioneer to do, and we stop running the auction.

ultra sound is already good at guessing the second-highest global bid blind, and more importantly we lead the field in just-in-time bid delivery. those are valuable properties for any builder running a sealed first-price game against the proposer.

to be explicit about preference order: we would rather run an auction that both builders and proposers pay into. with an auction worth competing in, there is no need to bestow the JIT advantage on anyone. but no bids means no auction, and the JIT advantage has to live somewhere. preferably, in our own builder (see below). selling it to a third-party builder is a distant alternative.

### in parallel: an ultra sound builder

we don't want the future of ethereum's high-performance, neutral transaction pipeline to depend on what a handful of builders decide to do in reaction to a market-structure shift with unclear outcomes. consistent with the framing at the top of this doc, some component of the pipeline will stay sophisticated and vertically integrated. we'd rather be a participant than a spectator. ultra sound is seriously exploring running a builder itself.

we'd also welcome proposers tipping the scales toward non-leading builders, e.g. allowing a designated non-leader to deliver its bid slightly later than the rest of the field. small structural edges are what keep a competitive field of builders alive.

## let's talk

post-ePBS market structure is uncertain enough that we don't want to write this doc and stop there. if you operate a large stake, or run searcher flow, your decisions are one of the main inputs to how this plays out, and we want to hear from you before the dust settles.

**staking operators.** [alex](https://t.me/smilingalex) would love to compare notes on where you agree, where you diverge, and especially:

- would you expect to keep accepting trusted bids from ultra sound, with no cap?
- what do you expect your config to look like towards other relays and builders: trusted vs. not, and if trusted, what cap?
- which builders, if any, are you planning to add directly?

we have opinions on the last one, but no urge to push hard; you know your situation better than we do.

**searchers.** if you'd like more than one efficient, neutral, private, low-latency path into ethereum to keep existing, consider sending your high-fee-paying bundles to us. our fees are very low, we expect many proposers will connect to us with no cap, and we have years of practice maintaining the liveness needed to keep high-cap trustless bids landing.

contacts at the bottom of this page.

## summary

- keep a professional auctioneer in the loop. it broadens builder participation and lifts your average.
- keep the no-cap bid lane open: trusted bids from ultra sound, with no ceiling. don't cap your top bid; the long tail is where most of your edge lives.
- expect us to try an authenticated open auction first, with a sealed second-price auction ready as a fallback.
- ultra sound is seriously exploring running its own builder, regardless of how the auction shakes out. if the auction is bypassed entirely, our builder is the natural home for our JIT-delivery edge.
- operators and searchers - do talk to us.

reach us: telegram [@ultrasoundrelay](https://t.me/ultrasoundrelay), [@ultrasoundbuilder](https://t.me/ultrasoundbuilder), [@smilingalex](https://t.me/smilingalex), or email [alex@ultrasound.money](mailto:alex@ultrasound.money).
