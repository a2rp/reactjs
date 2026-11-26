# Memoization, useId, useMemo, and useCallback - Q&A

## Memoization basics

### 1. What does memoization mean in React?

It means reusing a calculation result, function identity, or rendered component result while the relevant inputs remain unchanged. It is an optimization to skip some work, not a way to store application state.

### 2. What is the difference between useMemo, useCallback, and memo?

**useMemo** caches a calculated value. **useCallback** caches a function identity. **memo** can let a component skip a parent-caused render when its props compare equal.

### 3. What is useId for?

**useId** generates IDs for relationships such as a label and input or an input and its help text. It is not a memoization Hook and not a list-key generator.

### 4. Do components need to be memoized by default?

No. Most components render quickly, and React rendering is expected. Add memoization for a specific measured performance issue or identity-sensitive consumer, not automatically everywhere.

### 5. Does memoization make the initial render faster?

Usually no. **useMemo** calls the calculation on the initial render. It can avoid repeating that calculation on a later render when dependencies remain equal.

### 6. Is memoization a correctness guarantee?

No. It is a performance optimization. Code should remain correct if React recalculates a value, creates a new callback, or renders a memoized component again.

## useMemo

### 7. What does useMemo return?

It returns the result of calling its calculation function on the initial render. Later, it returns the cached result while every dependency is equal to the previous dependency by **Object.is**.

### 8. What is the basic useMemo syntax?

**useMemo(calculateValue, dependencies)**. The calculation function should be pure, take no arguments, and return the value to cache.

### 9. What belongs in a useMemo dependency list?

Every reactive value read by the calculation: props, state, and component-local variables or functions. The list should be inline, have a constant number of items, and follow the Hooks linter’s dependency guidance.

### 10. What happens when a useMemo dependency changes?

React calls the calculation again during rendering, stores its result, and returns the new value.

### 11. What happens if the useMemo dependency list is omitted?

The calculation runs on every render. To cache between renders, provide the dependencies as the second argument.

### 12. Can useMemo be called conditionally or in a loop?

No. It is a Hook and must be called at the top level of a component or custom Hook. Extract a component when work needs to be associated with each item in a loop.

### 13. Should a useMemo calculation mutate its inputs?

No. Keep it pure. For example, copy an array before sorting it rather than calling **sort()** on a prop array directly.

### 14. Why might a useMemo calculation run twice in development?

Strict Mode may call it twice to expose impure calculations. React ignores one result. A pure calculation should behave correctly either way.

### 15. Can my code depend on React keeping a useMemo cache forever?

No. The cache is a performance aid, and React may discard it in specific situations. If correctness or resource lifetime depends on the cached object existing, use the right state, ref, or Effect design instead.

### 16. What are reasonable cases for useMemo?

A calculation that profiling shows is noticeably expensive and whose inputs often stay the same; a value whose stable identity enables a memoized consumer to skip work; or a memoized value needed by another Hook after simpler alternatives are considered.

### 17. Should I use useMemo for every array filter?

No. Filtering small arrays is usually inexpensive. Memoize only when there is a clear performance or identity benefit.

### 18. Can useMemo be used to prevent an Effect from running too often?

Sometimes stable identity is useful, but first consider restructuring the Effect. Often the simpler fix is to create an object inside the Effect and depend on the primitive values it needs. Do not make the Effect’s correctness depend on a memo cache.

## useCallback

### 19. What does useCallback cache?

It caches the function itself and returns the same function identity while its dependencies remain equal. It does not call the function.

### 20. What is the basic useCallback syntax?

**useCallback(fn, dependencies)**. React returns **fn** on the initial render and may reuse that function on later renders while its dependencies are unchanged.

### 21. How is useCallback related to useMemo?

Conceptually, **useCallback(fn, deps)** is similar to **useMemo(() => fn, deps)**. **useCallback** is a shorter API for caching a function identity.

### 22. What belongs in a useCallback dependency list?

Every reactive value captured by the callback, including props, state, and component-local values or functions. Use the state setter form when it lets the callback avoid reading the current state value.

### 23. When can a stable callback help?

When passing it to a memoized child that can skip work if the callback prop stays equal, or when the callback itself is a dependency of another Hook.

### 24. Does every event handler need useCallback?

No. Creating a function during render is usually inexpensive and harmless. Add **useCallback** only when stable identity solves a concrete problem.

### 25. Does useCallback prevent the callback’s logic from running?

No. It only caches the function object. The function runs when it is called, such as when a user activates an event handler.

## memo and identity

### 26. What does memo do?

It creates a component that React may skip rendering when its props have not changed according to the comparison. By default, React compares each prop using **Object.is**.

### 27. Can a memoized component still render?

Yes. Its own state can change, consumed context can change, props can differ, and React may render it for other reasons. **memo** is not a never-render guarantee.

### 28. Why can a fresh object prop defeat memo?

Each object literal creates a new object identity. Even if its contents look the same, **Object.is** treats the two objects as different references.

### 29. Can a fresh callback prop defeat memo?

Yes. A new inline function has a new identity on each render. If stable identity matters to a memoized consumer, consider **useCallback** after confirming that it helps.

### 30. Is a custom memo comparison always safe?

No. It must compare every prop that affects output, including function props. Ignoring a changed callback can leave a child using a stale closure. Prefer the default comparison unless profiling justifies a custom one.

### 31. Does memo stop context updates from reaching a component?

No. A component that reads a context will update when that context value changes, regardless of **memo**.

### 32. Why do useMemo and useCallback often appear with memo?

They can keep array, object, or callback props stable so the memoized child’s shallow comparison can succeed. The full prop set must still remain equal for React to skip that render.

## useId and accessibility

### 33. How should I use useId in a form control?

Call it at the top level and use the returned ID to connect the label with **htmlFor** and the input’s **id**. It can also connect help or error text with **aria-describedby**.

### 34. Does useId take a parameter?

No. **useId()** takes no parameters and returns an ID string associated with that Hook call in that component instance.

### 35. Can I call useId inside a map to create each row’s ID?

No. Hooks cannot be called in loops. Extract a child component for each row; call **useId** at the top level inside that child if the row needs accessibility IDs.

### 36. Can useId generate keys for a list?

No. Keys identify data items among siblings and should come from stable data, such as **product.id**. A generated accessibility ID has a different job.

### 37. Is useId suitable for application data IDs or cache keys?

No. Use identifiers from the data or a domain-appropriate ID generator. **useId** is intended for UI accessibility relationships, not data identity or caching.

### 38. Does useId work with server rendering and hydration?

It is designed to produce matching IDs when the server and client render the same component tree. If the trees differ, the underlying hydration mismatch must be fixed.

### 39. What if a page contains multiple independent React roots?

Pass a distinct **identifierPrefix** to each root’s **createRoot** call to avoid generated ID collisions. With server rendering, use a matching prefix on the server and client hydration calls.

## React Compiler and design

### 40. Does every React app automatically use React Compiler?

No. It must be configured in the project build. When enabled and compiling components, it can automatically memoize many values, functions, and components, reducing the need for manual **useMemo**, **useCallback**, and **memo**.

### 41. Should I still understand manual memoization if React Compiler is used?

Yes. It helps explain identity, dependencies, and performance. But check compiler setup and diagnostics before adding manual memoization everywhere.

### 42. Is useId affected by React Compiler memoization?

No. Its purpose remains generating IDs for accessibility relationships. It should not be used for list keys.

### 43. What should I do before adding memoization to a slow interaction?

Profile the interaction, identify which calculation or component work is actually expensive, and then test a focused optimization. Also check for unnecessary Effects, state lifted too high, or impure rendering.

### 44. What is the quick rule for choosing among these APIs?

Use **useMemo** for a calculated value, **useCallback** for a function identity, **memo** for a component that can skip some renders, and **useId** for accessibility IDs. Use none of them unless their specific purpose is needed.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="059-refs-and-imperative-handles.md">059 - React 19 Refs, forwardRef, and Imperative Handles - Q&amp;A</a></td><td><a href="../lessons/060-memoization-useid-usememo-usecallback.md">Lesson</a></td><td><a href="../examples/060-memoization-useid-usememo-usecallback.md">Example</a></td><td><a href="061-rendering-and-committing.md">061 - Rendering, Committing, and Updating the UI - Q&amp;A</a></td></tr>
  </tbody>
</table>
