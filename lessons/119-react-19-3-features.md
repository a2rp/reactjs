# React 19.3: View Transitions, Fragment refs, and browser APIs

React 19.3 makes View Transitions and Fragment refs stable, adds a browser-only rendering signal for server-rendered applications, and improves compatibility with the browser's Trusted Types security API. This chapter explains what each feature does, how to use it, and the limits to keep in mind.

## Learning goals

After this lesson, you should be able to:

- identify when a React update is eligible for a View Transition;
- add enter, exit, update, and shared-element animations without taking over React's transition coordination;
- customize animations with CSS classes or the Web Animations API while respecting reduced-motion preferences;
- use a Fragment ref to work with sibling DOM nodes without adding a wrapper;
- make a component browser-only during server rendering with `use(browser())`;
- explain the role and limits of React's Trusted Types integration; and
- recognize the other React 19.3 Server Components Context improvement.

## Requirements and scope

The APIs in this chapter require **React and React DOM 19.3 or later**. Keep `react` and `react-dom` on compatible versions. View Transitions and Fragment refs currently apply to the DOM renderer; they are not React Native APIs.

React 19.3 also includes fixes and smaller changes across React and React DOM. This chapter focuses on the release features that affect application code. It does not replace the full changelog or the earlier chapters on transitions, Suspense, server rendering, and HTML injection.

## 1. View Transitions

### What React coordinates

A View Transition takes a snapshot of the old visual state, applies an update, and animates toward a snapshot of the new state. React's `<ViewTransition>` component connects this browser capability to React's rendering model.

React coordinates the browser View Transition API for you. Do **not** call `document.startViewTransition()` for the same React update: React needs to control when the DOM changes and when the snapshots are taken.

Import the component from React:

~~~jsx
import { ViewTransition } from "react";

function Card({ title }) {
  return (
    <ViewTransition>
      <article className="card">
        <h2>{title}</h2>
      </article>
    </ViewTransition>
  );
}
~~~

The wrapper is a React boundary, not an extra HTML element. React applies a temporary `view-transition-name` to the closest DOM node or nodes when the boundary participates in an animation.

### Which updates animate

A `<ViewTransition>` does not animate every render. React uses transitions to distinguish updates that can wait for an animation from urgent updates that should appear immediately.

A boundary can animate when it participates in:

- a React Transition started with `startTransition` or `useTransition`;
- a Suspense reveal; or
- an update rendered with `useDeferredValue`.

An ordinary urgent `setState` update does not activate a View Transition by itself. That keeps typing, pressing a button, and other immediate interactions responsive. If an interaction should animate a non-urgent screen change, mark that screen update as a Transition.

~~~jsx
import { startTransition, useState, ViewTransition } from "react";

function ResultPanel({ result }) {
  return (
    <ViewTransition enter="result-enter" exit="result-exit" default="none">
      <section className="result-panel">
        <h2>{result.title}</h2>
        <p>{result.description}</p>
      </section>
    </ViewTransition>
  );
}

export default function SearchResults({ results }) {
  const [selectedIndex, setSelectedIndex] = useState(0);
  const selected = results[selectedIndex];

  function selectResult(index) {
    startTransition(() => {
      setSelectedIndex(index);
    });
  }

  return (
    <>
      <nav aria-label="Search results">
        {results.map((result, index) => (
          <button
            key={result.id}
            type="button"
            aria-current={index === selectedIndex ? "true" : undefined}
            onClick={() => selectResult(index)}
          >
            {result.title}
          </button>
        ))}
      </nav>

      <ResultPanel result={selected} />
    </>
  );
}
~~~

The `ResultPanel` component returns its View Transition boundary as its first rendered node. That placement matters for enter and exit animations: a boundary must be the first component inserted or deleted in the Transition, and the boundary must appear before other DOM nodes in the component subtree that returns it. If an enter or exit animation does not run, inspect the boundary's position in the React tree.

### The four animation triggers

React chooses the trigger based on how the tree changed:

| Trigger | When it applies | Typical use |
| --- | --- | --- |
| `enter` | A boundary enters the transitioned tree | Reveal a panel or newly added item |
| `exit` | A boundary leaves the transitioned tree | Remove a panel or item |
| `update` | Content or layout inside an existing boundary changes | Cross-fade updated content |
| `share` | A named boundary leaves one place while a boundary with the same name enters another | Keep a thumbnail visually connected to its detail view |

By default, the browser supplies a smooth cross-fade. You can choose `auto`, `none`, a CSS class name, or an object that selects a class by Transition type:

~~~jsx
<ViewTransition
  default="none"
  enter={{
    forward: "slide-in-from-right",
    backward: "slide-in-from-left",
  }}
  exit={{
    forward: "slide-out-to-left",
    backward: "slide-out-to-right",
  }}
  update="content-cross-fade"
>
  <Page />
</ViewTransition>
~~~

Here `default="none"` disables unspecified animation types. The object values use Transition types added to the surrounding React Transition. A `default` key inside the object supplies a fallback for types that are not listed.

### Transition types

Use `addTransitionType` when the same state change can have different causes and should animate differently. Add the type inside the same Transition as the state update.

~~~jsx
import {
  addTransitionType,
  startTransition,
  useState,
  ViewTransition,
} from "react";

function Gallery() {
  const [index, setIndex] = useState(0);

  function go(direction) {
    startTransition(() => {
      addTransitionType(direction);
      setIndex((current) => current + (direction === "forward" ? 1 : -1));
    });
  }

  return (
    <>
      <button type="button" onClick={() => go("backward")}>
        Previous
      </button>
      <button type="button" onClick={() => go("forward")}>
        Next
      </button>

      <ViewTransition
        default="none"
        update={{
          forward: "move-left",
          backward: "move-right",
        }}
      >
        <article>
          <h2>Slide {index + 1}</h2>
        </article>
      </ViewTransition>
    </>
  );
}
~~~

A Transition type describes why that Transition happened; it does not change the state update itself. Keep the type names consistent between `addTransitionType` and the class map.

### Shared-element transitions and names

React gives each boundary a unique name automatically for ordinary animation. Add the `name` prop only when you intend to connect the same visual item across two different parts of the tree.

~~~jsx
<ViewTransition name="product-featured-image">
  <img src={product.image} alt={product.name} />
</ViewTransition>
~~~

For a shared-element animation, the old and new trees must contain a boundary with the same name in the same Transition. The name must be unique among all simultaneously mounted named boundaries in the app. If a list contains multiple products, include each product's stable ID in its name. Avoid assigning one shared name to every list row.

A shared transition can move and scale a captured image of the boundary. It does not individually animate every DOM element inside it. Add smaller boundaries where separate elements need independent motion.

### Styling with View Transition Classes

The `enter`, `exit`, `update`, `share`, and `default` props assign View Transition Classes. Style the browser's View Transition pseudo-elements using the class selector:

~~~css
::view-transition-new(.result-enter) {
  animation: result-enter 180ms ease-out both;
}

::view-transition-old(.result-exit) {
  animation: result-exit 140ms ease-in both;
}

@keyframes result-enter {
  from {
    opacity: 0;
    transform: translateY(0.5rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes result-exit {
  to {
    opacity: 0;
    transform: translateY(-0.25rem);
  }
}

@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(.result-enter),
  ::view-transition-group(.result-exit) {
    animation-duration: 1ms;
  }
}
~~~

The class name is applied to the transition snapshot, not as a lasting class on the rendered component. React does **not** automatically disable View Transition animation for people who prefer reduced motion. Define a reduced-motion rule for your own animations and avoid making motion necessary to understand the interface.

### Imperative animations

For cases where CSS is not enough, `onEnter`, `onExit`, `onShare`, and `onUpdate` receive an object exposing the old/new transition snapshots. They can be animated with the Web Animations API.

~~~jsx
<ViewTransition
  onEnter={(instance) => {
    const animation = instance.new.animate(
      [{ opacity: 0, transform: "scale(0.96)" }, { opacity: 1, transform: "scale(1)" }],
      { duration: 180, easing: "ease-out" }
    );

    return () => animation.cancel();
  }}
>
  <article>New content</article>
</ViewTransition>
~~~

Return a cleanup function from an event callback. React calls it when the View Transition finishes or is interrupted, so it can cancel the animation and release resources. Prefer CSS classes for reusable effects; use callbacks when the animation needs runtime values or imperative control.

### Practical limits

- View Transitions currently work with React DOM.
- Use `<ViewTransition>` as a boundary inside React's transition flow; do not manually start a competing browser transition.
- Keep urgent interactions urgent. A transition is appropriate for a non-blocking screen change, not for delaying a text field's value.
- Place enter/exit boundaries correctly and give shared names a unique namespace.
- Test reduced-motion behavior and the page with animations disabled.
- Design the final UI so it remains understandable if the animation is skipped or unavailable.

## 2. Fragment refs

A Fragment groups sibling elements without adding a wrapper to the DOM. In React 19.3, an explicit `<Fragment>` can also receive a ref. The ref value is a `FragmentInstance`, which provides a limited set of operations over the DOM children in the Fragment.

~~~jsx
import { Fragment, useEffect, useRef } from "react";

function ArticleGroup({ articles }) {
  const groupRef = useRef(null);

  useEffect(() => {
    const group = groupRef.current;
    if (!group) return;

    function handleGroupClick(event) {
      if (!(event.target instanceof Element)) return;

      const article = event.target.closest("[data-article-id]");
      if (!article) return;

      console.log("Selected article:", article.getAttribute("data-article-id"));
    }

    group.addEventListener("click", handleGroupClick);

    return () => {
      group.removeEventListener("click", handleGroupClick);
    };
  }, []);

  return (
    <Fragment ref={groupRef}>
      {articles.map((article) => (
        <article key={article.id} data-article-id={article.id}>
          <h2>{article.title}</h2>
          <button type="button">Open article</button>
        </article>
      ))}
    </Fragment>
  );
}
~~~

This adds one native listener to each first-level DOM child managed by the Fragment, while leaving the rendered HTML structure unchanged. For ordinary React interactions, use React event props such as `onClick`; Fragment refs are useful when a DOM API must operate on a group of sibling nodes or when a child component does not expose an element ref.

The shorthand `<>...</>` cannot take a ref. Import `Fragment` from React and write `<Fragment ref={...}>...</Fragment>`.

### FragmentInstance methods

| Method | What it does |
| --- | --- |
| `addEventListener` / `removeEventListener` | Add or remove a native listener on each first-level DOM child |
| `dispatchEvent` | Dispatch an event on the Fragment; it can bubble to the Fragment's DOM parent |
| `focus` / `focusLast` / `blur` | Focus the first or last focusable descendant, or blur focus inside the group |
| `observeUsing` / `unobserveUsing` | Attach or remove an `IntersectionObserver` or `ResizeObserver` for first-level DOM children |
| `getClientRects` | Read the rectangles of first-level DOM children |
| `getRootNode` | Get the root node of the Fragment's parent |
| `compareDocumentPosition` | Compare the Fragment's position with another node |
| `scrollIntoView` | Scroll the Fragment's first-level children into view |

### The direct-child rule

Most FragmentInstance methods target first-level host (DOM) children. If the Fragment contains:

~~~jsx
<Fragment ref={groupRef}>
  <article>...</article>
  <section>...</section>
</Fragment>
~~~

the `article` and `section` are first-level DOM children. If it contains a component that renders a `div`, that `div` is the host child React works with even though the component itself is not a DOM node.

Focus methods are different: `focus` and `focusLast` search nested children depth-first. Observer methods do not observe text nodes; observing a Fragment that contains only text is not useful. `scrollIntoView` takes the supported boolean alignment argument rather than an options object.

Use an actual semantic element when the UI needs one. A Fragment ref does not create a DOM element and cannot provide an element's attributes, layout box, role, or accessible name.

## 3. Browser-only rendering with `browser()`

Server-rendered components normally produce useful HTML on the server, then hydrate that HTML in the browser. Sometimes a component cannot produce meaningful server output because it needs a browser-only value such as `localStorage` or the device's local time zone.

React 19.3 adds `browser` from `react-dom`. Pass its result to React's `use` API:

~~~jsx
import { Suspense, use } from "react";
import { browser } from "react-dom";

function LocalTimeZone() {
  use(browser("The time zone is read from the browser."));
  const timeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;

  return <p>Your time zone is {timeZone}.</p>;
}

export default function Page() {
  return (
    <section>
      <h1>Local settings</h1>
      <Suspense fallback={<p>Loading local settings…</p>}>
        <LocalTimeZone />
      </Suspense>
    </section>
  );
}
~~~

During server rendering, `use(browser())` suspends the component. The closest `<Suspense>` fallback is included in the server HTML. In the browser, the same call does not suspend, so the component can read browser APIs and render normally.

### Rules and tradeoffs

- `use(browser())` must be inside a `<Suspense>` boundary during server rendering. Without one, server rendering fails.
- It belongs in a **Client Component**, not a Server Component. In a framework that treats files as Server Components by default, put `'use client'` at the top of the module or move the API call into a Client Component.
- Calling `browser()` alone has no effect. Pass its returned value to `use`; do not throw it.
- The optional reason is a diagnostic explanation used by the server renderer.
- The API intentionally opts the component out of server-rendered content. Use it only when server output is not meaningful; do not use it as a default replacement for SSR, hydration, or framework data loading.

When possible, give the server a stable initial value and render that value both on the server and on the first client render. That preserves meaningful initial HTML and avoids an unnecessary fallback. Use `browser()` when browser-only rendering is the actual requirement.

## 4. Trusted Types compatibility

Trusted Types is a browser security feature that can restrict values passed to DOM injection sinks such as `innerHTML`. A Content Security Policy such as:

~~~http
Content-Security-Policy: require-trusted-types-for 'script'
~~~

requires appropriate typed values, such as `TrustedHTML`, for protected sinks. Those values must come from an application policy connected to a real sanitizer or another trusted source.

Before React 19.3, React could coerce Trusted Types objects into strings before passing them to DOM APIs. React 19.3 passes these values through without that coercion, so a browser-enforced Trusted Types policy can validate them.

This is **compatibility support**, not sanitization:

- React does not make an unsafe string safe.
- Do not pass attacker-controlled HTML to `dangerouslySetInnerHTML`.
- Sanitize untrusted content and create Trusted Types values only through a carefully controlled policy.
- Keep CSP and sanitizer configuration in the application security design; React's integration does not replace either one.

The earlier [HTML injection, sanitization, and Trusted Types lesson](113-html-injection-and-trusted-types.md) covers the underlying security model.

## 5. Another React 19.3 Server Components change

React Server Components cannot create Context, but in React 19.3 a Server Component can import a Context object from a `'use client'` module and render it directly as a provider:

~~~jsx
// user-context.js
"use client";

import { createContext } from "react";

export const UserContext = createContext(null);
~~~

~~~jsx
// layout.jsx — a Server Component in a framework that supports RSC
import { UserContext } from "./user-context.js";

export default async function Layout({ children }) {
  const currentUser = await getCurrentUser();

  return (
    <UserContext value={currentUser}>
      {children}
    </UserContext>
  );
}
~~~

This removes a wrapper component whose only job was to pass a server-fetched value into the Context provider. It does not make Context creation available in Server Components and does not make arbitrary client-only Context consumers into Server Components. Follow the framework's Server/Client module boundary rules.

## 6. Choosing the right feature

| Need | Use |
| --- | --- |
| Animate a non-urgent React UI change | `<ViewTransition>` inside a Transition |
| Keep the same visual item continuous between two trees | A named `<ViewTransition>` pair |
| Work with sibling DOM nodes without changing markup | An explicit `<Fragment ref={...}>` |
| Render a component only in the browser during SSR | `use(browser())` inside a Client Component and a Suspense boundary |
| Put sanitized HTML into a protected DOM sink | A real sanitizer and a Trusted Types policy, with React 19.3 preserving the typed value |

Do not select a feature only because it is new. First identify the behavior you need, then choose the narrowest API that supports it.

## Implementation checklist

Before shipping code that uses these APIs, confirm:

- `react` and `react-dom` are both on a compatible 19.3-or-later version.
- View Transition state updates are marked as transitions when animation is desired; urgent updates remain immediate.
- Enter/exit boundaries are positioned so React can detect their insertion or removal.
- Shared-element names are stable and unique among mounted boundaries.
- Every custom animation has a reduced-motion rule and the UI still works without motion.
- Fragment refs use the explicit `Fragment` import; methods that target children are used with the first-level DOM rule in mind.
- Browser-only components are Client Components under Suspense in server-rendered applications.
- Trusted Types policies use sanitized values; React is not being treated as a sanitizer.
- Server Component Context providers follow the framework's import boundary rules.

## Key points

- React 19.3 stabilizes View Transitions and Fragment refs for React DOM.
- React starts and coordinates its own View Transitions; ordinary urgent state updates are not animated automatically.
- View Transition names are normally automatic. Add a name only to connect a shared element across trees.
- Fragment refs expose group operations without adding an element; most methods target first-level DOM children.
- `use(browser())` opts a Client Component out of server rendering behind a Suspense fallback.
- React's Trusted Types support preserves typed values for browser validation but does not sanitize them.
- React 19.3 also lets Server Components render Context objects imported from client modules as providers.

## Official references

- [React 19.3 release post](https://react.dev/blog/2026/09/09/react-19-3)
- [`<ViewTransition>` reference](https://react.dev/reference/react/ViewTransition)
- [`<Fragment>` and FragmentInstance reference](https://react.dev/reference/react/Fragment)
- [`browser` reference](https://react.dev/reference/react-dom/browser)
