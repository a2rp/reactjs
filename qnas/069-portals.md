# Portals and Rendering Outside the Parent DOM Node - Q&A

## Portal basics

### 1. What is a React portal?

A portal renders React children into a DOM node outside the DOM position where that component's surrounding JSX would normally appear.

### 2. Which function creates a portal?

`createPortal` from `react-dom`.

### 3. What arguments does `createPortal` accept?

It accepts the children to render, an existing target DOM node, and an optional key.

### 4. Does `createPortal` return a DOM element?

No. It returns a React node that can be included in a component's render output.

### 5. What can the children argument contain?

Anything React can render, including JSX, a component, a Fragment, strings, numbers, or arrays of renderable nodes.

### 6. Is a portal a new React root?

No. It remains in the same React tree as the component that created it.

### 7. What package should I import `createPortal` from?

Import it from `react-dom`, not from `react`.

### 8. What is a common reason to use a portal?

To render UI such as a dialog, tooltip, menu, or notification outside a DOM ancestor that clips or constrains it.

## Target DOM node

### 9. What should the second argument be?

An existing DOM node, such as the element returned by `document.getElementById`.

### 10. Can I pass the target's ID as a string?

No. Pass the actual DOM node, not a selector or string ID.

### 11. What happens if the target does not exist?

There is no valid DOM node to render into. Wait until the target exists or render nothing until it becomes available.

### 12. Must the target exist before the portal renders?

Yes. The target must already exist when React renders the portal.

### 13. What happens if I pass a different target node on an update?

React recreates the portal content in the new target. Children may unmount and remount, which can reset their local state.

### 14. Should the portal target change often?

Usually not. Keep the target stable unless you intentionally want the portal content recreated.

### 15. How can a client-only app provide a shared target?

Add an empty element such as `<div id="portal-root"></div>` beside the main root in the page HTML.

### 16. Can I render several portals into one target?

Yes. Multiple portal contents can be placed into a shared target. Use separate targets only when they serve a real layout or ownership purpose.

## React tree and DOM tree

### 17. What does a portal change?

It changes the physical DOM placement of the portal children.

### 18. Does a portal change the React parent of its children?

No. They remain children of the component that renders the portal in the React tree.

### 19. Do portal children receive context from their React ancestors?

Yes. Context follows the React tree, so portal children can read providers above the component that creates the portal.

### 20. How do React events bubble from a portal?

They bubble according to the React tree, not the physical DOM tree. A React handler on an ancestor component can run after an event in its portaled child.

### 21. Can a portal click call an ancestor React `onClick` outside the target DOM node?

Yes. If that component is an ancestor in the React tree, its React handler can receive the event.

### 22. How can I prevent a portal event from reaching a React ancestor?

Call `event.stopPropagation()` in an appropriate handler inside the portal, or move the portal higher in the React tree.

### 23. Do native DOM event relationships also follow the React tree?

No. Native DOM event propagation and DOM APIs follow the physical DOM structure. React's synthetic event propagation through a portal follows the React tree.

### 24. Do error boundaries and other React ownership relationships follow the React tree?

The portal remains part of its existing React tree, so React-level relationships such as context and ancestor boundaries follow that tree, subject to the feature's own behavior.

## CSS and layout

### 25. Does CSS inheritance follow the React tree?

No. CSS inheritance follows the physical DOM tree.

### 26. Will a portal inherit a CSS custom property defined only on `#root`?

Not if the portal target is outside `#root` in the DOM. Put shared variables on a common DOM ancestor or define them on the portal content or target.

### 27. Will a selector like `.app-shell .tooltip` match a portal tooltip?

Only if the tooltip is physically a descendant of an element with `app-shell`. React parentage does not affect CSS descendant selectors.

### 28. Does a portal always solve z-index problems?

No. It changes DOM placement, but stacking contexts and the target's own ancestors still matter. Choose a suitable target and styles.

### 29. Does rendering into a portal automatically position the content?

No. The portal controls the DOM destination. CSS still determines position, size, color, and layering.

### 30. Does a portal automatically make a dialog accessible?

No. You still need correct dialog semantics, an accessible name, keyboard interaction, focus management, and a way to dismiss it where appropriate.

### 31. Can I add `aria-modal="true"` and assume a dialog is complete?

No. That attribute does not implement focus movement, focus containment, background handling, or keyboard behavior.

### 32. What should I consider for a modal dialog portal?

Provide a name and dialog semantics, move focus appropriately, keep keyboard focus in the modal while it is open, support dismissal, and restore focus when it closes. Native `<dialog>` with `showModal()` supplies useful browser behavior, but the whole interaction still needs testing.

### 33. Does a non-modal popover need to claim it is a modal?

No. Use semantics that match its behavior. Do not claim modal behavior if users can still interact with the background.

## Server rendering and roots

### 34. Can I read `document.getElementById` at module scope in server-rendered code?

No. `document` does not exist on the server. Resolve the target in client-only code or after the component mounts.

### 35. Can `createPortal` be used with a target that only exists after an Effect?

Yes. Keep the target in state, set it in an Effect after mounting, and render the portal only once the target node is available.

### 36. Does the portal need a server-side target?

The portal can only render into a DOM node available to the client. Server-rendered applications need to account for client-only target discovery and hydration behavior.

### 37. What is the difference between `createPortal` and `createRoot`?

`createPortal` renders part of an existing React tree into another DOM node. `createRoot` creates a separate React root with a separate tree.

### 38. Why not call `createRoot` for each modal?

A separate root does not automatically share the original tree's context or state and adds another root to manage. Use a portal when the UI belongs to the same React application.

### 39. Can a portal render into a DOM node owned by a non-React page or widget?

Yes. This can integrate React content with server-rendered markup or a third-party widget. Keep ownership clear: React should manage the portal's children and the external system should manage the target node itself.

## Keys and state

### 40. Does portal content have state like any other component?

Yes. Its React state is managed according to its position and identity in the React tree.

### 41. Can changing the portal target reset child state?

Yes. Changing the target DOM node causes React to recreate the portal content.

### 42. What is the optional portal key for?

It gives the portal a stable identity among other rendered children, such as multiple portals in a list.

### 43. Should a portal in a list use an array index as its key?

Prefer a stable unique identifier that represents the item, just like other keyed React children.

## Practical usage

### 44. Should a portal be used for every overlay?

No. Use it when the physical DOM placement solves a real clipping, stacking, or ownership issue. Normal in-tree markup is simpler when it works.

### 45. Does a portal remove the component from its parent's lifecycle?

No. It remains in the same React tree, so its rendering and state are still managed by React from that tree.

### 46. Can I render a portal conditionally?

Yes. Render the portal node only when the UI should be present, or conditionally render the portal's children.

### 47. What if the portal target is missing during initial rendering?

Render nothing until the target exists, or ensure the target is included in the page before the React app runs.

### 48. Why might portal styles look different from the surrounding app?

The portal may no longer inherit CSS rules, custom properties, fonts, or selectors from the original DOM ancestors. Style it according to its actual target location.

### 49. What is the main rule about portal event handlers?

They follow React ancestry. Check whether an ancestor handler should receive the event, and stop propagation when that behavior would cause an unwanted side effect.

### 50. What is the main rule about portal accessibility?

A portal only changes DOM placement. It does not supply correct semantics, keyboard behavior, focus handling, or screen reader support.

### 51. When should I use one shared portal target versus several?

A shared target is usually enough for a small app. Several targets can be useful when separate overlays have different stacking, ownership, or integration requirements.

### 52. What is the key mental model for portals?

The DOM node is placed elsewhere, but the React component remains in its original React tree. Use the DOM tree to reason about CSS and physical layout, and the React tree to reason about context and React event propagation.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="068-error-boundaries-and-recovery-ui.md">068 - Error Boundaries and Recovery UI - Q&amp;A</a></td><td><a href="../lessons/069-portals.md">Lesson</a></td><td><a href="../examples/069-portals.md">Example</a></td><td><a href="070-react-dom-roots-and-client-apis.md">070 - React DOM Roots, Hydration, and Client APIs - Q&amp;A</a></td></tr>
  </tbody>
</table>
