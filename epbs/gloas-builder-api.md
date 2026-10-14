# Gloas builder API

*ultra sound relay — draft 2026-10-13. Companion to the [auction design](builder-auction.md).*

How to submit blocks to the ultra sound relay on a Gloas (ePBS) network, and how to test on glamsterdam-devnet-8 today. Everything not mentioned here works as pre-ePBS ([builders](../builders/)).

## 1. Endpoints and auth

Devnet host: `https://relay-plataberget.ultrasound.money` (hoodi and mainnet hosts stay as they are after their forks).

| Method / path | Purpose |
|------------------------------------------------|----------------------------------|
| `GET /relay/v1/builder/validators` | Lookahead: upcoming proposers with their effective preferences (fee recipient, gas limit, filtering, `max_execution_payment` when they accept trusted bids) |
| `GET /ultrasound/v1/builder/bid_caps[?slot=N]` | Stake caps for trustless bids: the relay's available stake; with your `X-Api-Token`, your mapped wallets' too |
| `POST /relay/v1/builder/blocks[?cancellations=1][&adjustments=1]` | Block submission (gloas format from the fork slot on) |
| `GET /relay/v1/data/bidtraces/builder_blocks_received?slot=N` | What we received and whether it simulated (v2: ns timestamps) |
| `GET /ws/v1/top_bid`, `/ws/v2/top_bid` | Top-bid websocket, unchanged |
| `GET /ultrasound/v1/auction/ip_test` | Connectivity check |

- `X-Api-Token`: your existing token; on the devnet your **hoodi** token works. Authenticated: 120 MiB of submissions per slot. Without a token: 50 requests/min per IP, attributed to builder `unknown`.
- Unknown builder pubkeys are accepted (no allowlist) and treated as non-optimistic.
- Optional headers as before: `X-Sequence` (monotonic per slot), `X-Hydrate` and `X-Mergeable` are **not** supported on gloas slots yet (400).

## 2. Bid types and payment

`X-Bid-Type` selects who pays the proposer and whose stake backs the on-chain bid. Missing header = `trusted`.

| `X-Bid-Type` | You put in the payload | Relay wraps as | Proposer paid by | Needs |
|------------|--------------------|--------------------|-----------------|------------------------------|
| `trusted` | payment to `proposer_fee_recipient`, as pre-ePBS | `value = 0`, `execution_payment = value / 1e9` | you, in the payload | a proposer who accepts execution payments (a cap in their preferences); simulated unless you are optimistic |
| `trustless-full` | payment of `message.value` to the **relay wallet** | `value = value / 1e9`, relay identity | relay, from its CL stake | nothing on-chain from you. **Start here.** |
| `trustless-partial` | nothing | `value = value / 1e9`, **your** identity | you, from your CL builder stake | a gloas builder deposit + a mapping submission pubkey → staked pubkey on our side (ask us) |
| `native` | — | — | — | not accepted yet |

- `message.value` is in wei as before; use whole-gwei values (the on-chain bid is in gwei).
- `message.proposer_fee_recipient` is always the proposer's announced fee recipient, also for `trustless-full` (only the payment itself goes to the relay wallet).
- Relay wallet on devnet: `0x2Cb4c37C65F7179e271D4E832C99AC3C6267469C`. The simulator checks the wallet's balance delta against `message.value`.
- Caps: `trustless-full` bids above the relay's spendable stake (net of bids in flight for other slots) are rejected; `trustless-partial` likewise against your mapped stake. Read them from `bid_caps`:

```json
{"slot":445208,"state_slot":445206,
 "relay":{"builder_index":12057,"pubkey":"0x8c9a…e63eb","available_stake_gwei":581437039407},
 "mapped_wallets":[{"builder_pubkey":"0x…","builder_index":2,"pubkey":"0x…","available_stake_gwei":16141800000}]}
```

- Trusted bids: only proposers whose lookahead entry has `preferences.max_execution_payment` (gwei) accept them; keep `message.value` at or below it.
- The on-chain bid commits you: once the proposer includes it, the payment settles even if the payload is withheld or invalid (`trustless-*`), or the payload pays (`trusted`).

## 3. Submission format

`POST /relay/v1/builder/blocks` with `Content-Type: application/json` or `application/octet-stream` (SSZ). The slot is read first (`message.slot`; first 8 LE bytes in SSZ) and selects the gloas pipeline from the fork slot on. JSON rejects unknown fields.

```text
SubmitBlockRequestGloas {
  message:                  BidTrace           // unchanged (slot, hashes, pubkeys,
                                               // fee recipient, gas, value)
  execution_payload:        ExecutionPayloadV3 // deneb shape
  block_access_list:        bytes              // EIP-7928 RLP from your EL; hex in JSON
  blobs_bundle:             BlobsBundle        // 128 cell proofs per blob; bodies required
  execution_requests:       ExecutionRequests  // deposits, withdrawals, consolidations,
                                               // builder_deposits, builder_exits (EIP-8282)
  parent_beacon_block_root: bytes32            // EIP-4788 root the payload builds on
  signature:                BLS                // over BidTrace, builder domain, unchanged
}
```

- SSZ: the container above, fields in this order; `execution_payload` as a plain `ExecutionPayloadV3`.
- `parent_beacon_block_root` must match the payload attributes our beacon nodes announced for `(slot, parent_hash)`; after an empty slot `(slot, parent_hash)` alone no longer identifies the block, which is why it is explicit.
- The block access list is validated by execution: the simulator rebuilds it and compares hashes.

### Adjustments

`?adjustments=1` with an `adjustment_data` object as the last field, exactly as in [bid adjustment](../builders/bid-adjustment.md): same placeholder (last tx, payment forwarder call, full bid in `value`), same proofs. Trusted bids only; trustless bids need no data and are adjusted by lowering the on-chain value. The relay rebuilds the adjusted block itself, including the block access list and the EIP-7708 transfer logs — nothing new for you to provide. Accounting is unchanged (50/50 of the landed delta).

## 4. Timing and lifecycle

- The relay snapshots the auction and publishes its signed `ExecutionPayloadBid` **1 s before** the slot starts (devnet setting). Submissions after that are accepted but cannot win the slot. Our test builder submits ~2.5 s before the slot.
- Once published, a bid cannot be cancelled. `?cancellations=1` works until then.
- The relay reveals the payload; you do nothing after submitting. Adjusted bids are revealed with the adjusted block; the on-chain bid shows the adjusted value, `builder_blocks_received` shows what you submitted.
- Proposers choose between relay bids and p2p bids; proposers that never published preferences cannot be won (`no proposer preferences for slot`).

## 5. Responses

`200 {"code":200,"message":"OK"}` with `X-Simulation-Mode: pessimistic|optimistic|skipped`; `202` below floor, stored unsimulated. Rejections:

| Status / message | Meaning |
|----------------------------------------------|------------------------------------|
| 400 `invalid x-bid-type header, expected one of: ...` | not one of the four types |
| 400 `bid type not supported yet` | `native` |
| 400 `dehydrated and mergeable submissions are not supported for gloas slots` | drop `X-Hydrate` / `X-Mergeable` |
| 400 `adjustable submissions must be trusted bids, got ...` | `?adjustments=1` on a trustless bid |
| 400 `failed to parse gloas json/ssz payload: ...` | shape error; JSON rejects unknown fields |
| 400 `slot is before glamsterdam` | gloas format on a pre-fork slot |
| 400 `blob bodies are required` / `expected 128 cell proofs per blob` | |
| 400 `unknown parent beacon block root` | not the root announced for `(slot, parent_hash)` |
| 400 `could not find slot duty` / `proposer pubkey does not match slot duty` / `fee recipient mismatch` | lookahead mismatch |
| 400 `no proposer preferences for slot` | proposer never published preferences |
| 400 `proposer does not accept trusted bids` | trusted bid, proposer without execution-payment cap |
| 400 `bid value exceeds relay available stake` | `trustless-full` above our spendable stake net of in-flight bids |
| 400 `no gloas wallet mapped for builder` / `mapped gloas wallet not active` / `bid value exceeds spendable stake` | `trustless-partial` without (sufficient) mapped stake |
| 400 `submission for past slot` / `already using a newer payload` | too late / superseded |
| 400 `<simulator message>` | simulation failed: payment, gas limit, block access list hash, state root, ... |
| 401 `invalid api token` | unknown `X-Api-Token` |
| 429 | rate limit (see §1) |
| 500 | relay-side error; retry, tell us if persistent |

## 6. Checking what happened

- `builder_blocks_received?slot=N` (or `?builder_pubkey=...`): your submissions, simulation outcome, received timestamps.
- Wins: the beacon block's `signed_execution_payload_bid` (builder index 12057 = ultra sound on devnet) and the revealed payload, e.g. on [Dora](https://dora.glamsterdam-devnet-8.ethpandaops.io). `proposer_payload_delivered` is not yet populated for gloas reveals.

## 7. Not supported yet, in planned order

1. Dehydrated submissions (block deltas) — block access lists make bodies larger; we plan per-account dedup of the list.
2. Header-only (v3) submissions — must carry the block access list to stay adjustable.
3. Block merging.
4. Native ePBS bids — depends on builder interest, see the design doc.

## 8. Devnet quick start (glamsterdam-devnet-8)

| | |
|---|---|
| Relay | `https://relay-plataberget.ultrasound.money` |
| Chain id | `7091047534` |
| Gloas fork | epoch 1536 = slot `49152` |
| Gas limit | 200M, announced per proposer in the lookahead |
| Config / checkpoint / explorer | https://config.glamsterdam-devnet-8.ethpandaops.io · https://checkpoint-sync.glamsterdam-devnet-8.ethpandaops.io · https://dora.glamsterdam-devnet-8.ethpandaops.io |
| Clients | `ethpandaops/{reth,lighthouse}:glamsterdam-devnet-8` |

1. Point your builder at the relay host with your hoodi `X-Api-Token`.
2. Submit `trustless-full` bids: pay `message.value` to the relay wallet in the last tx, keep `proposer_fee_recipient` as announced.
3. Watch `builder_blocks_received` and Dora. Roughly 1 in 16–20 slots has a proposer that pulls bids over the builder API; the rest are decided on p2p, where our test builder competes with lowball bids.
4. Ask us for optimistic status, a pubkey mapping (`trustless-partial`) or an adjustment wallet (`trusted` + adjustments) when you want those paths.

Questions and feedback: [@ultrasoundrelay](https://t.me/ultrasoundrelay).
