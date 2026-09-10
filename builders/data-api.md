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

## /ultrasound/v1/data/disallow

non-standard. the ofac disallow list the relay currently filters against, as a json array of execution addresses. takes no parameters.

```json
["0x098B716B8Aaf21512996dC57EB0615e2383E2f96", "0xa0e1c89Ef1a489c9C7dE96311eD5Ce5D32c20E4B", "..."]
```

## /relay/v1/data/merged_blocks

not part of the relay specs. block merging results for a single slot, matching the shape titan serves at the same path.

| parameter | description |
|---|---|
| `slot` | required, a specific slot |

returns one entry per merged block we built for that slot, not only the one delivered, ordered by `proposer_value` descending. slots we did not merge return `[]`.

| field | description |
|---|---|
| `original_block_hash` | hash of the base block before merging |
| `block_hash` | hash of the resulting merged block |
| `original_value` | value of the base block before merging |
| `proposer_value` | value paid to the proposer for the merged block |
| `total_merged_value` | value added by the merged in orders, the sum of every `contribution` |
| `base_builder_revenue` | the base block builder's share of the merged value |
| `relay_revenue` | the relay's share of the merged value |
| `original_tx_count`, `merged_tx_count` | transaction count before and after merging |
| `original_blob_count`, `merged_blob_count` | blob count before and after merging |
| `original_gas_used`, `merged_gas_used` | gas used before and after merging |
| `builder_inclusions` | contributing builders, keyed by coinbase. `contribution` is the value they added, `revenue` their share of it, `txs` the transaction hashes appended to the base block |

```json
[
  {
    "slot": "15184509",
    "block_number": "25946559",
    "original_block_hash": "0x292fc1ec652098c5e44cf1b584c53d8c14bf5a850ca5816fdfb070a90a13a3f7",
    "block_hash": "0x9fb4e6dd48d7f9905f596e3287c5f22f029056730ece34361e6e29cd167984f5",
    "original_value": "11597027056460525",
    "proposer_value": "11628962247064035",
    "total_merged_value": "143878148154040",
    "base_builder_revenue": "31935190603510",
    "relay_revenue": "31935190603510",
    "original_tx_count": "380",
    "merged_tx_count": "386",
    "original_blob_count": "4",
    "merged_blob_count": "4",
    "original_gas_used": 36764446,
    "merged_gas_used": 37242697,
    "builder_inclusions": {
      "0x4838b106fce9647bdf1e7877bf73ce8b0bad5f97": {
        "contribution": "143878148154040",
        "revenue": "31935190603510",
        "txs": [
          "0x84516dcb997ba8b0155be3b078669d9371c3836244eaac57e8a811e859fb8bc9",
          "0xbd5d00770d957be3a1bfe7ce2988421ce09b4f5f9b14ff0c3b7bc7bdfe5f7e3e"
        ]
      }
    }
  }
]
```
