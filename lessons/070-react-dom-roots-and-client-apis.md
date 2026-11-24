# React DOM Roots, Hydration, and Client APIs

React DOM connects a React tree to browser DOM nodes. In a client-rendered application, `createRoot` creates a root and `root.render` displays the tree. When a server or framework has already produced React HTML in the page, `hydrateRoot` attaches React to that existing markup instead.

The choice depends on what is already inside the target DOM node. Use a client root for an empty container that React should populate. Use hydration for HTML that was rendered from the same React tree on the server. Treat the root as application setup and lifecycle infrastructure; ordinary UI changes should come from component props, state, and context.

## Learning goals

After this lesson, you should be able to:

- import `createRoot` and `hydrateRoot` from `react-dom/client`;
- choose the correct API based on whether the container is empty or contains server-rendered React HTML;
- create a root once, render into it, and unmount it when its DOM container is being removed;
- explain why hydration requires the first client render to match the server output;
- diagnose common root and hydration mistakes;
- recognize the legacy root APIs removed in React 19.

## The client entry point

The client APIs live in the `react-dom/client` package entry point:

~~~jsx
import { createRoot, hydrateRoot } from "react-dom/client";
~~~

The two APIs have different jobs:

| Page contents at startup | API | What React does |
| --- | --- | --- |
| An empty DOM container for a client-rendered interface | `createRoot(container)` | Creates a root that can render React into the container |
| HTML previously rendered from the same React application | `hydrateRoot(container, <App />)` | Reuses that HTML and attaches React behavior |

Do not use `createRoot` on a server-rendered React container. Its initial render is a client render and clears the container's existing HTML. Do not use `hydrateRoot` for an empty client-only container; there is no server-rendered React markup to hydrate.

## Creating a client root

A typical client-rendered page provides an empty element for React:

~~~html
<div id="root"></div>
~~~

The entry module finds that element, creates the root once, and renders the application:

~~~jsx
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The React root element #root was not found.");
}

const root = createRoot(container);
root.render(<App />);
~~~

`createRoot` receives the actual DOM node, not the string "root" and not a React element. `root.render` receives the React content to display.

The root is normally created in the application entry module, outside of components. Creating a root during component rendering would repeatedly start separate root lifecycles instead of letting the existing tree update normally.

### Render updates through React

Once the root exists, components respond to changes in props, state, and context. For example, a click handler calls a state setter and React updates the affected UI:

~~~jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount((current) => current + 1)}>
      Count: {count}
    </button>
  );
}
~~~

The root does not need to be recreated for this update. Calling `root.render` again with a new tree is possible, but it is generally not the way to drive normal interactive changes. Use React's data flow within the mounted tree.

### What the initial render does

For a client root, the first `root.render` populates its container and clears any existing HTML inside that container. Keep content that should remain outside React as a sibling of the root element.

~~~html
<header>Site-wide navigation managed by the page</header>
<div id="root"></div>
<footer>Site-wide footer managed by the page</footer>
~~~

React owns the descendants it renders inside its root. Avoid having unrelated code manually edit those same descendants; external edits can conflict with React's view of the DOM.

## Hydrating server-rendered HTML

Server rendering can send useful HTML to the browser before the client JavaScript has loaded. A server rendering API or framework creates that HTML from the React application. The browser then calls `hydrateRoot` with the same initial React tree:

~~~jsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.jsx";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The server-rendered root element #root was not found.");
}

hydrateRoot(container, <App />);
~~~

Hydration is not a request to replace the HTML. React checks the existing output against the initial client render, reuses the matching DOM, and attaches event handling and React behavior. In production, a framework commonly coordinates server rendering, script loading, and hydration. A manually written HTML fragment can illustrate matching markup, but it is not itself a server-rendering setup.

### The first render must match

The server and browser must render the same initial content for the hydrated region. Differences can come from:

- rendering a different value on the server and first client pass, such as a random number or current time;
- branching on `window`, `document`, or a browser-only condition during rendering;
- locale-dependent formatting that produces different output;
- invalid HTML nesting that the browser repairs before React reads it;
- whitespace or markup differences between server and client templates;
- server data changing before the client receives the matching initial data.

A mismatch is a bug to investigate. React may report a recoverable hydration error or switch part of the tree to client rendering, but you should not depend on recovery to make inconsistent output correct. React does not guarantee that every mismatched attribute will be patched.

For values that truly depend on the browser, render the same initial value on the server and client, then read the browser-only value in an Effect after hydration:

~~~jsx
import { useEffect, useState } from "react";

function BrowserWidth() {
  const [width, setWidth] = useState(null);

  useEffect(() => {
    setWidth(window.innerWidth);
  }, []);

  return (
    <p>
      {width === null ? "Width not measured yet" : "Width: " + width + "px"}
    </p>
  );
}
~~~

This avoids reading `window` during server rendering and keeps the initial markup consistent. The post-hydration update may briefly show the initial text, so choose an initial state that remains understandable.

`suppressHydrationWarning` is a narrow escape hatch for an unavoidable one-level text or attribute difference. It does not fix the cause, suppresses only a limited warning, and does not tell React to patch the mismatch. Prefer deterministic initial output.

## Root lifecycle and ownership

A root represents React's ownership of one DOM container.

- Create the root once for that container.
- Render the top-level interface into it.
- Let React update the mounted tree through its normal state and props.
- Call `root.unmount()` when another system is removing the root container or when a separately mounted React island is being disposed.

Unmounting tells React to clean up the tree, including Effects and event handlers:

~~~jsx
const root = createRoot(container);
root.render(<Widget />);

// Later, when an external widget manager removes this container:
root.unmount();
~~~

After a root has been unmounted, do not render into it again. If the same DOM element will be used for a new React lifecycle, create a new root after the previous one has been fully disposed.

A standalone React page normally has one root. Multiple roots can be useful when adding isolated React islands to a page managed by another system, but the roots are independent: they do not share React context or component state. If two parts belong to one React application, keep them in one tree. If content needs a different physical DOM destination while remaining in the same tree, consider a portal.

## Root options and diagnostics

`createRoot` and `hydrateRoot` accept an optional options object. Current React versions provide error callbacks such as `onCaughtError`, `onUncaughtError`, and `onRecoverableError` for reporting errors to an application's diagnostics system.

~~~jsx
const root = createRoot(container, {
  onUncaughtError(error, errorInfo) {
    reportApplicationError(error, errorInfo);
  },
});

root.render(<App />);
~~~

Error callbacks should report enough context for diagnosis without exposing secrets or sensitive user data. A root-level callback does not replace an Error Boundary's user-facing fallback.

The `identifierPrefix` option provides a prefix for IDs generated with `useId`. It is useful when a document contains multiple independent roots. When hydrating, use the same prefix that was used while producing the server HTML, or generated IDs may not match.

Hydration also supports framework-specific or advanced options for restoring form state. Most applications should let their server-rendering framework configure this instead of inventing root options themselves.

## React 19 migration

React 19 removed the old root APIs `ReactDOM.render`, `ReactDOM.hydrate`, and `ReactDOM.unmountComponentAtNode`.

Use the client root APIs instead:

| Older API | Current API |
| --- | --- |
| `ReactDOM.render(<App />, container)` | `createRoot(container).render(<App />)` |
| `ReactDOM.hydrate(<App />, container)` | `hydrateRoot(container, <App />)` |
| `ReactDOM.unmountComponentAtNode(container)` | `root.unmount()` |

The modern client APIs should be imported from `react-dom/client`. Keep the root reference when later cleanup is needed.

## Common mistakes

- **Passing the wrong kind of value:** pass a DOM element to `createRoot` or `hydrateRoot`, not an element ID string.
- **Using the wrong API:** `createRoot` for existing React server HTML throws away that markup; `hydrateRoot` has no server output to reuse on an empty root.
- **Creating a root repeatedly:** create it once in the startup module, not in a component or on every state change.
- **Forgetting to render:** creating a client root alone does not display the app; call `root.render`.
- **Hydrating different content:** make the server output and first browser render deterministic and equivalent.
- **Rendering before hydration finishes:** do not call `root.render` on a hydrating root before its initial hydration has completed; doing so can force the content to switch to client rendering.
- **Removing the container without cleanup:** if a non-React system owns container removal, call `root.unmount()` first so React can clean up.
- **Treating a recoverable error as success:** inspect and correct hydration mismatches instead of ignoring reports.
- **Using a separate root for a modal or tooltip by default:** a portal usually keeps that UI in the same React tree while changing its DOM placement.

## Practical checklist

1. Is the root container empty, or does it hold HTML rendered from this React application?
2. Use `createRoot` for the empty client-rendered case and `hydrateRoot` for the server-rendered case.
3. Find the container before initializing and handle a missing element clearly.
4. Create the root once and render the initial application.
5. For hydration, make the server and initial client output match, including the initial data.
6. Keep normal updates inside the React tree; unmount only when the root's lifecycle is ending.
7. Investigate root and hydration errors instead of hiding them.

## Summary

`createRoot` starts a client-rendered React tree in a DOM container. `hydrateRoot` adopts existing HTML produced from the same React tree and attaches client behavior. Their first-render behavior is different, so choose based on the container's contents. Create a root once, update the UI through React, keep server and client output consistent during hydration, and unmount a root when its container is being disposed.
