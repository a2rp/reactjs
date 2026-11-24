# ReactJS Summary

A compact guide to React's core ideas, practical APIs, and application architecture. Use this for review after working through the lessons; use the [ReactJS keywords and concepts](reactjs-keywords.md) file when you need a short definition of a particular term.

## 1. What React is

React is a JavaScript library for building user interfaces from components. You describe the UI for the current inputs and state. React calculates updates and applies the necessary changes to its rendering target.

React is one part of an application stack:

- **React** provides components, elements, state, Hooks, transitions, and related UI APIs.
- **React DOM** renders React into browser DOM and provides browser client and server rendering APIs.
- **Build tools** transform JSX, bundle modules, and serve or produce browser-ready files.
- **Frameworks and libraries** may add routing, data loading, server rendering, Server Components, caching, styling, testing, and deployment conventions. Those choices are not all provided by React itself.

React is declarative: describe what the interface should look like for a given state, then update the state in response to user actions. Avoid manually changing DOM nodes to keep them in sync with React-rendered UI.

## 2. The component model

A component is a reusable function that returns React nodes. Components compose into a tree.

```jsx
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}

function App() {
  return <Greeting name="Ashish" />;
}
```

Important component rules:

- Component names start with a capital letter; lowercase JSX names refer to built-in HTML elements.
- Props are read-only inputs from a parent. Use callbacks to request that a parent change its state.
- `children` contains nested content between a component's JSX tags.
- Split components around useful responsibilities and reusable behavior. Avoid both giant components and unnecessary abstraction.
- Prefer composition: let a component accept children or another component when callers need to customize its contents.
- A component should return the same UI for the same inputs and avoid side effects during rendering.

### JSX

JSX is JavaScript syntax for describing UI. A build tool transforms it before a browser runs the code.

- Use one parent node or a Fragment for adjacent siblings.
- Put JavaScript expressions inside braces: `<p>{user.name}</p>`.
- Use `className` for a DOM class and `htmlFor` for a label association.
- Use `style={{ color: 'tomato' }}` for a JavaScript style object; property names are camel-cased.
- Escape untrusted text by rendering it as text. Treat raw HTML injection as a security-sensitive escape hatch.
- Use `<Fragment>` instead of adding a wrapper element when the DOM should remain unwrapped. Explicit Fragments can have keys; the short `<></>` form cannot.

### Lists and keys

Render collections from data, usually with `map`. Give each direct list child a stable key from the data, such as a database ID. A key lets React match items after insertion, deletion, or reordering. Do not use array indexes for lists that can change order or membership, and do not generate a new random key on each render. A key is not passed to the component as a prop; pass the needed ID separately.

### Conditional UI

Use ordinary JavaScript conditions to choose UI: `if`, a ternary, or `&&`. Ensure the expression produces the intended output; for example, a numeric value of zero on the left side of `&&` renders as zero.

## 3. Rendering and component identity

A simplified update has two phases:

1. **Render:** React calls components to calculate what the UI should be.
2. **Commit:** React applies the accepted changes to the DOM or another rendering target.

A render does not necessarily change the DOM. React may render again because state, context, or a parent changed, then find that the visible output is the same.

Rendering must be pure: do not mutate existing props or state, issue network requests, subscribe to external systems, or change the DOM while calculating JSX. React may repeat or abandon render work. Put interaction-specific work in event handlers and external synchronization in Effects.

React preserves state according to a component's type, position in the tree, and key. Changing a key can intentionally reset a subtree's state. A stable component type and key allow React to preserve state through ordinary updates.

Strict Mode adds development checks that can reveal impure rendering and missing Effect cleanup. Extra development calls are diagnostic; they are not production behavior.

## 4. Events, forms, and accessibility

Attach event handlers using JSX event props, such as `onClick`, `onChange`, and `onSubmit`. Pass a function rather than calling it during render.

```jsx
function SaveButton({ onSave }) {
  return <button type="button" onClick={onSave}>Save</button>;
}
```

When handling an event:

- Read the event's relevant values while they are available.
- Call `event.preventDefault()` only when intentionally replacing a browser default, such as form navigation.
- Use the correct semantic element. A `<button>` already supports keyboard activation; a clickable `<div>` does not.
- Keep user feedback visible: pending status, success, and errors should be understandable and accessible.

A **controlled input** gets its current value from React state and updates that state in `onChange`. An **uncontrolled input** keeps its current value in the DOM and can use `defaultValue`, `defaultChecked`, or a ref. Choose one model deliberately for a given field.

Use HTML forms and native validation where appropriate. Associate labels with controls, identify errors in text, connect help and error messages to fields, and manage focus when users need help finding a problem. Prefer semantic HTML; add ARIA only when native elements do not express the needed role, name, or state.

## 5. State and data flow

State is information that changes over time and affects rendered UI. Props are inputs from a parent; state is owned by a component or a state system.

Choose the **minimal state** needed to represent the changing data:

- Do not store values that can be calculated from existing props or state.
- Avoid duplicated copies of the same fact.
- Keep state close to the components that use it.
- If multiple components must coordinate, move the state to their closest common parent and pass values and event handlers down. This is called lifting state up.
- Use context when a value is needed across many levels, such as a theme or current user. Context distributes a value; it does not replace thoughtful state ownership.

A state value is a snapshot for one render. Calling a setter requests another render; it does not change the value captured by the currently running handler.

```jsx
const [count, setCount] = useState(0);

function increment() {
  setCount(current => current + 1);
}
```

React queues and batches updates. Use a functional updater when the next value depends on the previous queued value. Treat objects and arrays in state as immutable: create a replacement with the intended changes rather than mutating the existing value.

Use `useReducer` when related updates are easier to describe as actions handled by one pure reducer. Use an external state store only when the app's sharing or update needs justify that extra system.

## 6. Hooks

Hooks give function components access to React features. Call Hooks at the top level of function components or custom Hooks, in the same order on every render. Do not call them inside conditions, loops, nested functions, or event handlers. The `use` API is a documented exception that can read a Promise or context conditionally.

| Hook or API | Main purpose |
| --- | --- |
| `useState` | Store component state. |
| `useReducer` | Store state updated through a reducer and actions. |
| `useContext` | Read a value from the nearest matching context provider. |
| `useRef` | Keep a mutable value between renders or access a DOM node without triggering a render when it changes. |
| `useEffect` | Synchronize with an external system after commit. |
| `useLayoutEffect` | Read or change layout after DOM updates and before paint; use sparingly. |
| `useInsertionEffect` | Insert styles before layout Effects, mainly for CSS-in-JS library authors. |
| `useMemo` | Cache a calculation as a performance optimization. |
| `useCallback` | Cache a function identity when a consumer's optimization or API needs it. |
| `useImperativeHandle` | Expose a limited imperative API through a ref. |
| `useId` | Generate IDs for accessibility relationships; not for list keys. |
| `useSyncExternalStore` | Subscribe to an external store using a consistent snapshot contract. |
| `useDebugValue` | Add a label for a custom Hook in React Developer Tools. |
| `useTransition` | Mark non-urgent state updates and read whether the transition is pending. |
| `useDeferredValue` | Let non-critical UI use a deferred value while urgent UI updates first. |
| `use` | Read a Promise or context during render; a pending Promise can suspend to a Suspense boundary. |
| `useActionState` | Connect an Action to previous result state and a pending flag. |
| `useFormStatus` | Read the submission status of a parent form from a descendant. |
| `useOptimistic` | Show temporary optimistic state while an Action is pending. |
| `useEffectEvent` | Separate event-like logic called from an Effect from the Effect's reactive synchronization. **React 19.2.** |

A **custom Hook** is a function beginning with `use` that composes other Hooks to reuse logic. Calling a custom Hook in two components does not automatically share state; each call has its own Hook state unless it connects to context or an external store.

Memoization Hooks are performance tools, not ways to make incorrect code correct. Start with clear code and measure before optimizing. When React Compiler is enabled and configured, it can automatically optimize supported code; it does not remove the need for pure render logic.

## 7. Effects and refs

An Effect is for synchronizing a committed component with something outside React: a subscription, browser API, timer, network connection, or imperative third-party widget. It has setup and, when needed, cleanup.

```jsx
useEffect(() => {
  const connection = connect(roomId);
  connection.connect();

  return () => connection.disconnect();
}, [roomId]);
```

The dependency list contains every reactive value the Effect reads. When a dependency changes, React cleans up the old synchronization and starts the new one. The Hooks linter helps find missing dependencies.

Before adding an Effect, ask:

- Can this value be calculated during render instead?
- Is this work caused by a particular interaction? If so, put it in the event handler.
- Is there an external system that needs to stay synchronized? If so, an Effect may fit.
- Does setup have a matching cleanup?
- Can the code tolerate setup and cleanup being repeated during development checks?

A ref stores a mutable value in `.current` without scheduling a render. Use refs for DOM access or values that are not themselves rendered UI. Prefer props and state for information that should appear on screen. Use imperative refs sparingly and expose a small, intentional API.

## 8. Async UI, Suspense, and Actions

**Suspense** displays a fallback while supported child work is not ready, such as a lazy-loaded component or a Promise read through a supported React data integration. It is a boundary, not a general-purpose fetch API. Frameworks can connect their data loading to Suspense.

**Transitions** mark non-urgent updates so React can keep the interface responsive while preparing the new UI. Keep direct text input updates urgent; use a transition for a slower dependent view such as a large result panel.

React Actions integrate asynchronous work with pending states and forms:

- A form can use an `action` function to process submission.
- `useActionState` can hold the last result and expose pending status.
- `useFormStatus` lets a descendant submit button show the parent form's pending status.
- `useOptimistic` can display an expected result while an Action is pending.

For async work, represent loading, success, and error states deliberately. Handle failure, cancellation or outdated responses where relevant, and avoid allowing an older request to overwrite a newer user choice.

An Error Boundary can display fallback UI for rendering errors below it. It does not catch every error source, such as errors from ordinary event handlers. Use a boundary and recovery path that make sense for the feature.

## 9. Browser rendering and server architecture

A client-rendered app uses `createRoot` to mount React into a browser container. Server-generated HTML can be attached to with `hydrateRoot`; the first client render must agree with the server output to avoid hydration mismatches.

Server rendering produces HTML before client code runs. Streaming can send available parts while slower content is pending. These are rendering strategies and can be supplied by a framework.

**React Server Components (RSC)** describe where components execute in a server/client architecture. They are distinct from server-side rendering. RSC requires compatible framework or bundler integration:

- Server Components run in the server environment and can access server-side data. They do not use client-only state or Effects.
- Client Components are marked at a client module boundary in an RSC setup and can use interactions, state, and browser APIs.
- Values crossing the boundary must follow the framework's serialization rules.
- Server Functions and Server Actions require framework support. Validate authorization and input on the server; a directive does not make an endpoint secure by itself.

Use the application's framework and router to understand routes, data loading, caching, navigation, and deployment. React does not prescribe one universal router or backend.

### Version-aware React features

The roadmap uses stable React 19.3 as of November 24, 2026. Check the version and the framework integration when applying version-specific APIs.

- **React 19.2:** `<Activity>` can hide a subtree while preserving its state; `useEffectEvent` separates Effect event logic from synchronization.
- **React 19.3:** `<ViewTransition>` coordinates eligible React transitions with browser view animations; Fragment refs expose group operations on DOM children without adding a wrapper; Trusted Types values are passed through for browser validation.
- These APIs do not eliminate the need to check browser support, accessibility preferences, or the framework's support.

## 10. Portals, resources, and DOM access

React DOM provides APIs for common browser rendering needs:

- `createPortal` renders UI into another DOM node, for example an overlay, while preserving its place in the React tree.
- `createRoot` creates a client root; `hydrateRoot` attaches to server-generated markup.
- `flushSync` forces an update to flush immediately. It is an uncommon escape hatch and may hurt responsiveness.
- Refs give controlled access to DOM nodes for focus, measurement, and browser APIs.

React can coordinate resources such as stylesheets, images, fonts, and scripts with rendering in supported cases. Avoid manipulating React-owned DOM directly; use the DOM APIs only for a clearly bounded imperative need.

## 11. Styling, routing, and data choices

React does not require a styling system, router, or server data library. Choose tools based on project needs and keep their responsibilities clear:

- CSS files, CSS Modules, utility classes, and styling libraries are different styling approaches.
- A router maps URLs and navigation to UI; choose one suitable for the app or use the router included by its framework.
- Data fetching may live in a framework loader, a server component, a query/cache library, or a focused client Effect for simple external synchronization.
- Context is useful for broadly needed values, but passing props or composing components is often simpler for local needs.
- Keep server-only credentials and privileged operations on the server. Never treat values bundled into browser JavaScript as secret.

## 12. Testing, performance, and reliability

Test behavior from the user's point of view:

- **Unit tests** cover small logic or component behavior.
- **Integration tests** cover interactions among components, forms, and data flows.
- **End-to-end tests** cover complete user journeys in a browser-like environment.

Prefer queries based on accessible names and visible behavior. Include keyboard use, loading and error states, and important boundary conditions. Testing libraries and frameworks are ecosystem tools rather than React core.

For performance, first identify measured bottlenecks. Then consider smaller bundles, code splitting, list virtualization for very large collections, memoization, transitions, and avoiding unnecessary state or Effects. Use React Developer Tools and browser profiling to understand the cause before optimizing.

Reliability also depends on stable keys, correct Effect cleanup, error handling, accessible feedback, secure treatment of user input, and avoiding duplicated or stale state.

## 13. A practical problem-solving workflow

When building a feature:

1. Sketch the visible states and the user interactions that change them.
2. Split the design into components with clear responsibilities.
3. Identify which values are props, minimal state, or derived values.
4. Decide where each state value belongs and how updates flow.
5. Use semantic HTML, accessible names, and keyboard-friendly controls.
6. Add event handlers for user actions; add Effects only for external synchronization.
7. Handle pending, empty, success, and error states.
8. Test the behavior and important edge cases.
9. Measure performance before adding optimizations.
10. Check server/client boundaries and security before deployment.

## 14. Small example: searchable list

This example combines props, local state, a controlled input, derived data, and stable list keys:

```jsx
import { useState } from 'react';

function SearchableList({ items }) {
  const [query, setQuery] = useState('');

  const visibleItems = items.filter(item =>
    item.name.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <section>
      <label>
        Filter items
        <input
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
      </label>

      {visibleItems.length === 0 ? (
        <p>No matching items.</p>
      ) : (
        <ul>
          {visibleItems.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
    </section>
  );
}
```

The filtered list is calculated from props and state rather than copied into another state variable. React updates the UI when the query changes, and each item has a stable key.

## 15. Key ideas to remember

- Describe the UI from props and state; let React manage the DOM updates.
- Keep rendering pure and make state changes in response to events or Actions.
- Store the minimum changing information; calculate the rest.
- Give each state value a clear owner and use stable keys for changing lists.
- Follow the Rules of Hooks; use Effects to synchronize with external systems.
- Treat memoization as an optimization and measure before adding it.
- Use semantic HTML and accessible interaction patterns.
- Distinguish React APIs from browser APIs, frameworks, and third-party libraries.
- Understand whether code runs on the server, in the browser, or across an RSC boundary.
- Keep framework-specific server operations secure with server-side validation and authorization.
