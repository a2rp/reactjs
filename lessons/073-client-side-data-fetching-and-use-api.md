# Client-Side Data Fetching, the `use` API, and Async Values

A component often needs data that arrives after the first render. React applications can load it with an Effect and ordinary state, or read a cached Promise with React's `use` API inside a Suspense boundary. These are related approaches, but they have different mechanics and should not be mixed up.

This lesson introduces both patterns. The next lessons cover request races, complete loading and error states, and production caching in more depth.

## Learning goals

After this lesson, you should be able to:

- choose between an Effect-based request and a Suspense-enabled Promise;
- check HTTP responses and handle asynchronous failures;
- explain what happens when `use` reads a pending, fulfilled, or rejected Promise;
- place Suspense and Error Boundaries around the components that need them;
- keep Promise identity stable between client renders;
- recognize when a framework or data library should own fetching and caching.

## Choose a data-loading model

| Approach | What the component does | Where pending UI comes from | Good fit |
| --- | --- | --- | --- |
| Effect and state | Starts a request after commit and stores its result in state | Component state such as `loading` | Direct client-side synchronization with an external service |
| `use(promise)` with Suspense | Reads a Promise supplied by a stable cache or framework | The nearest Suspense boundary | A Suspense-compatible data source |
| Framework or data library | Uses its route, loader, query, or server-data conventions | The framework or library | Most production applications with routing, caching, or server rendering |

The Suspense pattern is not just a shorter spelling of an Effect. A Promise started inside an Effect is not automatically observed by Suspense. The data source must provide a Promise that React can read during rendering.

## Start with the browser's `fetch` API

`fetch()` returns a Promise for a `Response`. An HTTP error status such as 404 or 500 does not by itself reject that Promise. Check `response.ok` before reading the body:

~~~js
async function readJson(url, options) {
  const response = await fetch(url, options);

  if (!response.ok) {
    throw new Error("Request failed with status " + response.status);
  }

  return response.json();
}
~~~

`response.json()` is also asynchronous. It can reject if the body is not valid JSON or cannot be read. Treat both the request and body parsing as work that can fail.

A request should be tied to a clear event or lifecycle:

- If a user clicks **Save**, start the write in that event handler. The click caused the work.
- If a screen should load data when it appears or when a search term changes, an Effect can synchronize the component with that external system.
- If the component reads a Promise during render, that Promise must come from a Suspense-compatible source.

Avoid starting ordinary network requests during render. React may render more than once, pause, or retry work. Render should describe UI; the data-loading mechanism should control request identity and lifetime.

## Client-side fetching with an Effect

An Effect is useful when a mounted component needs to synchronize with a request. Store the request status and result in state, and include every reactive value used by the Effect in its dependencies:

~~~jsx
import { useEffect, useState } from "react";

function SearchResults({ query }) {
  const [result, setResult] = useState({ status: "loading" });

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      setResult({ status: "loading" });

      try {
        const response = await fetch(
          "/api/search?q=" + encodeURIComponent(query),
          { signal: controller.signal },
        );

        if (!response.ok) {
          throw new Error("Search failed: " + response.status);
        }

        const items = await response.json();
        setResult({ status: "success", items });
      } catch (error) {
        if (controller.signal.aborted) {
          return;
        }

        setResult({ status: "error", error });
      }
    }

    load();

    return () => controller.abort();
  }, [query]);

  if (result.status === "loading") {
    return <p>Loading results…</p>;
  }

  if (result.status === "error") {
    return <p role="alert">Could not load results: {result.error.message}</p>;
  }

  return (
    <ul>
      {result.items.map((item) => (
        <li key={item.id}>{item.label}</li>
      ))}
    </ul>
  );
}
~~~

This example aborts the previous request when the query changes or the component unmounts. Cancellation and stale-response ordering deserve careful treatment; see [lesson 074](074-request-cancellation-and-race-conditions.md). The basic rule is already useful: asynchronous work may finish after the UI has moved on, so cleanup must prevent obsolete work from updating current state.

An Effect runs after React commits a render. That means an initial render can show a loading state before the request starts. This is a normal consequence of this approach. An Effect-based request does not activate a Suspense fallback.

## Read a Promise with React's `use` API

React's `use` API can read a Promise during rendering:

~~~jsx
import { use } from "react";

function SearchResults({ resultsPromise }) {
  const items = use(resultsPromise);

  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.label}</li>
      ))}
    </ul>
  );
}
~~~

For a Promise:

- while it is pending, `use` suspends the component; the nearest `<Suspense>` boundary displays its fallback;
- when it fulfills, `use` returns the fulfilled value and React can render the component;
- when it rejects, the error is thrown to the nearest Error Boundary.

A parent can provide the Promise, a framework can provide it, or a Suspense-aware data layer can manage it. In a client component, the source must reuse the same Promise for the same resource across rerenders. If each render creates a new Promise, the component can keep suspending and restarting.

### Do not create a Promise inline during render

This is an unstable pattern:

~~~jsx
function SearchResults({ query }) {
  // Avoid: this starts a fresh request each time this component renders.
  const items = use(fetch("/api/search?q=" + encodeURIComponent(query))
    .then((response) => response.json()));

  return <Results items={items} />;
}
~~~

The render creates a new Promise, so React has no stable resource to read when it retries. A client-side cache or framework should own Promise creation and reuse. A small module-level cache can teach the concept, but it is not a full production cache:

~~~jsx
import { use } from "react";

const searchPromises = new Map();

function getSearchPromise(query) {
  const key = query.trim().toLowerCase();

  if (!searchPromises.has(key)) {
    const promise = fetch("/api/search?q=" + encodeURIComponent(key))
      .then((response) => {
        if (!response.ok) {
          throw new Error("Search failed: " + response.status);
        }

        return response.json();
      });

    searchPromises.set(key, promise);
  }

  return searchPromises.get(key);
}

function SearchResults({ query }) {
  const items = use(getSearchPromise(query));
  return <Results items={items} />;
}
~~~

This cache reuses a Promise for a normalized query, but it never expires or refreshes entries, has no memory limit, and is not scoped to a user session. Real applications should use their framework or a deliberate data layer for cache lifetime, invalidation, request deduplication, authentication boundaries, and retries. Those topics are covered in [lesson 076](076-caching-and-revalidation.md).

### Add the right boundaries

Suspense handles pending work. An Error Boundary handles failures. They solve different states:

~~~jsx
<ErrorBoundary fallback={<p role="alert">Search could not be loaded.</p>}>
  <Suspense fallback={<p>Loading search results…</p>}>
    <SearchResults query={query} />
  </Suspense>
</ErrorBoundary>
~~~

The Error Boundary is shown here as an application component; React does not provide a built-in function component API for declaring an Error Boundary. Use the boundary implementation or framework already chosen for the application.

A Suspense boundary only handles a Promise that suspends while React renders. If an Effect calls `fetch()` and later sets state, Suspense does not take over that request. The component must render its own loading state.

## Special rules for `use`

`use` is a React API, but it has a specific set of rules:

- Call it while rendering a function component or a custom Hook, not in an event handler or ordinary utility function.
- Unlike most Hooks, `use` may be called inside a condition or loop.
- When reading a Promise, do not inspect undocumented Promise fields or skip `use` because you believe the Promise has already settled. Read it through React each time that render needs the value.
- Do not wrap `use` in `try/catch` to handle a rejection. Let the nearest Error Boundary handle it.
- A Promise passed from a Server Component to a Client Component can also be read with `use`; the surrounding server and framework rules still apply.

`use` can also read context and may be useful when context access is conditional. It does not make arbitrary asynchronous code safe to start during render.

## Avoid using an Effect for the wrong reason

Not every calculation needs a request or an Effect:

- Derive filtered or formatted values during render when they can be computed from existing props and state.
- Start an action such as submitting a form in its event handler.
- Use an Effect when the component must stay synchronized with an external system, such as a request that follows the current screen or query.
- Use a Suspense-compatible source when the rendering tree is intentionally meant to wait for a Promise.

Strict Mode may run an extra setup-and-cleanup cycle for Effects in development. Correct cleanup makes this safe and helps reveal requests that are not managed properly.

## Common mistakes

- Assuming every non-2xx response rejects `fetch()` automatically.
- Forgetting that parsing the response body is asynchronous and can fail.
- Starting a request in render or in a state initializer without a stable resource design.
- Calling `use(fetch(url))` and creating a new Promise on every render.
- Expecting Suspense to catch a request started later inside `useEffect`.
- Treating a Suspense fallback as an error message; rejected Promises go to an Error Boundary.
- Adding an Effect for work that belongs directly in an event handler.
- Building a permanent module-level cache and assuming it handles expiry, invalidation, or user isolation.
- Catching a Promise rejection around `use` instead of using an Error Boundary.

## Practical checklist

1. What triggers this request: a user action, a screen appearing, or render-time data access?
2. Does this component need an Effect with status state, or does the app already provide a Suspense-compatible Promise?
3. Does the request check `response.ok` and handle parsing failures?
4. If using `use`, will the same Promise instance be reused across rerenders for the same resource?
5. Is a Suspense boundary near pending content and an Error Boundary near failures?
6. Can cleanup or a newer request make an earlier result obsolete?
7. Who owns cache expiry, invalidation, authentication scope, and retries?
8. Does the framework already provide a loader or data-fetching convention?

## Summary

Use Effects and state for straightforward client-side synchronization with requests, including explicit loading and error UI. Use `use` with Suspense when a framework or stable cache supplies a Promise designed for render-time reading. Pending Promises suspend, fulfilled Promises provide a value, and rejected Promises reach an Error Boundary. Never create a fresh fetch Promise directly during every client render.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="072-react-dom-resource-hints-and-browser-apis.md">072 - React DOM Resource Hints and Browser APIs</a></td><td><a href="../qnas/073-client-side-data-fetching-and-use-api.md">Q&amp;A</a></td><td><a href="../examples/073-client-side-data-fetching-and-use-api.md">Example</a></td><td><a href="074-request-cancellation-and-race-conditions.md">074 - Request Cancellation, Stale Responses, and Race Conditions</a></td></tr>
  </tbody>
</table>
