# Caching, Deduplication, Invalidation, and Revalidation

A cache can make repeat visits faster, avoid duplicate requests, and keep data visible while it refreshes. But reusing data is only correct when the cache key, lifetime, and invalidation rules match the meaning of that data.

This lesson separates the main cache layers, explains freshness and revalidation, and shows how mutations affect cached queries. It builds on [lesson 073](073-client-side-data-fetching-and-use-api.md) and [lesson 075](075-loading-empty-success-and-error-states.md).

## Learning goals

After this lesson, you should be able to:

- distinguish a React render cache, a client query cache, and the browser's HTTP cache;
- design a query key that identifies the full result;
- reuse in-flight requests to deduplicate work;
- choose a freshness lifetime and understand stale data;
- invalidate affected entries after a mutation;
- revalidate data while preserving useful old results;
- avoid leaking private data across users or sessions;
- understand which React cache APIs are limited to Server Components.

## Caching happens at different layers

| Layer | What it reuses | Typical owner |
| --- | --- | --- |
| Component state or memoization | A value within a component's render lifecycle | A React component |
| Client-side query cache | Data and in-flight requests shared by components | A data library or application cache |
| Browser HTTP cache | HTTP responses according to request and response caching rules | Browser and server headers |
| CDN or shared HTTP cache | Responses shared between clients where allowed | Infrastructure and HTTP policy |

These layers have different keys, lifetimes, and invalidation rules. Clearing a JavaScript query cache does not necessarily clear the browser's HTTP cache. A network request made by a query cache may still be fulfilled from the browser's HTTP cache.

React's `cache` API is for Server Components, not a general client-side query cache. `useMemo` memoizes a render-time calculation; it is not a durable, shared data cache with expiry and invalidation. For client fetching, use the framework's data mechanism or a query library when possible. A hand-written cache is useful for learning the concepts, but must deliberately implement the behaviors the app requires.

## Choose a complete query key

A cache key must identify all inputs that can change the result. For a product search, that may include:

- resource name;
- normalized search query;
- filters and sort order;
- page or cursor;
- locale or currency;
- user or tenant scope when results are permission-dependent.

If the key omits a relevant input, the cache can return a valid result for the wrong request. If equivalent inputs produce different keys, the cache misses and repeats work unnecessarily.

Use a stable, deliberate representation:

~~~js
function productsKey({ query, category, page, locale }) {
  return JSON.stringify([
    "products",
    query.trim().toLowerCase(),
    category ?? null,
    page,
    locale,
  ]);
}
~~~

Do not put raw credentials or access tokens into visible cache keys. For private data, use a safe user or tenant identifier as part of the scope, and clear or partition the cache when identity changes.

## Deduplicate in-flight requests

Two components may ask for the same data at nearly the same time. A query cache can store the pending Promise so both consumers share one request:

~~~js
const inFlight = new Map();

function loadOnce(key, loader) {
  if (inFlight.has(key)) {
    return inFlight.get(key);
  }

  const promise = loader().finally(() => {
    inFlight.delete(key);
  });

  inFlight.set(key, promise);
  return promise;
}
~~~

This example only deduplicates concurrent work. Once the Promise settles, it forgets the result. A useful query cache also stores successful data and its timestamp, tracks subscribers, and defines what to do after data becomes stale or a request fails.

Do not create one unrelated cache per component if the goal is sharing. The cache owner should be shared at the correct scope, and its key must match the data being requested.

## Freshness, retention, and staleness

A cached result can be stored but considered stale. These are separate decisions:

- **Freshness lifetime:** how long data can be used without revalidation.
- **Retention lifetime:** how long an unused entry stays in memory before removal.
- **Invalidation:** an event that makes data need an update before normal expiry.
- **Revalidation:** asking the source for current data, often while keeping stale data visible.

For example, a dashboard count may tolerate a short freshness window. A permission change or a successful edit may require immediate invalidation. A financial balance may need a stricter freshness policy than a help article.

A stale entry is not automatically wrong; it is data that needs revalidation according to the chosen policy. The application must decide whether to hide it, display it with a refreshing indicator, or use it temporarily if revalidation fails.

## Revalidate while keeping old data

A stale-while-revalidate UI can show cached data immediately and start a fresh request in the background:

~~~text
cached data available → show it as stale/refreshing
                          └─ request current data
                               ├─ success → replace cache and mark fresh
                               └─ failure → keep or discard old data by policy
~~~

This can make navigation feel faster, but old data must not be presented as newly confirmed. Keep the query key with the result and show a refreshing or refresh-failed state where useful.

HTTP caching has related but distinct behavior. For example, server response directives can allow reuse while fresh or require validation before reuse. Validators such as an ETag can let a server confirm that a stored representation is unchanged. These rules belong to the HTTP cache layer; they do not automatically invalidate an in-memory React query cache.

## Invalidate after mutations

When a user changes server data, any cached query that depends on it may need updating. Common strategies include:

1. **Invalidate and refetch:** mark affected entries stale, then refetch active queries.
2. **Update the cache directly:** apply the server-confirmed result to known entries.
3. **Optimistic update:** update UI immediately, then confirm or roll back when the write completes.

For a beginner flow, invalidation and refetching are often easiest to reason about:

~~~js
async function handleCreateProduct(input) {
  await createProduct(input);
  invalidateQueries(["products"]);
}
~~~

The invalidation function should match relevant queries, not indiscriminately discard unrelated data. A product update may affect a detail query, a category list, search results, and counts. Choose a clear key hierarchy or tags so affected entries can be marked stale.

An invalidation is not itself a fetch. The cache may refetch immediately, when an active component next reads it, or on another configured event. Know which policy your data layer uses.

## Retry errors without caching them as success

A failed request should not become a successful empty result. Most simple caches remove a rejected in-flight Promise so a later retry can try again. Some libraries briefly cache errors or apply retry backoff; those are explicit policies.

Avoid permanently caching an error under a key unless the application deliberately wants negative caching. On recovery, invalidate or retry the failed entry. Keep the previous successful data only if the UI policy allows it and labels it accurately.

## Scope private data carefully

A client-side cache can outlive the component that requested it. Consider:

- whether two accounts on one device could see each other's cached values;
- whether tenant, role, or permission changes should change the key;
- whether logout should clear user-scoped entries;
- whether a sensitive response is allowed in the browser or shared HTTP cache;
- whether a service worker or CDN has a separate caching policy.

Do not use a public shared cache for personalized content unless the server's caching rules explicitly make that safe. A cache key is not an authorization check; the server must still enforce access.

## Cancellation and shared requests

A component that unmounts may no longer need a request, but another component may still be using the same cached Promise. The cache, not an individual subscriber, should decide whether shared work can be aborted.

A cache can count active subscribers, let the request finish for reuse, or cancel after a grace period. The correct policy depends on the library and application. See [lesson 074](074-request-cancellation-and-race-conditions.md) for request cancellation and stale responses.

## Common mistakes

- Treating `useMemo` as a complete query cache.
- Assuming React's `cache` API is a general client-side cache.
- Leaving part of the query out of the cache key.
- Including raw credentials in cache keys or logs.
- Caching one Promise per component instead of sharing the cache owner.
- Deduplicating requests but never retaining successful data.
- Keeping data forever without freshness, invalidation, or memory-retention rules.
- Confusing freshness lifetime with how long an unused entry remains in memory.
- Assuming invalidation automatically triggers a fetch.
- Keeping stale data visible without indicating that it may be outdated.
- Reusing a rejected Promise forever so retries can never succeed.
- Clearing a shared request when another component still needs it.
- Assuming browser HTTP cache and application query cache are the same store.
- Forgetting to clear or partition user-specific data when identity changes.

## Practical checklist

1. What exact inputs define this result, and are they all in its key?
2. Which component, library, or framework owns the cache?
3. Does the cache reuse an in-flight request for identical keys?
4. How long is data considered fresh?
5. How long are unused entries retained?
6. Which mutation or event invalidates this data?
7. Does invalidation refetch now or only on a later read?
8. Should old data remain visible during revalidation or after a refresh error?
9. How can a failed request be retried without reusing a rejected Promise?
10. Is the cache scoped correctly for accounts, tenants, and permissions?
11. Is the browser or server also caching the HTTP response?
12. Does the application framework already provide a safer query or route cache?

## Summary

A cache is correct only when its key and lifecycle match the data. Deduplicate identical in-flight requests, define freshness separately from memory retention, and invalidate data after changes that can affect it. Revalidation can refresh stale results while keeping them visible, but communicate their freshness. Keep private data scoped safely, and use the framework or query library when it already manages these rules.