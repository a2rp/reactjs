# Transitions with useTransition and startTransition - Q&A

## Hook and API basics

### 1. What does useTransition do?

It lets a component mark some state updates as non-urgent Transition updates and read whether those updates are pending.

### 2. What does useTransition return?

It returns exactly **[isPending, startTransition]**. The first value is a boolean pending status. The second starts an Action that marks scheduled state updates as Transitions.

### 3. Does useTransition take arguments?

No. Call it without arguments at the top level of a component or custom Hook.

### 4. Can I call useTransition inside a click handler?

No. Hooks must be called at the top level of a component or custom Hook. Call the Hook during rendering, then call its returned **startTransition** function from the event handler.

### 5. What is an Action?

An Action is the function passed to **startTransition**. It can schedule state updates and can be asynchronous.

### 6. Does startTransition delay its callback?

No. React calls the callback immediately. The API assigns Transition priority to the state updates scheduled during the Action; it is not a timer.

### 7. Does startTransition return a value?

No. The **startTransition** function does not return a value. Use **isPending** from **useTransition** when the component needs a pending indicator.

### 8. When should I use the standalone startTransition function?

Use it when you can access a state setter but cannot call a Hook at that location, such as in a utility function or a data library. It does not provide component-level **isPending**.

### 9. Can I transition an update if I do not have its setter?

No. **startTransition** must wrap code that schedules the update, and you need access to that setter. If you only receive a changing prop, consider **useDeferredValue** in the slower consumer.

### 10. Can I pass isPending to a child?

Yes. A component can pass its pending status as a prop to the affected controls or view. Keep the status near the interaction it describes.

## Choosing what to transition

### 11. What updates are good candidates for a Transition?

Non-urgent updates that cause slower rendering, such as selecting a large tab panel, changing a chart, navigating to a page, or displaying a substantial result view.

### 12. Should I put a controlled input’s value update in a Transition?

No. A controlled input should update synchronously so it reflects each keystroke. Keep that state urgent and defer or transition the slower consumer instead.

### 13. Can I transition the results shown while typing?

Yes. Keep the input state urgent, then use **useDeferredValue** for the results consumer or store a separate results state that is updated in a Transition.

### 14. What if the value comes from props and I cannot access its setter?

Use **useDeferredValue** in the component that consumes the value slowly. It lets that consumer lag behind without requiring control over the original setter.

### 15. Does putting a calculation inside startTransition make the calculation interruptible?

No. A single long synchronous calculation still blocks the main thread until it returns. A Transition changes the priority of React state rendering; optimize or split the calculation, or move suitable work to a worker.

### 16. Should I wrap every state update in startTransition?

No. Keep immediate feedback urgent. Mark only the updates that can wait without making the interface feel unresponsive.

## Pending UI

### 17. What does isPending mean?

It indicates that a Transition started with that Hook still has pending work. It is a boolean status, not a progress percentage.

### 18. Will isPending always appear visibly?

No. Very fast work may finish before React paints a pending state. Do not add an artificial delay just to force a spinner.

### 19. How should pending feedback be displayed?

Prefer a local status message, subtle style, or pending state on the affected control. Keep already useful content visible when possible.

### 20. Should I disable the whole interface while a Transition is pending?

Usually not. Disable only an action when repeating it would be unsafe or confusing. Keep unrelated interactions available.

### 21. Does a pending Transition mean a network request is pending?

Not necessarily. A Transition can contain only rendering work. Track request status separately unless the request itself is awaited as part of an async Action.

### 22. Can several components use the same pending status?

Yes. Pass **isPending** from the component that owns the Hook to the parts of the interface that need it. If unrelated operations need separate status, consider where each Hook should live.

## Async Actions

### 23. Can the Action passed to startTransition be async?

Yes. In current React, an Action can be async. **isPending** can remain true while its awaited work is in progress and until the final state is shown.

### 24. How do I mark state updates after await as Transitions?

Wrap those state updates in another **startTransition** call:

~~~jsx
startTransition(async () => {
  const result = await saveDraft(draft);

  startTransition(() => {
    setSavedDraft(result);
  });
});
~~~

This is a current limitation of async context propagation.

### 25. Why might isPending stay true after the request resolves?

It can remain true while pending Actions finish and the resulting state is rendered. The request resolving does not necessarily mean React has committed the final UI yet.

### 26. Does an async Action make the network request run in the background?

No. It awaits normal asynchronous work. The browser and server handle the request; React’s Transition priority concerns UI updates and pending state.

### 27. Does a Transition automatically debounce requests?

No. Use a request policy such as debouncing, throttling, cancellation, or caching when the goal is to control how often requests are sent.

### 28. Does a Transition automatically cancel an older request?

No. A Transition does not cancel network operations. Cancellation and stale-result handling must be implemented by the application or supplied by a higher-level abstraction.

### 29. Can older requests overwrite newer results?

Yes, if the app allows requests to overlap and does not handle their completion order. Use request IDs, cancellation, serialized updates, or an abstraction that manages ordering.

### 30. Should I use useActionState or a form Action instead?

For common form submission and server mutation flows, consider **useActionState** or React form Actions. They provide higher-level patterns for pending state and action results, so you may not need to manage all of that state manually.

### 31. Where should I show a failed Action?

Choose an error boundary for errors that should replace the affected subtree, or catch the error and show inline error state. Keep urgent error feedback responsive.

### 32. Does isPending tell me whether the Action succeeded?

No. It only indicates pending work. Store or derive success and error results separately.

## Suspense and navigation

### 33. How can a Transition help with Suspense?

When a Transition update suspends, React can keep already revealed content visible instead of immediately hiding it behind the nearest fallback.

### 34. Does a Transition eliminate Suspense fallbacks?

No. A newly mounted or nested Suspense boundary may still show its fallback. Place boundaries around meaningful loading regions and decide which revealed content should stay visible.

### 35. Should an application wrap its router’s internal state setter?

Usually not. Use the router’s navigation API. Suspense-aware routers can mark navigation updates as Transitions themselves.

### 36. Why can users change tabs again while a Transition is pending?

Transition renders are interruptible. React can prioritize the newer interaction and prepare the latest selected view without waiting for the previous render attempt to finish.

## Common implementation details

### 37. Can state updates in setTimeout be part of the original Transition?

No. The timer callback runs after the synchronous Action callback has finished. If the later setter should be a Transition update, call **startTransition** inside the timer callback.

### 38. What happens if the Action throws or rejects?

With **useTransition**, an error can be handled by an Error Boundary around the component. Catch the error inside the Action when the application needs inline error state.

### 39. Are multiple concurrent Transitions always tracked separately?

Not necessarily. Current React batches multiple ongoing Transitions together in some cases. Design pending feedback around the user-visible operation instead of assuming one independent counter per call.

### 40. Does startTransition have to wrap the setter directly?

It must wrap the code that schedules the state update. A helper can be called inside the Action if it schedules the relevant setter synchronously.

### 41. How should an Action prop be invoked by a reusable component?

Wrap it in an Action and await it so the prop can be either synchronous or asynchronous:

~~~jsx
startTransition(async () => {
  await action();
});
~~~

This keeps the Transition pending until the callback’s asynchronous work finishes.

### 42. Does useTransition replace useDeferredValue?

No. Use **useTransition** when you control the setter for a non-urgent update and want pending status. Use **useDeferredValue** when a slower consumer should lag behind a value it receives.

### 43. What is the short rule for choosing urgent state?

Keep direct interaction state urgent, especially controlled input values. Transition the expensive view change or defer the value consumed by the slower view.

### 44. What does startTransition not do?

It does not delay its callback, make JavaScript multithreaded, interrupt one synchronous calculation, guarantee request order, cancel requests, or provide **isPending** by itself.

### 45. What should I verify if the pending UI never appears?

Check whether the Action actually schedules work or awaits an operation that remains pending. Fast work may complete before a frame can show the status, which is expected.

## Practical checks

### 46. What should I check if a tab click feels blocked?

Confirm that the slow view state update is marked as a Transition, that any long synchronous calculation is not running before the setter, and that the UI keeps already revealed content visible.

### 47. What should I check if a post-await setter seems urgent?

Wrap that setter in a nested **startTransition** call if the resulting update should stay non-urgent.

### 48. What should I check if an Action error replaces too much of the UI?

Review the nearest Error Boundary placement, or catch the error and show a local message when inline recovery is more useful.

### 49. What should I check if rapid submissions show stale data?

Inspect whether requests overlap and finish out of order. Add ordering or cancellation rules, or choose a higher-level action abstraction suited to the operation.

### 50. What is the core mental model?

An Action starts immediately, React tracks its pending UI work through **isPending**, and Transition priority keeps non-urgent rendering from blocking urgent interactions. Async request semantics and business-level error handling still belong to the application.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="063-concurrent-rendering-and-priorities.md">063 - Concurrent Rendering and Update Priorities - Q&amp;A</a></td><td><a href="../lessons/064-transitions-and-usetransition.md">Lesson</a></td><td><a href="../examples/064-transitions-and-usetransition.md">Example</a></td><td><a href="065-usedeferredvalue.md">065 - useDeferredValue - Q&amp;A</a></td></tr>
  </tbody>
</table>
