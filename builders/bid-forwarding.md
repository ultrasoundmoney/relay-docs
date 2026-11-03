# Bid forwarding

On the auction level, we gossip all top bids between our auction instances.

The auction instance closest to the proposer receives a stream of top bids from every other instance. At the reply deadline it serves the highest bid it knows about, whether that bid was submitted to it directly or forwarded from another instance. This has the following consequences:

1. The highest bid wins, regardless of which instance it was submitted to and regardless of how old it is.
2. A bid replaces your previous bid from the same pubkey at the instance you submitted it to. So to lower or cancel a bid, submit the lower (or below-floor) bid to the same instance you submitted the original to. A bid submitted to one instance never replaces or cancels a bid you hold at another instance, even under the same `builder_id`.
3. Bids are ranked by value only. There is no freshness rule: a newer, lower bid from another one of your pubkeys, or from another instance, does not displace your older, higher bid.

For how our top bid websocket endpoints reflect this, see [top-bid-websocket.md](top-bid-websocket.md).

Should it be valuable to you to ensure your bids are not forwarded, come talk to us [@ultrasoundrelay](https://t.me/ultrasoundrelay) or [@smilingalex](https://t.me/smilingalex). We don't think Ultra Sound should refrain from giving out a higher bid to a proposer because they happen to call in one geo vs another. We'd like them to select the latest bid, from the closest relay server. We do recognize some builders have reasons to want their bids to be short-lived, and only submit close to where a proposer will call to minimize end-to-end latency. For them we have a solution in mind. Again, please come talk to us.
