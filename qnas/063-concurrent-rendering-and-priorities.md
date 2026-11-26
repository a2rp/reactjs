# Concurrent Rendering and Update Priorities - Q&A

## Concurrent rendering

### 1. What does concurrent rendering mean in React?

React can prepare some UI updates as interruptible render work. It can pause or abandon that work when a more urgent update arrives, then continue with the latest inputs.

### 2. Does concurrent rendering run JavaScript components on multiple threads?

No. JavaScript component functions still run on the browser’s main thread. React can yield between units of render work; it cannot interrupt one long synchronous function halfway through.

### 3. Why can React abandon a render?

A newer, more urgent update may make the in-progress result stale. React can stop that render attempt and work on the latest state instead.

### 4. Does abandoned render work change the DOM?

An abandoned render is not committed. React commits a completed result; render code must be pure because it may run again without its result ever being committed.

### 5. Is every React update a Transition?

No. Ordinary updates remain ordinary. Use a Transition or deferred value when slower rendering should be interruptible and less urgent.

## useTransition and startTransition

### 6. What does useTransition return?

It returns **[isPending, startTransition]**. The function marks state updates as non-urgent, while **isPending** lets the component show that the Transition work is pending.

### 7. What is startTransition from React?

It is a standalone function that marks state updates scheduled synchronously inside its callback as Transition updates. It does not provide an **isPending** flag.

### 8. Does the callback passed to startTransition run later?

No. React calls it immediately. The state updates scheduled inside it get non-urgent priority; this is not a timer like **setTimeout**.

### 9. What kind of update should be placed in a Transition?

A non-urgent update that may involve slower rendering, such as switching a large tab panel, updating a chart, or rendering a substantial result view.

### 10. Can a Transition update be interrupted?

Yes. React can prioritize another update, such as a keystroke, and restart the Transition render with the latest inputs.

### 11. Can I use a Transition to control an input’s value?

No. A controlled input’s state update must happen synchronously in response to the input event so the field reflects each keystroke promptly.

### 12. How can typing stay urgent while a large result view updates in the background?

Keep input text in urgent state, then use **useDeferredValue** for the slower consumer or maintain a separate results state that is updated inside a Transition.

### 13. What does isPending tell me?

It indicates that a Transition started with that **useTransition** Hook has work pending. Use it for suitable feedback near the affected UI; it is not a progress percentage.

### 14. Does startTransition work in a helper outside a component?

Yes, if the helper receives the relevant state setter. It can call the standalone **startTransition** function. The helper cannot get **isPending** from that function.

### 15. Do state updates inside setTimeout belong to the surrounding Transition?

No. The timer callback runs after the synchronous Transition callback has finished. If that later state update should be non-urgent, call **startTransition** inside the timer callback.

### 16. What if an async Transition updates state after await?

Current React guidance requires wrapping state updates after **await** in another **startTransition** call if those updates should be Transition updates.

### 17. Does a Transition automatically prevent stale network responses from overwriting fresh ones?

No. If multiple requests can finish out of order, the app must handle request ordering, cancellation, or stale responses. Transition priority does not solve request races by itself.

### 18. Should I disable all buttons while isPending is true?

Not necessarily. Keep the interface usable when possible. Disable an action only when repeating it would be confusing or unsafe; otherwise, a local pending indicator may be enough.

## useDeferredValue

### 19. What does useDeferredValue do?

It returns a deferred version of a value. A slower consumer may continue using the previous value while React handles urgent work and then renders in the background with the latest value.

### 20. Does useDeferredValue take a state setter?

No. It receives a value and returns a deferred value. It is useful when the component has the changing value but does not control where that value is set.

### 21. What does the deferred value look like on the initial render?

It is usually the same as the supplied value because there is no previous value to keep showing. An optional initial value can be provided for a different initial behavior.

### 22. Is there a fixed delay before the deferred value updates?

No. React attempts the background render as soon as possible, and urgent updates can interrupt it. There is no built-in millisecond delay.

### 23. Does useDeferredValue make an expensive calculation faster?

No. It changes scheduling priority for rendering. The calculation still takes the time it takes; React may allow it to run without blocking more urgent UI work.

### 24. Does useDeferredValue reduce network requests?

No. It defers the rendering consumer, not network work. Use a request strategy such as debouncing, throttling, cancellation, or caching if the goal is to control requests.

### 25. Does useDeferredValue require a memoized child to help with typing responsiveness?

For the common pattern where the parent renders urgently with the old deferred value, the slow child should be able to skip that parent render, commonly by using **memo**. Otherwise, it may still redo the slow work with the unchanged deferred prop.

### 26. How can I indicate that a deferred result is stale?

For comparable primitive values, compare the current value with the deferred one, such as **query !== deferredQuery**, and use a subtle visual or accessible status to indicate that results are catching up.

### 27. Does a deferred result mean a network request is pending?

No. It means the rendered consumer may still be using an older value. Network status should be tracked separately if relevant.

### 28. Should I pass a new object literal to useDeferredValue on every render?

Avoid it. A fresh object has a different identity each time and can cause unnecessary background work. Prefer a primitive, a value created outside render, or appropriately stable state.

### 29. What happens if the deferred background render suspends?

With Suspense, React can keep showing the previous deferred content instead of replacing already revealed UI with a fallback while the background result is unavailable.

## Choosing a technique

### 30. When should I choose useTransition?

Use it when you control the setter for the non-urgent state update and want to mark that update as a Transition. Choose **useTransition** if the initiating component also needs **isPending**.

### 31. When should I choose useDeferredValue?

Use it when you have a changing value that a slower consumer should use later, especially when you do not control the setter that produced the value.

### 32. Can useTransition and useDeferredValue be used for the same update?

Usually that is redundant. A Transition is already non-urgent, and **useDeferredValue** does not start another deferred render for a value already updated inside a Transition.

### 33. How is useDeferredValue different from debouncing?

Debouncing waits for a chosen quiet interval before running work. **useDeferredValue** has no fixed timer and lets React prioritize and interrupt rendering based on current work and user interactions.

### 34. How is a Transition different from throttling?

Throttling limits how often an operation runs over time. A Transition changes the priority of React state rendering; it does not impose a maximum update rate.

### 35. Can debouncing and Transitions be combined?

Yes, if an app separately needs fewer requests and lower-priority rendering. Debounce or throttle the request policy, then use React scheduling for the resulting UI work when appropriate.

### 36. What should I do if the calculation itself is one long synchronous loop?

A Transition cannot interrupt that function in the middle. Optimize the calculation, split work into interruptible units, move it to an appropriate worker, or use another strategy suited to the workload.

### 37. Should I wrap every state setter in startTransition?

No. Immediate interactions should remain urgent. Mark only updates that can wait without making the UI feel unresponsive.

### 38. Does useDeferredValue automatically show a spinner?

No. It only returns a deferred value. The component decides whether and how to show a stale indicator or status.

### 39. Do transitions keep already revealed content visible?

They are designed to avoid unwanted loading jumps for already revealed UI, particularly with Suspense. Design pending feedback based on what the user needs to understand.

### 40. Does useTransition delay executing the Action function?

No. React calls it immediately, marks the state updates it schedules, and tracks pending Transition work. Any synchronous computation directly inside that callback still runs synchronously.

## Practical checks

### 41. What should I verify when a controlled field feels laggy?

Check that its own value update is urgent. If a slow result view also updates on every keystroke, separate its rendering input with **useDeferredValue** or a Transition.

### 42. What should I verify when a deferred value seems to do nothing?

Confirm that the slow child can skip rendering while its deferred prop is unchanged, and that there is actual render work whose priority can be adjusted. The API does not speed up the work itself.

### 43. What should I verify when a Transition never shows pending feedback?

Check that the Transition triggers work that remains pending long enough to observe. Very fast work may complete before a visible pending state is painted; do not add an artificial delay just to force a spinner.

### 44. What is the short rule for update priorities?

Keep immediate input feedback urgent. Use Transitions or deferred values for slower rendering that can catch up after urgent interactions.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="062-reconciliation-and-component-identity.md">062 - Reconciliation, Component Identity, and Keys - Q&amp;A</a></td><td><a href="../lessons/063-concurrent-rendering-and-priorities.md">Lesson</a></td><td><a href="../examples/063-concurrent-rendering-and-priorities.md">Example</a></td><td><a href="064-transitions-and-usetransition.md">064 - Transitions with useTransition and startTransition - Q&amp;A</a></td></tr>
  </tbody>
</table>
