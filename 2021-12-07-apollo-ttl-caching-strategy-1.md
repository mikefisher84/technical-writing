# Apollo Time-To-Live (TTL) Caching Strategy

Date: 2021-12-07

## Author

Mike Fisher (@mikefisher84)

## Status

Approved

## Context and Background

We have many types of data in our app that need to be kept up-to-date but are unlikely to change at a high frequency. For example, we want to show the latest subcategories that belong to a market category, so we want to fetch this data from the GraphQL API. However, if we fetch the same data five minutes later, it's very likely to be exactly the same as the first time we fetched, it since the market taxonomy changes infrequently. Therefore, we would like to be able to cache this kind of data rather than fetch it from the API every time it's needed, but we would like to refresh the data once the cached version has reached a certain age (TTL).

Unfortunately, Apollo Client gives us [limited options](https://www.apollographql.com/docs/react/data/queries/#setting-a-fetch-policy) out-of-the-box for controlling how long data lives in the cache and when to make network requests. In one option ('cache-first', the default), data is _never_ fetched again if it already exists in the cache, leading to stale data. The other two most common options ('cache-and-network', 'network-only') will always keep data fresh, but they also result in network requests every time the data is requested. So this leaves us on our own to devise a strategy for expiring cached items after a period of time.

### Scope of change

The TTL caching strategy that we have devised works like this:

- We created [`responseTimestamp`](https://github.com/*redacted*/*redacted-app-name*/blob/master/src/graphql/schema.ts#L18) query on the API, which returns an ID and a time the request was resolved. Since GraphQL can perform multiple query operations in a single request, the `responseTimestamp` query can be included in any other query for other data.
- On the client side, whenever we need to fetch data, we can first fetch it from the cache if available, (by using the default 'cache-first' fetch policy). We can then check the timestamp on the _cached_ data, and if it's too old, refetch the data.

Some advantages of this approach are: the request timestamp can be added to any query without needing to write any new server-side code, and custom cache TTL times can be added to any query with very minimal client-side code.

We created a [custom React hook](https://github.com/*redacted*/*redacted-app-name*/blob/master/src/utils/useRefetchOnFocus/index.ts) to facilitate adding TTL cache times to a query with a few lines of code. Queries are automatically refetched when the component is on the screen that currently has focus and the cached version is older than the maximum allowed age.

#### Examples

An example of our custom cache-managing hook in use can be found in the [`MarketScreen`](https://github.com/*redacted*/*redacted-app-name*/blob/master/src/screens/MarketCategory/index.tsx#L64) component.

### Other options considered

There are surprisingly few options available to manage this kind of caching with Apollo, which is what led us to devise our own solution. There is an npm package, [`apollo-invalidation-policies`](https://www.npmjs.com/package/apollo-invalidation-policies), which accomplishes a similar outcome with a very different strategy (it evicts items from the cache that have reached a certain age, forcing the data to need to be refetched), but it is still in beta and seems to be little-used.

## Decision

Use our custom `responseTimestamp` query and `useRefetchOnFocus` hook to manage TTL caching.
