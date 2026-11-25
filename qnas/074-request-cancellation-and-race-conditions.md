# Request Cancellation, Stale Responses, and Race Conditions — Q&A

## Understanding response races

### 1. What is a race condition in client-side fetching?

It is a bug caused by asynchronous operations finishing in an unexpected order. If an old request finishes after a newer request and both update the same state, the old result can replace the result for the user's current input.

### 2. Why can responses arrive out of order?

Network latency, server workload, response size, caching, and body parsing can vary per request. Starting request A before request B does not guarantee A will finish first.

### 3. What does “latest request wins” mean?

Only the most recently started relevant request is allowed to publish state. It is a useful policy for searches and selected-record details, but not necessarily for pagination where multiple requests may all contribute data.

### 4. Is an old response always harmful?

No. Some interfaces intentionally keep old content visible during a refresh. The UI should make clear that the content is previous data, or otherwise associate it with the key that produced it. Showing old results as if they match a new query is misleading.

### 5. What kinds of UI trigger stale-response problems?

Search boxes, autocomplete, route changes, filter controls, selected users, tabs that load data, and any view where a changed input makes earlier asynchronous work irrelevant.

## Effect cleanup and ignore guards

### 6. How does an ignore flag help?

Each Effect setup owns a local boolean. Cleanup sets it to true. When that request later completes, it checks the flag and avoids changing state if its setup is obsolete.

### 7. Why should the ignore flag be local to one Effect setup?

Each request needs its own relevance marker. A shared boolean can be reset by a newer Effect setup and accidentally allow an old request to update state.

### 8. Does an ignore flag cancel the fetch?

No. It prevents an obsolete result from affecting the UI. The browser and server may keep processing the request. Use an `AbortSignal` when the operation supports cancellation and stopping client-side work is useful.

### 9. Should stale errors also be ignored?

Yes. An obsolete request should not replace the latest query's success, loading, or error state. Guard every state update that belongs to that request.

### 10. When does React run an Effect's cleanup?

Before React runs that Effect again because its dependencies changed, and when the component unmounts. Development Strict Mode can add a setup-and-cleanup probe to expose missing cleanup.

### 11. Can I set React state in cleanup to mark the request canceled?

Usually there is no need. Cleanup should release or stop the work it owns. The next Effect can set the new loading state, and the rendered result can be keyed to the current query.

### 12. What if the result is for the previous query while the new Effect has not started yet?

Store the query or resource key with the result. Render it only when its key matches the current input, or intentionally display it as stale/previous content. This prevents a brief render from presenting the old result as the new one.

## AbortController and AbortSignal

### 13. How do I make a Fetch request abortable?

Create an `AbortController` and pass its `signal` to `fetch`:

~~~js
const controller = new AbortController();

const response = await fetch("/api/items", {
  signal: controller.signal,
});

controller.abort();
~~~

In a component Effect, return cleanup that calls `controller.abort()`.

### 14. Should I reuse a controller for the next query?

No. Once aborted, its signal remains aborted. Create a new controller for each independent request or Effect setup.

### 15. What does Fetch do when its signal is aborted?

The fetch Promise rejects, usually with an `AbortError` DOMException when no custom abort reason was supplied. Aborting can also stop consumption of a response body that has not yet been read.

### 16. Is checking `error.name === "AbortError"` enough?

It is a common check for Fetch's default abort error. Checking the specific controller's `signal.aborted` is often more robust, especially when custom abort reasons are used.

### 17. Should an abort show an error message to the user?

Usually not when the request was deliberately canceled because its query became obsolete or the view unmounted. Treat that cancellation as expected control flow. Do show genuine network, HTTP, parsing, and application failures.

### 18. Should every caught error be ignored if the signal might have been aborted?

No. Ignore only the cancellation or obsolete result. Do not hide unrelated failures. Also check `response.ok` because HTTP error statuses do not automatically reject `fetch`.

### 19. Does aborting after receiving response headers still matter?

It can. The response body may not have been read yet. Aborting the signal can reject body consumption, such as `response.json()`, with an abort error.

### 20. Does `AbortController` work with every asynchronous API?

No. The API must accept or otherwise observe the signal. If it cannot be canceled, use a relevance guard so the old result cannot update state.

### 21. Can one controller abort several operations?

An `AbortSignal` can be shared by operations designed to observe it, so aborting the controller signals all of them. In components, do this only when those operations share one lifecycle and should all stop together.

## Cancellation versus state correctness

### 22. Why might code use both abort and an ignore flag?

Abort asks supported work to stop and can save client resources. The ignore flag protects state correctness if completion races with cleanup, if a later processing step does not observe the signal, or if the operation cannot be canceled.

### 23. Is abort alone enough to ensure stale state is impossible?

For a particular API, an abort signal can reject its pending work. A relevance check is still useful when later work, custom APIs, or shared request ownership complicates the lifecycle. Understand the data layer's guarantees rather than assuming every operation reacts identically.

### 24. What is a request ID guard?

A monotonically increasing identifier is assigned when each request starts. A completion updates state only if its identifier still equals the latest identifier. It is useful for overlapping requests initiated from different handlers or flows.

### 25. Is a request ID always better than an ignore flag?

No. For one Effect instance at a time, an Effect-local ignore flag is simple and scoped correctly. Use IDs when they make a more complex overlapping-request policy easier to express.

### 26. What is the difference between “latest request wins” and “latest response wins”?

“Latest request wins” preserves the user's newest intent. “Latest response wins” means the request that happened to finish last determines the state, which can be an older intent. UI search normally wants the first policy.

### 27. Should an older request be canceled if both results are useful?

Not necessarily. Paginated data, parallel dashboard panels, and independent prefetches may all need their results. Cancel only work that is truly no longer needed, and scope the guard to the state it protects.

## Server and side-effect boundaries

### 28. Does aborting guarantee the server never received the request?

No. The request may already have reached the server before the client aborts. Aborting is a signal to stop supported client-side work; it is not proof that the server did nothing.

### 29. Can abort undo a purchase or save operation?

No. Client-side abort is not a transaction rollback. Start a user action from its event handler and give server-side writes appropriate correctness guarantees. Do not rely on an Effect cleanup to reverse a write.

### 30. Should one component abort a request shared with another component?

Not blindly. Another subscriber may still need it. A shared cache or data library should coordinate ownership and decide when the underlying request is no longer needed.

### 31. Does cancellation debounce typing?

No. If the app sends a request on every keystroke, cancellation can stop or ignore old requests after they start. Debouncing waits before starting a request. A search may use both.

### 32. If I use `Promise.race` with a timer, is the fetch canceled when the timer wins?

No. `Promise.race` only settles with the first Promise that settles; it does not cancel the losing operation. Use an abort signal for cancellable work, and clear any timer in cleanup.

## Strict Mode and debugging

### 33. Why do I see a request and then an aborted request in development?

Strict Mode can run an extra Effect setup-and-cleanup cycle in development. The first setup starts work, its cleanup aborts it, and the next setup starts current work. Correct cleanup should make this safe.

### 34. Should I use a ref to force an Effect to run only once in Strict Mode?

No. That hides the setup/cleanup problem rather than fixing it. Make the Effect safe to start, clean up, and start again.

### 35. How can I reproduce an out-of-order response bug?

Use a test endpoint or local demo that gives different query values different response delays. For example, make a short query slow and a longer query fast, then type quickly. Without a relevance guard, the short-query result may arrive last.

### 36. What should I inspect in browser developer tools?

Inspect request start times, response completion order, aborted requests, status codes, and response bodies. A request shown as aborted in the browser does not prove the server stopped processing it.

### 37. Does seeing an aborted request mean the implementation is broken?

No. It can mean cleanup successfully stopped work the view no longer needs. Ensure current requests still resolve and that aborts are not presented as real errors.

### 38. Can a query cache solve all race conditions automatically?

No. A data library can coordinate request keys, deduplication, caching, and cancellation, but the app still needs correct query keys and intentional policies for invalidation, stale data, and parallel requests.

### 39. How does this lesson relate to the previous lesson about Suspense and `use`?

The previous lesson explained reading a stable Promise with `use` and Suspense. This lesson focuses on request lifetimes and stale results, especially Effect-based fetching. A Suspense-compatible data source must own cancellation and sharing of its Promises; a component should not independently abort a Promise another consumer still needs.

### 40. What is the core rule for race-safe UI?

Tie each result to the request key that produced it. When that key stops being current, cancel supported work and prevent its completion from publishing stale success or error state. Keep genuine failures visible and treat cancellation as expected lifecycle behavior.