# Memoization and Optimization Decisions - Q&A

## Fundamentals

### 1. What does memo do?

memo returns a component that can skip some renders caused by its parent when the new props compare equal to the previous props. It is a performance optimization, not a guarantee that the component never renders.

### 2. How are props compared by default?

React compares each prop with Object.is. This is a shallow comparison of prop values. Equal primitive values compare equal; separately created objects and functions compare different because their references differ.

### 3. Does memo prevent every render?

No. The component can still render when its own state changes, a context it reads changes, or React otherwise needs to render it. memo only addresses some parent-driven renders when props are unchanged.

### 4. Can I rely on memo to make my component correct?

No. The component must behave correctly without memo. Fix missing updates, mutation, impure rendering, or incorrect Effect logic directly.

### 5. Does a render mean React changed the DOM?

No. A render means React called component logic to determine the next UI. React may find that the DOM does not need to change.

### 6. Should every component be wrapped with memo?

No. It is usually unnecessary for components that are already fast or whose props change on every render. Add it when a profiler shows repeated expensive work with unchanged props.

### 7. What is a good candidate for memo?

A component that renders often because its parent updates for unrelated reasons, receives stable props, and performs enough work that skipping the render is measurable.

### 8. What if a memoized component's prop is an object?

The default comparison checks the object's reference, not its fields. A new object with the same contents is still a different prop value to Object.is.

### 9. What if a prop is a function?

Function props are compared by reference too. A new function created during the parent render is considered a changed prop even if its body looks the same.

### 10. Does passing fewer props always improve performance?

Not automatically. Passing focused props can make a component interface clearer and can help a memo boundary when those values remain stable. Do not distort a good API solely to avoid an unmeasured render.

## Choosing an API

### 11. What is the difference between memo, useMemo, and useCallback?

memo can skip some renders of a component when its props are unchanged. useMemo caches the result of a calculation. useCallback caches a function definition's identity. These APIs solve related but different problems.

### 12. When should I use useMemo?

Consider it for a calculation that is measurably expensive, or when a value's stable identity is needed by another optimization or Hook. The calculation must be pure and its dependency list must include every reactive value it reads.

### 13. Should I use useMemo for every calculation?

No. Most calculations are cheap, and memoization adds work and complexity. Measure first. Directly computing a simple value is often clearer.

### 14. When should I use useCallback?

Consider it when stable function identity helps a memoized child or is needed as a dependency of another Hook. Ordinary event handlers do not need useCallback by default.

### 15. Does useCallback call the function?

No. useCallback returns the function you passed. Your event handler, child component, or other code decides when to call it.

### 16. Can useMemo or useCallback be used as state storage?

No. Their caches are optimization details and React can discard them in specific circumstances. Use state for values that drive UI and refs for mutable values that do not drive UI.

### 17. Can I call these Hooks inside a loop or condition?

No. Call Hooks at the top level of a component or custom Hook. If different list entries need independent Hook state, extract a component for each entry.

### 18. What is a dependency list?

It lists the reactive values read by the memoized calculation or callback. React compares each dependency with Object.is. The list must be complete; omitting a value can make the calculation or callback use stale data.

### 19. Can an updater function reduce callback dependencies?

Sometimes. If a callback only needs the previous state to compute its next value, use the state updater form. Then the callback may not need to close over that state value.

### 20. What happens if an object dependency is recreated every render?

Its reference changes, so React treats it as a changed dependency and recalculates the memoized value. Often the better fix is to create the object inside the calculation or simplify the dependency, if that matches the intended behavior.

## Prop comparison and component boundaries

### 21. What does a custom arePropsEqual comparator do?

It receives previous and next props and returns true only when the component would produce the same output and behavior with either set. Returning false tells React that the props changed.

### 22. Why are custom comparators risky?

They can be incomplete or more expensive than rendering. If they ignore a changed prop, the component may show stale output or keep an outdated callback. A deep comparison can also grow costly as data changes.

### 23. Must a custom comparator compare function props?

Yes, unless you can prove the function identity and captured values do not affect behavior. Ignoring a changed callback can preserve a closure with stale parent values.

### 24. Should I use a deep equality comparison?

Usually not. Deep comparisons may be slower than rendering and can become fragile as data structures change. Prefer the default comparison unless profiling justifies a carefully scoped custom comparator.

### 25. Does a context update bypass memo?

A component that reads a changed context still needs to update. If only a small piece of UI needs the context value, split the component so a small outer component reads context and passes focused props to an inner component.

### 26. Does a component's own state update bypass memo?

Yes. memo compares parent-provided props; it does not freeze the component's own state.

### 27. Can passing JSX children help avoid unnecessary rendering?

It can help structure updates. A wrapper that renders its children passed from a parent can update its own local state while React can preserve child work where appropriate. Choose this pattern when it fits the component's purpose, not as a blind optimization.

### 28. Why should I avoid defining a component inside another component?

A nested component definition creates a new component type whenever the parent renders. That can cause React to remount the nested component, reset its state, and redo work. Define component types at module scope.

### 29. Should I pass an entire data object or separate fields?

Choose the interface that best represents what the child needs. Separate fields may make dependencies and comparisons clearer; an object is natural when the child genuinely consumes that object. Memoization alone should not dictate the data model.

## Profiling and compiler

### 30. How do I know whether memoization helped?

Repeat the same realistic interaction and compare profiling results before and after. Check whether the targeted component did less work and whether the interaction improved. Keep the optimization only if the result justifies its complexity.

### 31. Should I optimize because a component renders many times?

Not from render count alone. Many renders are inexpensive, and a render does not necessarily cause a DOM update. Look at render duration and the actual user-visible delay.

### 32. Does React Compiler make memo, useMemo, and useCallback obsolete?

The compiler automatically memoizes eligible components, values, and functions, reducing the need for manual calls. The APIs remain supported. Follow the compiler setup and diagnostics for the project's toolchain.

### 33. Should I remove all existing manual memoization after enabling the compiler?

Do not remove everything mechanically. Review why each optimization exists, verify behavior, and profile meaningful interactions. Simplify code when the compiler makes a manual boundary unnecessary and the change remains correct.

### 34. Does manual memoization still make sense with the compiler?

It can, especially for a documented API or a measured case the compiler does not address. Prefer the simplest correct implementation and use diagnostics and profiling to guide decisions.

## Common mistakes

### 35. Why did a memoized child render after I added memo?

Check whether any prop is a new object, array, or function. Also check whether the child updates its own state, reads changed context, or is intentionally re-rendered for another reason.

### 36. Why is my useMemo calculation running twice in development?

Strict Mode can call render-related pure calculations more than once in development to help expose accidental impurities. Ensure the calculation has no side effects; do not use a render calculation to perform external work.

### 37. Can memo fix an Effect that runs too often?

Not directly. Effects should synchronize with external systems and declare their true dependencies. Correct Effect design and stable dependencies are separate concerns from skipping a component render.

### 38. What should I optimize first: component renders or calculation time?

Profile the interaction to identify the actual cost. If a component tree repeats work, a component boundary may help. If a specific calculation dominates, optimize that calculation. If time is spent in the browser, network, or server, memoization may not address it.

### 39. What is the safest optimization workflow?

1. Reproduce the slow interaction.
2. Profile it and identify the expensive work.
3. Choose one focused change.
4. Check behavior and correctness.
5. Repeat the same profile.
6. Keep the change only if it provides a useful improvement.

### 40. What is the central rule for memoization?

Memoize for measured performance, not correctness. Keep code simple until real evidence shows where optimization will help.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="108-react-compiler.md">108 - React Compiler - Q&amp;A</a></td><td><a href="../lessons/109-memo-and-optimization-decisions.md">Lesson</a></td><td><a href="../examples/109-memo-and-optimization-decisions.md">Example</a></td><td><a href="110-large-lists-and-virtualization.md">110 - Large Lists and Virtualization - Q&amp;A</a></td></tr>
  </tbody>
</table>
