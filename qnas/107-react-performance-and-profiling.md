# React Performance Fundamentals and Profiling: Questions and Answers

## Measuring the problem

### 1. What should I optimize first?

Start with a user-visible symptom, such as delayed typing, a slow route, or a list that stutters. Reproduce it with representative data, record the interaction, and determine whether React rendering, JavaScript, network activity, or browser layout and paint contributes to the delay.

### 2. Is render count a good performance score?

No. A render can be fast and may not change the DOM. One long calculation can be slow even if it runs once. Measure the work and the user's response rather than trying to minimize every render.

### 3. Which tools should I use?

Use React DevTools Profiler to understand React component rendering. Use browser Performance and Network tools to inspect JavaScript, requests, layout, paint, and loading. Combine the tools when the interaction spans both React and browser work.

### 4. Why test with realistic data?

A tiny development list may make an expensive calculation look instant. Representative data and device conditions help show whether the issue occurs for users and whether the proposed change matters.

### 5. Are development timings representative of production?

Not exactly. Development instrumentation, source maps, and Strict Mode checks can add work. Use development profiles to understand behavior, and compare timing in a production-like or profiling build on representative hardware.

## Rendering and state

### 6. Why can changing one state value render many components?

React evaluates children under an updated component as it calculates the next tree. If frequently changing state is owned high in the tree, a large subtree may be revisited. Profile the interaction and move transient state closer to its owner when unrelated parts do not need it.

### 7. Should I always keep state local?

Keep it at the nearest owner that needs to coordinate the value. A value shared by sibling features may belong in their common parent. The goal is not minimal state sharing at all costs, but avoiding broad update fan-out that the application does not need.

### 8. Why is derived state in an Effect often slower?

The component can render once with the changed source value, run the Effect, update a second state value, then render again. If the value is just calculated from current props or state, derive it during render instead of storing and synchronizing a duplicate.

### 9. When should I use `useMemo`?

Use it when a pure calculation is measurably expensive and can be skipped while its dependencies remain unchanged, or when the stable result is needed by a consumer that benefits from stable identity. It does not make the calculation cheaper when a dependency changes.

### 10. Does `useMemo` make the initial render faster?

Usually not. The calculation still has to run to produce its first value. The benefit is avoiding later recalculations when dependencies remain the same.

### 11. When should I use `useCallback`?

When a stable function identity benefits a consumer, such as a memoized child or a Hook dependency. It does not make the function body execute faster, and it adds dependencies that must remain correct.

### 12. When does `memo` skip a component render?

It can skip a parent-driven render when its props compare equal; by default React compares each prop with `Object.is`. The component can still render when its own state or a context it reads changes, and React treats memoization as an optimization hint rather than a correctness guarantee.

### 13. Why did `memo` not prevent a child render?

Check whether any prop received a new identity, such as an inline object or callback. Also check whether the child uses changing state or context. A new value in any one prop can prevent the default shallow comparison from skipping the render.

### 14. Should I write a custom `memo` comparison function?

Only in a measured, unusual case. It must compare every prop, including functions, and must be faster than rendering the child. A faulty comparison can preserve stale behavior; a deep comparison can cost more than the work it tries to avoid.

### 15. Can manual memoization fix incorrect UI behavior?

No. If the UI only works when a memoization hook is present, find the correctness bug first. `memo`, `useMemo`, and `useCallback` are performance tools, not semantic storage or state-management guarantees.

### 16. What if my project uses React Compiler?

React Compiler can automatically memoize many components, values, and functions when it is configured for the project. Follow the project's compiler setup and inspect its behavior before adding manual memoization everywhere. Compiler configuration and adoption are covered in the following chapter.

## Scheduling responsive work

### 17. What does `useDeferredValue` do?

It lets part of the UI use a less urgent, deferred version of a value. For example, a text input can update immediately while a large result list catches up. The deferred UI may temporarily show results for the previous value.

### 18. Is `useDeferredValue` the same as debouncing?

No. Debouncing waits for a time period before starting work. `useDeferredValue` lets React schedule a render as non-urgent so urgent updates can be shown first. It does not specify a fixed delay.

### 19. What does a transition do?

A transition marks certain state updates as non-urgent and interruptible, allowing more urgent interactions to take priority. It changes scheduling priority; it does not make the underlying synchronous calculation faster or move it to another thread.

### 20. Should the state controlling a text input be updated inside a transition?

No. Keep the controlled input's own value update urgent so typing stays responsive. Defer the downstream results or non-urgent view update instead.

### 21. Will a transition solve a long synchronous calculation?

Not by itself. A long uninterrupted JavaScript calculation can still block the main thread. Reduce the amount of work, split it across tasks where appropriate, or use a worker when the computation can run outside the UI thread.

### 22. Should I add a loading indicator for deferred content?

If users need to know that results are updating or that the displayed results are temporarily stale, communicate that state clearly. For example, compare the current query with the deferred query and use a subtle status. Do not imply that the current results match the new input if they do not.

## Lists, context, and loading

### 23. When does a list need virtualization?

When profiling shows that creating, reconciling, laying out, or painting a large number of rows harms interaction or scrolling. For small lists, rendering all rows is simpler. Virtualization limits the DOM to visible rows and a buffer, but it changes scrolling, focus, accessibility, and browser find behavior.

### 24. Do stable keys improve performance?

Keys primarily preserve item identity across list updates. Stable keys help React match rows after insertion, deletion, or reordering and prevent state from attaching to the wrong row. They are not a general speed switch.

### 25. How can context affect performance?

When a context value changes, its consumers may render to receive the new value. A broad context that changes frequently can notify many components. Split unrelated values or move frequently changing data closer to its consumers when profiling shows unnecessary fan-out.

### 26. How should I investigate a slow first screen?

Inspect the initial JavaScript bundle, request sequence, and browser startup work. Remove unused code, split rarely used routes or features when useful, and avoid waiting on data the first screen does not need. Check loading and error states so the page remains understandable while work continues.

### 27. Does code splitting always improve performance?

No. It can reduce the code needed initially, but it may add a later request or delay a feature until its chunk loads. Choose split points around navigation and feature use, then measure both startup and the later interaction.

### 28. Why might a React render be fast while the page still feels slow?

The bottleneck may be outside React: event-handler code, a network request, data parsing, layout, style calculation, paint, images, or fonts. Correlate the React profile with the browser timeline and Network panel.

## Practical profiling decisions

### 29. What is a good optimization experiment?

Record the original interaction, identify the work causing the symptom, make one targeted change, then repeat the same interaction with the same data and environment. Check both the measured work and the visible result.

### 30. Should I memoize a component just because it appears in the profiler?

No. It may be cheap or may need to render. Memoize only when the component often receives the same props, its render is costly enough to matter, and profiling shows the change improves the interaction.

### 31. Can I use console timing instead of a profiler?

A targeted `console.time` measurement can help explore one calculation. It does not show the whole React tree or total page cost, and logging itself can affect measurements. Use it as a narrow diagnostic and confirm with profiler and browser traces.

### 32. What does `baseDuration` tell me?

It estimates the cost of rendering the whole profiled subtree based on recent component durations. Compare it with `actualDuration` as context about work skipped during an update; treat it as an estimate, not a user's measured latency.

### 33. Is optimizing the highest-duration component always the right choice?

Not necessarily. Check whether it lies on the user-visible critical path, how often it runs, and whether its work delays an important response. A smaller cost repeated on every keystroke can matter more than a larger one during a rare background interaction.

### 34. What is the main rule for React performance work?

Measure the interaction that feels slow, identify which kind of work causes it, make one targeted change, and compare under the same conditions. Keep the simplest solution that improves the user's actual experience.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="106-profiling-during-development.md">106 - Profiling Component Behavior During Development: Questions and Answers</a></td><td><a href="../lessons/107-react-performance-and-profiling.md">Lesson</a></td><td><a href="../examples/107-react-performance-and-profiling.md">Example</a></td><td><a href="108-react-compiler.md">108 - React Compiler - Q&amp;A</a></td></tr>
  </tbody>
</table>
