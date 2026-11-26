# 038. State as a render snapshot

## Goal

A React state variable is the value supplied to one particular render of a component. Calling its setter requests a future render; it does not rewrite the value already captured by the running component function or event handler.

This chapter explains why a handler can request a state change and still read the previous value, how each render creates its own handlers, and why an asynchronous callback can use the values from the render that created it.

## Learning outcomes

After this chapter, you should be able to:

- Explain what a render snapshot contains.
- Predict what a handler reads before and after calling a state setter.
- Explain why a later click can use a newer value than an earlier click.
- Describe what a delayed callback captures.
- Distinguish the value requested for the next render from the current handler's value.
- Recognize why several direct setter calls can all calculate from the same snapshot.

## A render creates a snapshot

A component is a function. When React renders it, React calls that function with the props and state for that render. The function calculates local variables, JSX, and event handlers from those values.

For example:

~~~jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button type="button" onClick={handleClick}>
      Count: {count}
    </button>
  );
}
~~~

On the initial render, count is 0. The returned button displays 0, and its handleClick function was created while count was 0. That output and its handler belong to that render's snapshot.

A render snapshot includes:

- The props and state values used for that render.
- Local variables calculated while the component function runs.
- The JSX returned from that invocation.
- Event handlers created from those values.

React state is held by React between renders. The count variable in a particular invocation is an ordinary JavaScript binding to the value React supplied for that invocation. It is not a live window into React's state storage.

## A setter requests another render

When handleClick calls setCount(count + 1), it calculates the requested value using the current handler's count. React records the update and renders the component again. The new render receives the updated value and creates new JSX and new handlers.

Calling the setter does not assign a new value to the count binding inside the already-running handler:

~~~jsx
function handleClick() {
  console.log(count); // The value from this render.

  setCount(count + 1);

  console.log(count); // Still the value from this render.
}
~~~

If the handler started with count equal to 0, both logs read 0. The setter requests a render whose count will be 1; it does not make the second log read 1.

The setter does not return the next state value either. If the handler needs to use the calculated next value right away, calculate it explicitly:

~~~jsx
function handleClick() {
  const nextCount = count + 1;

  setCount(nextCount);

  console.log('Current snapshot:', count);
  console.log('Value requested for the next render:', nextCount);
}
~~~

Here count is still the current render's value. nextCount is an ordinary local variable holding the calculation the handler made.

## Later interactions use the newer render

After React renders and commits the updated interface, the button in that interface is associated with a handler created by the new render. A later click therefore starts with that render's count.

Think through two clicks, with React finishing the render between them:

1. The first click's handler was created with count equal to 0. It requests 1.
2. React renders the updated button with count equal to 1 and creates its next handler.
3. The second click uses that next handler. It starts with count equal to 1 and requests 2.

The first handler does not change into the second handler. React creates a new render and new functions. This is why ordinary repeated clicks can advance a counter even though each individual handler keeps its own snapshot.

## Several calls can read the same snapshot

Suppose one handler contains three direct updates:

~~~jsx
function handleAddThree() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
~~~

If this handler's snapshot has count equal to 0, each expression reads that same 0 and requests 1. None of these calls changes count inside the handler, so this code does not mean “read the newly requested value, then add one again” three times.

This example is about the value captured by the handler. How React processes multiple updates, and how to express a true sequence of increments, are covered in the next chapter on queued and functional updates.

## Asynchronous callbacks keep their render's values

A function created during a render closes over the values from that render. A timer or promise callback does not turn those captured values into live state.

~~~jsx
function handleSend() {
  setTimeout(() => {
    console.log(message, recipient);
  }, 1000);
}
~~~

If the handler was created when message was “Hello” and recipient was “Mira”, the scheduled callback uses those values. Updating the form after scheduling causes another render with new handlers, but it does not rewrite the callback that was already created.

This is often the intended behavior. A send action should normally use the message and recipient that the user submitted, even if they start editing the form afterward. A useful way to reason about it is: the action belongs to the snapshot from which it started.

If an operation should instead use data that is current at some later moment, define that behavior deliberately. Do not expect a state variable captured by an older handler to update itself. The upcoming chapters cover updater functions and other patterns for coordinating state changes.

## Props and local calculations are part of the snapshot too

State is not the only value tied to a render. Props, local variables, and functions created by the component also use the values from that render.

~~~jsx
function Greeting({ name }) {
  const label = 'Hello, ' + name;

  function handleClick() {
    console.log(label);
  }

  return <button onClick={handleClick}>Show greeting</button>;
}
~~~

If a parent later renders Greeting with a different name, React creates a new render of Greeting. The new button uses a new handler and label. A handler created by the previous render still refers to the previous label.

This does not mean props or state are globally frozen. It means one render and the functions created during it each use their own set of values.

## Rendering and the visible interface

A setter asks React to render again with an updated state value. React calls the component to calculate a new JSX snapshot and then updates the screen to match it. The JavaScript statement after a setter still runs in the current handler; React does not pause that function and replace its local variables midway through.

Avoid trying to make an immediate state change by editing the DOM manually. Render the UI from state, and use the next render to display the new value.

## Common misunderstandings

### “The setter changed my variable”

It did not change the local binding in the current render. It requested a future render with a different state value.

### “The next line should see the new state”

The next line is still running in the same handler, so it reads the same snapshot. Store the calculated next value in a local variable if the current handler needs that calculation.

### “A timeout should read whatever is in the form later”

A timeout callback uses values captured when that callback was created. Decide whether the operation should use the submitted values or intentionally read a later value.

### “Every render reuses the same handler”

A component render creates functions from that render's values. React uses the handlers from the rendered interface; a later render creates a new set.

### “Calling a setter changes the DOM immediately”

The setter requests a render. React calculates and commits an updated interface as part of rendering; it does not mutate the currently running handler's snapshot.

## Practice

Without running the code, assume count is 4 when handleClick runs:

~~~jsx
function handleClick() {
  setCount(count + 2);
  console.log(count);
}
~~~

Answer these questions:

1. What value does the log read?
2. What value does this handler request for the next render?
3. What value will the button display after React renders the update?

Then explain why this handler requests 5 rather than 6 when count starts at 4:

~~~jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
}
~~~

In your explanation, identify which render supplied count to the handler.

## Recap

- Each render receives a particular set of props and state and calculates JSX and handlers from them.
- An event handler keeps the values from the render that created it.
- Calling a state setter requests an update for a future render; it does not change the current handler's state variable.
- A later render creates new handlers that use the new state.
- A delayed callback keeps the values it captured when it was created.
- Multiple direct calculations in one handler all start from that handler's snapshot; queued and functional updates are the topic of the next chapter.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="037-state-with-usestate.md">037 - State with useState</a></td><td><a href="../qnas/038-state-as-a-render-snapshot.md">Q&amp;A</a></td><td><a href="../examples/038-state-as-a-render-snapshot.md">Example</a></td><td><a href="039-queued-batched-and-functional-updates.md">039 - Queued updates, batching, and functional updates</a></td></tr>
  </tbody>
</table>
