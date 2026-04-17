# get_header endpoint

Under the [mev-boost protocol](https://docs.flashbots.net/flashbots-mev-boost/introduction) relays run block auctions, block builders submit blocks, and proposers call relays to get the best available header. If the header is returned signed, the corresponding block is proposed to go on-chain.

As headers contain bid values, historically, builders would also call the header endpoint, ignoring most of the response and look at the bid to decide whether they would want to build a new block and outbid. On our relay a builder should use our [websocket endpoint](../builders/top-bid-websocket.md) instead.

To get a proposer the best bid (highest, most reliable, timely) it's helpful for us to know which call is the proposer's. Our API therefore attempts to identify to the proposer and only returns the top bid to them. When calling `GET /eth/v1/builder/header/{slot}/{parent_hash}/{pubkey}` you may see the following response codes:

```
1. 200 OK                - the top header
2. 204 No Content        - we do not have any header to give you
3. 403 Forbidden         - not a validator call
4. 403 Forbidden         - validator call, but not for the current slot
5. 429 Too Many Requests - validator call, for current slot, but already gave out a top header response.
```

All 4xx responses have JSON bodies with a message explaining the same as written above. If you see these responses >1% of the time, when you are the proposer for the current slot and call us for a header, then please come talk to us https://t.me/ultrasoundrelay . We want to get you the best header.

Our relay expects the proposer to call once, wait for a reply as long as they're comfortable, then either

1. hit a deadline and use the best bid they do have
2. receive all replies early and select the best of those.

Both mev-boost and Vouch interact well, by default, with the above, albeit sub-optimally. If you prefer non-default behavior, or want more optimal bid requesting behavior, we're flexible–please come talk to us so we can collaborate.
