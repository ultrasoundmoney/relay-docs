# rate limits

We've introduced rate limits to improve the quality, reliability and fairness of the auction across the network. There are two levels of rate limiting.

### unknown builders

By default unknown builders are subject to stricter rate limits than authenticated builders.

To become an authenticated builder simply [ask us to issue you an API token](https://t.me/ultrasoundrelay) and add it to your `X-Api-Token` header when sending through requests.

### authenticated builders

Authenticated builders are subject to a bandwidth rate limit of 60 MB per slot per region.

Based on our analysis this should be more than enough for most use cases, but we'll be monitoring usage and may adjust it over time as needed.

### managing rate limits

To ensure you get the most out of your rate limit we recommend:

1. getting an API token and authenticating your submissions
2. using SSZ encoding all submissions
3. considering implementing [dehydrated bids](https://docs.titanrelay.xyz/builders/builder-integration#block-deltas) or [optimistic-v3.md](optimistic-v3.md) which are more efficient and will allow for more requests per slot
4. ensuring your builder is set up to handle 429 (Too Many Requests) responses gracefully
5. testing your builder against our `hoodi` relays to avoid unnecessary usage of your bandwidth limit on `mainnet`.



If you find that you need more bandwidth than the free tier, we do have a paid option available.  Please reach out to [smilingalex](https://t.me/smilingalex) or the [ultra sound team](https://t.me/ultrasoundrelay) for details.
