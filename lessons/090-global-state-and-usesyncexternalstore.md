# Global State, `useSyncExternalStore`, and External Stores

## Learning goals

After this lesson, you should be able to:

- Distinguish local React state, Context, and an external store.
- Recognize when a value truly lives outside React.
- Connect an external store with `useSyncExternalStore`.
- Implement stable subscription and snapshot functions.
- Avoid mutable or freshly allocated snapshots that cause incorrect updates.
- Understand the server-rendering and hydration requirements.
- Keep external state scoped and use it only where its ownership is clear.

## 1. “Global state” is a placement question

An application's state does not automatically need one global store. For each value, first identify who owns it and which components need it.

Common options include:

- **Local state** with `useState` or `useReducer` for values owned by one component or a nearby group.
- **Context** when a value owned by a React component needs to reach distant descendants in that component tree.
- **An external store** when the source of truth is outside React or needs an independent subscription lifecycle.

Examples of data that may live outside React include:

- A state-management library's store.
- A legacy JavaScript module that updates independently.
- A browser API such as the online/offline status.
- A shared client-side cache owned outside the React tree.

Use the simplest owner that fits the data flow. A value used in several components is not, by itself, proof that it needs an external store. A reducer combined with Context can share state that still belongs to React.

## 2. What is an external store?

An external store owns mutable information outside the React component tree. It provides at least two capabilities:

1. A way for React to subscribe to changes.
2. A way for React to read the current snapshot.

React does not automatically know when that external value changes. A subscription connects the store to components that need to render it.

A store should update its own data first, then notify subscribers. React reads the new snapshot and updates the subscribed components.

## 3. The `useSyncExternalStore` API

The Hook accepts a subscribe function, a snapshot reader, and optionally a server snapshot reader:

```jsx
import { useSyncExternalStore } from "react";

const snapshot = useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot,
);
```

- **`subscribe(callback)`** registers React's callback with the store and returns an unsubscribe function.
- **`getSnapshot()`** returns the current immutable value that the component needs.
- **`getServerSnapshot()`** supplies the initial value during server rendering and hydration. It is required when rendering this component on the server.

When the store calls the subscribed callback, React reads the snapshot again. If the snapshot differs from the previous one according to `Object.is`, React renders the component with the new value.

## 4. Subscription functions need a stable identity

Define the subscribe function outside the component when it does not depend on component props:

```js
function subscribe(callback) {
  store.listeners.add(callback);

  return () => {
    store.listeners.delete(callback);
  };
}
```

If a new subscribe function is created during every render, React must unsubscribe and subscribe again. A module-level function is naturally stable. If a subscription genuinely depends on a component value, use a stable callback whose dependencies are explicit.

Always clean up subscriptions when the component no longer needs them. Returning an unsubscribe function from `subscribe` lets React perform that cleanup.

## 5. Snapshot identity must be stable and immutable

While the store has not changed, repeated calls to `getSnapshot()` must return the same value. For an object snapshot, keep and return the same object until the underlying data changes.

Incorrect:

```js
function getSnapshot() {
  // A new object is created for every read.
  return { count: store.count };
}
```

This appears to React as a new snapshot on every read and can cause repeated rendering or a cached-snapshot warning.

A simple immutable store can keep one object snapshot:

```js
let snapshot = { count: 0 };

function getSnapshot() {
  return snapshot;
}

function increment() {
  snapshot = { ...snapshot, count: snapshot.count + 1 };
  emitChange();
}
```

Do not mutate a snapshot object and return the same reference. React could then miss the change because the identity has not changed. Instead, create a new immutable snapshot when data changes and retain it between changes.

Primitive values such as booleans and numbers are naturally simple snapshots. For objects and arrays, stable immutable identity matters.

## 6. Notify only after updating the store

A small store can keep subscribers in a `Set`:

```js
const listeners = new Set();

function subscribe(callback) {
  listeners.add(callback);
  return () => listeners.delete(callback);
}

function emitChange() {
  for (const listener of listeners) {
    listener();
  }
}
```

When an action changes data, follow this order:

1. Compute and store the next snapshot.
2. Notify subscribers.

Calling subscribers before updating the snapshot can make React read the old value. Do not notify when an action makes no change unless consumers have another reason to be notified.

## 7. Do not hand-roll subscriptions with an Effect

A common first attempt is to read a store into React state inside an Effect and manually attach a listener. This creates two representations of the same value: the store snapshot and the component's state copy. It also means each component must manage subscription timing and cleanup correctly.

`useSyncExternalStore` is the purpose-built API for subscribing to data that changes outside React. It lets React coordinate the snapshot read with subscription and rendering, including concurrent rendering.

Use Effects for synchronizing with external systems when appropriate, but use this Hook when the specific job is subscribing to an external store snapshot.

## 8. Build a small store only to understand the contract

A minimal store can be useful for learning or a narrowly scoped use case. It should have:

- One clear owner and scope.
- Explicit actions that update its data.
- Immutable snapshot replacement when a value changes.
- A subscription function that returns cleanup.
- A stable snapshot reader.
- A tested way to initialize and reset the store when needed.

A hand-written store also means you own notification semantics, selector behavior, dev tooling, persistence, and server integration. Most apps should begin with React state and Context, and use a maintained state library when its features are needed.

## 9. Avoid making every component subscribe to everything

If every component reads a large global snapshot, every snapshot update can cause all those subscribers to check and potentially render with the new value.

To keep updates focused:

- Split state by ownership or update frequency.
- Subscribe near the components that use the data.
- Derive simple display values from the snapshot during render.
- Use a selector-aware library when consumers need selective subscriptions to portions of a large state.
- Keep unrelated feature data out of one universal state object.

A component should subscribe only to the information it needs.

## 10. Browser APIs are also external stores

A browser value can change without React knowing about it. For example, network connectivity is exposed as `navigator.onLine` and changes can be observed through browser events:

```js
function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);

  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function getSnapshot() {
  return navigator.onLine;
}
```

A Hook can pass these functions to `useSyncExternalStore`. This is better suited to a browser-owned value than copying it into component state and trying to keep the copy synchronized.

Browser APIs may be unavailable during server rendering. Provide an appropriate server snapshot when needed and avoid accessing browser globals at module initialization in code that also runs on the server.

## 11. Server rendering and hydration

When a component using an external store is server-rendered, provide `getServerSnapshot`. The initial server snapshot and the snapshot used during client hydration must represent the same initial data. If they differ, the server HTML and client render can disagree.

For a per-user or per-request store:

- Create the store in the correct request or application scope.
- Serialize only the initial data needed by the client.
- Initialize the client store from that same data for hydration.
- Avoid a mutable module-level singleton that can leak one server request's state into another.

A module-level singleton can be appropriate for one browser application instance, but its lifetime and scope are different in a server process that handles multiple users.

If a component is only used in a client-rendered app, the third argument may be omitted. Do not omit it for a component that can be rendered on the server.

## 12. External stores and transitions

External stores are not ordinary React state. A store can change independently while React is rendering. React checks snapshots so a commit does not leave parts of the screen showing inconsistent versions of the store.

A mutation during a non-blocking Transition can cause React to restart the update as blocking when the snapshot changed. Treat store updates as synchronous external changes; use React's own state APIs for work that should participate in React transitions.

## 13. State libraries and Context

A state-management library can provide APIs for stores, selectors, middleware, persistence, debugging, or server support. Its React integration should correctly subscribe React to store snapshots.

Context is a React mechanism for passing values through a component tree. It does not itself provide persistence or a separate external store. Combining a reducer and Context is often enough for complex state that belongs to a React subtree.

Choose based on ownership and required behavior, not on a rule that all shared state must use a global library.

## 14. Common mistakes

- Moving ordinary local state into a global store without a coordination need.
- Assuming Context and external stores solve the same problem.
- Mutating a snapshot object without changing its identity.
- Returning a fresh object from every `getSnapshot()` call.
- Creating a new subscribe function on every render unnecessarily.
- Forgetting to return an unsubscribe function.
- Notifying listeners before replacing the store snapshot.
- Maintaining a second copy of store data in component state.
- Making every component subscribe to one oversized store.
- Accessing browser APIs during server rendering without a safe snapshot.
- Sharing a module-level mutable store across independent server requests.
- Writing a custom store before checking whether built-in state or an existing library is enough.

## Key takeaways

- Use `useState`, `useReducer`, and Context when the data belongs to React.
- Use `useSyncExternalStore` when a value is owned and updated outside React.
- A correct external store exposes stable subscription and snapshot functions.
- Snapshots must be immutable and keep the same identity while unchanged.
- Server rendering requires an initial server snapshot that matches hydration.
- Keep store scope and subscriptions narrow; do not make global state a default destination.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="089-design-systems-and-component-libraries.md">089 - Design Systems, Shared Components, and Component Libraries</a></td><td><a href="../qnas/090-global-state-and-usesyncexternalstore.md">Q&amp;A</a></td><td><a href="../examples/090-global-state-and-usesyncexternalstore.md">Example</a></td><td><a href="091-typescript-with-react.md">091 - TypeScript with React Components and Hooks</a></td></tr>
  </tbody>
</table>
