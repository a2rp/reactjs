# useRef for DOM Nodes and Mutable Values - Q&A

## Ref basics

### 1. What does **useRef** return?

It returns a ref object with one mutable property, **current**. React preserves the same ref object for that component instance across renders.

### 2. What is the initial value of a ref?

It is the value passed to **useRef** on the initial render. Later arguments are ignored because React returns the same ref object.

### 3. Does changing **ref.current** trigger a render?

No. React does not track changes to the mutable property, so changing it alone does not update the screen.

### 4. When should a value go in state instead of a ref?

Use state when the value affects visible UI or determines what the component should render. Use a ref for information that must persist but does not itself need to render.

### 5. Can a ref hold values other than DOM elements?

Yes. It can hold any JavaScript value, such as a timer ID, browser object, or non-visual mutable bookkeeping.

### 6. Is a ref object the same as a state variable?

No. A ref is one persistent object whose **current** property you can mutate. State is a render snapshot updated through a setter, and a state update requests a render.

### 7. Does each component instance get its own ref?

Yes. A ref created by a component belongs to that component instance. Avoid moving it to module scope, where instances could accidentally share one mutable value.

## Refs and rendering

### 8. Can I read or write **ref.current** while rendering?

Usually no. React expects rendering to be pure, and ref mutations are invisible to React. Use the ref from an event handler, Effect, or callback ref instead.

### 9. Is there an exception for initializing a ref?

A predictable, side-effect-free one-time initialization may set the ref during rendering when **current** is still **null**. Do not use this exception to start timers, access the DOM, or create external resources that need cleanup.

### 10. Why should I not display **ref.current** in JSX?

Changing it does not cause a render, so the displayed value can remain stale. If the user needs to see the value, store it in state.

### 11. Can changing a ref update a value in the current screen if some other state update happens later?

A later render may read the mutated ref, but relying on unrelated renders to reveal ref changes is fragile. Use state for any value that belongs in the rendered output.

### 12. Is it useful to add **ref.current** to an Effect dependency array?

No. Mutating **current** does not cause React to re-render and compare dependencies. React cannot use that dependency to observe ref changes.

### 13. Does a ref update work synchronously?

The JavaScript assignment to **ref.current** takes effect immediately. That does not make it a reactive state update or cause React to render again.

### 14. What happens if I pass an expensive expression to **useRef**?

JavaScript evaluates the expression on every render, even though React uses only the first value. If the initialization is expensive but pure, a guarded one-time initialization may avoid repeating it. Do not create side-effectful resources during rendering.

## DOM refs

### 15. How do I attach an object ref to a DOM element?

Create it with **useRef(null)** and pass the ref object to the element's **ref** prop, such as **ref={inputRef}**.

### 16. What is in a DOM ref before the element is committed?

Initially **current** is **null**. React attaches the DOM node during the commit after rendering.

### 17. What happens when the DOM element is removed?

React clears the object ref to **null** when that node detaches. Check for **null** before using the ref when the node might not exist.

### 18. Where can I safely call **inputRef.current.focus()**?

In an event handler or an Effect after React has attached the input. Do not call it while calculating JSX.

### 19. What kinds of browser operations are good uses for DOM refs?

Imperative operations such as focusing an input, scrolling an element into view, measuring a committed node, or calling browser media methods like **play()** and **pause()**.

### 20. Should I manually remove an element that React rendered?

Usually no. React owns that DOM structure. Use state and conditional rendering to add or remove it. Imperative focus and scrolling are generally safer because they do not replace React-managed structure.

### 21. How do I get a ref to one of many dynamically rendered nodes?

Keep a single ref to a **Map** keyed by stable item IDs, and use callback refs to register and remove nodes. Do not call **useRef** inside a loop or **map()**.

### 22. What is a callback ref?

It is a function passed to the **ref** prop. React calls it when attaching a node. In current React, it may return a cleanup function to remove the node from your map when it detaches.

### 23. Can an inline callback ref cause repeated attach and cleanup work?

Yes. A new inline callback has a new identity on every render, so React may detach the old callback and attach the new one. Consider callback identity and cleanup costs when managing many nodes.

### 24. What does Strict Mode do to callback refs in development?

It performs an extra setup and cleanup cycle to help reveal callback refs that fail to release what they registered. Make the callback's cleanup match its setup.

## Non-visual mutable values

### 25. Why store a timer ID in a ref?

Event handlers need to start and later stop the same timer. The ID must persist between handlers but does not need to appear in the UI, so a ref is suitable.

### 26. Where should the timer itself be started?

In response to an interaction, such as the Start button's handler, or in an Effect when it should synchronize with rendered inputs. Do not start it during rendering.

### 27. Where should a timer ID be cleared?

In the Stop handler when the user stops it, and in cleanup when the component unmounts or the Effect that owns it ends.

### 28. Why store a running status in state if the timer ID is in a ref?

The running status changes visible buttons or labels, so it belongs in state. The timer ID is an implementation detail used to control the browser timer, so it belongs in a ref.

### 29. Can I store an event count in a ref?

Yes, if it is used only by event handlers or diagnostics and does not affect rendered output. If it should be displayed, use state.

### 30. Can I store an object used by an imperative library in a ref?

Yes, when the object is local to the component and not used to calculate JSX. If creating it starts external work or requires cleanup, create and release it in an Effect or an appropriate lifecycle, not during rendering.

## Rules and safety

### 31. Can I call **useRef** inside a condition or loop?

No. **useRef** is a Hook and must be called at the top level of a component or custom Hook in a consistent order.

### 32. What happens to refs in development Strict Mode?

React may call the component function twice and discard one render result to detect impure rendering. The accepted ref object remains local to the mounted component. Keep render-time ref initialization pure.

### 33. Can a ref hold mutable data that is also used for rendering?

Avoid mutating data that is used to calculate JSX. React will not know about the mutation, which can make output inconsistent. Keep rendered data in state and reserve refs for non-rendering uses.

### 34. Can I use refs to avoid lifting state or passing props?

A ref is not a general shared-state mechanism. It belongs to one component instance and does not trigger rendering. Use state ownership, props, context, or an external store for shared reactive data.

### 35. Should I use a ref for the value of a controlled input?

Usually no. Store the input's current value in state if React renders from it or validates it. Use a DOM ref when you need an imperative operation such as focusing the input.

### 36. Is a DOM ref available on the first render?

No. React attaches it during commit. On initial render it is normally **null**, so code that accesses the node must wait until an event after mount, a callback ref, or an Effect.

### 37. Can a custom component receive a ref?

It depends on the component and React version. Built-in DOM elements attach refs directly. Passing refs through a custom component and exposing a limited imperative API are covered in the next refs lesson.

### 38. What if an operation must happen after a rendered list changes?

Use a ref to access the committed node and an Effect to synchronize with the rendered change. If the operation must complete before the browser paints, use the appropriate layout-effect API rather than reading the ref during render.

## Quick recap

- Use state for values that determine UI; use refs for persistent values that do not.
- A ref's **current** value is mutable and changing it does not render.
- React attaches DOM refs during commit and clears them when nodes detach.
- Use DOM refs for focused imperative actions, and let React own rendered structure.
- Store timer IDs and similar non-visual values in refs; clean up external resources.
- Keep ref reads and writes out of rendering, except predictable one-time initialization.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="057-custom-hooks-and-usedebugvalue.md">057 - Custom Hooks and useDebugValue - Q&amp;A</a></td><td><a href="../lessons/058-useref.md">Lesson</a></td><td><a href="../examples/058-useref.md">Example</a></td><td><a href="059-refs-and-imperative-handles.md">059 - React 19 Refs, forwardRef, and Imperative Handles - Q&amp;A</a></td></tr>
  </tbody>
</table>
