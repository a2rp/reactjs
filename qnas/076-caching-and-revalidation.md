# Caching, Deduplication, Invalidation, and Revalidation — Q&A

## Cache layers and React

### 1. What does a client-side query cache store?

It can store successful data, timestamps, pending Promises, errors, and subscription or refetch information for query keys. Exact behavior depends on the cache library or implementation.

### 2. Is a query cache the same as component state?

No. Component state belongs to a component instance. A shared query cache can serve matching data to several components and can outlive the component that first requested it.

### 3. Does React provide a general client-side query cache?

No. React provides rendering and state APIs, but a general client-side query cache with keys, freshness, invalidation, and revalidation is usually provided by a framework or data library.

### 4. Is React's `cache` API a general cache for browser components?

No. React documents `cache` for Server Components. Do not use it as a client-side query cache.

### 5. Can `useMemo` replace a query cache?

No. `useMemo` memoizes a calculation in a component render lifecycle. It does not by itself provide shared request deduplication, time-based freshness, invalidation, persistence, or a cache lifecycle.

### 6. What is the difference between an application query cache and the browser HTTP cache?

A query cache stores JavaScript data under application-defined keys and policies. The HTTP cache stores HTTP responses according to request and response cache rules. A query cache can issue a request that the browser then satisfies from its own HTTP cache.

### 7. Does clearing a query cache clear the browser cache?

No. They are separate stores. Clearing or invalidating an application entry does not necessarily remove a browser-cached response, service-worker entry, or CDN response.

## Keys and scope

### 8. What is a query key?

A stable identifier for one result. It must include every input that can change that result, such as resource type, query, filters, sort order, page, locale, and relevant account scope.

### 9. What happens if the key omits a filter?

The cache may return data for a different filter because both requests map to the same key. The key must represent the full semantic request.

### 10. What happens if equivalent inputs create different keys?

The cache may miss and make duplicate requests. Normalize equivalent values when that normalization preserves the API's meaning.

### 11. Should raw access tokens be part of a cache key?

No. Do not expose credentials in keys, logs, or debugging output. Use a safe account or tenant scope when needed, and always enforce authorization on the server.

### 12. How should private data be scoped?

Partition it by the identity or tenant that affects access. Clear or otherwise isolate user-scoped entries on logout, account switch, or permission changes. A cache key is not an authorization mechanism.

## Deduplication and cache lifetime

### 13. What is request deduplication?

Sharing one in-flight request among consumers that ask for the same key at nearly the same time, instead of starting duplicate work.

### 14. How can two components share one pending request?

They must read through the same cache owner and use the same key. A cache can store the in-flight Promise and return it to later consumers.

### 15. Is deduplicating pending requests enough to be a complete cache?

No. Once the Promise settles, deduplication alone forgets the result. A query cache also needs an explicit policy for successful data, freshness, invalidation, errors, and memory retention.

### 16. What is the freshness lifetime?

How long cached data is treated as current enough to use without revalidation. It is often called a stale time in data libraries.

### 17. What is cache retention or garbage-collection time?

How long an unused entry remains stored before it can be removed from memory. This is separate from how long the data is considered fresh.

### 18. Can cached data be present but stale?

Yes. The cache can retain an old value while marking it as needing revalidation. Stale does not mean unusable in every interface; it means the configured freshness policy says to update it.

### 19. Does an entry refetch exactly when its freshness time expires?

Not necessarily. Expiry often makes an entry stale; a request may start on the next read, mount, focus event, reconnect, or explicit refresh according to the configured library policy.

### 20. Should a rejected Promise remain cached forever?

Usually no. A basic cache should clear or replace a rejected in-flight Promise so retry can make a new attempt. Some libraries apply a deliberate short error cache or backoff.

## Revalidation and HTTP

### 21. What is revalidation?

Asking the source whether cached data is still current or retrieving a newer version. It can replace stale data while the UI remains visible.

### 22. What is stale-while-revalidate in a UI?

Show cached data immediately, mark it as refreshing or stale, and request current data in the background. When the new response arrives, replace the old data.

### 23. Should stale data always be shown during revalidation?

No. It depends on whether the old value remains safe and useful. Search results for a changed query should not masquerade as current; a dashboard may keep old values visible with a refreshing indicator.

### 24. What is the HTTP `Cache-Control: no-cache` directive?

It permits storage but requires validation with the origin before reuse. It does not simply mean “do not store.” Use `no-store` when the response should not be stored under the relevant HTTP cache rules.

### 25. How is HTTP revalidation different from query-cache revalidation?

HTTP revalidation follows browser/server response directives and validators. Query-cache revalidation follows application keys and library policies. Both may occur for the same logical data request.

### 26. What are ETags used for?

An ETag can act as a server-provided validator. A cache may make a conditional request with that validator; if the representation is unchanged, the server can respond that the stored response remains valid without sending the full body again.

## Mutations and invalidation

### 27. What does invalidating a query mean?

Marking matching cached data as no longer fresh according to the application's policy. Invalidation may cause an immediate refetch or wait until the next read; it is not itself necessarily a network request.

### 28. When should data be invalidated?

After a mutation or event that may change the cached result, such as creating a product, updating a profile, deleting a record, or changing permissions.

### 29. Should every mutation clear the entire cache?

Usually not. Invalidate the affected keys or tags. A mutation may affect several related queries, but unrelated cached data should not be discarded without a reason.

### 30. What are the common strategies after a mutation?

Invalidate and refetch, directly update known cache entries with the confirmed result, or optimistically update the UI and reconcile when the server responds. Pick a strategy the data layer supports and the interface can explain.

### 31. Why might a mutation affect more than one query key?

A changed record can appear in a detail view, search results, category listings, counts, and related summaries. Identify all dependent views or use a deliberate tag/key hierarchy.

### 32. Should a request to create data start in an Effect?

No. A user-triggered write belongs in the submit or click event handler that caused it. The cache can be invalidated after that write succeeds.

### 33. What if an old request is still in flight when data is invalidated?

A cache must avoid letting the obsolete request overwrite newer data. It can version entries, cancel supported work, or ignore results from an earlier generation. Established query libraries handle this class of coordination.

## Errors, cancellation, and common pitfalls

### 34. What should happen to old successful data if revalidation fails?

That is a UI policy choice. The app may keep it with a visible refresh error if it remains safe, or clear it if stale information could mislead or cause harm.

### 35. Should the cache store a server error response as successful empty data?

No. Keep failure distinct from a successful empty result. Check HTTP status and response parsing, then use the error policy appropriate for the query.

### 36. Who should cancel a shared request when one component unmounts?

The shared cache should own that decision. Another component may still be subscribed and need the result. A single consumer should not abort shared work without checking ownership.

### 37. Can a service worker or CDN add another cache layer?

Yes. Those caches have separate scope and rules. Personalized data must not be shared unless the server and infrastructure policy make it safe.

### 38. What is a common memory bug in a hand-written cache?

Keeping an entry for every search string forever. Define retention or size limits, especially for free-form queries with many possible values.

### 39. How can I tell whether duplicate requests are caused by a cache miss?

Inspect the exact keys, normalized input values, cache owner scope, and whether the requests were simultaneous or sequential. A cache that only deduplicates in-flight requests may legitimately fetch again after the first Promise settles.

### 40. What should I use in a production React app?

Prefer the data-loading and caching conventions of the framework or query library already chosen for the app. If building a cache yourself, specify key semantics, deduplication, freshness, retention, invalidation, error retries, cancellation ownership, and account scoping.

### 41. Why does the example store a Promise as well as data?

The Promise lets a second consumer share work while the request is pending. The data and timestamp let later consumers reuse a successful response while it is fresh.

### 42. What is the core cache-design rule?

A cached value is safe to reuse only when the key identifies the exact data, the freshness policy is appropriate, and every relevant change can invalidate or revalidate it.