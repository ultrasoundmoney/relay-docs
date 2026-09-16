# connection guide

as a proposer connecting itself is easy. you'll need one of the popular beacon side-cars that enable higher paying blocks to be sourced from specialized block builders.

commonly used are:

* mev-boost - [https://github.com/flashbots/mev-boost](https://github.com/flashbots/mev-boost)
* vouch - [https://github.com/attestantio/vouch](https://github.com/attestantio/vouch)
* commit-boost - [https://www.commit-boost.org/](https://www.commit-boost.org/)

each provides own docs for how to configure. each will at one point instruct to add URLs of the relays you'd like to source blocks from. this is where you may add us.

when selecting a URL to choices are required.

1. which network is the proposer on?
2. does this proposer require every block offered has been filtered for OFAC transactions. for details on how we handle OFAC see [ofac.md](ofac.md).

```bash
# hoodi - unfiltered
https://0xb1559beef7b5ba3127485bbbb090362d9f497ba64e177ee2c8e7db74746306efad687f2cf8574e38d70067d40ef136dc@relay-hoodi.ultrasound.money

# hoodi - filtered
https://0xb1559beef7b5ba3127485bbbb090362d9f497ba64e177ee2c8e7db74746306efad687f2cf8574e38d70067d40ef136dc@relay-filtered-hoodi.ultrasound.money

# mainnet - unfiltered
https://0xa1559ace749633b997cb3fdacffb890aeebdb0f5a3b6aaa7eeeaf1a38af0a8fe88b9e4b1f61f236d2e64d95733327a62@relay.ultrasound.money

# mainnet - filtered
https://0xa1559ace749633b997cb3fdacffb890aeebdb0f5a3b6aaa7eeeaf1a38af0a8fe88b9e4b1f61f236d2e64d95733327a62@relay-filtered.ultrasound.money
```

## direct regional connections (advanced)

for most operators, use `relay.ultrasound.money` or `relay-filtered.ultrasound.money`. Cloudflare attempts to route these hostnames to the appropriate region using geo-based DNS resolution.

experienced staking operators who know which region is closest to the server their proposer connects from can instead configure a direct regional endpoint. the following hostnames are available on mainnet:

| region | unfiltered | filtered (OFAC) |
| --- | --- | --- |
| europe | `relay-eu.ultrasound.money` | `relay-filtered-eu.ultrasound.money` |
| united states | `relay-us.ultrasound.money` | `relay-filtered-us.ultrasound.money` |
| japan | `relay-jp.ultrasound.money` | `relay-filtered-jp.ultrasound.money` |

replace only the hostname in your mainnet relay URL, keeping `https://` and the same relay pubkey. for example, the unfiltered europe URL is:

```text
https://0xa1559ace749633b997cb3fdacffb890aeebdb0f5a3b6aaa7eeeaf1a38af0a8fe88b9e4b1f61f236d2e64d95733327a62@relay-eu.ultrasound.money
```

**choose only one ultra sound endpoint per connecting server: either the default hostname or one regional hostname, with your intended filtering preference. configuring the wrong region or adding multiple ultra sound endpoints is likely to hurt proposal performance and reduce rewards.** a regional endpoint replaces the default endpoint in your configuration. choose based on the location of the server making requests to the relay, not your operator's office or headquarters. if you're unsure, use the default hostname.

## faq

<details>

<summary>should i add filtered vs non-filtered?</summary>

several parties, usually US staking operators, have asked us to, in their slots, only accept blocks from builders which have been filtered for transactions containing addresses on the US OFAC sanctions list. what violation or compliance look like for you we cannot help you with. this is not legal advice. if you require filtering, the filtered URL is available.

</details>

<details>

<summary>is there a benefit to adding both filtered and non-filtered URLs?</summary>

no, please don't. the URL is treated as a stated preference. a config value if you will. your block building sidecar will periodically send a registration to our relay. adding both URLs means we see the same proposer pubkey requesting both, _conflicting_, preferences. as we cannot follow both, our relay will assume _unfiltered_ was the intended preference.

</details>

<details>

<summary>how many relays should i add, i see there are a handful available?</summary>

this is ultimately up to the operator. we'll offer our highest bid for a given slot and they'll offer theirs. your sidecar will automatically select the highest. we recommend adding several so we are forced to offer our best bids. several public dashboards exist like [relayscan.io](https://relayscan.io/) or [rated.network](https://explorer.rated.network/relays?network=mainnet&timeWindow=30d) to show how many slots different relays have the highest bid.

this recommendation is about different relay operators. use only one ultra sound endpoint, as explained under [direct regional connections](#direct-regional-connections-advanced).

</details>

<details>

<summary>the sidecar i'm setting up offers options to configure timeouts to improve bid value, what is a good value to set?</summary>

this is a complicated topic. our relay does a lot of clever stuff to try and work with whatever value you set. offering the highest bid possible within your deadline, without missing any slots. if you're eager to find the best possible values and get the highest possible bids, we don't have a comprehensive guide at the moment. come talk to us: [https://t.me/ultrasoundrelay](https://t.me/ultrasoundrelay).

</details>

<details>

<summary>why the long pubkey in the URL?</summary>

your client will use this information to verify any bids it receives are signed with that pubkey, i.e. came from the same party that wrote these docs.

</details>

<details>

<summary>i have a different question</summary>

[https://t.me/ultrasoundrelay](https://t.me/ultrasoundrelay)

</details>
