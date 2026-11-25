# React DevTools, Debugging, and Common Warnings: Questions and Answers

## React Developer Tools

### 1. What does React Developer Tools add to browser developer tools?

It adds a Components panel for inspecting the React component tree and a Profiler panel for recording React rendering work. The browser's built-in panels remain useful for DOM, CSS, JavaScript, network, storage, and overall page performance.

### 2. How is the Components panel different from the Elements panel?

The Components panel shows React components and their props, state, context, and Hooks. The Elements panel shows actual DOM nodes and styles. The two trees are related, but they are not the same: components can return fragments, other components, portals, or no DOM node.

### 3. What should I inspect when a component displays the wrong value?

Select it in the Components panel and inspect its props, state, context, and parent. Then trace the value from the component that created it to the one that displays it. If a prop is already wrong at the child, inspect the parent and data source next.

### 4. Can I change a value in DevTools to fix the application?

You can use an edited value to explore how the UI responds, but that edit is temporary. It does not change source code, persist application data, or prove that a user action updates the value correctly.

### 5. What is the React Profiler for?

It records React rendering work while you perform an interaction. Use it to find which components participated in a commit and where React spent time. It helps locate a performance question; it does not automatically tell you the right optimization.

### 6. What is a React commit?

A commit is a batch of React work applied to the screen. In a Profiler recording, select the commit associated with the interaction you want to understand, then inspect the components that rendered in that commit.

### 7. Does every component render indicate a performance problem?

No. A component may need to render because its state, props, or context changed. Investigate a render when it contributes to a measurable slowdown or does unnecessary work. Optimize after profiling identifies a real cause.

### 8. How is React Profiler different from the browser Performance panel?

React Profiler focuses on React component rendering and commits. The browser Performance panel can show broader work such as JavaScript execution, layout, paint, and event handling. Use the tool that covers the suspected bottleneck; sometimes the issue spans both.

### 9. Is profiling free of overhead?

No. Profiling adds measurement overhead. The normal production build may omit profiling instrumentation, so compare measurements in a consistent environment and use an appropriate profiling build when production-like React timings are needed.

## Debugging workflow

### 10. What should I do first when the UI is wrong?

Reproduce the issue and note the exact route, action, data, and environment. Then read the first relevant console error or warning, since later failures may follow from it.

### 11. Why should I use the component stack in a warning?

It helps identify which React components were involved when the problem appeared. Use that stack to inspect props, state, and parent data instead of guessing from the visible DOM alone.

### 12. When should I use a source breakpoint?

Use one when you need to inspect values at a specific point, such as an event handler, effect, render calculation, or response handler. A breakpoint can show the current call stack and local values without adding temporary logging throughout the component.

### 13. What are source maps useful for?

They map transformed or bundled code shown in browser developer tools back to the source files you authored. This makes breakpoints and stack traces more useful during development. Production source maps should follow the application's deployment and security policy.

### 14. Should I fix several warnings at once?

Prefer one targeted change at a time. That keeps the connection between the warning, the correction, and the observed result clear. If several messages share one cause, fix the shared cause and reproduce the original failure again.

### 15. Why can `console.log` show the old state right after a setter?

A setter requests a future render; it does not replace the state snapshot already captured by the current event handler. Log the next value before setting it if needed, or inspect the state in the next render with DevTools.

## Common React warnings

### 16. What does “Each child in a list should have a unique key” mean?

React needs a stable identity for each rendered sibling so it can match items when the list changes. Give each mapped element a key based on a stable item ID that is unique among those siblings.

### 17. Why can an array index be a bad key?

If items are inserted, removed, or reordered, the same index can refer to a different item. React may then associate state or input values with the wrong row. Use an ID that belongs to the data item instead.

### 18. Why should I not use `Math.random()` for keys?

A random key changes on each render. React treats the element as a new item each time, which can recreate its DOM, lose local state, and do extra work. Create a persistent ID when the item is created instead.

### 19. Does React pass `key` to the child as a prop?

No. React uses `key` for reconciliation. If the child needs the same ID, pass it separately as a normal prop, such as `taskId={task.id}`.

### 20. What causes an “Invalid Hook call” error?

Common causes include calling a Hook outside the top level of a function component or custom Hook, incompatible React and renderer versions, or more than one React copy in the application. Check the call site and dependency tree before changing unrelated code.

### 21. What is the correct place to call a Hook?

Call it at the top level of a function component or custom Hook, in the same order on every render. Do not place it inside a condition, loop, event handler, nested callback, or after an early return.

### 22. How do I check for incompatible or duplicate React packages?

Inspect the dependency tree with `npm ls react react-dom`. Check linked local packages and libraries too. A component and its renderer must resolve to the same React module instance for Hooks to work correctly.

### 23. What does “uncontrolled input to controlled input” mean?

The input first gets its value from the DOM, then later gets a `value` or `checked` prop from React. This often happens when the controlled value starts as `undefined` or `null` and later becomes a string or boolean.

### 24. How do I keep a text input controlled?

Initialize its state to a string, such as `''`, and synchronously update that state in `onChange`. If data may be missing, normalize it with a fallback such as `user.name ?? ''`.

### 25. What should I use for an uncontrolled input?

Use `defaultValue` or `defaultChecked` when the DOM should own the value after initialization. Choose controlled or uncontrolled behavior intentionally and avoid switching modes during the input's lifetime.

### 26. Why does React warn about setting state during render?

Render should calculate UI from the current props, state, and context. An unconditional setter during render requests another render and can create a loop. Derive a value instead, or update state in the event or synchronization path that owns the change.

### 27. Why might an Effect seem to run twice after mounting?

With Strict Mode enabled, React performs an extra development-only setup and cleanup cycle to expose Effects that do not clean up properly. This is a diagnostic check; it is not the same as production repeating every Effect.

### 28. Should I remove Strict Mode when an Effect runs twice?

Usually not. First make the Effect's cleanup undo its setup, and check whether the work belongs in an event handler rather than an Effect. A “run once” ref can hide the missing cleanup and leave the production bug in place.

### 29. What causes a hydration mismatch?

The server-rendered HTML and the client's initial render differ. Common causes include time or random values, locale formatting, browser-only branches, changing external data, and invalid HTML nesting. Make the initial server and client output consistent.

### 30. What should I do with a minified production React error code?

Keep the code and stack trace as evidence, then reproduce the problem in a development build where React can provide a fuller message and component context. Trace the failing component and inputs rather than guessing from the compact code alone.

## Error handling and common mistakes

### 31. What does an Error Boundary catch?

It can render fallback UI when a descendant throws during rendering or supported lifecycle work. It does not automatically catch errors in event handlers, ordinary asynchronous callbacks such as timers, server rendering, or the boundary itself.

### 32. How should event-handler errors be handled?

Handle expected failures at the event or service boundary where the application can show a useful message or recovery action. Use an Error Boundary for rendering failures; it is not a substitute for handling form submissions or request rejections.

### 33. Is it okay to suppress a warning with `console.error` filters?

Not as a fix. Suppressing warnings removes evidence and can hide future problems. Correct the source of the warning, and only filter known, intentional noise in a narrowly scoped tooling setup when there is a clear reason.

### 34. How should I react to an unexpected extra render in development?

Check whether Strict Mode is enabled, whether rendering mutates inputs or causes side effects, and whether an Effect has proper cleanup. Use the Profiler if the question is performance. Extra development checks are useful clues, not a reason to assume React is broken.

### 35. What is a reliable debugging habit?

Reproduce one issue, inspect the first useful message and relevant component inputs, make one root-cause correction, and repeat the same path. Keep the warning visible until the actual cause is fixed.
