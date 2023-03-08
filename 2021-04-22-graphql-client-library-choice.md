# GraphQL Client Library Choice

Date: 04-22-2021

## Author

Mike Fisher (@mikefisher84)

## Status

Closed

## Context

### The state of affairs

We currently use [`@shopify/react-graphql`](https://www.npmjs.com/package/@shopify/react-graphql), in tandem with Shopify's GraphQL code generation tool, [`graphql-type-definitions`](https://github.com/Shopify/react-graphql-web/tree/master/packages/graphql-typescript-definitions). `@shopify/react-graphql` combines many of the Apollo client stack packages and adds some [modifications and additional tooling](https://www.npmjs.com/package/@shopify/react-graphql).

### Decision drivers

The rationale for why `@shopify/react-graphql` was introduced to the app is well documented in [this commit message](https://github.com/*redacted*/*redacted-app-name*/commit/55a83b7e361118e2037f451872a0c27eed743175).

Our version of `@shopify/react-graphql` (v6.0.8) is tied to `apollo-client` v2, and does not include some of the newer hooks provided in [`@apollo/react-hooks`](https://www.apollographql.com/docs/react/api/react-hooks/), such as `useLazyQuery`, meaning we have to import them directly from `@apollo/react-hooks` to use them. `apollo-client` v3, while officially still in beta, is very stable, and offers [numerous bugfixes](https://github.com/apollographql/apollo-client/blob/master/CHANGELOG.md) and improvements](https://github.com/apollographql/apollo-client/blob/master/CHANGELOG.md).

`graphql-type-definitions` does automatically generate types for GraphQL query and mutation results, and it creates namespaced subtypes for nested query results. However, the type names can be very verbose and imperative rather than clearly expressing the intent of that data in the requery result and how it will be used. Some examples:

```
ProductsInSectionsQueryData.SectionsEdges // This is a namespaced type for a nested CategoryEdge type in the query result

SectionsEdgesNodeMarketProductsEdgesNodePricingSubscriptionDiscounts // Yikes
```

`@shopify/react-graphql`, while reasonably popular, is a niche product compared to the Apollo stack it is based upon. Using Shopify's opinionated approach to GraphQL hooks ties us to Shopify's point of view and their continued support and maintenance of the library. There's also far less documentation, community blog posts, Github issues, etc. about usage of Shopify's tools to aid in usage and determining bug fixes, etc. when compared to the vastly popular vanilla Apollo stack.

One major benefit of Shopify's stack is the `@shopify/graphql-testing` library. However, this library isn't coupled to the rest of Shopify's tools and can continue to be used with Apollo's tools too.

[`@graphql-codegen`](https://github.com/dotansimha/graphql-code-generator) is a GraphQL community-supported option for automatic code generation. It offers feature parity for the most part with `graphql-type-definitions`, albeit with some different approaches:

- It does not provide subtypes for nested query responses.
- Instead, the recommended approach is to use GraphQL fragments in the query operation, which then do receive their own types.
- It creates named hooks per query/mutation, with automatically typed results. ex: `useProductsInSectionsQuery()`.

This is pretty different from Shopify's approach, and which approach is better is debatable. However, some potential advantages to this approach
are:
More human-friendly type names. Fragments can be named in such a way to create type names that are more declarative about their actual use:

```
ProductsInSectionsQueryData.SectionsEdges
could become
CategorySectionsFragment

SectionsEdgesNodeMarketProductsEdgesNodePricingSubscriptionDiscounts

could become
SubscriptionDiscountsFragment
```

Fragments can also make large query operations easier to understand and make their intent more clear.

However, a big drawback of this approach with our query-per-component approach, is that fragments must be uniquely named, and there's no way to enforce the naming of fragments apart from establishing some name conventions to prevent name collisions.

Examples of the fragment approach in action are demonstrated in [this PR](https://github.com/*redacted*/*redacted-app-name*/pull/150).

## Considered options

- Remove `@shopify/react-graphql`, use `@apollo/client` v3 and `@apollo/react-hooks` exclusively, in tandem with `@graphql-codegen`.
- Remove `@shopify/react-graphql`, use `apollo-client` v2 and `@apollo/react-hooks` exclusively, in tandem with `@graphql-codegen`.
- Use `@shopify/react-graphql` exclusively, in tandem with `graphql-type-definitions`, upgrade `@shopify/react-graphql` to latest version (6.1.8).
- Use `@shopify/react-graphql`, in tandem with `graphql-type-definitions`, use `@apollo/react-hooks` to fill in missing hooks from `@shopify/react-graphql` (the status quo).

## Proposed Decision

Remove `@shopify/react-graphql`, use `@apollo/client` v3 and `@apollo/react-hooks` exclusively, in tandem with `@graphql-codegen`. Keep using `@shopify/graphql-testing`.

## Reasoning

- `@apollo/client` covers our use cases without the need for the additional tooling `@shopify/react-graphql` adds on top.
- We get the added improvements of Apollo Client v3.
- `@apollo/client` is a more popular library, resulting in better documentation, more frequent updates, more bug resolution guidance through GitHub issues, etc.
- One of the primary benefits of `@shopify/react-graphql` was tooling providing increased type safety. However, `@apollo/client` combined with `@graphql-codegen` provides most of the same benefits while sticking to a more community-supported, conventional approach. Codegen produces easier-to-read type names, and nice custom React hooks (hooks per query definition, with type-checking, ex: `useProductsQuery()` instead of `useQuery(query)` ).
- `@graphql-codegen` is also the [tool being used in _redacted-app-name_](https://github.com/*redacted*/*redacted-app-name*/blob/master/codegen.yml), so using it for the mobile app too would add to tooling consistency across projects.

## Consequences

### Implementation changes

- Remove `@shopify/react-graphql` and replace usage with equivalent `@apollo/client`/ `@apollo/react-hooks` implementations.
  Replace `graphql-type-definitions` with the more conventional [`graphql-code-generator`](https://github.com/dotansimha/graphql-code-generator) for automatic Typescript type generation.

### Estimated complexity

Moderately high -- It involves refactoring quite a bit of the GraphQL operations and typings in the client code. However, the immediate cost is somewhat reduced by the fact that much of the affected code is being refactored anyway as part of the [products queries refactor](https://github.com/*redacted*/*redacted-app-name*/pull/150), which is why this is coming up at this particular time.

## Alternatives

Either:

- Use `@shopify/react-graphql` exclusively, in tandem with `graphql-type-definitions`, upgrade `@shopify/react-graphql` to latest version (6.1.8). This choice may solve the current issue of needing to import some hooks from `@apollo/react-hooks`.
  or
- Use `@shopify/react-graphql`, in tandem with `graphql-type-definitions`, use `@apollo/react-hooks` to fill in missing hooks from `@shopify/react-graphql` (the status quo).

Either of these options would require less of a refactor. The question is really whether the benefits of Apollo Client v3, named custom hooks, and the fragments/type names pattern is enough benefit to justify a significant refactor and change to the current GraphQL usage patterns in the app.

## Conclusion

Upon further investigation, we decided that our current implementation is deeply coupled with Shopify's tools, particularly the testing tools, in a way that makes this change significantly larger in scope than initially assumed.

In addition to replacing `@shopify/react-graphql` and `graphql-type-definitions`, we would need to rewrite a significant amount of our tests to remove the Shopify test tools. Also, the process of re-generating all of our types and refactoring the GraphQL operations to use fragments is significant in scope too. In light of those findings, we have decided to close this ADR, and re-open a new ADR that more clearly outlines the broad scope of changes proposed.
