# data api

data api is available both local, through builder subdomains as found in [builder-getting-started.md](builder-getting-started.md), and global, through the relay-analytics. block submissions and their adjusted counterparts are available locally and are aggregated with some delay into a single, global, relay-analytics view. all other relay data api resources only exist globally.

```bash
# eg local rbx
https://relay-builders-eu.ultrasound.money/relay/v1/data/bidtraces/builder_blocks_received

# eg global
https://relay-analytics.ultrasound.money/relay/v1/data/bidtraces/builder_blocks_received
```

For bid submission records such as `builder_blocks_received`, `timestamp`, `timestamp_ms` and `timestamp_ns` refer to when the relay received the bid submission (`received_at` internally). This is distinct from top bid websocket update timestamps, which describe when a bid became eligible/top after decoding and auction-state comparison.

## /relay/v1/data/bidtraces/builder_blocks_received

replicates [getReceivedBids](https://flashbots.github.io/relay-specs/#/Data/getReceivedBids) from the relay specs.

query parameters, at least one of the filters is required:

| parameter | description |
|---|---|
| `slot` | a specific slot |
| `block_hash` | a specific block hash |
| `block_number` | a specific execution block number |
| `builder_pubkey` | all submissions from one builder pubkey |
| `limit` | max rows returned, default and maximum 500. only applies when filtering by `builder_pubkey` alone; slot, hash and number queries return all matching rows |

results are ordered by slot descending, most recently inserted first. `optimistic_submission` is not part of the spec schema; it is a de-facto extension also returned by the flashbots relay, which our response matches key-for-key.

```json
[
  {
    "slot": "3635889",
    "parent_hash": "0x11629d94dc6f4f0cc5efc530b055f41c5cbe5c7382f0ac775fee112a14d49418",
    "block_hash": "0x52753878bb890715f1f8b91e18d65cdf0bd2d2b722126c87aae59a215df66081",
    "builder_pubkey": "0x80ff91f2b5db3628ddc2863d3317e5baca972c32e86c1b4b9bc98c3424c8e36fd318d105c1fcd99f94f898a15d13cb8a",
    "proposer_pubkey": "0xa1e27f5820549640bc3ae51127a3bcfc18f3ed77e0565baa2346ef178f38542dae5a0a8057ded413a6733f00df3e3731",
    "proposer_fee_recipient": "0x09a43fd8ff63b79035f3c3bbe2e95c943fc0ed48",
    "gas_limit": "60000000",
    "gas_used": "58058609",
    "value": "6983905659119621",
    "num_tx": "40",
    "block_number": "3350783",
    "timestamp": "1785844068",
    "timestamp_ms": "1785844068170",
    "optimistic_submission": true
  }
]
```

## /relay/v2/data/bidtraces/builder_blocks_received

identical to v1, same hosts, same query parameters, same semantics, with one added response field:

| field | description |
|---|---|
| `timestamp_ns` | nanosecond unix timestamp of when the relay received the bid submission. same instant `timestamp` and `timestamp_ms` truncate. `null` for submissions recorded before the field existed |

```json
[
  {
    "slot": "3635889",
    "...": "...",
    "timestamp": "1785844068",
    "timestamp_ms": "1785844068170",
    "timestamp_ns": "1785844068170783602",
    "optimistic_submission": true
  }
]
```

the v2 api is not part of the relay specs and has not stabilized yet; we may make further changes to it.

## /relay/v1/data/bidtraces/proposer_payload_delivered

replicates [getDeliveredPayloads](https://flashbots.github.io/relay-specs/#/Data/getDeliveredPayloads) from the relay specs.

## /relay/v1/data/validator_registration

replicates [getValidatorRegistration](https://flashbots.github.io/relay-specs/#/Data/getValidatorRegistration) from the relay specs.

## /ultrasound/v1/data/registration

when a proposer registration was last received. complements the standard `validator_registration` endpoint, which does not expose the receive time of repeated identical registrations. see [recent-registration.md](recent-registration.md).

## /ultrasound/v1/data/adjustments

non-standard. pre-adjustment values for adjusted bids. see [bid-adjustment.md](bid-adjustment.md).
