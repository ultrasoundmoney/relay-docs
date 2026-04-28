# optimistic relaying

An onboarding guide for builders interested in low-latency optimistic relaying with [the ultra sound relay](https://relay.ultrasound.money).

**TLDR**

1. Share with us over Telegram or Discord the list of builder pubkeys you want promoted for optimistic relaying. We will manually review recent bid submissions from those pubkeys to ensure a low historical rate of bad bids. A bad bid is one with an invalid block or an insufficient payment to the proposer.
2. Post a maximum of 64 stETH collateral to `relay.ultrasound.eth` and share the transaction details with us. The transaction sender must be an address publicly associated with one of your builder pubkeys, ideally your primary fee recipient address.
3. The relay will automatically demote you for submitting a single bad bid to the relay. You will only be re-promoted after the underlying reason for submitting a bad bid is addressed.
4. A bad bid that wins the auction and is signed by the proposer will cause an on-chain incident, i.e. a missed slot or an insufficient proposer payment. We expect you to directly compensate the proposer the bid value plus a fixed 0.01 ETH penalty within 48 hours and send us the transaction details.
5. Without receiving proof the proposer was compensated within 48 hours we may use your collateral to compensate the proposer ourselves.

### Purpose

This document outlines key aspects of optimistic relaying with the ultra sound relay. Optimistic relaying allows builders to significantly reduce the latency of their bid submissions through asynchronous simulation. For more detail see [the proposal](https://github.com/michaelneuder/opt-relay-docs/blob/main/proposal.md), [the implementation](https://github.com/flashbots/mev-boost-relay/pull/285), and the discussion in [MEV community call #0](https://collective.flashbots.net/t/mev-boost-community-call-0-23-feb-2023/1348).

### Optimistic logic

The optimistic relay implementation adds three DB fields to every builder pubkey:

1. `is_optimistic`: This boolean, which defaults to `false`, indicates whether or not the pubkey is eligible for optimistic relaying. Promoting a builder pubkey for optimistic relaying is a manual process. When a builder submits a bad bid `is_optimistic` is reset to `false` before moving to the next slot, with demotion details recorded in the relay DB.
2. `collateral`: This integer reflects the collateral value in wei backstopping the value of optimistic bids. That is, optimistic relaying happens when `is_optimistic` is `true` and `collateral` is at least as large as the bid value.
3. `builder_id`: This string is used to share collateral across multiple pubkeys. The demotion of a pubkey will result in the simultaneous demotion of all pubkeys sharing the same builder ID.

Consider the example below:

```
 builder_pubkey | is_optimistic | collateral         | builder_id
----------------+---------------+--------------------+------------------
 0xaaaaaa...    | true          | 990000000000000000 | mike
 0xbbbbbb...    | true          | 990000000000000000 | mike
 0xcccccc...    | false         | 990000000000000000 | flashbots
 0xdddddd...    | false         |                  0 | bloxroute
```

Pubkeys `0xaaaaaa` and `0xbbbbbb` share the same builder ID `mike` and collateral of 0.99 ETH. (0.99 ETH would be assuming 1 ETH collateral minus 0.01 ETH for the fixed penalty.) Since `is_optimistic` is `true` any bid with a value less than or equal to 0.99 ETH will be relayed optimistically. A larger bid, e.g. with 10 ETH of value, will not be relayed optimistically. If either pubkey submits an invalid bid both pubkeys will be demoted before the next slot.

Pubkey `0xcccccc` also has 0.99 ETH of collateral but `is_optimistic` is `false` so their bids will not be relayed optimistically. Builder `0xdddddd` has no collateral so their bids will also not be relayed optimistically.

### Collateral

Collateral for optimistic relaying must be posted to `relay.ultrasound.eth` from an address publicly associated with one of your builder pubkeys, ideally your primary fee recipient address. The maximum collateral per pubkey is currently 64 stETH—this value may be increased or decreased from time to time. The stETH reward paid by Lido is kept by ultra sound to fund the relay. If you prefer to send ETH, we reserve the right to stake and return your collateral 1:1, as stETH. Please contact us if you wish to stop optimistic relaying and have your collateral returned.

### Builder ID

For collateral efficiency you may reuse the same piece of collateral for multiple builder pubkeys. Let us know which pubkeys you want to share a builder ID. The demotion of a pubkey will result in the demotion of all pubkeys sharing the same builder ID.

### Promotions and demotions

We will manually promote your pubkeys by setting `is_optimistic` to `true` after collateral is posted and you have indicated readiness for optimistic relaying. For transparency we intend to publicly disclose `is_optimistic`, `collateral`, `builder_id` for every pubkey.

When a bad bid is submitted, even if the bid does not get signed by the proposer, the demotion logic resets `is_optimistic` to `false` before the next slot. Only after the root cause of a demotion is understood and fixed can we manually reset `is_optimistic` to `true`.

### On-chain incidents

Optimistic relaying means a bad block can occasionally make it on-chain. When that happens ultra sound aims to help resolve the incident, with how much we cover depending on the cause:

- **Foul play** — e.g. a builder paying themselves instead of the proposer in the last transaction, a safety fault. We will use the builder's collateral to compensate the proposer in full.
- **Liveness failure** — no foul play, a rare bug or operational fault. We aim to compensate proposers up to 10 ETH of bid value on a best-effort basis. We hold ourselves to the same standard for liveness failures on our own side.

When the builder's collateral is in scope we ask the builder to send ETH directly to the proposer's fee recipient and share the transaction with us. If we don't receive that proof within 48 hours we may use the collateral to compensate the proposer ourselves.
