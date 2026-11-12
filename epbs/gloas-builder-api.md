# Gloas builder API

*Ultra Sound relay — draft 2026-10-20. Companion to the [auction design](builder-auction.md).*

How to submit blocks to the Ultra Sound relay on a Gloas (ePBS) network, and how to test on glamsterdam-devnet-8 today. Everything not mentioned here works as pre-ePBS ([builders](../builders/)).

## 1. Endpoints and auth

Devnet host: `https://relay-plataberget.ultrasound.money` (Hoodi and mainnet hosts stay as they are after their forks).

| Method / path | Purpose |
|------------------------------------------------|----------------------------------|
| `GET /relay/v1/builder/validators` | Lookahead: upcoming proposers with their effective preferences (fee recipient, gas limit, filtering, `max_execution_payment` when they accept trusted bids) |
| `GET /ultrasound/v1/builder/bid_caps[?slot=N]` | Stake caps for trustless bids: the relay's available stake; with your `X-Api-Token`, your mapped wallets' too |
| `POST /relay/v1/builder/blocks[?cancellations=1][&adjustments=1]` | Block submission (Gloas format from the fork slot on) |
| `GET /relay/v1/data/bidtraces/builder_blocks_received?slot=N` | What we received and whether it simulated (v2: ns timestamps) |
| `GET /ws/v1/top_bid`, `/ws/v2/top_bid` | Top-bid websocket, unchanged |
| `GET /ultrasound/v1/auction/ip_test` | Connectivity check |

- `X-Api-Token`: your existing token; on the devnet your **Hoodi** token works. Authenticated: 120 MiB of submissions per slot. Without a token: 50 requests/min per IP, attributed to builder `unknown`.
- Unknown builder pubkeys are accepted (no allowlist) and treated as non-optimistic.
- Optional headers as before: `X-Sequence` (monotonic per slot); `X-Hydrate` (dehydrated submissions, needs `X-Api-Token`, see [§3](#dehydrated-submissions-x-hydrate)); `X-Mergeable` is **not** supported on gloas slots (400).

## 2. Bid types and payment

`X-Bid-Type` selects who pays the proposer and whose stake backs the on-chain bid. Missing header = `trusted`.

| `X-Bid-Type` | You put in the payload | Relay wraps as | Proposer paid by | Needs |
|------------|--------------------|--------------------|-----------------|------------------------------|
| `trusted` | Payment to `proposer_fee_recipient`, as pre-ePBS | `value = 0`, `execution_payment = value / 1e9` | You, in the payload | A proposer who accepts execution payments (a cap in their preferences); simulated unless you are optimistic |
| `trustless-full` | Payment of `message.value` to the **relay wallet** | `value = value / 1e9`, relay identity | Relay, from its CL stake | Nothing on-chain from you. **Start here.** |
| `trustless-partial` | Nothing | `value = value / 1e9`, **your** identity | You, from your CL builder stake | A Gloas builder deposit + a mapping submission pubkey → staked pubkey on our side (ask us) |
| `native` | — | — | — | Not accepted yet |

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

`POST /relay/v1/builder/blocks` with `Content-Type: application/json` or `application/octet-stream` (SSZ) and `Eth-Consensus-Version: gloas`. The header selects the Gloas format as it selects every other fork's; without it the slot (`message.trace.slot`; first 8 LE bytes in SSZ) decides from the fork slot on. A header that disagrees with the slot is rejected. JSON rejects unknown fields.

```text
SignedBidSubmissionGloas {
  message: GloasBidTrace {
    trace:              BidTrace          // unchanged (slot, hashes, pubkeys,
                                          // fee recipient, gas, value)
    parent_block_root:  bytes32           // EIP-4788 parent beacon block root the
                                          // payload builds on
  }
  execution_payload:    ExecutionPayload  // consensus gloas shape: the deneb fields, then
                                          //   block_access_list: bytes   EIP-7928 RLP from
                                          //                              your EL; hex in JSON
                                          //   slot_number:       uint64  = message.trace.slot
  blobs_bundle:         BlobsBundle       // 128 cell proofs per blob; bodies required
  execution_requests:   ExecutionRequests // deposits, withdrawals, consolidations,
                                          // builder_deposits, builder_exits (EIP-8282)
  signature:            BLS               // over hash_tree_root(message), builder domain
}
```

- The container, field order and signing root are helix's `SignedBidSubmissionGloas` / `GloasBidTrace`; `execution_payload` is the consensus-specs Gloas `ExecutionPayload` (block access list and slot number inside).
- `signature` is the builder-domain signature (`DOMAIN_APPLICATION_BUILDER`, genesis fork version, as pre-ePBS) over `hash_tree_root(message)`, i.e. over `trace` *and* `parent_block_root`, not over the bid trace alone.
- `execution_payload.slot_number` must equal `message.trace.slot`.
- `message.parent_block_root` must match the payload attributes our beacon nodes announced for `(slot, parent_hash)`; after an empty slot `(slot, parent_hash)` alone no longer identifies the block, which is why it is signed.
- The block access list is validated by execution: the simulator rebuilds it and compares hashes.

### Dehydrated submissions (`X-Hydrate`)

Send `X-Hydrate: true` with your `X-Api-Token` to reference transactions, blobs and block access list entries already sent in the same slot. The container is the one above with a dehydrated `execution_payload` and `blobs_bundle`; SSZ or JSON, `?adjustments=1` appends `adjustment_data` as usual.

```text
DehydratedSignedBidSubmissionGloas {
  message:              GloasBidTrace               // as above
  execution_payload:    DehydratedExecutionPayload  // the gloas payload, except:
                                                    //   transactions: each full bytes, or its
                                                    //     8-byte key (see below)
                                                    //   block_access_list: DehydratedBlockAccessList
  blobs_bundle:         DehydratedBlobsBundle
  execution_requests:   ExecutionRequests
  signature:            BLS                         // over hash_tree_root(message), as above
}

DehydratedBlockAccessList {
  refs:       List[bytes32]  // keccak256 of each raw RLP AccountChanges item, in list order
  new_items:  List[bytes]    // the raw items not sent yet this slot, in the same order
}

DehydratedBlobsBundle {
  commitments:  List[KzgCommitment]  // every blob of the block
  new_items:    List[BlobItem]       // blobs not sent yet: {proof: List[KzgProof] (128 cell
                                     // proofs), commitment, blob}
}
```

- Transactions and blobs work as pre-ePBS [block deltas](../builders/builder-getting-started.md#block-deltas): a transaction's key is the FxHash (`u64`, little-endian) of its last 67 bytes; transactions shorter than 67 bytes are rejected. The gloas container has no `tx_root`.
- Block access list: split your EL's `blockAccessList` into its top-level RLP items, send `refs` for all of them and the bytes of the unseen ones in `new_items`. The relay reassembles `rlp_list(items in refs order)` byte-exactly, so the hash matches the payload's `block_access_list_hash`; it does not re-sort.
- Keep a per-slot set of what you sent (transaction keys, blob commitments, item hashes); clear it on a new slot. The relay caches per builder pubkey for the current and previous slot; the pubkey must belong to your token.
- Items are keyed by content, which includes block access indices: changing only the trailing payment transaction re-sends a few accounts, inserting or reordering transactions re-sends every account touched after the change.
- On a 400 `hydration: unknown …` (cache miss, e.g. after a relay restart): clear your set and resend the submission in full. Everything sent in full in the rejected submission is cached anyway.

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
| 400 `invalid x-bid-type header, expected one of: ...` | Not one of the four types |
| 400 `bid type not supported yet` | `native` |
| 400 `mergeable submissions are not supported for gloas slots` | Drop `X-Mergeable` |
| 401 `dehydrated submissions require an api token` | `X-Hydrate` without `X-Api-Token` |
| 400 `hydration: unknown tx hash at index N` / `unknown blob commitment … at index N` / `unknown block access list entry … at index N` | Cache miss: clear what you sent, resend in full |
| 400 `hydration: invalid tx length …` / `too many …` / `undecodable block access list: …` | Malformed dehydrated submission |
| 400 `builder_pubkey does not match authenticated identity` | `X-Hydrate` for a pubkey not owned by your token |
| 400 `adjustable submissions must be trusted bids, got ...` | `?adjustments=1` on a trustless bid |
| 400 `invalid eth-consensus-version header. expected one of: ...` | Unknown `Eth-Consensus-Version` |
| 400 `gloas submissions are not enabled` | `gloas` header on a relay without the fork configured |
| 400 `invalid block submission version, expected gloas` (or `expected fulu`) | `Eth-Consensus-Version` disagrees with the slot: pre-Gloas header on a Gloas slot, or `gloas` before the fork |
| 400 `failed to parse gloas json/ssz payload: ...` | Shape error; JSON rejects unknown fields |
| 400 `slot_number N does not match message.slot M` | `execution_payload.slot_number` disagrees with `message.trace.slot` |
| 400 `signature verification failed` | Signature not over `hash_tree_root(message)` (trace + `parent_block_root`) |
| 400 `blob bodies are required` / `expected 128 cell proofs per blob` | |
| 400 `unknown parent beacon block root` | Not the root announced for `(slot, parent_hash)` |
| 400 `could not find slot duty` / `proposer pubkey does not match slot duty` / `fee recipient mismatch` | Lookahead mismatch |
| 400 `no proposer preferences for slot` | Proposer never published preferences |
| 400 `proposer does not accept trusted bids` | Trusted bid, proposer without execution-payment cap |
| 400 `bid value exceeds relay available stake` | `trustless-full` above our spendable stake net of in-flight bids |
| 400 `no gloas wallet mapped for builder` / `mapped gloas wallet not active` / `bid value exceeds spendable stake` | `trustless-partial` without (sufficient) mapped stake |
| 400 `submission for past slot` / `already using a newer payload` | Too late / superseded |
| 400 `<simulator message>` | Simulation failed: payment, gas limit, block access list hash, state root, ... |
| 401 `invalid api token` | Unknown `X-Api-Token` |
| 429 | Rate limit (see §1) |
| 500 | Relay-side error; retry, tell us if persistent |

## 6. Checking what happened

- `builder_blocks_received?slot=N` (or `?builder_pubkey=...`): your submissions, simulation outcome, received timestamps.
- Wins: the beacon block's `signed_execution_payload_bid` (builder index 12057 = Ultra Sound on devnet) and the revealed payload, e.g. on [Dora](https://dora.glamsterdam-devnet-8.ethpandaops.io). `proposer_payload_delivered` is not yet populated for Gloas reveals.

## 7. Not supported yet, in planned order

1. Header-only (v3) submissions — must carry the block access list to stay adjustable.
2. Block merging.
4. Native ePBS bids — depends on builder interest, see the design doc.

## 8. Devnet quick start (glamsterdam-devnet-8)

| | |
|---|---|
| Relay | `https://relay-plataberget.ultrasound.money` |
| Chain id | `7091047534` |
| Gloas fork | Epoch 1536 = slot `49152` |
| Gas limit | 200M, announced per proposer in the lookahead |
| Config / checkpoint / explorer | https://config.glamsterdam-devnet-8.ethpandaops.io · https://checkpoint-sync.glamsterdam-devnet-8.ethpandaops.io · https://dora.glamsterdam-devnet-8.ethpandaops.io |
| Clients | `ethpandaops/{reth,lighthouse}:glamsterdam-devnet-8` |

1. Point your builder at the relay host with your Hoodi `X-Api-Token`.
2. Submit `trustless-full` bids: pay `message.value` to the relay wallet in the last tx, keep `proposer_fee_recipient` as announced.
3. Watch `builder_blocks_received` and Dora. Roughly 1 in 16–20 slots has a proposer that pulls bids over the builder API; the rest are decided on p2p, where our test builder competes with lowball bids.
4. Ask us for optimistic status, a pubkey mapping (`trustless-partial`) or an adjustment wallet (`trusted` + adjustments) when you want those paths.

Questions and feedback: [@ultrasoundrelay](https://t.me/ultrasoundrelay).
