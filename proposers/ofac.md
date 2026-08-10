# ofac

the list of ofac sanctioned addresses the relay currently filters against is served live by the data api:

```bash
https://relay-analytics.ultrasound.money/ultrasound/v1/data/disallow
```

it returns a json array of execution addresses. see [data-api.md](../builders/data-api.md) for the rest of the data api.

the [ofac-ethereum-addresses](https://github.com/ultrasoundmoney/ofac-ethereum-addresses) repo previously used to track these addresses is deprecated in favor of the endpoint above.

if you are an operator with strict requirements and have questions, please contact us: [https://t.me/ultrasoundrelay](https://t.me/ultrasoundrelay) or email [contact@ultrasound.money](mailto:contact@ultrasound.money).
