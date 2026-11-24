# Effect Hooks and Synchronization — Q&A

## Understanding Effects

### 1. What problem does an Effect solve?

An Effect synchronizes a committed React component with an external system, such as a browser event listener, timer, network connection, or third-party widget. It lets React start, update, and stop that synchronization as the component's relevant values change.

### 2. What does “external system” mean?

It is something whose state or behavior is not owned by React rendering. Examples include the browser DOM outside React's managed updates, browser APIs, a WebSocket, a timer, a subscription, or a non-React widget.

### 3. Is an Effect a callback that always runs immediately after rendering?

No. React runs Effects after a render is committed, and timing relative to browser paint depends on the Hook and the kind of update. Use `useEffect` for most synchronization. Use `useLayoutEffect` only when work must happen before repaint.

### 4. How is an Effect different from a component lifecycle callback?

An Effect describes one synchronization process: setup, cleanup, and possibly setup again. It may re-synchronize several times while its component stays mounted. Thinking only in terms of “mount” and “unmount” misses dependency changes and repeated synchronization.

### 5. Where may I call an Effect Hook?

Call it at the top level of a function component or custom Hook. Do not call it inside a condition, loop, event handler, or nested function. React relies on Hooks being called in the same order on every render.

### 6. Can I call an Effect Hook from a class component?

No. Hooks are for function components and custom Hooks. Class components use their supported lifecycle APIs, or can be wrapped/refactored into function components.

## Setup, cleanup, and dependencies

### 7. What is Effect setup?

Setup is the function passed to an Effect Hook. It starts or configures the synchronization after React commits the component's UI.

### 8. What can setup return?

Setup can return nothing or a cleanup function. React uses that function to stop or undo the synchronization. Do not return a Promise from setup.

### 9. Why should an Effect callback not be declared async?

An async function always returns a Promise. React expects setup to return either a cleanup function or nothing, so the Promise cannot serve as cleanup. Declare an async function inside setup, call it there, and return cleanup synchronously.

### 10. When does React run cleanup?

React runs the previous cleanup before starting setup again for changed dependencies, and it runs the last cleanup when the component is removed. In development Strict Mode, React also performs an extra setup-and-cleanup check.

### 11. What should cleanup do?

Cleanup should stop or undo the work started by its corresponding setup: unsubscribe the same listener, clear the timer, disconnect the same connection, or abort the request. It should not perform unrelated work.

### 12. Does every Effect require cleanup?

No. Cleanup is needed when setup creates or starts work that should stop, be released, or be undone. A one-way synchronization that leaves no resource to release may not need cleanup.

### 13. What belongs in a dependency list?

Every reactive value read by setup or cleanup: props, state, and values or functions declared in the component body. React compares dependencies with `Object.is`.

### 14. May the dependency list have a variable length?

No. Keep its number and order of entries constant and write it inline. A dependency list whose structure changes between renders cannot be compared reliably.

### 15. What happens when a dependency changes?

React runs the previous cleanup using the previous render's values, then runs setup using the new render's values. The Effect now synchronizes with the new dependency values.

### 16. What does omitting the dependency argument mean?

The Effect runs after every committed render. This is rarely the desired behavior for an ongoing external synchronization, because it may repeatedly tear down and restart that synchronization.

### 17. What does an empty dependency array mean?

It means setup and cleanup read no reactive values that should cause this Effect to re-synchronize. It does not mean “ignore dependencies” or “guarantee that setup runs only once in every environment”; Strict Mode adds a development check.

### 18. Can I leave out a dependency to make an Effect run less often?

No. Omitting a reactive dependency can make the Effect use a stale value. Include the values the code reads, then restructure the code if the synchronization runs more often than intended. Do not silence the dependency linter to force a schedule.

### 19. Why do objects and functions often cause extra Effect runs?

Objects and functions created during a component render get new identities on later renders. Since React compares dependencies by identity using `Object.is`, an Effect that depends on one of those new values may re-run every time.

### 20. How can I avoid an unnecessary object dependency?

If the object is only used by the Effect, create it inside the Effect and list the primitive or stable values that define it. If a helper does not depend on component values, move it outside the component. Do not hide a real reactive dependency.

### 21. Why can a state updater remove a dependency?

When a state update only needs the previous state, pass an updater function such as `setCount(current => current + 1)`. The Effect no longer needs to close over the current count just to calculate its next value.

## Cleanup patterns and async work

### 22. Why must an event listener be removed with the same function?

The browser identifies a listener by its event type, function reference, and relevant options. Defining a new function during cleanup does not remove the function registered by setup. Keep the handler in setup's scope and pass that same reference to removal.

### 23. How should an Effect manage a timer?

Create the timer in setup, store its ID in the setup's local scope, and return cleanup that clears that ID. If the timer updates state based on its previous value, use a functional updater so the Effect does not capture a stale value.

### 24. How can an Effect avoid showing a response for an old search query?

When the query changes, cleanup can abort the request with an `AbortController` or mark its result as obsolete. Handle an abort separately from genuine request errors. Framework or query-library data APIs may provide additional caching and race handling.

### 25. Why should an Effect not directly return the result of an async function?

An async function returns a Promise, not a cleanup function. React cannot use that Promise to unsubscribe, abort, or release resources. Call a nested async function and return cleanup from the outer synchronous setup.

### 26. Does using an Effect for data fetching provide a complete data layer?

No. A basic Effect does not automatically provide caching, request deduplication, retries, preloading, or server rendering. Use framework data APIs or a query library when those capabilities are needed.

## When not to use an Effect

### 27. Should an Effect calculate derived state?

Usually no. If a value can be calculated from current props and state, calculate it during render. Copying it into state with an Effect creates an extra render and can leave duplicate state out of sync.

### 28. Should an Effect respond to a button click?

Usually no. Put the work caused by a known user action in that event handler. A state flag plus an Effect hides the cause of the action and can make the code harder to follow.

### 29. When is memoization appropriate for a derived value?

When the calculation is measurably expensive and avoiding its recomputation matters. Memoization is a performance optimization; it is not needed to make a derived value correct.

### 30. What question helps decide whether an Effect is appropriate?

Ask: “What external system is this Effect synchronizing with?” If there is no clear system, the logic likely belongs in rendering, an event handler, or a state transition instead.

## Choosing the Effect Hook

### 31. When should I use `useEffect`?

Use it for most synchronization with external systems when an update after the commit is suitable. It is the default choice for subscriptions, connections, timers, and similar work.

### 32. When should I use `useLayoutEffect`?

Use it when you must read layout or make a DOM-related correction after React commits changes but before the browser paints, such as positioning a tooltip without showing a misplaced frame. It blocks painting, so prefer `useEffect` otherwise.

### 33. Can `useLayoutEffect` measure server-rendered HTML?

No. It only runs in the browser, where layout exists. Server-rendered output must be useful before layout measurement; alternatively, defer the measurement-dependent UI until the client has mounted.

### 34. What is `useInsertionEffect` for?

It is a specialized Hook mainly for CSS-in-JS library authors who need to insert style rules before layout Effects execute. Ordinary components should generally use CSS stylesheets, extracted CSS, or normal style props.

### 35. Can I use `useInsertionEffect` to read refs or measure layout?

No. Refs are not attached at its timing, and the DOM may not yet represent the final committed updates. It is not a layout-reading Hook.

### 36. Can `useInsertionEffect` update state?

No. State updates are not allowed from this Hook. It has a narrow style-insertion purpose, not general application logic.

## Strict Mode and server rendering

### 37. Why does Strict Mode appear to run an Effect twice in development?

When Strict Mode is enabled, React performs an extra setup-and-cleanup cycle to check that cleanup mirrors setup. This is a development-only check, not an extra production subscription.

### 38. Should I use a ref to prevent Strict Mode's second setup?

No. That hides incomplete cleanup instead of fixing it. Make setup and cleanup safe to repeat. The same correctness is needed when dependencies change or a component is removed and later mounted again.

### 39. Do Effects run during server rendering?

No. Effects run only on the client. Server-rendered HTML cannot depend on an Effect having already run.

### 40. Why should browser globals not be read carelessly during render?

There is no `window` or `document` in a server runtime. Reading them while rendering can crash a server-rendered application. Keep browser-only work in client code and Effects, and make the initial server-rendered UI safe.

## Troubleshooting and review

### 41. An Effect keeps reconnecting. What should I inspect?

Check whether a dependency changes on every render, especially an object or function created in the component. Also confirm that all dependencies are necessary and that effect-only values can be created inside setup. Do not fix a reconnect loop by omitting a real dependency.

### 42. The Effect uses an old prop or state value. What is the likely cause?

A reactive value read by setup or cleanup is missing from the dependency list, or the code needs a different structure such as a state updater. Let the dependency linter identify stale closure risks.

### 43. The component updates twice after adding an Effect. Is that always a bug?

Not necessarily. In development Strict Mode, the extra setup-and-cleanup check is intentional. Also, an Effect state update itself causes another render. Consider whether the state is derived and can instead be calculated during render.

### 44. How do I review an Effect before shipping it?

Identify the external system, verify that setup starts the intended synchronization, verify that cleanup stops exactly that work, check every reactive dependency, and confirm that derived state or event handling has not been moved into an unnecessary Effect. Check the behavior in Strict Mode and make sure server-rendered output does not depend on client Effects.

## Quick comparison

| Hook | Typical purpose | Timing and main caution |
| --- | --- | --- |
| `useEffect` | General external-system synchronization | After commit; usually does not block paint. |
| `useLayoutEffect` | Pre-paint measurement or visual correction | After DOM commit and before repaint; blocks painting. |
| `useInsertionEffect` | Insert styles from a CSS-in-JS library | Before layout Effects; refs and layout are unavailable, and state updates are not allowed. |

## Key points

- Effects are for external synchronization, not arbitrary post-render code.
- Setup starts synchronization; cleanup stops or reverses that same work.
- Dependencies come from the reactive values setup and cleanup read.
- React compares dependencies using `Object.is`.
- Strict Mode's extra development cycle checks that setup and cleanup are repeatable.
- Derive values during render and handle known user actions in event handlers.
- Choose `useEffect` by default; use the other two Hooks only for their specialized timing needs.
- Effects do not run during server rendering.
