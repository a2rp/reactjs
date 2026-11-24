# 039. Queued updates, batching, and functional updates

## Goal

Calling a state setter does not change the state variable in the current render. Instead, React records an update and calculates the next state before rendering again. When several updates to the same state are queued together, a direct value replaces the pending value, while an updater function calculates from the pending value.

This chapter explains how React processes those updates, when batching happens, and how to choose between a direct value and a functional updater.

## Learning outcomes

After this chapter, you should be able to:

- Explain why several setter calls do not create several immediate renders.
- Distinguish a replacement update from an updater function.
- Calculate the result of a sequence of queued updates.
- Use an updater when the next value depends on pending state.
- Keep updater functions pure and free of side effects.
- Recognize when a direct value is simpler and correct.
- Explain why batching and render snapshots are related but different ideas.

## State setters queue work

A setter call is a request to update state. It does not alter the state variable in the currently running render. React records the update so it can calculate the next state and render the interface with that value.

~~~jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count);
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
~~~

If the handler started with count equal to 0, it still logs 0. The setter has requested a future value of 1; the current handler keeps its render snapshot.

When React processes the update, it calls the component again. The next render receives the new count and returns JSX based on it.

## Batching groups updates

When an event handler calls multiple setters, React queues those updates and processes them after the handler has finished. This lets React render the result of the interaction together instead of showing an intermediate interface after every setter call.

~~~jsx
function handleComplete() {
  setIsSaving(true);
  setMessage('Saving your changes...');
  setError(null);
}
~~~

These setters update separate state variables. React can process them as part of the same update and render a consistent interface that shows the saving state, message, and cleared error together.

Batching does not mean that setter calls disappear or that different state variables merge into one. Each state variable has its own next value. Batching groups the work of rendering the resulting interface.

Modern React can batch updates from more than traditional React event handlers, depending on how the app is mounted and where the updates happen. Avoid depending on an exact render count as a correctness rule. Write setters so the intended next state is correct.

## Direct values are replacement updates

When you pass a value to a setter, React treats it as the value to use for that state:

~~~jsx
setCount(10);
setName('Sam');
setIsOpen(false);
~~~

For a queue of updates to count, a direct value is a replacement. It says “use this value as the pending result.” It does not mean “add this value to the result already queued.”

Consider a count that starts at 0:

~~~jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
~~~

All three expressions use count from the same render snapshot. Each one calculates 1. React receives three replacement updates with the value 1, so the final count is 1, not 3.

This is the same snapshot behavior from the previous chapter. The queue stores the updates, but it does not change the count variable in the handler while the handler is running.

## Updater functions calculate from pending state

Pass a function when the next value needs to be calculated from the state value that is pending in the queue:

~~~jsx
function handleClick() {
  setCount((currentCount) => currentCount + 1);
  setCount((currentCount) => currentCount + 1);
  setCount((currentCount) => currentCount + 1);
}
~~~

Each updater receives the result produced by the previous queued update:

| Update | Pending value it receives | Value it returns |
| --- | ---: | ---: |
| First updater | 0 | 1 |
| Second updater | 1 | 2 |
| Third updater | 2 | 3 |

The next render receives count equal to 3.

The updater parameter is the pending state for this particular state variable. It is not a live reference to every value in the component, and it does not change the value captured by the handler.

## Two kinds of queue entries

A state queue can contain both replacement values and updater functions. React processes them in order.

Start with count equal to 0:

~~~jsx
function handleClick() {
  setCount(count + 5);
  setCount((currentCount) => currentCount + 1);
}
~~~

The handler's count is 0, so the first call queues a replacement with 5. The updater then receives 5 and returns 6. The next render receives 6.

Reverse the calls:

~~~jsx
function handleClick() {
  setCount((currentCount) => currentCount + 1);
  setCount(count + 5);
}
~~~

The updater first calculates 1 from 0. The next call is a replacement with 5, because count in this handler is still 0. The final result is 5.

A later replacement can replace the result of earlier queued work. Order matters.

Another example makes that explicit:

~~~jsx
function handleClick() {
  setCount(count + 5);
  setCount((currentCount) => currentCount + 1);
  setCount(42);
}
~~~

With an initial count of 0, the queue is:

1. Replace the pending value with 5.
2. Add 1 to the pending value, producing 6.
3. Replace the pending value with 42.

The final count is 42.

## Choose the form that matches the calculation

Use a direct value when the next value is already known or comes from an event value:

~~~jsx
function handleNameChange(event) {
  setName(event.target.value);
}

function handleClose() {
  setIsOpen(false);
}
~~~

Use an updater when the next value depends on the pending value of that same state:

~~~jsx
function handleIncrement() {
  setCount((currentCount) => currentCount + 1);
}

function handleToggle() {
  setIsOpen((wasOpen) => !wasOpen);
}
~~~

For one ordinary user action, using the current render's state to calculate one next value is usually fine:

~~~jsx
function handleIncrement() {
  setCount(count + 1);
}
~~~

React processes each intentional click separately so that the next click uses the updated interface. An updater is particularly useful when multiple updates to the same state are queued together, or when the update is calculated later and an older closure could otherwise provide an outdated value.

Do not add updater functions mechanically when a direct value expresses the intent more clearly. Choose based on what the new value depends on.

## Updaters must be pure

An updater should calculate and return a next value from its argument. React may call updater functions more than once in development Strict Mode to detect accidental side effects. Only one calculated result is used.

Correct:

~~~jsx
setCount((currentCount) => currentCount + 1);
~~~

Incorrect:

~~~jsx
setCount((currentCount) => {
  sendAnalyticsEvent('incremented');
  return currentCount + 1;
});
~~~

Do not put network requests, timer creation, logging required for application behavior, or other side effects inside an updater. React may call the updater again while checking it. Perform side effects in the appropriate event handler or Effect, and keep the updater focused on calculating state.

Do not mutate the pending value. For objects and arrays, return a new value instead of changing the existing one; those state structures are covered in later chapters.

## Updaters calculate one state variable

An updater receives the pending value for the state variable whose setter it belongs to:

~~~jsx
setCount((currentCount) => currentCount + 1);
~~~

It does not automatically receive the latest value of another state variable:

~~~jsx
setCount((currentCount) => currentCount + step);
~~~

Here step is still read from the render that created this handler. That is appropriate when this action should use that render's step. If multiple pieces of state must always change together based on each other, consider how the state is structured; later chapters cover those designs.

An updater is not a way to make every captured prop, local variable, or state value refresh. It only calculates the next value for the state being updated.

## Async work and repeated completions

Suppose several requests can finish at different times. If each completion should subtract one from the latest pending count, use an updater:

~~~jsx
function handleRequestComplete() {
  setPending((currentPending) => currentPending - 1);
}
~~~

Each completion calculates from the pending value in the queue instead of calculating from the pending count captured by an older render.

For paired changes, use an updater for each state variable that depends on its own previous value:

~~~jsx
function handleRequestComplete() {
  setPending((currentPending) => currentPending - 1);
  setCompleted((currentCompleted) => currentCompleted + 1);
}
~~~

The two updates can be processed together, while each updater computes from its own state's pending value.

Be explicit about values from other state or props that the asynchronous operation needs. An updater for pending does not make a captured request ID or message current. Capture operation inputs when the action begins, or use a state design that represents the operation clearly.

## Common mistakes

### Repeating a direct update and expecting it to accumulate

~~~jsx
setCount(count + 1);
setCount(count + 1);
~~~

Both expressions use count from the current render. Use updater functions when both operations should build on the pending value.

### Using the state variable inside an updater

~~~jsx
setCount((currentCount) => count + 1);
~~~

This ignores the pending value argument and uses the captured count instead. Write:

~~~jsx
setCount((currentCount) => currentCount + 1);
~~~

### Putting side effects in an updater

Updater functions should be pure calculations. Put the side effect in the event handler or an Effect, not in the updater.

### Awaiting a setter

A setter does not return a promise for the render. Writing await setCount(...) does not wait for the screen to update. If later work depends on an interaction, structure that work explicitly rather than treating the setter as an asynchronous function.

### Assuming batching means one exact render

Batching helps React process related updates together, but render counts are not a correctness contract. Do not make application behavior depend on a particular number of component invocations.

## Practice

Assume count is 2 when this handler runs. What value does the next render receive?

~~~jsx
function handleClick() {
  setCount(count + 4);
  setCount((currentCount) => currentCount * 2);
}
~~~

Work through the queue in order:

1. The direct expression uses the handler's count and queues a replacement.
2. The updater receives that replacement value and doubles it.

Now reverse the calls. Explain why the result changes:

~~~jsx
function handleClick() {
  setCount((currentCount) => currentCount * 2);
  setCount(count + 4);
}
~~~

Finally, rewrite this handler so it adds three:

~~~jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
~~~

## Recap

- Setter calls queue state updates; they do not mutate the current render's state variable.
- Batching lets React process related updates together and render a consistent result.
- Passing a direct value queues a replacement.
- Passing an updater function queues a calculation based on the pending state.
- React applies queue entries in order; later replacements can replace earlier results.
- Use functional updaters when the next value depends on pending state, especially for multiple updates to the same state.
- Keep updater functions pure and return the next value without side effects.
