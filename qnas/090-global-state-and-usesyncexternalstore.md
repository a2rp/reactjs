# Global State, `useSyncExternalStore`, and External Stores — Q&A

## Choosing where state belongs

### 1. What does “global state” mean?

It usually means state that multiple distant parts of an application need to read or update. It does not tell you where that state should live. Choose an owner based on its source, lifetime, and consumers.

### 2. Should all shared state use an external store?

No. State can be lifted to a common React parent or passed through Context. Use an external store when the source of truth is outside React or when a store library's behavior is needed.

### 3. What is the difference between Context and an external store?

Context passes a value from a React provider to descendants. The provider's value is still owned by React unless it wraps some other source. An external store owns data outside the React tree and notifies subscribers when it changes.

### 4. When should I use `useState` or `useReducer` instead?

Use React state when the value belongs to a component or a React subtree. A reducer helps when related transitions or actions are becoming complex. React recommends built-in state APIs when they fit the problem.

### 5. Does using a custom Hook make state global?

No. The Hook's state belongs to each component that calls it. A custom Hook can share logic, but a common owner or external store is needed to share one value.

## The external-store Hook

### 6. What does `useSyncExternalStore` return?

It returns the current snapshot read by the provided `getSnapshot` function. The component uses that snapshot during rendering.

### 7. What does the `subscribe` function do?

It registers a callback with the store and returns a cleanup function that removes that callback. The store calls the callback after its snapshot changes so React can read the latest value.

### 8. What does `getSnapshot` do?

It reads the current value needed by the component. It must return the same value while the store has not changed and a different value when the relevant snapshot changes.

### 9. Why does `getSnapshot` need a stable result?

React compares snapshot values with `Object.is`. Returning a new object on every read makes React think the store changed even when its data did not, which can cause repeated renders or a cached-snapshot warning.

### 10. Can the store mutate an object and return the same reference?

It should not. React may see the same reference and miss the change. Replace the snapshot with an immutable object when the data changes, and keep that snapshot stable between changes.

### 11. Why should the subscribe function be stable?

If a different function is passed after each render, React will unsubscribe and subscribe again. Define it outside the component when it has no component-specific dependencies, or stabilize a genuinely dependent subscription.

### 12. What should the unsubscribe function do?

It should remove that subscriber and release any resources owned for that subscription. This prevents callbacks from lingering after a component no longer needs the store.

### 13. Should I call the subscriber before or after updating the store?

After updating the store. React will read the snapshot when notified, so the new snapshot must already be available.

### 14. Is `useSyncExternalStore` a replacement for all Effects?

No. It is for reading and subscribing to an external store snapshot. Effects still synchronize with many other external systems, such as setting up a connection. Use the dedicated subscription Hook for the store-subscription job.

### 15. Why not subscribe in an Effect and copy the store value into `useState`?

That creates two copies that can get out of sync and makes each component manage subscription timing and cleanup. `useSyncExternalStore` is designed to connect React rendering to external snapshots.

## Snapshots and updates

### 16. Should a snapshot contain the entire store?

It can, but large snapshots may make many components react to unrelated changes. Prefer focused subscriptions or store designs that let components read only what they need.

### 17. Can I derive a value from the snapshot during render?

Yes, for simple derivations. For expensive selectors or selective subscriptions, use a selector-aware store integration or a carefully designed subscription API. Avoid creating a new object directly in `getSnapshot` on every call.

### 18. Does a component re-render every time a store notifies listeners?

React reads the snapshot and compares it to the previous one. If the snapshot is unchanged according to `Object.is`, React does not need to render that component with a new value.

### 19. Should all components subscribe to one large global store?

Usually not. Split state by ownership, scope, or update frequency, and subscribe near the consumers. A large universal store can make unrelated parts of the interface update and makes ownership less clear.

### 20. Are external-store updates ordinary React state updates?

No. The store may change independently of React. React coordinates snapshot reads to keep committed UI consistent. Use React state APIs for state that should participate in React's scheduling features.

## Server rendering and browser APIs

### 21. What does `getServerSnapshot` do?

It returns the initial value used during server rendering and client hydration. If a component is server-rendered, this argument is required. The initial client hydration value should match the server's value.

### 22. Can I omit `getServerSnapshot`?

You can omit it when the component is not rendered on the server. If it is rendered on the server without this function, React will throw an error.

### 23. What if the external data changes between server rendering and hydration?

Serialize the initial data used by the server and initialize the client store from that same data for hydration. After hydration, the store can expose newer values. If the initial snapshots disagree, server HTML and the first client render may not match.

### 24. Is a module-level store safe for server-rendered apps?

A mutable singleton can be shared by every request in one server process. That can leak data between users. Create per-request state and provide the correct instance to that request's React tree.

### 25. Can a module-level singleton be fine in a browser app?

It can be appropriate when the app has one client-side instance and the store's lifetime is intended to match that app. Still consider how tests, multiple roots, logout/reset, and microfrontends should scope the store.

### 26. How can I use a browser API as an external store?

Provide a stable snapshot reader and a subscription that listens to the API's change events. For example, `navigator.onLine` can be read directly and updated in response to `online` and `offline` events.

### 27. Do all browser APIs work during server rendering?

No. APIs such as `window` and `navigator` may not exist on the server. Avoid reading them at module initialization in shared server/client code and provide a suitable server snapshot when rendering requires one.

## Libraries and practical use

### 28. Should I write my own external store?

Usually only for learning or a narrow need. A production store also needs decisions about selectors, notification behavior, debugging, persistence, server scope, and integration. Use React state where it works and evaluate established libraries when more features are required.

### 29. How can I avoid unnecessary store notifications?

Compare the next data with the current data and do not notify when the action makes no meaningful change. Keep snapshot updates immutable and make action behavior explicit.

### 30. Can a component use both Context and an external store?

Yes. For example, Context can provide a store instance or configuration to a subtree, while a Hook subscribes to that store. Keep the responsibilities clear: Context supplies the instance; the external-store subscription supplies changing snapshots.

### 31. Can I use `useSyncExternalStore` to make updates low priority?

No. External store mutations are not ordinary React state updates. React may restart a transition as a blocking update if the store snapshot changes during it. Use React state for updates that need React transition scheduling.

### 32. How do I test an external-store integration?

Check that a component renders the initial snapshot, responds to a store update, ignores unchanged snapshots, and unsubscribes on unmount. For server rendering, verify that the initial server and hydration snapshots match. Also test the store's action and notification behavior.

### 33. Does browser storage automatically work as an external store?

No. Storage APIs provide values and events with their own behavior. For example, the `storage` event is not a general same-document notification for every write. A storage-backed store needs an explicit notification strategy for writes in the current app as well as updates from other browsing contexts.

### 34. What is a good rule of thumb?

Start with local state, reducer, or Context when React owns the value. Use `useSyncExternalStore` when the changing value is owned outside React. Keep the store's scope narrow and the snapshot contract correct.

## Quick review

- `subscribe` registers a callback and returns cleanup.
- `getSnapshot` must be stable while unchanged and immutable across updates.
- `getServerSnapshot` supplies matching initial server and hydration data.
- Context passes React-owned values through a tree; an external store owns data outside it.
- Avoid duplicate state copies and oversized global subscriptions.
- A module singleton in SSR can accidentally share state across requests.
