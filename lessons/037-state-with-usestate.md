# 037. State with useState

## Goal

React state is information that a component remembers between renders. The useState Hook gives a component a state value and a setter that asks React to render again with the next value.

This chapter introduces state as component memory, declares state with useState, and updates the interface from state. The following chapters explain render snapshots, queued updates, and object and array updates in more depth.

## Learning outcomes

After this chapter, you should be able to:

- Explain why a regular local variable cannot remember an update for the next render.
- Declare a state value with useState and read its setter.
- Update UI-changing data from a React event handler.
- Choose an appropriate initial value and name for state.
- Use more than one state variable when a component remembers different facts.
- Explain that state belongs to a particular rendered component instance.
- Follow the basic Hook rule: call useState at the top level of a component or custom Hook.

## When a regular variable is not enough

A component can calculate values during rendering:

~~~jsx
function Greeting() {
  const name = 'Mira';

  return <h1>Hello, {name}</h1>;
}
~~~

The local variable name is available while this invocation of Greeting runs. It is not React-managed memory. If an event handler changes a regular variable, React does not know that the UI needs another render:

~~~jsx
function ClickCounter() {
  let count = 0;

  function handleClick() {
    count += 1;
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
~~~

Clicking this button changes the local variable in that render's function call, but it does not request a new render. A later render would also create a fresh local count starting at zero.

For information that must both survive another render and update the UI, use state.

## Declare state with useState

Import useState from React and call it at the top level of the component:

~~~jsx
import { useState } from 'react';

function ClickCounter() {
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

useState returns a pair:

1. The current state value for this render.
2. A setter function that asks React to update that state and render again.

The array destructuring names are chosen by the developer. A common convention is to name them as a pair: count and setCount, isOpen and setIsOpen, or query and setQuery.

When the button is activated, the setter requests the next value. React renders ClickCounter again, useState provides the updated count, and the button text is calculated from that value.

## State drives the rendered interface

Treat the JSX as a description of the interface for the current state. When state changes, render the right output from the new value:

~~~jsx
function Disclosure() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <section>
      <button
        type="button"
        aria-expanded={isOpen}
        onClick={() => setIsOpen(!isOpen)}
      >
        {isOpen ? 'Hide details' : 'Show details'}
      </button>

      {isOpen && <p>The additional details are now visible.</p>}
    </section>
  );
}
~~~

The boolean isOpen represents whether the details are shown. The button label, aria-expanded value, and paragraph all come from the same state. This keeps the visual and accessible interface aligned.

Do not update the DOM manually to show the paragraph. Change the state and let React render the corresponding JSX.

## Choose state for information that changes

State is useful for information that changes over time and affects what the component displays, such as:

- A selected tab or current gallery item.
- Whether details are expanded.
- The current value of a controlled input.
- A count displayed in the interface.
- Data received from an interaction or external operation.

A value that can be calculated from props or other state usually does not need separate state. For example, if the UI shows whether a number is positive, render number > 0 instead of storing both the number and a separate isPositive boolean.

Avoid putting every variable in state. Constants, temporary calculations, and values that never affect rendered output can remain ordinary variables.

## Choose an appropriate initial value

The argument to useState is the initial value for that state variable:

~~~jsx
const [count, setCount] = useState(0);
const [isOpen, setIsOpen] = useState(false);
const [name, setName] = useState('');
const [selectedItem, setSelectedItem] = useState(null);
~~~

Choose an initial value that matches what the component needs to render before the user interacts with it. Keep the value's type consistent: a text field should generally use a string from its first render, and a collection should generally start as an array.

React uses the initial value when initializing the component's state. Passing a different value to useState on a later render does not reset that state. Resetting state when component identity changes is covered in a later chapter.

## Use more than one state variable

A component can call useState more than once to remember different facts:

~~~jsx
function StudyPanel() {
  const [completedCards, setCompletedCards] = useState(0);
  const [showHint, setShowHint] = useState(false);

  return (
    <section>
      <p>Cards completed: {completedCards}</p>

      <button
        type="button"
        onClick={() => setCompletedCards(completedCards + 1)}
      >
        Complete one card
      </button>

      <button
        type="button"
        aria-expanded={showHint}
        onClick={() => setShowHint(!showHint)}
      >
        {showHint ? 'Hide hint' : 'Show hint'}
      </button>

      {showHint && <p>Try explaining the idea in your own words.</p>}
    </section>
  );
}
~~~

The count and hint visibility are separate state variables because they represent different facts. A component can have several state variables, and each useState call should have a clear purpose.

When many values change together or form one coherent record, a single object or a reducer may be easier to manage. The later state-structure chapters cover those choices.

## State belongs to a component instance

State is local to the component instance that declares it. If the same component appears twice, each rendered instance has its own state:

~~~jsx
function App() {
  return (
    <>
      <StudyPanel />
      <StudyPanel />
    </>
  );
}
~~~

Each StudyPanel has its own completedCards and showHint values. Activating a button in one panel does not change the other panel.

A parent cannot read or directly change a child's private state. If multiple components need to share one value, move that state to a shared parent and pass the value and event handlers through props. That pattern is covered in the state-sharing chapters.

## Call useState at the top level

Call useState unconditionally at the top level of a component or custom Hook:

~~~jsx
function Settings({ enabled }) {
  const [theme, setTheme] = useState('light');

  if (!enabled) {
    return null;
  }

  return <button onClick={() => setTheme('dark')}>{theme}</button>;
}
~~~

Do not call a Hook inside a condition, loop, event handler, or nested function:

~~~jsx
function Settings({ enabled }) {
  if (enabled) {
    // Incorrect: the Hook call is conditional.
    const [theme, setTheme] = useState('light');
  }

  return null;
}
~~~

React relies on Hooks being called in the same order on every render. Declare the Hook first, then use ordinary conditions to decide what interface to return.

## State setters and the next render

Call the setter to request a state change. Do not assign directly to the state variable:

~~~jsx
setCount(count + 1);
~~~

The setter does not return the updated value. React will render again and provide the next state value to the component. The current function invocation continues with its current render's value; the next chapter explains this snapshot behavior in detail.

A setter can receive a direct next value. It can also receive an updater function for a value calculated from previous state:

~~~jsx
setCount(count + 1);
setCount((previousCount) => previousCount + 1);
~~~

The updater form is useful when the next value depends on the previous value, especially when several updates need to be composed. The following chapter on queued updates explores that behavior fully.

Keep updater functions pure: calculate and return the next value without changing external state or causing side effects. In development Strict Mode, React may call initializer or updater functions an extra time to expose impure logic.

If the next value is equal to the current value according to Object.is, React may skip rendering the component's children. Write code that depends on state values, not on whether React happens to perform a particular render optimization.

## Functions as state values

A function passed as the initial argument to useState is treated as an initializer function. A function passed to a setter is treated as an updater function. If a component truly needs to store a function itself, wrap it in another function:

~~~jsx
const [callback, setCallback] = useState(() => initialCallback);

function replaceCallback(nextCallback) {
  setCallback(() => nextCallback);
}
~~~

This is an uncommon case. Most components store data such as numbers, strings, booleans, objects, or arrays.

## Avoid mutating state objects

State can contain an object or an array, but React does not automatically merge object properties for a useState update. A setter replaces the state value with the next value. Treat stored objects and arrays as read-only and create a new value when changing them:

~~~jsx
const [profile, setProfile] = useState({
  name: 'Mira',
  city: 'Pune',
});

function updateCity() {
  setProfile({
    ...profile,
    city: 'Jaipur',
  });
}
~~~

Do not change profile.city directly. Mutating an object and passing the same object back may not produce the expected render. Dedicated chapters cover object and array updates in more depth.

## Common mistakes

- **Using a local variable for UI state:** it does not persist as React state and changing it does not request a render.
- **Assigning to the state variable:** call its setter instead.
- **Calling a setter during rendering:** state changes should normally come from an event, prop-driven synchronization, or another deliberate React update path.
- **Calling useState conditionally:** Hooks must keep the same call order.
- **Using an unsuitable initial type:** initialize text as a string, booleans as booleans, and collections as arrays when that is the expected shape.
- **Assuming a setter returns the new state:** it returns no value; the next render receives the updated state.
- **Expecting a new initial argument to reset state:** React uses the initializer for the component's state initialization, not as a reset command on every render.
- **Mutating an object or array:** create and set a new value instead.

## Practice

1. Build a counter with increment, decrement, and reset buttons.
2. Build a disclosure that shows or hides a paragraph from boolean state.
3. Render two copies of the same stateful component and confirm their state is independent.
4. Add an input whose displayed value is controlled by a string state variable.
5. Inspect a component and decide which values need state and which can be calculated during rendering.

## Summary

useState gives a component a value that persists between renders and a setter that requests a new render with the next value. Call it at the top level, choose a useful initial value, and render the interface from state. State is private to each component instance; use props and shared ownership when another component needs the same information.