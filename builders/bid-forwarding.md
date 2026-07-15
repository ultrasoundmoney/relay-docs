# bid forwarding

on the auction level, we gossip all top bids between our auction instances.

the closest auction instance to the proposer will receive a stream of bids from every auction. whenever the latest possible moment to respond to a proposer arrives, it will consider the freshest bid received by every builder, and reply with the highest among them. this has two important consequences:

1. the highest global bid wins.
2. for builders which submit to multiple auctions, it is not possible a more stale, high, bid wins, when a newer, lower, bid is globally winning also. cancellations are notable edge case should your builder want to entirely cancel, not replace, their freshest bid.

for how our top bid websocket endpoints reflect this, see [top-bid-websocket.md](top-bid-websocket.md).
