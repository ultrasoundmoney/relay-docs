# data api

data api is available both local, through builder subdomains as found in [builder-getting-started.md](builder-getting-started.md), and global, through the relay-analytics. block submissions and their adjusted counterparts are available locally and are aggregated with some delay into a single, global, relay-analytics view. all other relay data api resources only exist globally.

```bash
# eg local rbx
https://relay-builders-eu.ultrasound.money/relay/v1/data/bidtraces/builder_blocks_received

# eg global
https://relay-analytics.ultrasound.money/relay/v1/data/bidtraces/builder_blocks_received
```

