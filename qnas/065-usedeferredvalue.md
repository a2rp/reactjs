# useDeferredValue - Q&A

## Basics

### 1. What does useDeferredValue do?

It returns a deferred version of a value so a slower UI consumer can update at a lower priority than urgent parts of the interface.

### 2. What arguments does useDeferredValue accept?

It accepts the value to defer and an optional initial value for the first render.

### 3. Where can I call useDeferredValue?

Call it at the top level of a function component or custom Hook. Do not call it inside an event handler, condition, or loop.

### 4. What does it return on the initial render?

Without an explicit initial value, it returns the same value that was passed in. If an initial value is supplied, that value is returned for the initial render.

### 5. Is useDeferredValue a state setter?

No. It returns a deferred value. Update the source state with its normal setter, then pass the source value to **useDeferredValue**.

### 6. Can the value be an object?

Yes. It can have any type, but object identity matters. Avoid passing a newly created object on every render because React sees it as a new value.

### 7. How does React decide whether the value changed?

React compares the previous and next value with **Object.is**.

## Urgent and deferred rendering

### 8. How should I use it with a controlled search field?

Keep the input controlled by the immediate state. Pass the deferred query to the slower result view.

### 9. Why should the input not use deferredQuery as its value?

The input should reflect each keystroke promptly. If the input uses the lagging value, it can feel delayed or behave incorrectly.

### 10. What happens after the source value changes?

React can first render urgent UI while returning the previous deferred value, then attempt a background render using the new value.

### 11. Is the background render interruptible?

Yes. A newer urgent update can interrupt or discard an in-progress background render. React can later retry using the latest value.

### 12. Is there a fixed delay before the deferred value changes?

No. The Hook has no fixed millisecond delay. React schedules background rendering according to its current work and priorities.

### 13. Will the value always visibly lag behind?

No. On a fast device or with a small UI, the deferred value may catch up before a visible stale period appears.

### 14. Does the Hook make a slow component render faster?

No. It changes when React attempts the lower-priority render. The component’s work still takes the time it takes.

### 15. Can React interrupt one long synchronous calculation inside that component?

No. React cannot interrupt a single JavaScript function halfway through. Optimize the calculation, split the work, use a worker where suitable, or apply a large-list technique.

### 16. How can I tell whether content is stale?

For comparable primitive values, compare the current and deferred values, such as **query !== deferredQuery**.

### 17. What should the stale indicator say?

Describe the displayed content as catching up or out of date. Do not say a request is loading unless a request is actually pending.

### 18. Should I hide stale content while the new render is prepared?

Usually keep it visible if it is still useful. Use a subtle stale style or local status rather than replacing the whole region without a reason.

## Memoization and child rendering

### 19. Why is memo commonly used with useDeferredValue?

The parent can render urgently with the old deferred prop. A memoized slow child can skip that render when its props have not changed.

### 20. What happens if the slow child is not memoized?

It may render during the urgent parent render anyway, even though its deferred value is still old. That can defeat the common responsiveness pattern.

### 21. Is memo always required?

No. It is useful for the common case where the child is genuinely slow and should skip a render when its props are unchanged. It is not a universal requirement for correctness.

### 22. Can a new callback prop defeat memo?

Yes. A new callback has a different identity, so a memoized child may render even while the deferred value itself is unchanged. Keep relevant props stable when the optimization depends on them staying equal.

### 23. Can a new array prop defeat memo?

Yes. Creating a new array during every parent render changes its identity. Derive it in the memoized child from the deferred primitive, or stabilize it appropriately.

### 24. Should I wrap every value in useMemo before useDeferredValue?

No. Use **useMemo** only when you need a stable derived object. If a string or number expresses the value to defer, pass that primitive directly.

### 25. What if I need to defer several related values together?

You can defer a stable object containing them, or defer the individual values. Ensure the object is not recreated unnecessarily and keep values that must stay consistent together in one stable object.

## Object values and initial value

### 26. Why is this problematic: useDeferredValue({ query })?

That object is newly created during each render. Since its identity changes, React can repeatedly schedule background renders.

### 27. How can I defer an object safely?

Create it outside rendering if it is constant, or derive it with **useMemo** from stable dependencies before passing it to **useDeferredValue**.

### 28. When should I pass an initialValue?

When the first render should intentionally use a different value in the deferred region, such as a selected initial dataset. It is not needed for the usual controlled-input pattern.

### 29. Does the initialValue remain in use forever?

No. It applies to the initial render. Subsequent updates follow the normal deferred-value behavior.

### 30. Can initialValue be an object?

Yes, but keep its identity intentional and make sure the initial content is appropriate for the component.

## Suspense and data

### 31. How does useDeferredValue interact with Suspense?

If rendering with the new deferred value suspends, React can keep showing the old deferred content instead of replacing already revealed UI with the fallback.

### 32. Does useDeferredValue eliminate Suspense fallbacks?

No. A nested boundary or content that has not been revealed may still display its fallback. The result depends on boundary placement and what is already visible.

### 33. Does passing a query to useDeferredValue make normal fetch code suspend?

No. A deferred value does not turn a regular request into a Suspense-enabled data source. The data source must support Suspense for that behavior.

### 34. Will Effects run for a background render that is abandoned?

No. Effects run only for a render that commits. An interrupted or discarded background attempt does not run its Effects.

### 35. Does the Hook prevent extra network requests?

No. It defers rendering the consumer. Code that triggers requests may still issue requests for values it processes; use a separate request policy when needed.

### 36. Does it cancel stale requests?

No. Handle cancellation or stale responses separately with an appropriate request strategy.

## Choosing between related tools

### 37. How is useDeferredValue different from useTransition?

Use **useDeferredValue** when a slower consumer should lag behind a value it receives. Use **useTransition** when you control the state setter and want to mark that update as non-urgent, with an optional pending flag.

### 38. Should I combine useDeferredValue and useTransition for the same update?

Usually not. If the value is already being updated inside a Transition, **useDeferredValue** returns the new value without starting another deferred render.

### 39. How is useDeferredValue different from debouncing?

Debouncing waits for a chosen quiet interval before running work. **useDeferredValue** has no fixed timer and lets React schedule interruptible rendering based on current priorities.

### 40. How is it different from throttling?

Throttling limits how often an operation runs over time. **useDeferredValue** changes rendering priority and does not impose an update-rate limit.

### 41. Can I combine deferring with debouncing?

Yes, when the app has two separate needs: fewer requests and more responsive rendering. Debounce request work and use React scheduling for the UI as appropriate.

### 42. When should I choose neither?

If the UI is already responsive and there is no slow consumer, extra deferred state may add complexity without helping the user.

## Practical troubleshooting

### 43. What if the deferred value seems to update immediately?

That can be expected when the device and render are fast. The Hook does not promise a visible pause.

### 44. What if typing still feels slow?

Check whether the controlled input’s own update is urgent, whether the slow child can skip the urgent render, and whether another unstable prop is forcing it to render.

### 45. What if the results render twice?

The parent may render urgently with the old value and then render again with the new deferred value. That is the intended pattern when the input changes.

### 46. What if the list keeps rendering on every input update?

Check that the slow child is memoized and its other props remain stable while the deferred value is unchanged.

### 47. What if a slow synchronous filter still blocks typing?

A single synchronous filter function cannot be interrupted halfway through. Optimize or split the work, move suitable computation to a worker, or use a more suitable data and list strategy.

### 48. What if stale results are mistaken for current results?

Show a local status or subtle stale treatment while the source and deferred values differ. Keep the label accurate and accessible.

### 49. What if useDeferredValue receives a context value?

It can defer any value the component can read, but consider whether that consumer should be isolated and memoized so urgent parent work can skip its slow rendering.

### 50. What is the short mental model?

Keep the source state current. Let a slower consumer use the previous value briefly while React prepares an interruptible render with the latest value.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="064-transitions-and-usetransition.md">064 - Transitions with useTransition and startTransition - Q&amp;A</a></td><td><a href="../lessons/065-usedeferredvalue.md">Lesson</a></td><td><a href="../examples/065-usedeferredvalue.md">Example</a></td><td><a href="066-suspense-boundaries-and-fallbacks.md">066 - Suspense Boundaries and Fallbacks - Q&amp;A</a></td></tr>
  </tbody>
</table>
