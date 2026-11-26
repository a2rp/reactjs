# React 19 Refs, forwardRef, and Imperative Handles - Q&A

## Refs across component boundaries

### 1. What happens when a parent passes a ref to a function component?

The ref is a request for the child component to expose something. The child chooses what the ref points to. It may pass the ref to a DOM node or expose a custom handle.

### 2. Can a function component receive ref as a prop?

Yes, starting with React 19. The component can declare **ref** in its props and forward it or pass it to **useImperativeHandle**.

### 3. How did a function component receive a ref in React 18?

It used **forwardRef**. React passed the ref as the second argument to the wrapped render function.

### 4. Is forwardRef required in React 19?

No. React 19 lets function components receive **ref** as a prop. **forwardRef** remains usable for existing code and React 18 compatibility, but React marks it deprecated for a future release.

### 5. Can I remove forwardRef from every component now?

No. First check which React versions the project supports. A component written only with the React 19 ref-prop form will not receive refs correctly in React 18.

### 6. Does a ref on a built-in input work the same way?

Yes. **<input ref={inputRef} />** tells React to attach the DOM node to the ref after commit. The distinction is that a custom function component must decide how to expose a ref.

### 7. What does a parent receive if a ref is passed to a class component?

A ref to a class component refers to its class instance. React 19’s ref-as-prop pattern applies to function components; it does not turn a class instance ref into an ordinary prop.

### 8. Should every component forward a ref?

No. Forward or expose a ref only when it is part of the component’s useful public API. If a parent has no imperative need, ordinary props are enough.

## useImperativeHandle

### 9. What does useImperativeHandle do?

It customizes the value a parent receives through a ref. The child can return an object with selected methods instead of exposing its complete DOM node.

### 10. What is the basic signature?

**useImperativeHandle(ref, createHandle, dependencies)**. The first argument is the received ref, the second returns the exposed value, and the optional dependency list controls when React recreates that value.

### 11. Does the handle have to be an object?

No. The returned handle can be any value. A small object of named methods is common because it documents the actions the parent may request.

### 12. Why expose a custom handle instead of the input DOM node?

A custom handle limits the parent to supported actions, such as **focus()** and **select()**. The child can change its markup without giving the parent access to implementation details.

### 13. How do I use useImperativeHandle in a React 19 function component?

Receive **ref** as a prop, create an internal DOM ref if needed, and call **useImperativeHandle(ref, createHandle, dependencies)** at the component’s top level.

### 14. How do I use useImperativeHandle in React 18?

Wrap the function component in **forwardRef**. Its render function receives **props** and **ref**; pass that ref to **useImperativeHandle**.

### 15. Must useImperativeHandle be called at the top level?

Yes. Call it unconditionally at the top level of the component or custom Hook, following the Rules of Hooks. Do not call it in a branch, loop, event handler, or nested callback.

### 16. What belongs in the dependency list?

Every reactive prop, state value, or component-local function read by **createHandle** or its methods. Keep the list inline and keep its number of entries constant.

### 17. When is an empty dependency list appropriate?

When the handle methods only use stable values, such as a ref object returned by **useRef**, and do not capture changing props, state, or local functions.

### 18. What happens if I leave out the dependency list?

React recreates the exposed handle on each render. Prefer a correct dependency list and heed the Hook linter instead of omitting dependencies as a workaround.

### 19. What if a handle method reads a prop or state value?

Make sure the handle is recreated when that reactive value changes by including it in the dependency list, or redesign the method so it safely reads the current value from an appropriate ref. Do not rely on a stale closure.

### 20. Is the handle available during the child’s first render?

No. The ref is attached during commit. It is initially **null**, then receives the handle after React commits the child.

### 21. Can a parent assume ref.current is never null?

No. It can be null before attachment and after detachment or unmount. Use optional chaining, a guard, or a lifecycle-aware Effect.

### 22. When should a parent call a handle method?

Usually from an event handler or an Effect after the child has committed. Do not read or invoke the handle during rendering.

### 23. Does creating the handle perform its methods?

No. The factory creates and returns the handle. The methods do work only when the parent invokes them. Keep handle creation pure; do not start timers or attach listeners there.

### 24. Can a handle expose a DOM node and methods together?

It can, but consider whether that weakens the boundary. Expose only what callers need. Returning the full node gives the parent access to many browser properties and methods.

## Declarative props versus imperative methods

### 25. Should a parent call ref.current.open() to show a dialog?

Usually not. If open or closed is visible UI state, model it with an **isOpen** prop and a callback such as **onClose**. This keeps rendering declarative.

### 26. When is an imperative method appropriate?

For a focused operation that is naturally imperative and difficult to express as desired UI state: focus, selection, scrolling, measurement, animation, or a browser API action.

### 27. Can refs replace state for visible values?

No. Mutating **ref.current** does not trigger a render. Use state or props for values that determine what appears on screen.

### 28. Can a handle clear a controlled input by setting input.value directly?

That is usually the wrong design. A controlled value belongs to React state. Expose a parent callback or update the state through normal props so the DOM and React stay in sync.

### 29. Is it safe to remove or add React-rendered DOM nodes through a handle?

Avoid it. React owns the structure it renders; manual structural changes can conflict with reconciliation. Prefer non-destructive browser operations or state updates that describe the desired structure.

### 30. Is focusing an input through a handle safe?

Yes. Focusing is a non-destructive browser operation and a common use for an imperative handle.

### 31. Should a reusable component expose every internal method?

No. A small, intentional API is easier to use and lets the component change its internal implementation safely.

## Development behavior and design checks

### 32. Does Strict Mode change how handles should be written?

Keep rendering and handle creation pure. Strict Mode can repeat render-related work in development to reveal impurities. Do not perform external side effects merely while creating the handle.

### 33. Should I depend on the handle object’s identity staying constant?

Generally, no. React may recreate it when dependencies change. Design around the documented methods rather than object identity unless identity stability is deliberately guaranteed.

### 34. What is the difference between forwarding a DOM ref and exposing a custom handle?

Forwarding a DOM ref gives the parent that DOM node and its browser API. A custom handle gives the parent only the values or methods returned by **useImperativeHandle**.

### 35. Can forwardRef and useImperativeHandle be used together?

Yes. That is a standard React 18 pattern: **forwardRef** receives the ref, and **useImperativeHandle** controls what the parent receives. React 19 can use **ref** as a prop instead of **forwardRef**.

### 36. What should I check before changing an existing forwardRef component?

Check the minimum supported React version, the public ref contract, the component’s consumers, and whether the parent needs the DOM node or only a narrow custom API.

### 37. What is a good rule of thumb for a component API?

Use props to say what the component should render. Use a ref handle to request a small imperative action that cannot be expressed cleanly as a prop.

### 38. What does the React 19 version of a custom input look like in one sentence?

It accepts **ref** as a prop, holds its internal input in **useRef**, and uses **useImperativeHandle** to expose only intentional methods such as **focus()**.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="058-useref.md">058 - useRef for DOM Nodes and Mutable Values - Q&amp;A</a></td><td><a href="../lessons/059-refs-and-imperative-handles.md">Lesson</a></td><td><a href="../examples/059-refs-and-imperative-handles.md">Example</a></td><td><a href="060-memoization-useid-usememo-usecallback.md">060 - Memoization, useId, useMemo, and useCallback - Q&amp;A</a></td></tr>
  </tbody>
</table>
