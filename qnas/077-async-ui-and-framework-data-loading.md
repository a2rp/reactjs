# Async UI, Suspense, and Framework Data Loading — Q&A

## Suspense fundamentals

### 1. What problem does Suspense solve?

It lets a component tree declare fallback UI for a subtree that cannot finish rendering because a supported dependency is pending.

### 2. Is Suspense a data-fetching library?

No. It coordinates rendering. A framework or library must provide the data-loading integration and resource reuse.

### 3. What is a Suspense boundary?

A Suspense component with a fallback. When a descendant suspends, React shows the fallback instead of that boundary's children.

### 4. Does Suspense start a request?

Not by itself. The framework, library, loader, or code-splitting mechanism owns and starts the work.

### 5. What does the fallback replace?

It replaces the children of the nearest boundary that catches the suspended render. Content outside that boundary can remain visible.

### 6. What can activate Suspense?

Documented cases include lazy-loaded code, reading a Promise with React's use API, and integrations supplied by a Suspense-enabled framework or library.

### 7. Does fetch in useEffect activate Suspense?

No. The Effect runs after rendering and its pending request does not automatically activate a boundary.

### 8. Does an async click handler activate Suspense?

No. Track the interaction's pending and result states explicitly, or use the data API your framework recommends.

### 9. Does any pending Promise activate Suspense?

No. The resource must be read through an integration that React recognizes. Merely creating or storing a Promise is not enough.

### 10. Is a Suspense fallback the same as an isLoading flag?

No. An isLoading flag is application state. Suspense chooses a fallback when a descendant suspends beneath a boundary. A library can offer both models.

## Promises and use

### 11. What does use(promise) do while the Promise is pending?

The component suspends. React shows the nearest Suspense fallback and retries rendering after the Promise settles.

### 12. What happens when the Promise resolves?

The retried render receives the resolved value from use.

### 13. What happens when it rejects?

The rejection is thrown during rendering and can be handled by the nearest Error Boundary.

### 14. Why must the Promise stay stable across renders?

React may retry a suspended render. Recreating the Promise can start duplicate work and prevents React from reusing the original pending resource.

### 15. Why is use(fetch(url)) inside a component usually wrong?

It creates a new fetch Promise during render. Use a framework loader, Suspense-compatible library, or stable resource cache instead.

### 16. Can use be called conditionally?

Yes. The use API can be called in conditions and loops, unlike most Hooks. It still must be called from a component or custom Hook.

### 17. Can try/catch handle a Promise read with use?

No. Suspense uses internal thrown signals during rendering. Handle rejected resources with an Error Boundary.

### 18. Should I build a Promise cache myself?

Usually not for production. A real cache needs keying, freshness, invalidation, retries, cancellation, user-data isolation, and memory limits. Prefer the framework or library already used by the app.

## Boundary design and updates

### 19. Where should a boundary go?

Around a meaningful UI region whose loading behavior can be described as one unit. Keep useful page structure outside it if that structure should appear sooner.

### 20. Why use nested boundaries?

They let independent regions reveal separately. A slow chart can keep its fallback while the heading and a ready activity panel are visible.

### 21. Can one boundary wrap the whole app?

It can, but then unrelated content may share a single fallback. Use a global boundary only when that shared experience is intentional.

### 22. What makes a good fallback?

It says what is loading, keeps the layout steady when practical, avoids unnecessary motion, and uses appropriate status semantics for assistive technology.

### 23. What if visible content suspends during an update?

React may show the fallback again. A transition or useDeferredValue can allow already revealed content to stay visible while React prepares the next view.

### 24. Does keeping old content visible mean the data is current?

No. Communicate stale or refreshing status when freshness matters. Transitions affect presentation, not data correctness.

### 25. What is a request waterfall?

A sequence where a later request starts only after an earlier request completes, even though both could have started in parallel.

### 26. How can frameworks reduce waterfalls?

Route loaders or framework data APIs can start independent requests together and coordinate the results with navigation, caching, and rendering.

## Frameworks, rendering, and errors

### 27. What can a framework add to React data loading?

Depending on the framework: route loaders, server access, request caching and deduplication, parallel loading, preloading, server-rendered HTML, streaming, and coordinated error handling.

### 28. Are framework data APIs interchangeable?

No. APIs, caching rules, and server/client boundaries vary. Follow the selected framework's version-specific conventions.

### 29. Does Suspense make a client-only Vite app stream HTML from a server?

No. Streaming needs server-rendering infrastructure that supports it. Client-only apps can still use Suspense for supported resources and lazy code.

### 30. Is Suspense a cache?

No. It chooses fallback UI for pending rendering. The framework or data layer owns resource reuse, expiry, and invalidation.

### 31. What does an Error Boundary catch?

Errors thrown while rendering descendants, including rejected Promises read with use. It does not replace error handling in every event handler or asynchronous callback.

### 32. Can a parent function's try/catch catch a child's rendering error?

No. Use an Error Boundary component around the region that may fail.

### 33. Why can a retry keep failing?

The UI may be reading the same rejected Promise again. Clear or replace the failed resource before retrying.

### 34. What should an error recovery action do?

Retry with a new request, return to a safe route, preserve useful user input, or offer a reload when the app cannot recover locally.

### 35. Does the Suspense fallback handle a network error?

No. It describes pending work. A rejection belongs to an Error Boundary or the explicit error state owned by the data library.

## Choosing the right pattern

### 36. When is explicit loading state preferable?

When the source does not support Suspense, a request is tied to a form or interaction, or the view needs explicit pending, success, stale, and error states.

### 37. Can explicit state and Suspense coexist?

Yes, when each has a clear responsibility. Avoid having two systems independently issue and cache the same request.

### 38. Should every component fetch its own data?

Not necessarily. Deep component fetching can introduce waterfalls and duplicate requests. Consider route-level loading or a shared query cache.

### 39. How should accessibility affect loading UI?

Announce meaningful progress without repeatedly announcing trivial changes, preserve focus, and show a clear message when loading fails.

### 40. What should I confirm before using Suspense for data?

Confirm the source supports it, Promise identity is stable, boundaries match the user experience, and errors and retries have a clear owner.
