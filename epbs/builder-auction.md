# ePBS (Gloas) auction — builder integration sketch

*ultra sound relay — draft for builder feedback, 2026-07-31*

## The problem

Post-ePBS, a block only reaches the chain via a signed
[`ExecutionPayloadBid`](https://github.com/ethereum/consensus-specs/blob/master/specs/gloas/beacon-chain.md#executionpayloadbid)
— signed by a registered builder identity whose on-chain stake covers the bid
value. This holds for **every** bid — including trusted ones.

## Design space

Three ways to run a relay auction on top:

1. **Partially mediated**: you stake a builder wallet and share its key with
   us; builders submit blocks as today and the relay forms the on-chain bid,
   signed with *your* identity.
2. **Fully mediated**: one shared staked wallet operated by ultra sound; you
   pay per included block, essentially as pre-ePBS.
3. **ePBS-native**: builders construct and sign valid ePBS bids themselves
   and send those to the relay.

## Overview

|                             | (1) Partially mediated    | (2) Fully mediated | (3) ePBS-native        |
|-----------------------------|---------------------------|--------------------|------------------------|
| **Bid signed by**           | relay                     | relay              | you                    |
| **Staked identity**         | per builder               | one per relay      | per builder            |
| **# Signed bids**           | one per slot              | one per slot       | every submission       |
| **Staked wallet**           | yours, key shared with us | ultra sound's      | yours, you operate     |
| **Upfront capital**         | your stake                | none               | your stake             |
| **Submission API**          | unchanged[^payload]       | unchanged[^payload] | new[^native]          |
| **When you pay**            | attestation[^attestation] | block inclusion    | attestation[^attestation] |
| **Whom you pay**            | proposer                  | ultra sound        | proposer               |
| **How you pay (trustless)** | CL withdrawal             | EL tx              | CL withdrawal          |
| **How you pay (trusted)**   | EL tx                     | EL tx              | EL tx                  |
| **Adjustments**[^adjustments] | yes                     | yes                | no                     |
| **Payload publishing**      | relay and/or you          | only relay         | only you               |
| **Relay simulation**        | optional[^simulation]     | required           | none                   |

[^payload]: some payload details change irrespective of this design decision:
    the extended `execution_requests` (EIP-8282) and block access lists
    (EIP-7928).
[^native]: submissions become ePBS bids, constructed and signed by the
    builder.
[^attestation]: independent of whether the block is actually included, i.e.
    whether the payload has been revealed.
[^adjustments]: see [bid adjustments](../builders/bid-adjustment.md).
[^simulation]: still required for trusted bids / ofac filtering.

## Our preference

A mediated auction (1 or 2), mainly because ePBS-native rules out
adjustments: the bid value is fixed by your own signature, so the relay
cannot re-sign it on your behalf. That said, we are open to also supporting
ePBS-native submissions — though likely as a paid feature.

## Status

- A prototype of the partially mediated design (1) is implemented and running
  end-to-end on **glamsterdam-devnet-6**: submission → auction → signed
  on-chain bid → reveal, with a test builder.
- Migration to **devnet-7** is next; deployment on **hoodi** as soon as it
  forks.
- If you want to join the testing, we can expose / share the required
  endpoints with you.

## Open questions (feedback wanted)

1. **API differentiation of trustless vs trusted bids** — separate endpoint,
   a submission flag, or inferred (presence of an in-payload payment /
   `execution_payment`)? Preferences welcome, especially if you plan to run
   both kinds side by side.
2. **Partially vs fully mediated** — would you rather operate your own
   staked wallet (partially mediated), or have ultra sound manage the staked
   balance and pay per included block (fully mediated)?
