# ePBS (Gloas) auction — builder integration

*Ultra Sound relay — updated 2026-10-13 (first draft 2026-07-31). Feedback wanted, see the end.*

## The forcing function

Post-ePBS, a block only reaches the chain via a signed [`ExecutionPayloadBid`](https://github.com/ethereum/consensus-specs/blob/master/specs/gloas/beacon-chain.md#executionpayloadbid) — signed by a registered builder identity whose on-chain stake covers the bid value. This holds for **every** bid — including trusted ones.

## Design space

Three ways to run a relay auction on top:

1. **Partially mediated**: you stake a builder wallet and share its key with us; builders submit blocks as today and the relay forms the on-chain bid, signed with *your* identity.
2. **Fully mediated**: one shared staked wallet operated by Ultra Sound; you pay per included block, essentially as pre-ePBS.
3. **ePBS-native**: builders construct and sign valid ePBS bids themselves and send those to the relay.

Independent of the design, a bid pays the proposer in one of two ways:

- **Trustless**: from the staked identity's CL balance, via the bid `value`.
- **Trusted**: inside the EL payload, as today, via `execution_payment`.

## Overview

Three bid types are implemented so far. Any design can carry trusted bids; for now we offer them on the fully mediated design (2) only — the relay signs with its own identity, nothing on-chain from you. ePBS-native submissions are deferred until builder feedback shows demand.

<!-- small -->
|                      | Trusted                 | Fully mediated          | Partially mediated      | ePBS-native             |
|--------------------|---------------------------|---------------------------|------------------------|-----------------|
| **Status**           | Live (devnet)           | Live (devnet)           | Live (devnet)           | **Not yet**[^native-status] |
| **Bid signed by**    | Relay                   | Relay                   | Relay (your key)        | **You**                 |
| **Staked identity**  | Relay's                 | Relay's                 | **Yours**               | Yours                   |
| **Key custody**      | Relay                   | Relay                   | **Shared**              | You                     |
| **Upfront capital**  | None                    | None                    | Full max bid            | Full max bid            |
| **Submission API**[^payload] | Unchanged       | Unchanged               | Unchanged               | **New**[^native]        |
| **Paid in payload**  | Proposer                | **Relay**               | None                    | None                    |
| **When you pay**[^attestation] | Payload inclusion | Payload inclusion   | Block inclusion      | Block inclusion         |
| **How you pay**      | EL tx                   | EL tx                   | CL withdrawal           | CL withdrawal           |
| **Bid capped by**    | `max_execution_payment` | Relay stake             | Your stake              | Your stake              |
| **Adjustments**[^adjustments] | Yes            | Yes                     | Yes                     | No                      |
| **Payload reveal**   | Relay                   | Relay                   | Relay                   | **You**                 |
| **`X-Bid-Type`**[^header] | `trusted` (default) | `trustless-full`       | `trustless-partial`     |                         |
<!-- /small -->

[^native-status]: Not accepted yet; see feedback.
[^header]: Request header on the block submission that selects the bid type; missing header means `trusted`.
[^payload]: Some payload details change irrespective of this design decision: the extended `execution_requests` (EIP-8282) and block access lists (EIP-7928).
[^native]: Submissions become ePBS bids, constructed and signed by the builder.
[^attestation]: Once the proposer includes your signed bid in a CL block, you are committed to pay, independent of whether the EL payload is revealed or valid.
[^adjustments]: See [bid adjustments](../builders/bid-adjustment.md).

## What this means for you

- **All types**: the payload gains the Glamsterdam EL fields — block access list (EIP-7928), extended execution requests (EIP-8282), explicit `parent_beacon_block_root`. Fork changes, not ePBS changes.
- **`trusted`**: nothing else. Pay the proposer in the payload as today; adjustments as today. Only for proposers with a `max_execution_payment` in the lookahead.[^lookahead]
- **`trustless-full`**: set the header; pay `message.value` to the relay wallet in the payload instead of the proposer. Nothing on-chain.
- **`trustless-partial`**: set the header; no payload payment. Needs a Gloas builder deposit and your key shared with us.
- **Test now** on glamsterdam-devnet-8: `https://relay-plataberget.ultrasound.money`, your Hoodi `X-Api-Token` works. Details, formats and rejections: [Gloas builder API](gloas-builder-api.md).
- **Not yet** on Gloas: dehydrated submissions, header-only (v3), merging.
- **Hoodi** forks on 2026-10-26 17:42 UTC (epoch 132352); mainnet date to be announced.

[^lookahead]: `/relay/v1/builder/validators`, `preferences.max_execution_payment` (gwei); absent means the proposer takes trustless bids only. Stake caps: `GET /ultrasound/v1/builder/bid_caps` (relay stake; with your `X-Api-Token` also your mapped stake).

## Feedback wanted

1. **Native bids** — builder keeps its key; submissions become signed bids, builder releases the payload, no adjustments. Interest? Free basic tier + paid simulation tier?
2. **Partially vs fully mediated** — your staked wallet (key shared) or ours (invoiced per block)?
3. **Priorities** — dehydration (block access lists make bodies bigger; per-account dedup planned), header-only, merging.
4. **Payment flow** — anything in paying the relay wallet (`trustless-full`) or `execution_payment` (`trusted`) that doesn't fit your builder?

Talk to us on Telegram: [@ultrasoundrelay](https://t.me/ultrasoundrelay).
