# React Performance Fundamentals and Profiling

## Learning goals

After this lesson, you should be able to:

- Describe performance in terms of the user's interaction, not only component render counts.
- Find whether a slow interaction is caused by React rendering, JavaScript, the network, or browser rendering.
- Reduce unnecessary work by keeping state local and avoiding redundant update chains.
- Apply `memo`, `useMemo`, and `useCallback` only when a measured case benefits from them.
- Keep urgent input responsive while deferring non-urgent rendering work.
- Recognize when a long list or large initial bundle needs a different strategy.
- Compare an optimization against the same baseline and data.

## 1. Start with a user-visible performance problem

Performance includes more than React render duration. A page can feel slow because JavaScript blocks input, a request takes time, the browser performs expensive layout or paint, or too much code must load before the user can interact.

Start with a concrete symptom:

- Typing into search feels delayed.
- Opening a menu takes too long.
- A route shows content late because it waits on several requests.
- Scrolling a large list stutters.
- The first usable screen takes too long to load.

Reproduce the same interaction with representative data. Use React DevTools Profiler for React render work and browser Performance and Network tools for the rest of the page. Chapter 106 covers how to record and read those traces during development.

A useful investigation answers three questions:

1. What does the user observe, and when?
2. Which work occurs between the action and the visible response?
3. Which smallest change removes work that is unnecessary for that response?

Do not treat the number of renders as a performance score. A render may be fast, and one long calculation can be slow even if it happens only once.

## 2. Keep rendering pure and update chains short

A component should calculate UI from props, state, and context. It should not mutate external data or perform side effects while rendering. Pure rendering is easier for React to repeat, interrupt, and measure.

Avoid an Effect that copies a value from one state variable into another when the second value can be calculated during render:

```jsx
// Unnecessary extra state and update
const [query, setQuery] = useState('');
const [visibleItems, setVisibleItems] = useState(items);

useEffect(() => {
  setVisibleItems(filterItems(items, query));
}, [items, query]);
```

Prefer deriving it directly:

```jsx
const [query, setQuery] = useState('');
const visibleItems = filterItems(items, query);
```

If a calculation is genuinely expensive and profiling shows it runs unnecessarily, `useMemo` can cache its result between renders. Do not use state plus an Effect as a substitute for a pure calculation.

## 3. Keep transient state near its owner

When a state value changes, its owning component renders again. React then evaluates its children unless it can skip some work. A form field's draft, an open menu, or a selected tab often belongs near the UI that owns that interaction.

If typing into one small field updates state in a page-level component with a large unrelated subtree, the profile may show broad work for every keystroke. Move that state into a focused child when other regions do not need it. Pass values or callbacks down only where they are used.

This does not mean all state must be local. Shared state belongs at the nearest common owner that needs to coordinate it. Context and external stores are useful when appropriate, but a frequently changing broad context can notify many consumers. Split unrelated context values or move high-frequency state closer to its consumers when a profile identifies that fan-out as a cost.

## 4. Understand `memo`, `useMemo`, and `useCallback`

These APIs are performance tools. They should not be required for the UI to be correct.

### `memo`

`memo` can skip a parent-driven re-render when the component receives props that compare equal. By default React compares each prop with `Object.is`. A memoized component still renders when its own state or a context it uses changes, and memoization is a hint rather than a guarantee.

```jsx
import { memo } from 'react';

const ResultRow = memo(function ResultRow({ item, onSelect }) {
  return (
    <button type="button" onClick={() => onSelect(item.id)}>
      {item.label}
    </button>
  );
});
```

This helps only if the parent often renders while the row's props remain stable and rendering the row is worthwhile to skip. If the parent creates a new `item` object or `onSelect` function each time, the props may no longer compare equal.

### `useMemo`

`useMemo` caches the result of a pure calculation until one of its dependencies changes:

```jsx
const visibleItems = useMemo(
  () => filterItems(items, query),
  [items, query],
);
```

This does not make a changed query cheaper; the calculation still needs to run when `query` changes. It can avoid recalculating for unrelated renders when `items` and `query` are unchanged.

### `useCallback`

`useCallback` caches a function identity between renders while its dependencies remain unchanged:

```jsx
const handleSelect = useCallback((itemId) => {
  setSelectedId(itemId);
}, []);
```

It can be useful when passing a callback to a memoized child or when function identity matters to a Hook dependency. It does not make the callback's work faster when it is invoked.

### Measure before keeping a memoization change

Memoization adds dependency and identity rules to reason about. A custom comparison function can itself cost more than rendering, and an incorrect comparison can preserve stale behavior. Use a memoization tool only when a profile identifies repeated work that matters, then compare the same interaction afterward.

When React Compiler is configured, it can automatically memoize many components, values, and functions. Chapter 108 covers the compiler. Avoid layering manual memoization without checking how the project is configured and what the profile shows.

## 5. Keep urgent updates responsive

Some updates should reflect immediately, such as typing into a controlled input. Other work, such as rendering a large filtered result set, may be allowed to lag briefly behind the input.

`useDeferredValue` lets a less urgent part of the UI use a deferred version of a value:

```jsx
const [query, setQuery] = useState('');
const deferredQuery = useDeferredValue(query);
const results = useMemo(
  () => filterLargeDataset(items, deferredQuery),
  [items, deferredQuery],
);
const isShowingPreviousResults = query !== deferredQuery;
```

The input should still use `query`, so typing remains urgent. The results can continue showing the previous query briefly while React works on the deferred render. The UI can communicate that it is updating if the distinction matters.

`useTransition` and `startTransition` let you mark certain state updates as non-urgent. A transition is interruptible rendering work; it does not move a long synchronous JavaScript calculation off the main thread. If the calculation itself blocks for a long time, reduce the work, split it, or consider a worker.

Do not mark the controlled input's immediate state update as a transition. Keep the input responsive and defer the downstream content that does not need to block it.

## 6. Handle large lists deliberately

A list with a few dozen rows is often fine. A very large list can cost time to create React elements, reconcile, lay out, and paint, even if each row is simple.

For large or frequently updated lists:

- Profile with realistic record counts and row content.
- Keep stable data keys so row identity survives inserts, deletes, and reordering.
- Avoid rendering data the user cannot currently see when the list size makes that work significant.
- Consider list virtualization or windowing so the DOM contains only visible rows and a small buffer.
- Preserve keyboard access, focus, screen-reader context, and expected scrolling when virtualizing.

Virtualization changes how content exists in the DOM and can affect accessibility and browser find behavior. It is a trade-off, not a default for every list.

## 7. Reduce initial loading work

A React interface can render quickly after its code loads while still taking too long to become usable. Inspect the browser Network and Performance panels for large JavaScript bundles, duplicate dependencies, slow requests, and work that blocks startup.

Useful steps include:

- Remove unused dependencies and imports that increase shipped code.
- Split routes or rarely used features so they load when needed.
- Avoid blocking the first screen on data it does not need.
- Give loading and error states a clear, usable interface.
- Cache or deduplicate data requests using the application's established data layer.

React's lazy loading and Suspense are covered earlier in the course. Choose code splitting around user navigation and product boundaries, then measure its effect on startup and navigation.

## 8. Use a repeatable optimization loop

For every proposed optimization:

1. Record the original behavior with the same user action and representative data.
2. Identify whether React rendering, calculation, request latency, or browser work contributes to the delay.
3. Change one cause, such as moving state, removing redundant derived state, reducing a list, or caching an expensive pure calculation.
4. Repeat the same recording under the same conditions.
5. Check the visible behavior, not only the timing number.
6. Keep the change only if it improves the user's experience without adding disproportionate complexity.

For more representative timing, test a production or profiling build on hardware similar to the target devices. Development traces are excellent for understanding component behavior, but development instrumentation and Strict Mode checks add work.

## 9. Common performance mistakes

- **Adding `memo` everywhere:** first identify a repeated, expensive render with a profile.
- **Memoizing a calculation that is already fast:** caching and dependency maintenance may add more complexity than value.
- **Using `useCallback` without a consumer that benefits from stable identity:** the callback body does not become faster.
- **Keeping derived data in state:** this often creates an extra render from an Effect.
- **Lifting input state too high:** unrelated regions may render on every keystroke.
- **Stabilizing props while passing a new object elsewhere:** one changing prop can still defeat memoization.
- **Treating transitions as faster computation:** they change scheduling priority, not the cost of synchronous work.
- **Virtualizing a small list:** it adds complexity without a measured need.
- **Optimizing React while the delay is on the network or in layout:** use the browser timeline to find the actual boundary.
- **Comparing different devices, data, or build modes:** repeat the same experiment under consistent conditions.

## Summary

React performance work begins with a user-visible symptom and a repeatable profile. Keep render pure, avoid Effects that synchronize derived state, and place transient state near the interaction that owns it. Use `memo`, `useMemo`, and `useCallback` only for profiled cases; transitions prioritize work but do not make synchronous calculations cheaper. For large lists and startup costs, measure DOM and bundle work as well as React rendering, then compare one targeted change against the same baseline.
