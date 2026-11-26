# Client-Side Data Fetching, the `use` API, and Async Values - Q&A

## Choosing a data-loading approach

### 1. What are the two approaches introduced in this lesson?

A component can start a request in an Effect and store its status and result in state, or it can read a Promise during rendering with React's `use` API and let Suspense handle the pending state. A framework or data library may provide the Promise and own the request lifecycle.

### 2. Are Effects and Suspense two ways to write the same request?

No. An Effect runs after a render commits and updates state when the request finishes. Suspense responds when a component reads a pending Promise during render. A request started inside an Effect does not automatically activate Suspense.

### 3. When is an Effect a reasonable choice for fetching?

When a mounted client component needs to synchronize with an external service as its inputs change, an Effect can start the request and update explicit loading, success, and error state. Use cleanup so obsolete work cannot update the current view.

### 4. When should a request start in an event handler?

When the user's action causes the request. For example, a form submission or a **Save** button should start its write in the corresponding handler, rather than in an Effect watching a flag that the handler just changed.

### 5. Should every app use `use` for all data fetching?

No. `use` is useful for reading a Promise from a Suspense-compatible source. Framework loaders, server data APIs, or query libraries may provide better fetching, caching, revalidation, and error handling. Use the conventions already chosen by the application.

## Fetch and response handling

### 6. Does `fetch()` reject for a 404 or 500 response?

Usually no. It resolves to a `Response` for an HTTP error status. Check `response.ok` and throw or otherwise handle a non-success status.

~~~js
const response = await fetch("/api/items");

if (!response.ok) {
  throw new Error("Request failed: " + response.status);
}
~~~

### 7. Can `response.json()` fail?

Yes. It returns a Promise and can reject if the body cannot be read or is not valid JSON. Handle this in the same asynchronous error path as request failures.

### 8. What does a rejected fetch Promise usually mean?

It often means a network-level failure, request cancellation, or a similar failure to complete the request. It does not mean that every HTTP error status will reject automatically.

### 9. Why encode a search term used in a URL?

Encoding makes the term safe to place in a query-string value and preserves special characters correctly. Prefer `URL` and `URLSearchParams` for more complex URL construction.

### 10. Why must an Effect include `query` in its dependency array?

The request uses the current query. Including it tells React to clean up the old synchronization and start a new one when the query changes. Omitting a reactive dependency can leave the request using stale data.

## Effects and asynchronous state

### 11. Why does an Effect-based request often show a loading state first?

Effects run after React commits. The first render can therefore display loading UI, and the Effect starts the request after that commit.

### 12. Does the Suspense fallback appear for a request started in `useEffect`?

No. Suspense responds to work that suspends during rendering. An Effect-based request needs loading UI represented by component state or by its data library.

### 13. Why return cleanup from a request Effect?

The component may unmount or the query may change while a request is pending. Cleanup can abort the request or mark its result obsolete so stale work does not affect the current screen. Lesson 074 covers cancellation and races in detail.

### 14. Does aborting a request always prevent all related work from finishing?

No. Cancellation behavior depends on where the request is in its lifecycle and on the operation being performed. Code should still avoid applying results after the request is no longer relevant.

### 15. Why can Strict Mode make an Effect request appear to run twice in development?

Strict Mode performs an extra setup-and-cleanup cycle for Effects in development to reveal missing cleanup. A correct Effect tolerates setup, cleanup, and setup again. This is not a reason to disable Strict Mode or hide the behavior with a fragile global flag.

### 16. Should derived data be put in state with an Effect?

Usually not. If the data can be calculated from current props or state during render, derive it directly. An Effect adds another update and can briefly produce out-of-sync UI.

## React's `use` API and Suspense

### 17. What does `use(promise)` do?

It reads the Promise's value during rendering. A pending Promise suspends the component, a fulfilled Promise provides its value, and a rejected Promise is thrown for an Error Boundary to handle.

### 18. Which boundary handles a pending Promise?

The nearest parent `<Suspense>` boundary displays its fallback while the component is suspended.

### 19. Which boundary handles a rejected Promise read by `use`?

The nearest Error Boundary handles the thrown error. Suspense handles pending work; it is not an error message or error boundary.

### 20. Does React provide a built-in function component called `ErrorBoundary`?

No. Error Boundaries are declared with a class component or supplied by a framework or library. An application can reuse its established Error Boundary implementation.

### 21. Can `use` read anything besides a Promise?

Yes. It can also read a React context value. This API's conditional-call behavior is useful for context access that is conditional.

### 22. Can `use` be called inside a condition or loop?

Unlike most Hooks, `use` can be called conditionally or inside a loop, as long as it is still called while rendering a function component or custom Hook. Follow the Promise-specific rule: pass the Promise to `use` rather than trying to skip the read based on its status.

### 23. Can `use` be called in an event handler?

No. Call it while rendering a function component or custom Hook. An event handler can initiate work or update state, but it cannot call `use` to read a Promise.

### 24. Can `use` be wrapped in `try/catch` to handle a rejected Promise?

Do not do that. Let the rejection reach the nearest Error Boundary. A local `try/catch` around `use` interferes with React's suspension and error handling.

### 25. Can a Promise be passed from a Server Component to a Client Component?

Yes, in React's Server Component model a Promise can be passed to a Client Component and read with `use`. Framework and server-rendering constraints still apply; this does not mean arbitrary client-created Promises are automatically cached.

### 26. Can I write `use(fetch(url))` directly in a client component?

Avoid it. That expression creates a new Promise during each render. When React retries a suspended render, another Promise can be created and the component may suspend repeatedly.

### 27. Why does Promise identity matter?

React needs to see the same pending resource when a suspended component retries. A stable Promise lets React track the resource's pending, fulfilled, or rejected state. Recreating it on each render makes it a different resource.

### 28. Where should the Promise be created?

In a framework loader, a Suspense-aware data source, a stable cache, or another owner that can supply the same Promise to repeated renders. If an event starts a Promise, the parent can store and pass that Promise as state; do not recreate it from the child's render.

### 29. Does a module-level Promise map count as a complete production cache?

No. It can demonstrate Promise reuse, but a plain map does not automatically provide expiration, invalidation, memory limits, retries, deduplication policy, or user/session isolation. Lesson 076 covers caching and revalidation.

### 30. Is a cached Promise the same as the browser's HTTP cache?

No. A JavaScript Promise cache reuses a particular in-memory computation or request result. The browser's HTTP cache follows HTTP caching rules. They are different layers with different lifetimes and invalidation behavior.

## Boundaries and UI states

### 31. Is Suspense enough to design every loading state?

Not necessarily. Suspense provides a fallback while a child suspends. Applications still need deliberate UI for empty results, errors, refreshes, and other states. The next loading-state lesson covers those cases.

### 32. Where should a Suspense boundary go?

Place it around content that can suspend, at a level where the fallback is useful and understandable. A boundary too high can replace a large area of the interface; a boundary too low may not provide a meaningful loading experience.

### 33. Can a single Error Boundary and Suspense boundary be used together?

Yes. A common arrangement is an Error Boundary outside Suspense: Suspense handles pending content inside, and the Error Boundary handles a rejection from that subtree.

### 34. If the Error Boundary shows an error, will changing the query clear it automatically?

Not always. Whether the boundary resets depends on its implementation and how the tree is keyed or reset. A query change may need to remount/reset the boundary or use the framework's reset mechanism.

### 35. What happens if the response contains an empty array?

The Promise fulfills normally. Render an intentional empty state after reading the value; an empty result is not a loading state or a request error.

## Architecture and common mistakes

### 36. Why not start a normal request during render?

React may render, retry, pause, or discard work. A direct request in render can be duplicated and has no deliberate cancellation or cache policy. Render-time Promise reads should be backed by a Suspense-compatible resource that owns these concerns.

### 37. Is every Promise safe to pass to `use`?

No. The Promise must be created and managed by an appropriate source. In a client component, ensure the source reuses the Promise across renders for the same resource. A fresh Promise created during every render is not safe.

### 38. What are the three common Promise outcomes for `use`?

- Pending: the component suspends and the nearest Suspense fallback appears.
- Fulfilled: `use` returns the value.
- Rejected: the error is delivered to the nearest Error Boundary.

### 39. What is the difference between a loading error and an empty result?

An error means the operation failed and should be explained or made recoverable. An empty result means the operation succeeded but found no matching data. Keep those states distinct in the UI.

### 40. Which approach should a beginner choose for a small client-only demo?

An Effect with explicit state can be easiest to understand and debug. If the goal is specifically to learn Suspense, use a stable Promise source and boundaries. For a real routed application, first check the framework's recommended data-loading approach.

### 41. What should I inspect when the Suspense fallback never disappears?

Check that the Promise eventually settles, that the same Promise is reused rather than recreated on each render, and that the component is beneath the intended Suspense boundary. Also verify the data source is not returning a new Promise on every read.

### 42. What should I inspect when an error appears instead of the Suspense fallback?

A rejected Promise goes to an Error Boundary. Inspect the request, status check, and response parsing, then show a useful error state. Suspense only represents pending work.

### 43. What is the safest general rule for data fetching in React?

Let one deliberate system own each request lifecycle. Check HTTP status and parsing, represent loading, empty, success, and error states, clean up obsolete Effect work, and use Suspense only with a Promise source designed for it.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="072-react-dom-resource-hints-and-browser-apis.md">072 - React DOM Resource Hints and Browser APIs - Q&amp;A</a></td><td><a href="../lessons/073-client-side-data-fetching-and-use-api.md">Lesson</a></td><td><a href="../examples/073-client-side-data-fetching-and-use-api.md">Example</a></td><td><a href="074-request-cancellation-and-race-conditions.md">074 - Request Cancellation, Stale Responses, and Race Conditions - Q&amp;A</a></td></tr>
  </tbody>
</table>
