# Q&A: React 19.3 View Transitions, Fragment refs, and browser APIs

## React 19.3 overview

### 1. Which APIs are the focus of this chapter?

React 19.3 makes the DOM `<ViewTransition>` component and refs on explicit `<Fragment>` stable. It also adds `browser()` from `react-dom` for intentionally browser-only rendering during SSR, improves Trusted Types compatibility, and lets Server Components render certain Context providers imported from Client Components.

### 2. Which versions do I need?

Use React and React DOM 19.3 or later, keeping the two packages on compatible versions. View Transitions and Fragment refs currently apply to React DOM, not React Native.

### 3. Does this chapter cover every React 19.3 changelog item?

No. It covers the major APIs in the chapter title and the related Context change. React 19.3 also includes fixes and smaller changes; check the official release notes when upgrading an application.

## View Transitions

### 4. What does React's `<ViewTransition>` do?

It marks a part of the React tree for browser View Transition animations when React commits an eligible update. The browser captures visual snapshots and animates between them. The component does not add an HTML wrapper element.

### 5. Should I call `document.startViewTransition()` myself?

Not for a React update managed by `<ViewTransition>`. React coordinates the browser API with its own render and commit. Starting another transition manually can conflict with React's coordination.

### 6. Does every state update animate?

No. Regular urgent `setState` updates are immediate and do not activate View Transitions by themselves. A boundary can animate when its change is part of a React Transition, a Suspense reveal, or an update from `useDeferredValue`.

### 7. How do I animate a state change?

Mark a non-urgent UI update as a Transition, for example with `startTransition`:

~~~jsx
startTransition(() => {
  setSelectedPage("details");
});
~~~

Keep genuinely urgent values, such as text currently being typed, outside the Transition when they need to update immediately.

### 8. What are `enter`, `exit`, `update`, and `share`?

- `enter`: a boundary is inserted into the transitioned tree;
- `exit`: a boundary is removed;
- `update`: content or layout inside an existing boundary changes; and
- `share`: a named boundary leaves one subtree while a boundary with the same name enters another in the same Transition.

React chooses the trigger from the tree changes.

### 9. What animation runs if I do not set a class?

The browser's default animation is a smooth cross-fade. You can keep it, disable it for selected trigger types, or give a trigger a CSS class with `<ViewTransition>` props.

### 10. What does `default="none"` mean?

It turns off animation for triggers that are not explicitly assigned. For example, set `default="none"` and `enter="slide-in"` if only entering content should animate.

### 11. How do View Transition Classes work?

A value such as `enter="slide-in"` tells React to add that class to the transition snapshot for the selected animation. CSS styles the snapshot through selectors such as `::view-transition-new(.slide-in)`. The class is not a regular persistent class on the component.

### 12. What is a Transition type?

A Transition type labels the reason or direction for a Transition so the same trigger can use different animations. Call `addTransitionType("forward")` inside the `startTransition` callback, then map `forward` to a class in a `<ViewTransition>` prop.

### 13. What is the `name` prop for?

Use `name` to connect the same conceptual visual item across two different positions or trees in a shared-element transition. React generates unique names automatically for normal animations, so a manual name is not needed for ordinary enter, exit, or update effects.

### 14. What rules apply to named shared elements?

The old and new trees must contain a boundary with the same name during the same Transition. There must not be two simultaneously mounted boundaries with the same name anywhere in the app. In lists, include a stable item ID in each name to prevent collisions.

### 15. Why is my enter or exit animation not running?

Check that the boundary participates in a Transition and is the first component inserted or deleted for that change. Also check its placement: for enter/exit, the `<ViewTransition>` should be the first rendered component in the component subtree that returns it, before other DOM nodes in that subtree.

### 16. Does React animate every nested element's layout independently?

No. The browser animates a captured visual representation of the boundary. This can create smooth movement and scaling, but nested elements do not all behave like individually animated layout objects. Add more boundaries if specific inner pieces need their own motion.

### 17. Can I animate a View Transition with JavaScript?

Yes. The `onEnter`, `onExit`, `onShare`, and `onUpdate` callbacks receive access to the old/new transition pseudo-elements. They can use the Web Animations API. Return a cleanup function, such as `() => animation.cancel()`, so React can clean up when a transition finishes or is interrupted.

### 18. Does React automatically respect reduced-motion settings?

No. React does not automatically turn these animations off for users who prefer reduced motion. Add `@media (prefers-reduced-motion: reduce)` rules for your View Transition CSS and make sure the interface still works without animation.

### 19. Can I use `<ViewTransition>` in React Native?

Not currently. The stable API in this release is for React DOM.

## Fragment refs

### 20. What problem do Fragment refs solve?

They let a component coordinate certain DOM operations across sibling elements without inserting a wrapper just to hold a ref. This is helpful when a component renders multiple host nodes or does not expose one ref-able element.

### 21. Does a Fragment ref add an element to the DOM?

No. A Fragment still groups its children without changing the resulting DOM structure. It has no element box, tag name, attributes, role, or accessible name.

### 22. Can the shorthand Fragment `<>...</>` receive a ref?

No. Import `Fragment` from React and use the explicit form:

~~~jsx
import { Fragment, useRef } from "react";

const groupRef = useRef(null);

return (
  <Fragment ref={groupRef}>
    <article>First</article>
    <article>Second</article>
  </Fragment>
);
~~~

### 23. What does `fragmentRef.current` contain?

It contains a `FragmentInstance`, not a DOM `Element`. The instance exposes a limited set of operations such as event listener management, focus, observation, measurement, and scrolling.

### 24. Which children do FragmentInstance methods target?

Most methods target the Fragment's first-level host (DOM) children. For example, `addEventListener`, `observeUsing`, and `getClientRects` apply to those direct DOM children, not to all DOM descendants recursively.

### 25. Are there methods that search nested children?

Yes. `focus` and `focusLast` search nested descendants depth-first for focusable elements. `blur` removes focus if the active element is inside the Fragment.

### 26. What observer types work with `observeUsing`?

It accepts an `IntersectionObserver` or a `ResizeObserver` and observes the first-level DOM children. It does not observe text nodes, so a Fragment containing only text has no useful element targets.

### 27. What does `dispatchEvent` do on a FragmentInstance?

It dispatches an event on the Fragment's children as a group; listeners registered through the Fragment instance are called, and a bubbling event can bubble to the DOM parent. Prefer normal React state and event props unless a native DOM event is specifically needed.

### 28. Should I use a Fragment ref instead of a semantic element?

No. Use a real element such as `<section>`, `<ul>`, or `<fieldset>` when the interface needs its semantics or attributes. A Fragment ref is for grouping DOM behavior without changing markup; it cannot provide element semantics or a layout box.

### 29. Can I use a Fragment ref for every click handler?

Usually no. React's `onClick` and other event props are simpler and remain part of React's event model. A Fragment ref is useful when native DOM APIs need to operate on several sibling roots or on components that do not expose their own DOM refs.

## Browser-only rendering

### 30. What is `browser()`?

It is an API imported from `react-dom` that returns a browser-only rendering signal. Pass that signal to React's `use` API:

~~~jsx
import { use } from "react";
import { browser } from "react-dom";

function BrowserOnlyPanel() {
  use(browser("This panel reads localStorage."));
  const savedValue = localStorage.getItem("saved-value");

  return <p>{savedValue}</p>;
}
~~~

Calling `browser()` alone has no effect. Do not throw its return value; pass it to `use`.

### 31. What does `use(browser())` render on the server?

It suspends that component during server rendering, so the closest `<Suspense>` boundary's fallback is included in the server HTML. In the browser, it does not suspend and the component continues rendering.

### 32. Does it require Suspense?

Yes, when rendering on the server. Without a surrounding `<Suspense>` boundary, the server render fails.

### 33. Can a Server Component call `use(browser())`?

No. The call must be inside a Client Component. In frameworks where modules are Server Components by default, add `'use client'` to the module or put the call in a Client Component below that boundary.

### 34. Should I replace SSR with `browser()` whenever I use `window`?

No. Prefer useful, stable server output whenever possible. Use `browser()` when the component truly cannot produce meaningful output without browser-only state. It deliberately leaves the Suspense fallback in the server HTML and delays that component's real content until the browser renders it.

### 35. What is the optional reason argument?

It is a string or function explaining why the component needs the browser. The server renderer can use it as the cause for browser-only bailout diagnostics. It does not alter what the component renders.

## Trusted Types and Context

### 36. What changed for Trusted Types in React 19.3?

React now passes Trusted Types values through to DOM APIs without first coercing them to plain strings. This allows the browser to validate typed values under a policy such as `require-trusted-types-for 'script'`.

### 37. Does React's Trusted Types support sanitize HTML?

No. React does not sanitize or bless an unsafe string. Untrusted content still needs a carefully configured sanitizer and Trusted Types policy before it reaches an injection sink such as `dangerouslySetInnerHTML`.

### 38. What Context change does React 19.3 make for Server Components?

A Server Component can import a Context object from a `'use client'` module and render it directly as a provider. That can remove a wrapper component whose only purpose was to pass a server-fetched value to the Context. Server Components still cannot create Context themselves.

### 39. Where should I read about the general HTML injection risks?

See the earlier [HTML injection, sanitization, and Trusted Types lesson](../lessons/113-html-injection-and-trusted-types.md). React 19.3's compatibility change does not change the basic rule: do not send untrusted HTML to an injection sink.

## Quick review

### 40. What are the safest defaults for these features?

- Let React coordinate View Transitions and keep urgent updates immediate.
- Use CSS View Transition Classes first; include reduced-motion styles.
- Leave names automatic unless building a shared-element transition.
- Use an explicit Fragment ref only when group-level DOM access is needed.
- Keep `use(browser())` inside a Client Component and a server-side Suspense boundary.
- Treat Trusted Types as a browser-enforced safety layer, not as a substitute for sanitization.
