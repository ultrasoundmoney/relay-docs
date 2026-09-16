# Connection guide

As a proposer connecting itself is easy. You'll need one of the popular beacon side-cars that enable higher paying blocks to be sourced from specialized block builders.

Commonly used are:

* mev-boost - [https://github.com/flashbots/mev-boost](https://github.com/flashbots/mev-boost)
* Vouch - [https://github.com/attestantio/vouch](https://github.com/attestantio/vouch)
* Commit-Boost - [https://www.commit-boost.org/](https://www.commit-boost.org/)

Each provides own docs for how to configure. Each will at one point instruct to add URLs of the relays you'd like to source blocks from. This is where you may add us.

When selecting a URL to choices are required.

1. Which network is the proposer on?
2. Does this proposer require every block offered has been filtered for OFAC transactions. For details on how we handle OFAC see [ofac.md](ofac.md).

```bash
# Hoodi - unfiltered
https://0xb1559beef7b5ba3127485bbbb090362d9f497ba64e177ee2c8e7db74746306efad687f2cf8574e38d70067d40ef136dc@relay-hoodi.ultrasound.money

# Hoodi - filtered
https://0xb1559beef7b5ba3127485bbbb090362d9f497ba64e177ee2c8e7db74746306efad687f2cf8574e38d70067d40ef136dc@relay-filtered-hoodi.ultrasound.money

# Mainnet - unfiltered
https://0xa1559ace749633b997cb3fdacffb890aeebdb0f5a3b6aaa7eeeaf1a38af0a8fe88b9e4b1f61f236d2e64d95733327a62@relay.ultrasound.money

# Mainnet - filtered
https://0xa1559ace749633b997cb3fdacffb890aeebdb0f5a3b6aaa7eeeaf1a38af0a8fe88b9e4b1f61f236d2e64d95733327a62@relay-filtered.ultrasound.money
```

## Direct regional connections (advanced)

For most operators, use `relay.ultrasound.money` or `relay-filtered.ultrasound.money`. Cloudflare attempts to route these hostnames to the appropriate region using geo-based DNS resolution.

Experienced staking operators who know which region is closest to the server their proposer connects from can instead configure a direct regional endpoint. The following hostnames are available on mainnet:

| Region | Unfiltered | Filtered (OFAC) |
| --- | --- | --- |
| Europe | `relay-eu.ultrasound.money` | `relay-filtered-eu.ultrasound.money` |
| United States | `relay-us.ultrasound.money` | `relay-filtered-us.ultrasound.money` |
| Japan | `relay-jp.ultrasound.money` | `relay-filtered-jp.ultrasound.money` |

Replace only the hostname in your mainnet relay URL, keeping `https://` and the same relay pubkey. For example, the unfiltered Europe URL is:

```text
https://0xa1559ace749633b997cb3fdacffb890aeebdb0f5a3b6aaa7eeeaf1a38af0a8fe88b9e4b1f61f236d2e64d95733327a62@relay-eu.ultrasound.money
```

**Choose only one Ultra Sound endpoint per connecting server: either the default hostname or one regional hostname, with your intended filtering preference. Configuring the wrong region or adding multiple Ultra Sound endpoints is likely to hurt proposal performance and reduce rewards.** A regional endpoint replaces the default endpoint in your configuration. Choose based on the location of the server making requests to the relay, not your operator's office or headquarters. If you're unsure, use the default hostname.

## FAQ

<details>

<summary>Should I add filtered vs non-filtered?</summary>

Several parties, usually US staking operators, have asked us to, in their slots, only accept blocks from builders which have been filtered for transactions containing addresses on the US OFAC sanctions list. What violation or compliance look like for you we cannot help you with. This is not legal advice. If you require filtering, the filtered URL is available.

</details>

<details>

<summary>Is there a benefit to adding both filtered and non-filtered URLs?</summary>

No, please don't. The URL is treated as a stated preference. A config value if you will. Your block building sidecar will periodically send a registration to our relay. Adding both URLs means we see the same proposer pubkey requesting both, _conflicting_, preferences. As we cannot follow both, our relay will assume _unfiltered_ was the intended preference.

</details>

<details>

<summary>How many relays should I add, I see there are a handful available?</summary>

This is ultimately up to the operator. We'll offer our highest bid for a given slot and they'll offer theirs. Your sidecar will automatically select the highest. We recommend adding several so we are forced to offer our best bids. Several public dashboards exist like [relayscan.io](https://relayscan.io/) or [rated.network](https://explorer.rated.network/relays?network=mainnet&timeWindow=30d) to show how many slots different relays have the highest bid.

This recommendation is about different relay operators. Use only one Ultra Sound endpoint, as explained under [direct regional connections](#direct-regional-connections-advanced).

</details>

<details>

<summary>The sidecar I'm setting up offers options to configure timeouts to improve bid value, what is a good value to set?</summary>

This is a complicated topic. Our relay does a lot of clever stuff to try and work with whatever value you set. Offering the highest bid possible within your deadline, without missing any slots. If you're eager to find the best possible values and get the highest possible bids, we don't have a comprehensive guide at the moment. Come talk to us: [https://t.me/ultrasoundrelay](https://t.me/ultrasoundrelay).

</details>

<details>

<summary>Why the long pubkey in the URL?</summary>

Your client will use this information to verify any bids it receives are signed with that pubkey, i.e. came from the same party that wrote these docs.

</details>

<details>

<summary>I have a different question</summary>

[https://t.me/ultrasoundrelay](https://t.me/ultrasoundrelay)

</details>
