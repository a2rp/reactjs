# 038. State as a render snapshot - Q&A

## Understanding render snapshots

### 1. What does “state as a render snapshot” mean?

When React renders a component, it supplies state values for that render. The component's JSX, local calculations, and event handlers are created using those values. They remain the values for that render's code.

### 2. What belongs to a render snapshot?

The props and state supplied to that render, the local variables calculated while the component runs, the JSX it returns, and the functions it creates are all based on that render's inputs.

### 3. Where does state live between renders?

React keeps the component's state between renders. A component invocation receives the state value for that render; its local JavaScript variable is not a live reference to React's stored state.

### 4. Is a snapshot a copy of the entire application?

No. It is a way to describe the values and UI calculated for one render of a component. Other components and later renders have their own values and work.

### 5. Are state values permanently frozen?

No. State can change for a later render. “Snapshot” means a particular render's variables and handlers keep the values from that render; it does not mean the state can never update.

### 6. How are JSX and an event handler related?

The component calculates both from the same render's values. For example, a button might display count from a render and use a click handler created from that same count.

## Calling a setter

### 7. What does calling setCount(count + 1) do?

It calculates count + 1 using the current handler's count and asks React to render again with that requested value. It does not change count in the currently running code.

### 8. What does this handler log if count is 2?

~~~jsx
function handleClick() {
  setCount(count + 1);
  console.log(count);
}
~~~

It logs 2. The handler still has count from the render that created it, even though it requested 3 for a later render.

### 9. Does JavaScript pause the handler until React renders?

No. The setter call does not pause the running function and replace its local variables. The remaining statements run with that handler's snapshot.

### 10. Does a state setter return the next state?

No. A state setter does not return the next value. React supplies updated state to the component when it renders again.

### 11. How can the current handler use the calculated next value?

Calculate it in a local variable:

~~~jsx
const nextCount = count + 1;
setCount(nextCount);
console.log(nextCount);
~~~

nextCount is the calculation made by this handler. count still means the value from the current render.

### 12. If I call setCount(10), does count become 10 on the next line?

No. The current handler's count remains unchanged. React is asked to render with 10; a later render will receive that value.

### 13. Does calling the setter change the DOM immediately?

No. It requests an update. React renders the component with the next state and updates the visible interface to match that render.

## Handlers from different renders

### 14. Why can a later click see a different count?

React renders the component after an update. The new render creates a new handler using the new count. A later click handled by that rendered interface starts with the newer snapshot.

### 15. Does the first handler turn into the new handler?

No. The first function continues to refer to the values from the render that created it. A later render creates another function.

### 16. Which handler runs when I click a button?

The rendered interface is connected to the handler returned for that UI. After React commits an updated render, a later interaction with that updated interface uses the handler from that render.

### 17. If count starts at 0, what happens over two ordinary clicks?

Assuming React renders the update between the clicks, the first handler starts with 0 and requests 1. The next render creates a handler with 1. The second click starts with 1 and requests 2.

### 18. Why does one click handler keep using the same value even if it calls a setter?

The handler is ordinary JavaScript created during a render. Its references use the values from that render. Calling a React setter requests another render; it does not rewrite that function's already-created bindings.

## Multiple updates in one handler

### 19. What value is requested by this handler when count is 4?

~~~jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
}
~~~

Both expressions read 4, so both direct calls request 5. They do not read a value changed by the earlier call.

### 20. Why doesn't calling a setter three times automatically add three?

Each direct expression such as count + 1 uses the same count from the handler's render. The expressions do not successively read each other's requested values.

### 21. Is this only a logging issue?

No. It applies to any code in the handler that reads that state variable. The local variable keeps its snapshot whether it is used in a log, a calculation, or a callback.

### 22. How should I request three increments based on the pending value?

Use updater functions that calculate from the pending state, for example:

~~~jsx
setCount((currentCount) => currentCount + 1);
setCount((currentCount) => currentCount + 1);
setCount((currentCount) => currentCount + 1);
~~~

The next chapter explains update queues, updater functions, and how React processes them. The key idea here is that these functions receive a pending value instead of reading the handler's fixed count variable.

### 23. Is “snapshot” the same thing as “batching”?

No. A snapshot explains which value a running render and its handlers read. Batching describes how React processes state updates together. Several direct expressions can all read the same snapshot regardless of how many renders React performs for them.

## Asynchronous callbacks

### 24. What does a timer callback capture?

It captures values through the function created when the timer was scheduled. If that function reads message and recipient, it reads the values from the render that created the scheduling handler.

### 25. If I schedule a message to Mira and then change the form to Sam, who does the scheduled callback use?

It uses Mira if the callback was created by a handler whose recipient value was Mira. The form edit causes another render, but it does not replace the already-scheduled callback's captured values.

### 26. Is an older callback always a bug?

No. It is often correct for an operation to use the data that was submitted when it began. For example, a scheduled message should usually use the submitted recipient and message, even if the user edits the form afterward.

### 27. What if an asynchronous operation should use the latest value later?

Choose a pattern that explicitly reads or receives the intended latest value. Do not assume an older handler's state variable will become current. Updater functions and other coordination patterns are covered in later chapters.

### 28. Does an await make the handler's state variable refresh?

No. An async function still uses the values captured by the render that created it. Resuming after await does not replace those local bindings with values from a later render.

### 29. Can I save submitted values before starting asynchronous work?

Yes. Saving values in local constants makes the operation's input explicit:

~~~jsx
const submittedMessage = message;
const submittedRecipient = recipient;
~~~

Those constants represent the values this handler chose to submit. The callback can use them to make that intention clear.

## Props, calculations, and rendering

### 30. Do props also belong to a render snapshot?

Yes. Props supplied to one render are used to calculate that render's JSX, local variables, and functions. If a parent supplies different props in another render, the child runs again with those new values.

### 31. Do local variables update when state changes?

No. A local variable calculated in one component invocation belongs to that invocation. A later render calculates its own local variables from the new state and props.

### 32. Can a handler use a value calculated from a prop?

Yes. The handler captures the calculated value from the render where it was created. If the prop changes later, the next render creates a new calculation and handler.

### 33. Is React changing the state variable behind my back?

No. React supplies state for each render and tracks updates between renders. The local variable in your currently running JavaScript function stays as JavaScript assigned it for that invocation.

### 34. Should I mutate the DOM to make the new value appear before the next render?

No. Keep the interface derived from state and let React update it. Manual DOM edits can make the DOM disagree with the JSX React calculates.

### 35. Is logging state during a render a reliable way to track user actions?

A render log shows the value supplied to that particular component invocation, and rendering can happen for reasons beyond a user click. For understanding a specific handler, log or display both its current value and the next value it calculated.

## Check your understanding

### 36. If count is 4, what does this code log and request?

~~~jsx
function handleClick() {
  setCount(count + 2);
  console.log(count);
}
~~~

It logs 4 and requests 6. After React renders that update, the interface can display 6.

### 37. If count is 4, why do two direct calls to setCount(count + 1) both request 5?

Both expressions read count from the same handler snapshot. The first setter does not assign 5 to that handler's count variable before the second expression runs.

### 38. A user schedules a callback and then changes a controlled input. What determines what the callback reads?

The render that created the callback determines the captured state values. The later input change creates a newer render and handlers, but does not alter that existing callback.

### 39. What is the difference between count and nextCount in this code?

~~~jsx
const nextCount = count + 1;
setCount(nextCount);
~~~

count is the value supplied to the current render. nextCount is a regular local value calculated by this handler and passed to the setter as the requested next state.

### 40. What is the main debugging question when a value looks “stale”?

Ask which render created the function that is reading the value. Then identify the state or props supplied to that render and decide whether the operation should use that snapshot or an explicitly chosen later value.

## Summary

- A render provides a snapshot of state and props to a component invocation.
- JSX, local calculations, and event handlers are created from that render's values.
- A setter requests a future render; it does not update the current handler's variable.
- A later render creates new handlers with its new values.
- Asynchronous callbacks retain values from the render that created them.
- Direct repeated expressions read the same snapshot; queued and functional updates are explained in the next chapter.
