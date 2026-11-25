# Q&A: React quick reference and glossary

## Mental model and JSX

### 1. What is React responsible for?

React provides a component model and a way to calculate and update UI from current props, state, and context. React DOM connects that model to browser DOM nodes. Routing, data storage, a backend, styling choices, and deployment are usually supplied by other tools or application code.

### 2. What happens during a render?

React calls components to calculate the next UI description. A render may lead to DOM changes, but it may also produce the same UI and no visible DOM update. Render must stay pure because React may repeat or abandon render work.

### 3. What is the difference between render and commit?

Render calculates the next UI. Commit applies the accepted changes to the host environment, such as the browser DOM, and runs the related commit work.

### 4. Why should a component be pure?

Purity means the same inputs produce the same UI without observable side effects during rendering. It lets React safely repeat, prioritize, or abandon work and helps expose accidental mutation.

### 5. What is JSX?

JSX is JavaScript syntax for describing UI. A build tool transforms it; browsers do not execute JSX directly. JSX uses JavaScript expressions inside braces and React DOM conventions such as className and htmlFor.

### 6. Why do component names start with a capital letter?

In JSX, a capitalized tag such as Profile refers to a JavaScript component. A lowercase tag such as button refers to a built-in browser element.

### 7. What is a React node?

A React node is any child value React can render, including elements, text, numbers, arrays of nodes, portals, and empty values such as null.

### 8. Why does a component need one parent result?

A component can return one React node. To return multiple sibling elements, group them with a Fragment or an appropriate semantic element. A Fragment adds no wrapper to the DOM.

### 9. What does a key do?

A key identifies a child among its siblings so React can match it across insertion, removal, or reordering. Use stable data IDs when possible. A key is not passed to the child as a prop.

### 10. When is an array index a bad key?

When list membership or order can change. React can then associate a component's state with the wrong item. An index is acceptable only when the list is truly static and items will never be reordered or removed.

## State, props, and identity

### 11. How are props and state different?

Props are read-only inputs passed by a parent. State is data owned by a component or state system that can change over time and affect rendered UI.

### 12. Does calling a state setter change the current value immediately?

No. A setter queues an update and requests a future render. The running event handler still sees the state snapshot from the render that created it.

### 13. When should I use a functional state updater?

Use one when the next state depends on the previous queued state, such as incrementing a counter more than once in one event:

~~~jsx
setCount((current) => current + 1);
~~~

### 14. Should I mutate an object or array in state?

No. Create a replacement value, for example with object spread, map, or filter. Mutating an existing value can make updates difficult to detect and reason about.

### 15. What state should be stored?

Store the minimum information that changes over time and cannot be calculated from other current inputs. Calculate derived values during render instead of copying them into another state variable.

### 16. When should state move to a parent?

When multiple components need to coordinate the same state, move it to their closest common parent and pass data and callbacks through props. This is called lifting state up.

### 17. When is Context useful?

Context is useful when many descendants need a value and passing it through every intermediate component becomes noisy. It distributes a value; it does not decide who owns the state or replace every prop.

### 18. What determines whether React preserves component state?

React matches component identity using its type, position in the tree, and key. A changed key or type can create a new identity and reset that subtree's state.

### 19. How can a key intentionally reset a form?

Give the form component a key derived from the entity whose state it owns. When that entity changes, React creates a fresh component instance and resets its local state.

### 20. What is the difference between state and a ref?

State changes can trigger a render. A ref holds a mutable value between renders, and changing its current property does not trigger a render. Use refs for DOM nodes or non-visual mutable data, not for UI that must update.

## Events, forms, and Effects

### 21. Should I call an event handler during render?

No. Pass the function to the event prop. Calling it during render runs it immediately instead of waiting for the interaction.

### 22. When should I call preventDefault?

Only when the application intentionally replaces a browser default, such as preventing a form's normal navigation while handling its submission in JavaScript.

### 23. What is a controlled input?

A controlled input gets its current value from React props or state and reports edits through an event handler. Text fields use value/onChange; checkboxes use checked/onChange.

### 24. What is an uncontrolled input?

An uncontrolled input keeps its current value in the DOM. React may set its initial value with defaultValue or defaultChecked; a ref or form submission can read the eventual value.

### 25. What is an Effect for?

An Effect synchronizes a committed component with an external system, such as a subscription, timer, browser API, imperative widget, or network connection. It is not a general tool for calculating UI data.

### 26. What belongs in an Effect cleanup?

Undo the setup: unsubscribe, clear a timer, remove a listener, abort an owned request, or disconnect an observer. Cleanup runs before React repeats the Effect and when the component is removed.

### 27. Should I use an Effect to derive filtered data?

Usually no. Calculate the filtered data from current props/state during render. An Effect that copies derived data to state creates a second source of truth and an extra update.

### 28. What are Effect dependencies?

They are the reactive values read by the Effect. Include props, state, and component-body values the Effect uses. If dependencies cause unwanted synchronization, restructure the code rather than hiding a genuine dependency.

### 29. What is Strict Mode?

Strict Mode enables extra development-only checks intended to reveal issues such as impure rendering and missing Effect cleanup. Its diagnostic behavior is not extra production rendering.

### 30. How should a form expose validation errors?

Use an associated label, explain the error in text, connect help/error text to the control when useful, and move focus when the user's next step would otherwise be unclear. Use native validation where it fits.

## Hooks and built-in APIs

### 31. What are the Rules of Hooks?

Call Hooks only from function components or custom Hooks, and usually call them unconditionally at the top level so their order stays the same on every render. React's use API has specific conditional-call semantics; this exception does not apply to other Hooks.

### 32. What makes a function a custom Hook?

It is a JavaScript function that composes Hooks to reuse stateful logic and follows the Hook rules. Calling the same custom Hook twice does not automatically share its internal state between those calls.

### 33. When should I use useReducer instead of useState?

Use useReducer when state updates are easier to describe as actions handled by one pure reducer, especially for several related values or coordinated transitions. useState is simpler for straightforward independent values.

### 34. What is useContext for?

It reads and subscribes to the nearest matching provider above the component. If that provider value changes, consumers may render with the new value.

### 35. What is useId for?

It creates IDs that remain stable for a component and can connect labels, help text, and controls. It is not a data ID and should not be used as a list key.

### 36. What is useEffectEvent for?

It extracts event-like logic called by an Effect so that logic can read current props and state without making the Effect re-synchronize for those values. It must not be used to hide dependencies that genuinely control synchronization.

### 37. What is useSyncExternalStore for?

It connects a component to a store that exists outside React through a subscription and snapshot-reading contract. Prefer a store library's supported Hook when one is available.

### 38. What is useActionState for?

It manages the result and pending state of an Action. A form can use the returned action function as its action; the Action itself still needs to perform validation, persistence, or other work.

### 39. What is useFormStatus for?

It reads the pending status and related information for the nearest parent form. Call it in a descendant component inside that form, such as a submit button component.

### 40. What is useOptimistic for?

It lets an interface show a temporary optimistic value while an Action is pending. The authoritative result still needs to reconcile the UI with the server or data source.

### 41. What is use for?

It reads a supported resource such as a Promise or Context. A pending Promise can suspend to the nearest Suspense boundary. React 19.3 also uses use(browser()) for an intentional browser-only SSR bailout.

### 42. What does Suspense do?

A Suspense boundary displays a fallback while supported child work is not ready. It does not automatically detect every arbitrary Promise or data-fetching library; the source must integrate with Suspense.

### 43. How are Suspense and Error Boundaries different?

Suspense handles supported pending work by showing a fallback. An Error Boundary handles errors thrown while rendering descendants and can show recovery UI. They solve different states and are often used together.

## Scheduling and performance

### 44. What is a Transition?

A Transition marks an update as non-urgent. React can keep urgent interactions responsive while preparing the transitioned UI. It is not a timer, debounce, or promise that the update will finish in a particular duration.

### 45. What is the difference between useTransition and useDeferredValue?

useTransition lets code mark a state update and read its pending status. useDeferredValue lets a value used by part of the UI lag behind an urgent value. Neither API automatically debounces or cancels network requests.

### 46. When should I use memo, useMemo, or useCallback?

Use them after identifying unnecessary work that matters. memo can skip a component render when props compare equal; useMemo can cache a calculation; useCallback can cache a function identity. Code must remain correct if React recomputes or re-renders.

### 47. How should I investigate a slow screen?

Reproduce a user-visible interaction, capture a browser performance trace or React profiling data, identify where time is spent, and change the measured cause. Do not assume every slow interaction is caused by React rendering.

## DOM, accessibility, and security

### 48. What is a Portal?

A Portal renders a React subtree into a different DOM location, often for a dialog or overlay. The subtree remains part of the same React ownership tree and can still use context.

### 49. When should I use flushSync?

Only when an integration truly requires the DOM to reflect an update before the next line of code. It can hurt performance and interfere with scheduling; normal application updates rarely need it.

### 50. Does React sanitize HTML passed to dangerouslySetInnerHTML?

No. Treat untrusted content as text or sanitize it with a carefully selected sanitizer before HTML injection. Trusted Types helps the browser enforce typed values but does not sanitize a string.

### 51. Does React provide routing?

No. React describes UI. Use a router or framework for navigation and related route/data-loading conventions.

## SSR and React 19.3

### 52. What is SSR?

Server-side rendering generates HTML outside the browser so content can arrive before the client application is ready. A framework often manages the server rendering entry point.

### 53. What is hydration?

Hydration attaches React's client behavior to compatible server-generated HTML. The initial client render should match the server output; browser-only values read too early can cause mismatches.

### 54. What is a Server Component?

A Server Component renders in a separate server/build environment and is not sent as an interactive component implementation to the browser. It cannot use client-only state or Effects.

### 55. Does 'use server' mark a Server Component?

No. There is no directive for Server Components. 'use server' marks a Server Function in a compatible framework/runtime. 'use client' establishes a Client Component module boundary in an RSC framework.

### 56. What is the difference between a Server Component and a Server Function?

A Server Component describes UI rendered in the RSC environment. A Server Function is callable server-side logic invoked through the framework/runtime's supported mechanism.

### 57. What changed for Context in React 19.3 Server Components?

A Server Component can import a Context object from a Client Component module and render that Context directly as a provider. Server Components still cannot create Context themselves.

### 58. What does browser() do in React 19.3?

When a Client Component passes browser() to use, React suspends that component during server rendering. The nearest Suspense fallback appears in the server HTML; the component continues normally in the browser.

### 59. Does browser() require Suspense?

Yes, during server rendering. The call must also be inside a Client Component. Calling browser() without passing its result to use has no effect.

### 60. What changed for View Transitions in React 19.3?

ViewTransition became stable for React DOM. React coordinates the browser View Transition API for eligible React Transitions. Ordinary urgent state updates do not automatically animate.

### 61. What changed for Fragment refs in React 19.3?

An explicit Fragment can receive a ref whose value is a FragmentInstance. It exposes limited group DOM operations without inserting a wrapper. Most operations target first-level DOM children; focus methods search nested descendants.

### 62. Does React 19.3 Trusted Types support make HTML safe?

No. React passes Trusted Types values through without coercing them back to strings so the browser can validate them. The application must still sanitize untrusted input and configure its Trusted Types policy.

### 63. Which React 19 APIs replace removed React 18 DOM APIs?

Use createRoot instead of ReactDOM.render, hydrateRoot instead of ReactDOM.hydrate, refs instead of findDOMNode, and root.unmount() instead of unmountComponentAtNode. Use current react-dom/server APIs instead of removed legacy streaming methods.

## Quick troubleshooting

### 64. A component renders more times than I expected. Is that a bug?

Not necessarily. A render is React recalculating UI and does not guarantee a DOM mutation. Check whether the work affects user experience, keep render pure, and profile before adding memoization.

### 65. My state update appears to use an old value. Why?

The handler is reading the snapshot from its render. Use a functional updater when the new value depends on queued previous state, and avoid expecting a setter to mutate the current local variable.

### 66. My state resets after a list changes. What should I inspect?

Check keys, component types, and positions. Unstable keys, indexes in a changing list, or changing the component type can cause React to associate a different identity or create a fresh one.

### 67. My Effect runs too often. What should I inspect first?

Check its dependencies and whether it is doing work that belongs in render or an event handler. Do not suppress dependency warnings to force a timing pattern; model the synchronization the Effect actually owns.

### 68. My server-rendered page reports a hydration mismatch. What should I inspect?

Compare server output with the first client render. Look for browser-only APIs, current time, random values, locale/time-zone differences, invalid HTML nesting, and data that differs between environments.

### 69. An animation or API is missing in an older React project. What should I inspect?

Check the installed react and react-dom versions, the matching framework/runtime support, and the feature's release notes. The presence of an API in the latest docs does not mean an older installed version exports it.

## Final review

### 70. What should I remember after finishing the course?

Describe UI from current inputs, keep rendering pure, model state once, use semantic and accessible HTML, synchronize external systems with Effects, profile before optimizing, and check the framework/runtime boundary for server features. Use the roadmap and the root summary/glossary to return to the full lesson for any topic.
