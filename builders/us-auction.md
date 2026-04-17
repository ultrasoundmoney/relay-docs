# US auction

we have a second auction instance in the US. proposer requests are routed to the geographically closest instance.

the US instance runs on OVH bare metal machines in the VIN (Vint Hill) data center, [roughly 2.5ms from AWS us-east-1](https://was1-vin.smokeping.ovh.net/smokeping??&target=USA.AS16509-us-east-1). co-location requires an account on [OVH US](https://us.ovhcloud.com/).

see [builder-getting-started.md](builder-getting-started.md) for URLs.

the following routes are available for the EU and US instances:

* `/relay/v1/builder/blocks`
* `/relay/v1/builder/validators`
* `/eth/v1/builder/header`
* `/ws/v1/top_bid`
