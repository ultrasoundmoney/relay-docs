# bid forwarding

on the auction level, we gossip all top bids between our auction instances.

the auction instance closest to the proposer receives a stream of top bids from every other instance. at the reply deadline it serves the highest bid it knows about, whether that bid was submitted to it directly or forwarded from another instance. this has the following consequences:

1. the highest bid wins, regardless of which instance it was submitted to and regardless of how old it is.
2. a bid replaces your previous bid from the same pubkey at the instance you submitted it to. so to lower or cancel a bid, submit the lower (or below-floor) bid to the same instance you submitted the original to. a bid submitted to one instance never replaces or cancels a bid you hold at another instance, even under the same `builder_id`.
3. bids are ranked by value only. there is no freshness rule: a newer, lower bid from another one of your pubkeys, or from another instance, does not displace your older, higher bid.

until 2026-09-14 the auction applied a freshness rule across instances (the newest bid per `builder_id` replaced older ones regardless of value). that rule is gone; if you relied on it to cancel bids globally with a single submission, cancel at each instance instead.

for how our top bid websocket endpoints reflect this, see [top-bid-websocket.md](top-bid-websocket.md).

should it be valuable to you to ensure your bids are not forwarded, come talk to us [@ultrasoundrelay](https://t.me/ultrasoundrelay) or [@smilingalex](https://t.me/smilingalex). we don't think ultra sound should refrain from giving out a higher bid to a proposer because they happen to call in one geo vs another. we'd like them to select the latest bid, from the closest relay server. we do recognize some builders have reasons to want their bids to be short-lived, and only submit close to where a proposer will call to minimize end-to-end latency. for them we have a solution in mind. again, please come talk to us.
