# missed slot reimbursement

If a proposer signs a header delivered by the ultra sound relay and the block does not make it on chain, the slot is missed — the proposer can no longer propose a different block and loses both the bid and the consensus-layer proposer reward. When the miss is caused by us or by a collateralized builder, we aim to make the proposer whole:

**reimbursement = bid value + estimated CL proposer reward**

### bid value

The value of the header we delivered — the same value your sidecar logged from `getHeader`, verifiable via the [data API](../builders/data-api.md) (`proposer_payload_delivered`). For liveness failures (a rare bug or operational fault, no foul play) this component is capped at 10 ETH. For foul play by a builder, the builder's full collateral is in scope — see [optimistic relaying](../builders/optimistic-relaying-builder-guide.md).

### estimated CL proposer reward

A missed proposal forgoes the consensus-layer proposer reward — the proposer's share for including attestations and the sync aggregate. (There is no additional consensus-layer penalty for a missed proposal; the forgone reward is the entire loss.) The exact reward depends on the contents of a block that was never built, so we estimate it as the **average proposer reward of the nearest 10 valid slots on each side** of the missed slot:

- rewards are read from the standard beacon API: `GET /eth/v1/beacon/rewards/blocks/{slot}` (`data.total`, in gwei) — the same value beaconcha.in displays as a block's proposer reward.
- a slot is **valid** if it was proposed and its immediate predecessor was also proposed. Slots directly after a miss absorb the missed slot's attestations and earn a distorted (roughly doubled) reward, so they are excluded and the averaging window extends outward instead.

As of mid-2026 this component typically comes to ~0.05 ETH.

### insufficient proposer payment

If a bad block lands on chain with a short proposer payment (instead of a missed slot), no CL reward was lost — reimbursement is the payment shortfall, with the same caps.

### payment

Payment is made to the fee recipient in the validator's registration at the missed slot, within 48 hours of the incident. When a builder is at fault we ask the builder to pay the proposer directly and share the transaction with us; without proof of payment within 48 hours we may use the builder's collateral — see [optimistic relaying](../builders/optimistic-relaying-builder-guide.md). We hold ourselves to the same standard for failures on our own side.

## questions?

ping us on telegram: [@ultrasoundrelay](https://t.me/ultrasoundrelay) or email [contact@ultrasound.money](mailto:contact@ultrasound.money).
