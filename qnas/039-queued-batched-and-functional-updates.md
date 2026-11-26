# 039. Queued updates, batching, and functional updates - Q&A

## Queueing and batching

### 1. What happens when I call a state setter?

React records a request to update that state and render the component again. The call does not change the state variable in the currently running render.

### 2. What is a state update queue?

It is React's ordered collection of updates for a state variable. React processes the entries to calculate the value supplied to a later render.

### 3. What does batching mean?

React groups related state updates so it can render the resulting interface together instead of committing an intermediate UI after every setter call.

### 4. Does batching merge all my state variables into one value?

No. Each state variable keeps its own state and update queue. Batching groups rendering work; it does not combine unrelated state values.

### 5. Why does React batch updates?

It avoids unnecessary intermediate renders and helps the interface reflect a consistent result of an interaction.

### 6. Does batching promise exactly one component render?

No. Do not rely on an exact render count for correctness. Development Strict Mode and React's rendering behavior can make the component function run more than once.

### 7. Does React batch only updates from React event handlers?

Modern React can batch updates from additional sources too, depending on how the app is created and the update context. Prefer code whose result is correct regardless of the exact batching boundary.

### 8. Does a setter run the next render immediately in the middle of my handler?

No. The handler continues to run with its existing render snapshot. React processes the requested updates and renders the resulting interface afterward.

## Replacement values

### 9. What does passing a direct value to a setter mean?

It queues a replacement: use that value as the pending state for the next render.

### 10. What does setCount(10) do?

It requests that count be 10 in a future render. It does not add 10 to the current count.

### 11. If count is 0, what does this handler request?

~~~jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
~~~

Every expression reads count from the same render snapshot, so each queues the replacement value 1. The resulting count is 1, not 3.

### 12. Why don't direct setter calls read earlier setter calls?

A direct expression is calculated by JavaScript before the setter receives it. Calling the setter does not change the handler's local count variable, so later expressions still read the same snapshot.

### 13. When is a direct value the right choice?

Use a direct value when it is already known or comes from the current event, such as setIsOpen(false) or setQuery(event.target.value).

### 14. Is using a direct value wrong for every update based on current state?

No. For one ordinary user action, React processes that interaction before the next intentional one, so setCount(count + 1) is usually fine. Updaters are especially useful when multiple updates to the same state are queued together or when calculation from pending state is needed.

## Updater functions

### 15. What is an updater function?

It is a function passed to a state setter that receives the pending value for that state and returns its next value.

### 16. What does setCount(currentCount => currentCount + 1) mean?

It asks React to calculate the next count by taking the pending count and adding one.

### 17. How does an updater differ from a direct value?

A direct value replaces the pending state. An updater function calculates a new pending state from the value produced by the preceding queue entry.

### 18. How can three updater calls add three?

~~~jsx
setCount((currentCount) => currentCount + 1);
setCount((currentCount) => currentCount + 1);
setCount((currentCount) => currentCount + 1);
~~~

React processes them in order. If the queue starts at 0, they calculate 1, then 2, then 3.

### 19. What value does the first updater receive?

It receives the state value from which React begins processing that state variable's queue.

### 20. What does the second updater receive after the first one returns 3?

It receives 3 as its pending input, then returns its own next value.

### 21. Does an updater change the state variable inside the current handler?

No. It describes how React should calculate the next state. The handler's local variable remains the value from its render.

### 22. What does this sequence produce when count starts at 0?

~~~jsx
setCount(count + 5);
setCount((currentCount) => currentCount + 1);
~~~

The direct expression queues a replacement with 5. The updater receives 5 and returns 6, so the next render receives 6.

### 23. What does the reversed sequence produce when count starts at 0?

~~~jsx
setCount((currentCount) => currentCount + 1);
setCount(count + 5);
~~~

The updater first returns 1. The direct expression still reads the handler's count of 0 and queues a replacement with 5. The final result is 5.

### 24. Can a later direct value override earlier updater results?

Yes. Queue entries are processed in order, and a replacement later in the queue replaces the result calculated earlier.

### 25. What does this sequence produce when count starts at 0?

~~~jsx
setCount(count + 5);
setCount((currentCount) => currentCount + 1);
setCount(42);
~~~

The queue replaces 0 with 5, the updater calculates 6, and the final replacement sets 42. The next render receives 42.

### 26. Can I pass a function to a setter as a direct function value?

React treats a function passed to a state setter as an updater. To store a function itself in state, wrap it in another function so React returns it as the value. This is an uncommon case.

## Choosing update style

### 27. Should I always use an updater function?

No. Use the form that matches the calculation. Direct values are clear for known values and event input; updater functions are useful when the next value depends on the pending value of that same state.

### 28. Should I use an updater for a toggle?

If the next value should invert the pending state, use:

~~~jsx
setIsOpen((wasOpen) => !wasOpen);
~~~

If a handler intentionally sets a known state such as false, a direct value is clearer.

### 29. How should I handle a text input change?

Use the event's current value directly:

~~~jsx
function handleChange(event) {
  setQuery(event.target.value);
}
~~~

The new query comes from the input event; it is not calculated by incrementing the previous query.

### 30. Can an updater read other component values?

It can close over props or other values, but those values still come from the render that created the updater. The updater's parameter is only the pending value for its own state variable.

### 31. Does an updater make every captured value current?

No. It receives pending state only for the state variable being updated. It does not refresh captured props, other state variables, local variables, or request data.

### 32. How can an updater help with asynchronous completions?

If each completion should change the latest pending count, queue the change from that count:

~~~jsx
setPending((currentPending) => currentPending - 1);
~~~

Each updater calculates from the pending value rather than using an older handler's captured pending count.

### 33. If two state variables change together, can each use an updater?

Yes. Each setter can receive an updater for its own state. React can process both updates together while each calculation uses its corresponding pending value.

## Purity and common mistakes

### 34. What must an updater function do?

It should be a pure calculation: receive a pending state value and return the next value without side effects or mutation.

### 35. Why should I keep side effects out of updaters?

React may call an updater more than once in development Strict Mode to find impure logic. Requests, analytics, timer creation, and other side effects could then run unexpectedly.

### 36. Can an updater mutate an object or array it receives?

No. Return a new object or array instead of mutating the existing state. Immutable object and array updates are covered in later chapters.

### 37. What is wrong with this updater?

~~~jsx
setCount((currentCount) => count + 1);
~~~

It ignores currentCount and calculates from count captured by the render. Write:

~~~jsx
setCount((currentCount) => currentCount + 1);
~~~

### 38. Does await setCount(...) wait for the render?

No. A state setter does not return a promise for a render. Awaiting its return value does not wait for the interface to update.

### 39. Does batching mean the interface shows every intermediate queued value?

Usually no. React renders the result of the processed updates together, instead of displaying each intermediate state from the queue.

### 40. Is batching the same idea as a render snapshot?

No. A snapshot explains which state values the current render and its handlers read. Batching explains how React groups update processing and rendering work.

## Check your understanding

### 41. If count is 2, what does this sequence produce?

~~~jsx
setCount(count + 4);
setCount((currentCount) => currentCount * 2);
~~~

The first call queues 6. The updater receives 6 and returns 12.

### 42. If count is 2, what does the reversed sequence produce?

~~~jsx
setCount((currentCount) => currentCount * 2);
setCount(count + 4);
~~~

The updater first calculates 4. The direct expression uses the handler's count of 2 and queues a replacement with 6. The final result is 6.

### 43. How do I fix three direct calls that should add three?

Use an updater for each increment:

~~~jsx
setCount((currentCount) => currentCount + 1);
setCount((currentCount) => currentCount + 1);
setCount((currentCount) => currentCount + 1);
~~~

### 44. When can I use a direct value for a counter increment without an updater?

For a single increment in one ordinary click handler, setCount(count + 1) is normally correct. Each intentional click is handled separately, so the next click uses the next rendered count.

### 45. What should I ask when deciding between a value and an updater?

Ask whether the next value is already known, comes from the event, or must be calculated from the pending value of this same state variable. Use a direct value for the first cases and an updater for the last.

## Summary

- React queues setter requests and groups related rendering work through batching.
- A direct value queues a replacement.
- An updater function calculates from the pending state and is applied in queue order.
- A later replacement can override results from earlier updates.
- Use an updater when the next value depends on pending state, but do not add one when a direct value expresses the intent more clearly.
- Updaters must be pure and must not contain side effects.
- An updater only receives pending state for its own state variable; it does not refresh other captured values.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="038-state-as-a-render-snapshot.md">038 - State as a render snapshot - Q&amp;A</a></td><td><a href="../lessons/039-queued-batched-and-functional-updates.md">Lesson</a></td><td><a href="../examples/039-queued-batched-and-functional-updates.md">Example</a></td><td><a href="040-updating-object-state-immutably.md">040 - Updating object state immutably - Q&amp;A</a></td></tr>
  </tbody>
</table>
