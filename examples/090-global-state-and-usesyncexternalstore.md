# Global State, `useSyncExternalStore`, and External Stores - Examples

This example connects a small JavaScript counter store to React with `useSyncExternalStore`. The store exists outside the component tree, and two separate components subscribe to its cached snapshot.

This is a client-rendered Vite example. Its module-level store is scoped to one browser app instance. A server-rendered app needs a request-scoped store and matching server and hydration snapshots.

## 1. File structure

```text
src/
  main.jsx
  app/
    App.jsx
    app.css
  features/
    counter/
      counterStore.js
      useCounterSnapshot.js
      CounterStatus.jsx
      CounterControls.jsx
```

This example keeps the store inside the counter feature. It does not use React state as a duplicate copy of the counter.

## 2. Create the external store

Create `src/features/counter/counterStore.js`:

```js
const listeners = new Set();
const minimum = -10;
const maximum = 10;

let snapshot = Object.freeze({ count: 0 });

function subscribe(callback) {
  listeners.add(callback);

  return () => {
    listeners.delete(callback);
  };
}

function getSnapshot() {
  return snapshot;
}

function setCount(nextCount) {
  const count = Math.max(minimum, Math.min(maximum, nextCount));

  if (count === snapshot.count) {
    return;
  }

  // Replace the immutable snapshot only when the data changes.
  snapshot = Object.freeze({ count });

  // Notify React after the new snapshot is available.
  for (const listener of listeners) {
    listener();
  }
}

export const counterStore = {
  subscribe,
  getSnapshot,

  increment() {
    setCount(snapshot.count + 1);
  },

  decrement() {
    setCount(snapshot.count - 1);
  },

  reset() {
    setCount(0);
  },
};
```

The snapshot object has stable identity while the count is unchanged. Each real update replaces it before notifying subscribers. The unsubscribe function removes the callback from the listener set.

## 3. Connect the store to React

Create `src/features/counter/useCounterSnapshot.js`:

```js
import { useSyncExternalStore } from "react";
import { counterStore } from "./counterStore.js";

export function useCounterSnapshot() {
  return useSyncExternalStore(
    counterStore.subscribe,
    counterStore.getSnapshot,
  );
}
```

The subscribe and snapshot functions are defined outside the Hook, so their identities stay stable between renders. Each component that calls this Hook subscribes to the same store.

## 4. Create separate subscribers

Create `src/features/counter/CounterStatus.jsx`:

```jsx
import { useCounterSnapshot } from "./useCounterSnapshot.js";

export default function CounterStatus() {
  const { count } = useCounterSnapshot();

  return (
    <p className="counter-status" aria-live="polite">
      Shared counter value: <output>{count}</output>
    </p>
  );
}
```

Create `src/features/counter/CounterControls.jsx`:

```jsx
import { counterStore } from "./counterStore.js";
import { useCounterSnapshot } from "./useCounterSnapshot.js";

export default function CounterControls() {
  const { count } = useCounterSnapshot();

  return (
    <section className="counter-controls" aria-label="Counter controls">
      <button
        type="button"
        onClick={counterStore.decrement}
        disabled={count <= -10}
      >
        Decrease
      </button>

      <button
        type="button"
        onClick={counterStore.reset}
        disabled={count === 0}
      >
        Reset
      </button>

      <button
        type="button"
        onClick={counterStore.increment}
        disabled={count >= 10}
      >
        Increase
      </button>
    </section>
  );
}
```

Both components subscribe to the same external snapshot. Clicking a control changes the store and notifies both subscribers; React reads the new snapshot and updates the UI.

## 5. Assemble the application

Create `src/app/App.jsx`:

```jsx
import CounterControls from "../features/counter/CounterControls.jsx";
import CounterStatus from "../features/counter/CounterStatus.jsx";
import "./app.css";

export default function App() {
  return (
    <main className="app-shell">
      <p className="app-eyebrow">External store example</p>
      <h1>Shared counter</h1>
      <p className="app-description">
        The status and controls are separate components reading one store.
      </p>

      <div className="counter-card">
        <CounterStatus />
        <CounterControls />
      </div>
    </main>
  );
}
```

Create `src/app/app.css`:

```css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #192235;
  background: #f3f5fa;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button {
  font: inherit;
}

.app-shell {
  width: min(100% - 2rem, 640px);
  margin-inline: auto;
  padding-block: 4rem;
}

.app-eyebrow {
  margin: 0 0 0.5rem;
  color: #3448b5;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.app-shell h1 {
  margin: 0 0 0.5rem;
  font-size: clamp(2rem, 5vw, 3rem);
}

.app-description {
  margin: 0 0 1.5rem;
  color: #626d81;
  line-height: 1.6;
}

.counter-card {
  display: grid;
  gap: 1.25rem;
  border: 1px solid #dce1ec;
  border-radius: 14px;
  padding: 1.5rem;
  background: #fff;
  box-shadow: 0 10px 30px rgb(25 39 74 / 6%);
}

.counter-status {
  margin: 0;
  color: #35405a;
  font-size: 1.1rem;
}

.counter-status output {
  font-weight: 800;
}

.counter-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.counter-controls button {
  min-height: 44px;
  border: 0;
  border-radius: 8px;
  padding: 0.6rem 1rem;
  background: #3448b5;
  color: #fff;
  font-weight: 700;
}

.counter-controls button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.counter-controls button:focus-visible {
  outline: 3px solid #8897ed;
  outline-offset: 2px;
}
```

Create `src/main.jsx`:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./app/App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

The existing app entry page must provide an element with `id="root"`, as in a standard Vite React starter.

## 6. Observe the subscription contract

Try these interactions:

1. Select **Increase**. The value in `CounterStatus` changes even though that component did not handle the click.
2. Select **Decrease**. Both components continue to reflect the same store value.
3. Select **Reset**. The snapshot returns to zero and the reset button becomes disabled.
4. Increase to 10. The increase button becomes disabled; the store does not notify for a change that would leave the count at 10.
5. Unmount either subscriber in a React DevTools experiment. Its subscription is cleaned up by the function returned from `subscribe`.

## 7. Common incorrect snapshot

Do not create a new object every time React asks for the snapshot:

```js
function getSnapshot() {
  // Incorrect: a fresh object makes every read look like a change.
  return { count: snapshot.count };
}
```

Return the cached `snapshot` object as the example does. Replace it only when the store changes.

Also avoid mutating the current object:

```js
function setCount(nextCount) {
  // Incorrect: its identity stays the same.
  snapshot.count = nextCount;
  notifyListeners();
}
```

With an unchanged identity, React may not detect the update. Replace the object and then notify instead.

## 8. When not to use this pattern

For a counter used only inside one React subtree, `useState` or `useReducer` would be simpler. If a reducer-owned value only needs to reach distant descendants, Context may be enough.

Use this external-store pattern when there is a meaningful source outside React or when integrating a store that already has its own subscription API. In larger applications, a state library may provide selectors, devtools, persistence, and server-rendering integration.

This example does not implement persistence or server rendering. A module-level mutable singleton should not be shared across independent server requests. For SSR, create a store per request or app instance and ensure the initial server snapshot matches the snapshot used during hydration.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="089-design-systems-and-component-libraries.md">089 - Design Systems, Shared Components, and Component Libraries - Examples</a></td><td><a href="../lessons/090-global-state-and-usesyncexternalstore.md">Lesson</a></td><td><a href="../qnas/090-global-state-and-usesyncexternalstore.md">Q&amp;A</a></td><td><a href="091-typescript-with-react.md">091 - TypeScript with React: Reading List Example</a></td></tr>
  </tbody>
</table>
