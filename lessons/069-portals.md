# Portals and Rendering Outside the Parent DOM Node

A React portal lets a component render some children into a different DOM node while keeping those children in the same React tree. Portals are useful when the physical DOM location causes layout or stacking problems, such as a floating help panel inside an overflow-clipped card.

A portal changes **where the DOM nodes are placed**. It does not move the component to a different React parent. React context and React event propagation still follow the original React tree.

## Learning goals

After this lesson, you should be able to:

- create a portal with `createPortal` from `react-dom`;
- provide a DOM target that already exists;
- distinguish physical DOM ancestry from React tree ancestry;
- explain how context and React events behave through a portal;
- avoid unnecessary separate React roots;
- account for CSS inheritance, accessibility, and focus when choosing a portal target;
- render portals safely when the target is not available during server rendering.

## The portal API

The function takes children, a target DOM node, and an optional key:

~~~jsx
import { createPortal } from "react-dom";

function HelpTip({ target }) {
  return createPortal(
    <aside className="help-tip">Helpful information</aside>,
    target,
  );
}
~~~

The first argument can be any React-renderable content, such as an element, component, Fragment, string, or array. The second argument must be an existing DOM node, for example one returned by `document.getElementById`.

A portal is a React node. Return it from a component or include it in JSX just like other React elements.

## Why use a portal?

A portal is useful when a component must appear outside a DOM ancestor that clips, constrains, or stacks its children in an undesirable way.

Common examples include:

- modal dialogs that need to layer over the page;
- tooltips and menus that would otherwise be clipped by an ancestor with `overflow: hidden`;
- notifications that should appear in a global notification area;
- React content placed into a DOM node managed by a non-React widget.

A portal does not itself create a modal, trap keyboard focus, add an overlay, or guarantee the right `z-index`. It only changes the physical destination of its children. Layout and accessibility still need to be designed.

## Choose a target that exists

A common client-rendered app places a portal target beside its React root:

~~~html
<body>
  <div id="root"></div>
  <div id="portal-root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
~~~

The application can look up the target and use it:

~~~jsx
const portalTarget = document.getElementById("portal-root");

function App() {
  return portalTarget
    ? createPortal(<HelpPanel />, portalTarget)
    : null;
}
~~~

The target node must already exist when `createPortal` renders. Do not pass `null`, a string ID, or a React ref object where a DOM node is required.

For a client-only app, the target can be read from the document during rendering. In server-rendered applications, `document` is not available on the server. Resolve the target on the client, for example after mounting, and render the portal only after the node is available. Avoid reading `document` at module scope in code that is also evaluated on the server.

~~~jsx
import { useEffect, useState } from "react";
import { createPortal } from "react-dom";

function HelpPortal({ children }) {
  const [target, setTarget] = useState(null);

  useEffect(() => {
    setTarget(document.getElementById("portal-root"));
  }, []);

  if (target === null) {
    return null;
  }

  return createPortal(children, target);
}
~~~

The target should be stable. Passing a different DOM node on an update causes React to recreate the portal content.

## DOM tree and React tree are different

Consider a component rendered under a page shell that creates a portal into `#portal-root`:

~~~jsx
function PageShell() {
  return (
    <div onClick={() => console.log("PageShell received a React click")}>
      <h1>Dashboard</h1>
      {createPortal(
        <button type="button">Portal action</button>,
        document.getElementById("portal-root"),
      )}
    </div>
  );
}
~~~

In the browser's DOM, the button is inside `#portal-root`. In the React tree, it remains a child of `PageShell`.

That distinction has two important effects:

- **Context follows the React tree.** The portal content can read context provided by its React ancestors.
- **React events bubble through the React tree.** Clicking the portal button can invoke the `PageShell` React `onClick` handler even though the button is not a DOM descendant of that `div`.

If this bubbling causes unwanted parent behavior, stop propagation at the portal content or move the portal higher in the React tree.

## CSS follows the physical DOM

React context follows the React tree, but CSS inheritance and DOM selectors follow the physical DOM tree. A portal rendered beside `#root` does not inherit styles that only come from a class on `#root`, and selectors such as `.app-shell .help-tip` will not match when `.help-tip` is not a DOM descendant of `.app-shell`.

Place shared design tokens high enough in the DOM, use classes that match the portal's real location, or style the portal directly. Also consider stacking contexts and the target's own ancestors; a portal does not make every positioning or layering issue disappear.

## Portal target changes and component identity

React treats the portal's children as part of the rendering component's tree. Their state behaves like other component state when their React position and identity remain stable.

However, if the `domNode` argument changes, React recreates the portal content in the new target. This can unmount the old children and reset their local state. Keep the target stable when state should be preserved.

The optional `key` identifies the portal among other rendered children, just as keys identify elements in lists. Use a stable key when a list or collection contains multiple portals.

## Portals are not separate React roots

A portal is still part of the same React tree. It can share state, context, and React event relationships with its parent.

`createRoot` creates a separate React root. A separate root has a different React tree and does not automatically share the original root's context or state. Do not create an extra root just to render a modal or tooltip elsewhere in the DOM; use a portal when the UI is still one React application.

A portal can also place React content into a node owned by a server-rendered page or a third-party widget. Keep a clear ownership rule: React should manage the portal's child content, while the outside system should manage the target node itself.

## Portals and accessible dialogs

A portal is a common way to place dialog markup outside a clipping container, but it does not make a dialog accessible by itself. A modal needs a clear accessible name, appropriate dialog semantics, keyboard interaction, sensible initial focus, focus containment while modal, focus restoration when closed, and a way to dismiss it where appropriate.

Native HTML `<dialog>` opened with `showModal()` provides built-in modal behavior in modern browsers, but you still need to label it, manage its open/close state, and test the complete interaction. If implementing focus management manually, follow an established accessible dialog pattern or use a well-maintained component library. Do not assume that adding `aria-modal="true"` alone provides keyboard behavior.

For non-modal portals such as help panels or notifications, do not claim they are modal. Keep their controls reachable and choose suitable semantics for what the content does.

## Multiple portals and fragments

A component can return ordinary content together with one or more portals:

~~~jsx
function App() {
  return (
    <>
      <main>
        <h1>Page content</h1>
      </main>
      {createPortal(<Toast />, document.getElementById("toast-root"))}
      {createPortal(<HelpPanel />, document.getElementById("help-root"))}
    </>
  );
}
~~~

Each target must exist before its portal is rendered. Keep separate targets only when they serve a real layout or ownership purpose; a single shared portal root is often enough for small applications.

## Common mistakes

- Importing `createPortal` from `react` instead of `react-dom`.
- Passing a selector string or an unavailable target instead of a DOM node.
- Rendering to a target before that node exists.
- Reading `document` at module scope in code that also runs during server rendering.
- Assuming a portal changes React context or event ancestry.
- Assuming React event propagation follows the browser DOM tree.
- Forgetting that CSS inheritance and selectors do follow the physical DOM tree.
- Expecting a portal to automatically fix focus, stacking, or accessibility.
- Creating a separate React root for content that should share the current tree.
- Changing the target DOM node and unintentionally remounting portal children.
- Letting both React and an external widget independently manage the same child DOM nodes.

## Summary

- `createPortal(children, domNode, key?)` renders children into a different DOM node.
- The target node must exist, and changing it recreates the portal content.
- A portal changes physical DOM placement while remaining in the same React tree.
- Context and React event propagation follow the React tree; CSS inheritance and selectors follow the DOM tree.
- Use portals for UI that needs a different DOM placement, not as a replacement for a separate root or accessibility work.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="068-error-boundaries-and-recovery-ui.md">068 - Error Boundaries and Recovery UI</a></td><td><a href="../qnas/069-portals.md">Q&amp;A</a></td><td><a href="../examples/069-portals.md">Example</a></td><td><a href="070-react-dom-roots-and-client-apis.md">070 - React DOM Roots, Hydration, and Client APIs</a></td></tr>
  </tbody>
</table>
