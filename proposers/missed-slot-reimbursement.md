# Missed slot reimbursement

If a proposer signs a header delivered by the Ultra Sound relay and the block does not make it on chain, the slot is missed — the proposer can no longer propose a different block and loses both the bid and the consensus-layer proposer reward. When the miss is caused by us or by a collateralized builder, we aim to make the proposer whole:

**Reimbursement = bid value + estimated CL proposer reward**

### Bid value

The value of the header we delivered — the same value your sidecar logged from `getHeader`, verifiable via the [data API](../builders/data-api.md) (`proposer_payload_delivered`). How much of it we cover depends on whether the failure was lose-lose or win-lose:

- **Liveness failure (lose-lose)** — the block that would have paid you did not land, e.g. a missed slot from a bug or operational fault. Nobody wins. Capped at 10 ETH.
- **Foul play (win-lose)** — the builder intentionally did not pay what their bid promised, e.g. by paying themselves instead. Covered in full: from the builder's collateral for bids up to the amount they posted, and by simulation before we deliver the header for anything above it.

See [on-chain incidents](../builders/optimistic-relaying-builder-guide.md#on-chain-incidents) for detail.

### Estimated CL proposer reward

A missed proposal forgoes the consensus-layer proposer reward — the proposer's share for including attestations and the sync aggregate. (There is no additional consensus-layer penalty for a missed proposal; the forgone reward is the entire loss.) The exact reward depends on the contents of a block that was never built, so we estimate it as the **average proposer reward of the nearest 10 valid slots on each side** of the missed slot:

- Rewards are read from the standard beacon API: `GET /eth/v1/beacon/rewards/blocks/{slot}` (`data.total`, in gwei) — the same value beaconcha.in displays as a block's proposer reward.
- A slot is **valid** if it was proposed and its immediate predecessor was also proposed. Slots directly after a miss absorb the missed slot's attestations and earn a distorted (roughly doubled) reward, so they are excluded and the averaging window extends outward instead.

As of mid-2026 this component typically comes to ~0.05 ETH.

### Insufficient proposer payment

If a bad block lands on chain with a short proposer payment (instead of a missed slot), no CL reward was lost — reimbursement is the payment shortfall. The same distinction applies: a shortfall the builder intended, e.g. paying themselves, is foul play; otherwise it is a liveness failure.

### Payment

Payment is made to the fee recipient in the validator's registration at the missed slot, within 48 hours of the incident. When a builder is at fault we ask the builder to pay the proposer directly and share the transaction with us; without proof of payment within 48 hours we may use the builder's collateral — see [optimistic relaying](../builders/optimistic-relaying-builder-guide.md). We hold ourselves to the same standard for failures on our own side.

## Questions?

Ping us on Telegram: [@ultrasoundrelay](https://t.me/ultrasoundrelay) or email [contact@ultrasound.money](mailto:contact@ultrasound.money).
