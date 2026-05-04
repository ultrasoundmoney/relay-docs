# floor bid

every submission carries a `cancellations` flag, communicated via a query param, e.g. `cancellations=true` (defaults to `false` if omitted). non-cancellable submissions are commitments: by sending one, you are telling the relay this bid stands and you will not pull it back to a lower value.

to enforce that commitment, the relay tracks a per-auction **floor**: the highest non-cancellable value seen so far in that `(slot, parent_hash, proposer_pubkey)`. once set, the floor only goes up — a higher non-cancellable bid replaces it; nothing lowers it within the slot.

## what hits the floor

on every new submission the relay loads the current floor and compares:

- `cancellations=false` and `value <= floor` → the bid is dropped. it is not stored, not simulated, and cannot win.
- `cancellations=true` and `value < floor` → if you have a previous bid at-or-above the floor, that previous bid is cancelled (deleted). the new bid itself is not stored as a candidate.
- `value > floor` → remaining validation runs.

in the first two cases the relay returns:

```
HTTP 202 Accepted
accepted bid below floor, skipped validation
```

**this is a 202, not a rejection.** the request was well-formed and accepted; the relay simply skipped processing it because no value at-or-below the floor can affect the auction outcome. client-side log pipelines that key off the response body sometimes flag this as a failure — it isn't.

## related

- [bid forwarding](bid-forwarding.md) — the floor is per auction instance; gossiped top bids are evaluated separately at the edge.
