# bid forwarding

on the auction level, we gossip all top bids between our auction instances.

the closest auction instance to the proposer will receive a stream of bids from every auction. whenever the latest possible moment to respond to a proposer arrives, it will consider the freshest bid received by every builder, and reply with the highest among them. this has two important consequences:

1. the highest global bid wins.
2. if the same builder is the highest bid in more than one geo (local or forwarded, keyed by builder_id) then we pick the freshest instead of highest for that builder. we then pick the highest among different builders. if you'd prefer we always pick your highest bid regardless of freshness you can submit non-cancellable bids.

for how our top bid websocket endpoints reflect this, see [top-bid-websocket.md](top-bid-websocket.md).

should it be valuable to you to ensure your bids are not forwarded, come talk to us [@ultrasoundrelay](https://t.me/ultrasoundrelay) or [@smilingalex](https://t.me/smilingalex). we don't think ultra sound should refrain from giving out a higher bid to a proposer because they happen to call in one geo vs another. we'd like them to select the latest bid, from the closest relay server. we do recognize some builders have reasons to want their bids to be short-lived, and only submit close to where a proposer will call to minimize end-to-end latency. for them we have a solution in mind. again, please come talk to us.
