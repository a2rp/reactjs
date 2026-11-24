# Effect Hooks and Synchronization

## Effects connect React to external systems

Rendering should calculate the UI from props, state, and context. An Effect is for synchronizing a committed component with something outside React, such as a browser event listener, timer, network connection, or third-party widget.

An Effect is not a general-purpose callback for “after rendering” work. It is also not the place to calculate values that can be derived during render or to handle an event that already has a direct event handler.

This lesson covers three related Hooks:

- `useEffect` for most synchronization with external systems;
- `useLayoutEffect` for work that must happen after DOM updates but before the browser paints;
- `useInsertionEffect` for style-insertion libraries that must insert styles before layout Effects run.

## Learning goals

After this lesson, you should be able to:

- recognize when work belongs in an Effect and when it does not;
- describe an Effect as a setup and cleanup synchronization cycle;
- declare every reactive dependency and understand how React compares dependencies;
- clean up subscriptions, timers, and asynchronous work correctly;
- explain how Strict Mode checks Effect cleanup in development;
- choose between `useEffect`, `useLayoutEffect`, and `useInsertionEffect`;
- account for client-only execution and server-rendering behavior.

## The basic shape

Import the Hook from React and call it at the top level of a component or custom Hook:

~~~jsx
import { useEffect } from "react";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return <h1>Chat room: {roomId}</h1>;
}
~~~

The function passed to `useEffect` is the setup function. It runs after React commits the rendered UI. Setup may return a cleanup function. React calls cleanup before running setup again for changed dependencies and when the component is removed.

The Effect above keeps one connection synchronized with the current `roomId`. When the room changes, React disconnects the old room and connects to the new one. The Effect is not simply “component mounted” logic: it can stop and start synchronization multiple times while the component remains mounted.

An Effect setup may return either nothing or a cleanup function. Do not make the setup function itself `async`: an async function returns a Promise, but React expects a cleanup function or no return value.

## Effect lifecycle: start, stop, and synchronize again

Think about each Effect as an independent synchronization process:

1. React commits a render.
2. If this Effect needs to start or re-synchronize, React runs its setup.
3. When dependencies change, React runs the previous cleanup with the old values.
4. React then runs setup with the new values.
5. When the component unmounts, React runs the last cleanup.

Cleanup should undo or stop what the matching setup started. For a subscription, unsubscribe. For a timer, clear it. For a connection, disconnect it. Setup and cleanup form a pair; cleanup should not try to “undo the component” generally.

If setup did not create anything that needs stopping, it may not need a cleanup function. Do not add cleanup that performs unrelated state changes.

## Dependencies describe what setup reads

A dependency list tells React when the synchronization is out of date. Include every reactive value read by the setup or cleanup: props, state, and variables or functions declared in the component body.

~~~jsx
function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => connection.disconnect();
  }, [serverUrl, roomId]);

  return <h1>Chat room: {roomId}</h1>;
}
~~~

The dependency list must have a constant number of entries and be written inline. React compares each dependency with its previous value using `Object.is`. If a dependency changes, React cleans up the previous synchronization and starts the new one.

Common forms:

~~~jsx
// After every commit. Usually avoid this unless that is truly required.
useEffect(() => {
  synchronizeSomething();
});

// After the initial committed render, then cleanup on unmount.
// Strict Mode adds a development-only setup/cleanup check.
useEffect(() => {
  const subscription = subscribe();
  return () => subscription.unsubscribe();
}, []);

// Re-synchronize when a reactive value changes.
useEffect(() => {
  const subscription = subscribeToRoom(roomId);
  return () => subscription.unsubscribe();
}, [roomId]);
~~~

An empty array does not mean “run this whenever I want.” It means the Effect reads no reactive values that need to trigger re-synchronization. If the Effect reads `roomId`, include it. Do not suppress the dependency linter to force a preferred schedule. Change the code so its dependencies match the intended synchronization.

### Avoid accidental object and function dependencies

Objects and functions created in the component body get a new identity on each render. If an Effect depends on one of them, it may re-run on every render even when its meaningful data did not change.

~~~jsx
function ChatRoom({ roomId }) {
  const options = { roomId, retry: true };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return <h1>{roomId}</h1>;
}
~~~

Here `options` is a new object on every render. Prefer creating the object inside the Effect and depending on the values that define it:

~~~jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const options = { roomId, retry: true };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>{roomId}</h1>;
}
~~~

Do not remove a real dependency just to stop an Effect from running. Restructure the code so the Effect reads fewer reactive values, or use a state updater when the next state depends on the previous state.

## Example: subscribe to a browser event

A browser event listener is an external subscription. The Effect should add it and remove the same listener during cleanup:

~~~jsx
import { useEffect, useState } from "react";

function NetworkStatus() {
  const [online, setOnline] = useState(() => navigator.onLine);

  useEffect(() => {
    function updateStatus() {
      setOnline(navigator.onLine);
    }

    window.addEventListener("online", updateStatus);
    window.addEventListener("offline", updateStatus);

    return () => {
      window.removeEventListener("online", updateStatus);
      window.removeEventListener("offline", updateStatus);
    };
  }, []);

  return <p>{online ? "You are online" : "You are offline"}</p>;
}
~~~

The same function reference is passed to `addEventListener` and `removeEventListener`. Removing a different function would not unsubscribe the listener created by setup.

This initial state reads a browser global, so this exact version is for a client-rendered component. In server-rendered applications, browser globals are unavailable on the server. Initialize with a server-safe value and read browser state after mounting, or use the platform integration supplied by the framework.

## Example: timer with cleanup and a state updater

A timer is an external system. The Effect owns its lifetime, and cleanup clears the timer:

~~~jsx
import { useEffect, useState } from "react";

function Stopwatch() {
  const [running, setRunning] = useState(false);
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    if (!running) {
      return;
    }

    const timerId = window.setInterval(() => {
      setSeconds((currentSeconds) => currentSeconds + 1);
    }, 1000);

    return () => window.clearInterval(timerId);
  }, [running]);

  return (
    <section>
      <p>Elapsed: {seconds} seconds</p>
      <button type="button" onClick={() => setRunning((value) => !value)}>
        {running ? "Pause" : "Start"}
      </button>
      <button type="button" onClick={() => setSeconds(0)}>
        Reset
      </button>
    </section>
  );
}
~~~

The functional state update reads the latest `seconds` value without making `seconds` an Effect dependency. If `seconds` were added to the dependency list and used to recreate the interval, the timer would restart after every tick.

When `running` changes from true to false, React runs cleanup and clears the interval. When it changes back to true, setup starts a new interval.

## Asynchronous work and race conditions

For a request tied to a changing value, stop or ignore obsolete work during cleanup. This example uses `AbortController` so a previous request is cancelled when the query changes or the component unmounts:

~~~jsx
import { useEffect, useState } from "react";

function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    async function loadResults() {
      setError(null);

      try {
        const response = await fetch(
          "/api/search?q=" + encodeURIComponent(query),
          { signal: controller.signal }
        );

        if (!response.ok) {
          throw new Error("The search request failed.");
        }

        const data = await response.json();
        setResults(data.results);
      } catch (requestError) {
        if (requestError.name !== "AbortError") {
          setError(requestError);
        }
      }
    }

    loadResults();

    return () => controller.abort();
  }, [query]);

  if (error) {
    return <p role="alert">{error.message}</p>;
  }

  return <ResultsList results={results} />;
}
~~~

The async function is declared inside the synchronous Effect setup. Cleanup aborts the request created for the previous query. This prevents an obsolete response from winning after the user has already entered a newer query.

Real applications often use a framework's data-loading features or a dedicated query library for caching, deduplication, retries, and server rendering. An Effect is a useful low-level synchronization tool, but it does not provide those data-layer features by itself.

## Most Effects do not need to set derived state

If a value can be calculated from the current props or state during render, calculate it there. Do not store a duplicate and update it from an Effect:

~~~jsx
function ProductList({ products, query }) {
  const normalizedQuery = query.trim().toLowerCase();

  const visibleProducts = products.filter((product) =>
    product.name.toLowerCase().includes(normalizedQuery)
  );

  return <ul>{visibleProducts.map((product) => <li key={product.id}>{product.name}</li>)}</ul>;
}
~~~

Adding an Effect to copy `query` into another state variable or to filter products after rendering creates an extra render and risks stale or contradictory state. Use memoization only when measurement shows that the calculation is expensive; memoization is a performance optimization, not a requirement for correctness.

Likewise, handle a known user action in its event handler. If clicking a button submits an order, submit it in that button's handler. An Effect that watches a `shouldSubmit` state flag obscures which user action caused the request.

A useful question is: “What external system does this Effect synchronize with?” If there is no clear answer, the logic probably belongs in render, state transitions, or an event handler.

## `useLayoutEffect`: measure before paint

`useLayoutEffect` has the same setup and cleanup shape as `useEffect`, but React runs it after committing DOM changes and before the browser repaints. Use it when the user must not see an intermediate layout, such as measuring a tooltip and correcting its position before it appears.

~~~jsx
import { useLayoutEffect, useRef, useState } from "react";

function MeasuredPanel() {
  const panelRef = useRef(null);
  const [height, setHeight] = useState(0);

  useLayoutEffect(() => {
    const panel = panelRef.current;
    if (!panel) return;

    setHeight(panel.getBoundingClientRect().height);
  }, []);

  return (
    <section>
      <div ref={panelRef}>Measure this panel after it is placed in the DOM.</div>
      <p>Measured height: {height}px</p>
    </section>
  );
}
~~~

A state update inside `useLayoutEffect` causes React to render again before paint, so the user can see the corrected result without a visible intermediate frame. This blocks painting and can hurt responsiveness. Prefer `useEffect` unless pre-paint measurement or correction is necessary.

`useLayoutEffect` only runs in the browser. It cannot measure server-rendered layout. For content that must render on the server, render a layout that works without measurement, or defer the measurement-dependent UI until the component has mounted.

## `useInsertionEffect`: a specialized styling hook

`useInsertionEffect` is intended primarily for authors of CSS-in-JS libraries that need to insert style rules before layout Effects run. It is not a general application hook for updating styles, measuring layout, or synchronizing arbitrary data.

It runs before layout Effects, but it is not a general “before DOM mutation” callback: at this point refs are not attached, and the DOM may or may not have been updated. Do not read layout or expect DOM refs to be available. State updates are not allowed from this Hook. Prefer static CSS, extracted stylesheets, or ordinary style props when those meet the need.

~~~jsx
import { useInsertionEffect } from "react";

// Illustrative library-level pattern only.
function useInsertGeneratedRule(rule) {
  useInsertionEffect(() => {
    insertRuleIntoStyleSheet(rule);
  }, [rule]);
}
~~~

A style library also needs a sound policy for deduplicating and removing rules; the small illustration is not a complete CSS-in-JS implementation. Like the other Effects here, `useInsertionEffect` is client-only and does not run during server rendering.

## Strict Mode checks whether cleanup mirrors setup

When Strict Mode is enabled, React runs an extra setup-and-cleanup cycle for Effects in development. A newly mounted component can therefore appear to connect, disconnect, and connect again. This is a development check, not an extra production subscription.

The check helps expose Effects whose cleanup is missing or incomplete. Make setup and cleanup symmetrical. Do not hide the extra cycle with a ref that makes setup run only once: that masks the bug and does not solve remounting or dependency changes.

Effects should remain correct if React starts synchronization, stops it, and starts it again with the same values.

## Common mistakes

### Using an Effect for every state change

Effects are for synchronization with external systems. Derive UI during render, update state in event handlers, and use reducers or state updater functions for transitions within React.

### Leaving out dependencies

A missing dependency can make the Effect use an old render's value. Include the reactive values the Effect reads, then restructure if the resulting synchronization is not what you want.

### Adding dependencies that change identity every render

A new object or function can cause unnecessary re-synchronization. Move static helper definitions outside the component, create effect-only values inside setup, or otherwise stabilize the design without suppressing real dependencies.

### Forgetting cleanup

An event listener, subscription, observer, or timer that is never stopped may continue after the UI no longer needs it. Return cleanup that releases exactly what setup created.

### Choosing `useLayoutEffect` by default

It blocks browser painting. Start with `useEffect`; switch only when a visible pre-paint correction is required.

### Treating `useInsertionEffect` as an application styling shortcut

It is a specialized hook for style insertion libraries. It has constraints that make it unsuitable for normal component logic.

## Effects and server rendering

Effects run only on the client. They do not run while React renders HTML on the server. Server-rendered output must therefore be meaningful without an Effect having run.

Do not put essential content, initial data needed for the HTML response, or server-side work exclusively in an Effect. Use the framework's server data and rendering APIs where available. Avoid reading `window`, `document`, or other browser-only globals during render when that component can render on the server.

## A practical Effect review

Before keeping an Effect, check:

1. What external system does it synchronize with?
2. Does setup start or configure that synchronization?
3. Does cleanup stop or undo the corresponding setup?
4. Does the dependency list include every reactive value read?
5. Can a derived value be calculated during render instead?
6. Can a user-triggered action be handled directly by an event handler?
7. Is `useLayoutEffect` truly needed before paint?
8. Is this one of the rare library-level cases for `useInsertionEffect`?
9. Does the behavior remain correct through Strict Mode's extra development cycle?
10. Does server-rendered output work before client Effects run?

## Key takeaways

- Effects synchronize committed React UI with external systems.
- Every Effect has its own start/stop lifecycle and may re-synchronize when dependencies change.
- Include every reactive value the Effect reads; dependencies are compared with `Object.is`.
- Cleanup runs before re-synchronization and on unmount; it should mirror setup.
- Strict Mode deliberately tests that setup and cleanup can repeat safely in development.
- Avoid Effects for derived state, render calculations, and known user events.
- Prefer `useEffect`; reserve `useLayoutEffect` for necessary pre-paint layout work.
- Reserve `useInsertionEffect` for CSS-in-JS library authors.
- Effects run only on the client, not during server rendering.
