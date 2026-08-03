# ePBS (Gloas) auction — builder integration sketch

*ultra sound relay — draft for builder feedback, 2026-07-31*

## The forcing function

Post-ePBS, a block only reaches the chain via a signed [`ExecutionPayloadBid`](https://github.com/ethereum/consensus-specs/blob/master/specs/gloas/beacon-chain.md#executionpayloadbid) — signed by a registered builder identity whose on-chain stake covers the bid value. This holds for **every** bid — including trusted ones.

## Design space

Three ways to run a relay auction on top:

1. **Partially mediated**: you stake a builder wallet and share its key with us; builders submit blocks as today and the relay forms the on-chain bid, signed with *your* identity.
2. **Fully mediated**: one shared staked wallet operated by ultra sound; you pay per included block, essentially as pre-ePBS.
3. **ePBS-native**: builders construct and sign valid ePBS bids themselves and send those to the relay.

## Overview

|                             | (1) Partially mediated    | (2) Fully mediated | (3) ePBS-native        |
|-----------------------------|---------------------------|--------------------|------------------------|
| **Bid signed by**           | relay                     | relay              | you                    |
| **Staked identity**         | per builder               | one per relay      | per builder            |
| **# Signed bids**           | one per slot              | one per slot       | every submission       |
| **Staked wallet**           | yours, key shared with us | ultra sound's      | yours, you operate     |
| **Builder upfront capital** | Full max bid              | None               | Full max bid           |
| **Submission API**          | unchanged[^payload]       | unchanged[^payload] | new[^native]          |
| **When you pay**            | CL block inclusion[^attestation] | EL payload inclusion | CL block inclusion[^attestation] |
| **Whom you pay**            | proposer                  | ultra sound        | proposer               |
| **How you pay (trustless)** | CL withdrawal             | EL tx              | CL withdrawal          |
| **How you pay (trusted)**   | EL tx                     | EL tx              | EL tx                  |
| **Adjustments**[^adjustments] | yes                     | yes                | no                     |
| **Payload publishing**      | relay and/or you          | only relay         | only you               |
| **Relay simulation**        | optional[^simulation]     | default required[^simulation][^collaboration] | default none[^simulation] |

[^payload]: Some payload details change irrespective of this design decision: the extended `execution_requests` (EIP-8282) and block access lists (EIP-7928).
[^native]: Submissions become ePBS bids, constructed and signed by the builder.
[^attestation]: Once the proposer includes your signed bid in a CL block, you are committed to pay, independent of whether the EL payload is revealed or valid.
[^adjustments]: See [bid adjustments](../builders/bid-adjustment.md).
[^simulation]: Still required for trusted bids / OFAC filtering. The proposer is paid whether or not the EL payload is valid, making simulation doubly important for builders. Ultra Sound is happy to continue offering this as a service.
[^collaboration]: May be skipped in collaboration through e.g. optimistic collateral.

## Our preference

We prefer a mediated auction (1 or 2): these designs let builders focus on execution payloads while Ultra Sound handles the rest. They also preserve bid adjustments, which benefit larger builders and Ultra Sound. We are also open to ePBS-native submissions, with at least a free basic tier or higher paid service tier.

## Status

- A prototype of the partially mediated design (1) is implemented and running end-to-end on **glamsterdam-devnet-6**: submission → auction → signed on-chain bid → reveal, with a test builder.
- Migration to **devnet-7** is next; deployment on **hoodi** as soon as it forks.
- If you want to join the testing, we can expose / share the required endpoints with you.

## Open questions (feedback wanted)

1. **API differentiation of trustless vs trusted bids** — separate endpoint, a submission flag, or inferred (presence of an in-payload payment / `execution_payment`)? Preferences welcome, especially if you plan to run both kinds side by side.
2. **Partially vs fully mediated** — would you rather operate your own staked wallet (partially mediated), or have ultra sound manage the staked balance and pay per included block (fully mediated)?
