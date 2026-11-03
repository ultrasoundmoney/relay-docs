# OFAC

The list of OFAC-sanctioned addresses the relay currently filters against is served live by the data API:

```bash
https://relay-analytics.ultrasound.money/ultrasound/v1/data/disallow
```

It returns a JSON array of execution addresses. See [data-api.md](../builders/data-api.md) for the rest of the data API.

The [ofac-ethereum-addresses](https://github.com/ultrasoundmoney/ofac-ethereum-addresses) repo previously used to track these addresses is deprecated in favor of the endpoint above.

If you are an operator with strict requirements and have questions, please contact us: [https://t.me/ultrasoundrelay](https://t.me/ultrasoundrelay) or email [contact@ultrasound.money](mailto:contact@ultrasound.money).
