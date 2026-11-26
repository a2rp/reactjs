# React 19 Refs, forwardRef, and Imperative Handles

## Let a parent request a small imperative action

A function component does not expose its internal DOM nodes to its parent by default. In React 19, a function component can receive **ref** as a regular prop. The component can pass that ref to a DOM node, or use **useImperativeHandle** to expose a small custom API.

This is useful for actions that cannot be expressed naturally as ordinary props, such as focusing or selecting an input, scrolling a list, or starting an animation. For visible state and component configuration, continue to use props and state.

~~~jsx
function SearchField({ ref, value, onChange }) {
  return <input ref={ref} value={value} onChange={onChange} />;
}
~~~

## Learning goals

After this lesson, you should be able to:

- explain how React 19 lets a function component receive **ref** as a prop;
- pass a ref through a component to a DOM node;
- use **useImperativeHandle** to expose a narrow set of methods;
- understand the dependency list and lifetime of an imperative handle;
- recognize the React 18 **forwardRef** pattern and migrate it thoughtfully;
- decide when an ordinary prop is a better API than a ref;
- avoid exposing implementation details or mutating React-managed DOM unnecessarily.

## A ref does not automatically pass through a function component

This parent can attach a ref to a built-in DOM element such as **input**:

~~~jsx
const inputRef = useRef(null);

return <input ref={inputRef} />;
~~~

A custom function component is different. A ref on the custom component is an API request to that component. The component must decide what the parent receives. It might forward the ref to one DOM element, or expose a custom handle.

This distinction lets a component hide its internal structure. A parent that needs to focus a text field should not need to know whether the child renders an **input**, a **textarea**, or several wrapper elements.

## React 19: receive ref as a prop

Starting with React 19, function components may declare **ref** in their props and pass it onward:

~~~jsx
function SearchField({ ref, label, value, onChange }) {
  return (
    <label>
      {label}
      <input ref={ref} value={value} onChange={onChange} />
    </label>
  );
}
~~~

A parent can then attach a DOM ref through the component:

~~~jsx
function SearchPage() {
  const searchRef = useRef(null);

  function focusSearch() {
    searchRef.current?.focus();
  }

  return (
    <>
      <SearchField
        ref={searchRef}
        label="Search"
        value=""
        onChange={() => {}}
      />
      <button onClick={focusSearch}>Focus search</button>
    </>
  );
}
~~~

When **SearchField** passes the received ref to its **input**, the parent ref points to that input DOM node after React commits it. The parent may call browser APIs such as **focus()** from an event handler.

### Ref remains special at a DOM element

A component may receive **ref** as a prop in React 19, but an intrinsic element such as **input** still uses the JSX **ref** attribute to tell React which ref to attach. Do not spread a ref into an unrelated DOM element or treat it like an ordinary HTML attribute.

A ref to a class component has different behavior: it refers to the class instance. This lesson’s React 19 ref-as-prop pattern is for function components.

## Expose a custom API with useImperativeHandle

Passing a DOM node directly gives the parent access to the node’s full browser API. Sometimes a child should offer only a few supported actions. **useImperativeHandle** lets the component return a custom handle instead:

~~~jsx
import { useImperativeHandle, useRef } from "react";

function SearchField({ ref, value, onChange }) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current?.focus();
    },
    select() {
      inputRef.current?.select();
    },
  }), []);

  return <input ref={inputRef} value={value} onChange={onChange} />;
}
~~~

The child keeps its actual DOM node in **inputRef**. The parent receives an object with **focus()** and **select()**, not the input element itself:

~~~jsx
const searchFieldRef = useRef(null);

function handleFocus() {
  searchFieldRef.current?.focus();
}

return <SearchField ref={searchFieldRef} value={query} onChange={handleChange} />;
~~~

The parent cannot inspect or change the input’s styles or value through that handle. The child remains in control of its implementation, while the parent can perform the two documented actions.

## useImperativeHandle signature and dependencies

The Hook has this form:

~~~jsx
useImperativeHandle(ref, createHandle, dependencies);
~~~

- **ref** is the ref received by the component.
- **createHandle** returns the value assigned to the parent ref. This is commonly an object containing methods.
- **dependencies** lists every reactive value read by **createHandle**. React compares dependencies with **Object.is** and recreates the handle when one changes.

The dependency list must have a constant number of entries and be written inline. If the returned methods only use a stable ref object, an empty dependency list is appropriate because the ref object itself is stable. If a method closes over a prop, state value, or component-local function, include that reactive value or restructure the method so it reads the latest value safely.

If the dependency list is omitted, React recreates the handle on every render. Avoid omitting dependencies as a shortcut. Follow the Hook linter’s dependency guidance.

Call **useImperativeHandle** at the top level of the component. Do not call it conditionally, in a loop, or inside an event handler.

## Handle attachment and cleanup

Before the child is committed, a parent’s ref is **null**. After React commits the child, the ref receives the exposed handle. When the child unmounts or the ref changes, React detaches it; the parent must not assume that **current** is always non-null.

Use optional chaining or a deliberate guard when calling a method:

~~~jsx
function handleClick() {
  searchFieldRef.current?.focus();
}
~~~

As with DOM refs, use the handle from an event handler or an Effect, not while rendering. Do not mutate a ref during render to make a handle appear earlier.

## React 18 and earlier: forwardRef

Before React 19, function components could not receive **ref** as an ordinary prop. **forwardRef** supplied the ref as the second argument to a render function:

~~~jsx
import { forwardRef } from "react";

const SearchField = forwardRef(function SearchField(props, ref) {
  return (
    <label>
      {props.label}
      <input
        ref={ref}
        value={props.value}
        onChange={props.onChange}
      />
    </label>
  );
});
~~~

The parent usage looks the same:

~~~jsx
<SearchField ref={searchRef} label="Search" value={query} onChange={handleChange} />
~~~

To expose a custom handle in React 18, receive the ref from **forwardRef** and pass it to **useImperativeHandle**:

~~~jsx
import { forwardRef, useImperativeHandle, useRef } from "react";

const SearchField = forwardRef(function SearchField(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      inputRef.current?.focus();
    },
  }), []);

  return <input ref={inputRef} value={props.value} onChange={props.onChange} />;
});
~~~

In React 19, **forwardRef** is no longer necessary for new function components that receive refs. React marks it deprecated for a future release, but existing React 18 and React 19 code can still use it. When supporting React 18, keep the **forwardRef** wrapper; a component using the React 19 ref-prop form alone will not receive a ref correctly in React 18.

Do not mechanically remove **forwardRef** from code that needs older React versions. Check the project’s React version and supported consumers first.

## Prefer props for state and ordinary behavior

Imperative handles should stay small. If a value describes what the component should display, use props or state:

~~~jsx
<Dialog isOpen={isDialogOpen} onClose={closeDialog} />
~~~

This is usually clearer than making the parent call **dialogRef.current.open()** and **dialogRef.current.close()**. A prop-driven dialog can participate in normal rendering, state updates, server rendering, and parent-child data flow.

A ref is a good fit when the parent needs to request an imperative browser action that is hard to describe declaratively:

- focus or select a text field;
- scroll a specific child into view;
- start or stop a browser animation;
- measure a DOM node;
- trigger a narrowly scoped action on a media or canvas API.

Avoid a handle that exposes every internal node or a large collection of setters. A focused public API reduces coupling and lets the component change its markup without breaking its parent.

## Do not use a handle to bypass React state

Use React state to represent whether a dialog is open, which tab is selected, or what text should appear. Calling **element.style.display = "none"** or directly changing controlled input values can make the DOM disagree with React’s next render.

Safe imperative work generally uses browser APIs without changing the structure React owns: focusing, selecting, scrolling, measuring, or interacting with a browser object. Avoid adding, removing, or reordering React-rendered children manually.

## Strict Mode and purity

In development Strict Mode, React may call component render functions more than once to expose impure code. Keep rendering and handle creation pure: create the handle description without performing external side effects. Methods should do their imperative work only when the parent calls them.

A handle should not start timers, attach listeners, or mutate the DOM just because it was created. Manage resources with the appropriate Effect and cleanup. Never depend on a handle object retaining identity unless that is an explicit API contract; React may recreate it when dependencies change.

## A practical design checklist

Before exposing a ref, ask:

1. Is this operation genuinely imperative, or can a prop describe the desired UI?
2. Should the parent receive a DOM node, or only a custom method or two?
3. Does every method handle the node being unavailable during attach/detach?
4. Does the dependency list include values captured by the handle?
5. Does the project target React 19, or is a React 18-compatible **forwardRef** required?
6. Could this API let the parent mutate UI that should remain controlled by React?

## Summary

- In React 19, a function component may receive **ref** as a prop and forward it to a DOM node or another component.
- A ref passed to a component is an API boundary; the component decides what its parent receives.
- **useImperativeHandle** exposes a custom handle, often a small object of methods, instead of the complete DOM node.
- Include reactive values used by the handle in its dependency list.
- React 18 and earlier use **forwardRef** to let a function component receive a ref; React 19 does not require it for new function components.
- Use props and state for declarative UI. Reserve handles for focused imperative operations.
- Keep ref operations out of rendering and avoid destructive edits to React-managed DOM.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="058-useref.md">058 - useRef for DOM Nodes and Mutable Values</a></td><td><a href="../qnas/059-refs-and-imperative-handles.md">Q&amp;A</a></td><td><a href="../examples/059-refs-and-imperative-handles.md">Example</a></td><td><a href="060-memoization-useid-usememo-usecallback.md">060 - Memoization, useId, useMemo, and useCallback</a></td></tr>
  </tbody>
</table>
