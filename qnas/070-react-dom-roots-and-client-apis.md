# React DOM Roots, Hydration, and Client APIs — Q&A

## Core concepts

### 1. What is a React root?

A root is the connection between a React application tree and a DOM container. The root is created at the application boundary; React manages the descendants it renders inside that container.

### 2. Where do `createRoot` and `hydrateRoot` come from?

Import both client APIs from `react-dom/client`. They are not imported from `react`.

~~~jsx
import { createRoot, hydrateRoot } from "react-dom/client";
~~~

### 3. When should I use `createRoot`?

Use it when the target container is empty and React should render the interface into it on the client.

### 4. When should I use `hydrateRoot`?

Use it when the container already contains HTML rendered from the same React application on the server. Hydration reuses that HTML and attaches React behavior.

### 5. What is the key difference between creating and hydrating a root?

`createRoot` starts a client render. `hydrateRoot` expects matching server-rendered HTML to already be present and adopts it.

### 6. Can I hydrate an empty container?

No. There is no server-rendered React markup to reuse. Use `createRoot` for an empty client-only container.

### 7. Can I call `createRoot` on server-rendered markup?

You should not. The initial client render clears the container's existing HTML. Use `hydrateRoot` for HTML rendered from React on the server.

### 8. Does React create the DOM container?

No. The page or framework provides a DOM node, commonly a `<div id="root">` element. The client code looks it up and passes the actual node to the root API.

### 9. Does `createRoot("root")` work?

No. The API expects a DOM node, not a string ID. Look up the element first, for example with `document.getElementById("root")`.

### 10. What does `root.render(<App />)` do?

It tells a client root what React content to display in its container. Creating the root alone does not render the application.

### 11. Where should root initialization usually happen?

In the client entry module, once per root container. It should not happen inside a component render or on every UI update.

### 12. Should I recreate the root when component state changes?

No. State setters, props, and context update the mounted tree. The root is infrastructure for mounting and disposing of the application, not the usual mechanism for interactive updates.

## Hydration behavior

### 13. What does hydration mean?

Hydration is React's process of reusing server-rendered HTML and attaching client-side React behavior, such as event handling, to the existing DOM.

### 14. Does `hydrateRoot` immediately replace the server HTML?

Its purpose is to adopt matching markup rather than replace it with a fresh client render. React expects the initial client output to match the existing server output.

### 15. Why must the server and first client render match?

React needs to associate the React tree with the correct existing DOM. A mismatch can cause hydration warnings, recoverable errors, or client rendering of affected content; recovery does not make inconsistent markup correct.

### 16. What commonly causes a hydration mismatch?

Different initial data, current-time or random values, locale-dependent formatting, browser-only branches, invalid HTML nesting, or differences in server and client templates can produce different output.

### 17. Can I read `window` while rendering on the server?

No. `window` and `document` are browser globals and are unavailable during server rendering. For browser-dependent values, render a consistent initial value and read the browser value in an Effect after hydration.

### 18. What is a safe way to show browser-only information after hydration?

Use the same initial content on server and client, then update state in `useEffect`. This keeps the hydration pass consistent, though the initial content may appear briefly.

### 19. What does `suppressHydrationWarning` do?

It suppresses a warning for an unavoidable difference at one level of an element's content or attributes. It is a narrow escape hatch; it does not fix the mismatch or guarantee that React will patch it.

### 20. Should I ignore a recoverable hydration error?

No. Treat it as a signal to investigate the server and client output. Error recovery is not a substitute for deterministic markup.

### 21. Can I call `root.render` immediately after `hydrateRoot`?

Do not use `root.render` to replace the content before hydration completes. Doing so can cause the server HTML to be discarded and the tree to switch to client rendering. Let hydration finish and update through normal React state and props.

### 22. Can a plain HTML mockup prove hydration works?

It can demonstrate how matching markup should look, but hand-written HTML is not proof of server rendering. A real hydration setup needs HTML generated from the same React tree by a server-rendering API or framework.

## Root lifecycle and composition

### 23. What does `root.unmount()` do?

It unmounts the React tree from its container and lets React clean up Effects and event handlers. It is useful when a non-React system is removing the root container.

### 24. Should I call `root.unmount()` after every render?

No. Unmount only when that root's lifecycle is ending or its container is being disposed. It is not part of ordinary state updates.

### 25. Can I reuse the same root object after unmounting it?

No. A root is no longer usable after it has been unmounted. If the container is being used for a new React lifecycle, dispose the old root first and create a new one.

### 26. How many roots does a normal React application need?

A standalone React application usually has one root. Multiple roots may be appropriate for isolated React islands embedded in a page owned by another system.

### 27. Do independent roots share context or React state?

No. Roots are independent React trees. Context and component state do not cross from one root into another.

### 28. Should a modal usually use a separate root?

Usually not. A portal can render a modal in another DOM location while keeping it in the same React tree, preserving context and React event relationships.

### 29. What if another library edits DOM nodes inside my React root?

Avoid competing ownership. React expects to manage the descendants it renders; external edits can conflict with its DOM updates. Coordinate ownership or isolate the other system in its own container.

## Options, errors, and migration

### 30. What are root error callbacks for?

Callbacks such as `onCaughtError`, `onUncaughtError`, and `onRecoverableError` let an application report errors to diagnostics. They do not replace user-facing error handling or Error Boundaries.

### 31. What is `identifierPrefix` for?

It prefixes IDs generated by `useId`. It can help avoid collisions when a document contains multiple roots. The server and client must use the same prefix for a hydrated root.

### 32. What should error reporting avoid?

Do not expose passwords, tokens, personal data, or other sensitive details in logs or user-facing error messages. Report enough safe context to diagnose the failure.

### 33. Which old root APIs were removed in React 19?

The legacy `ReactDOM.render`, `ReactDOM.hydrate`, and `ReactDOM.unmountComponentAtNode` APIs were removed.

### 34. What replaces `ReactDOM.render`?

Use `createRoot(container)` and then call `root.render(<App />)`.

### 35. What replaces `ReactDOM.hydrate`?

Use `hydrateRoot(container, <App />)`. It creates and initializes the hydration root in one call.

### 36. What replaces `ReactDOM.unmountComponentAtNode`?

Keep the root reference and call `root.unmount()`.

### 37. Where do I import modern root APIs?

Import them from `react-dom/client`, including when migrating a React 19 application.

## Troubleshooting

### 38. Why is my page blank after calling `createRoot`?

Creating a root only prepares the container. Make sure the code calls `root.render(<App />)` and that the entry script runs after the container exists.

### 39. Why is my container lookup returning `null`?

The element may be missing, its ID may be misspelled, or the script may execute before the HTML has been parsed. Ensure the container exists before initializing React and handle the missing-element case clearly.

### 40. Why did existing HTML disappear?

A client root's first render clears existing content in the container. If that content was rendered by React on the server, use hydration instead.

### 41. Why did the browser show static content but clicking does nothing?

Server-rendered HTML can appear before client JavaScript loads. Event handlers and interactive React behavior become active when hydration succeeds; confirm the client entry loads and the initial markup matches.

### 42. Why do matching-looking pages still warn about hydration?

The mismatch can be subtle: whitespace, invalid nesting repaired by the browser, different data, or locale-specific output may differ even when the page looks similar. Compare the actual first server and client output.

### 43. Why should I keep the root reference?

Keeping it allows explicit lifecycle cleanup with `root.unmount()` and makes the root boundary clear. It also avoids trying to locate or recreate a root during teardown.

### 44. When is `flushSync` relevant?

`flushSync` is a separate, uncommon escape hatch for forcing an update to flush synchronously, usually to integrate with an external browser or third-party API. It is not needed to create or hydrate a root and should not be used for routine rendering.

### 45. What is the simplest decision rule?

If React should create the UI in an empty container, use `createRoot`. If React already rendered that UI to HTML on the server, use `hydrateRoot` with the same initial tree and data.
