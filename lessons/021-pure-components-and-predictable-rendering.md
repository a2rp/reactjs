# 021. Pure components and predictable rendering

## Learning goals

By the end of this lesson, you should be able to:

- Explain what it means for a component's render to be pure.
- Keep props, state, and context read-only while rendering.
- Identify side effects that must not happen during render.
- Distinguish safe local mutation from changing data that existed before the render.
- Move user-triggered work into event handlers and external synchronization into Effects when needed.
- Use Strict Mode's development behavior to find accidental side effects.

## Think of rendering as a calculation

A React component receives inputs and returns a description of the UI. A pure component produces the same JSX when it receives the same props, state, and context. It does not change values that existed before the render or do other observable work while calculating its output.

~~~jsx
function Recipe({ servings }) {
  const waterCups = servings;
  const teaSpoons = servings;

  return (
    <section>
      <h2>Tea for {servings}</h2>
      <p>Water: {waterCups} cups</p>
      <p>Tea: {teaSpoons} spoons</p>
    </section>
  );
}
~~~

For a given `servings` value, `Recipe` returns the same description of the recipe. A different prop can produce different output; purity does not mean a component is static. It means the output is determined by its inputs rather than hidden changes or external events during render.

## React may render more than once

Rendering calculates what the UI should look like. React may call a component again to handle updates, prioritize work, or check for mistakes. Components should not rely on a particular render count or order. If calculating a component changes something outside itself, an extra render can produce different results or duplicate work.

This is why components should behave like formulas: read the current inputs, calculate the output, and return JSX.

## Keep render free of side effects

A side effect is work beyond calculating the return value. Do not perform observable work during render, such as:

- Changing the DOM directly.
- Writing to a global or module-level variable.
- Mutating a prop, state value, context value, or an object supplied by another component.
- Making a network request, starting a timer, or registering a listener.
- Generating a value that changes each time, such as `Math.random()` or the current time, when that value is not an input.

For example, this component changes shared data while React is asking it what to render:

~~~jsx
let guestNumber = 0;

function GuestLabel() {
  guestNumber += 1;
  return <p>Guest {guestNumber}</p>;
}
~~~

The same call can now return different text depending on how many times it ran and what other components changed the variable. That makes the result unpredictable.

Pass the value as an input instead:

~~~jsx
function GuestLabel({ guestNumber }) {
  return <p>Guest {guestNumber}</p>;
}
~~~

Now the output follows the prop. React can render the component again without incrementing or changing shared data.

## Treat props, state, and context as read-only inputs

Props, state, and context describe the inputs for one render. Do not assign to them or mutate objects they reference. If you need to transform a value for display, derive a new value:

~~~jsx
function Profile({ person }) {
  const displayName = person.name.trim();

  return <h2>{displayName}</h2>;
}
~~~

Do not modify the incoming object:

~~~jsx
function Profile({ person }) {
  person.name = person.name.trim(); // Avoid changing a prop object.
  return <h2>{person.name}</h2>;
}
~~~

If the user changes something, handle the interaction and update state with its setter. The next render receives the updated state. Do not try to change a state variable directly while rendering.

## Local mutation is safe

Purity does not forbid every use of methods such as `push`. It is safe to mutate an object that was created locally during the current render and is not shared with other code:

~~~jsx
function GuestList({ guests }) {
  const rows = [];

  for (const guest of guests) {
    rows.push(<li key={guest.id}>{guest.name}</li>);
  }

  return <ul>{rows}</ul>;
}
~~~

`rows` is created inside `GuestList` each time it runs. No earlier render or other component can observe its contents changing. Using `map()` is also fine:

~~~jsx
function GuestList({ guests }) {
  const rows = guests.map((guest) => (
    <li key={guest.id}>{guest.name}</li>
  ));

  return <ul>{rows}</ul>;
}
~~~

The important distinction is whether the code changes a value that existed before the render or only builds a temporary local result.

## Put side effects in event handlers

An event handler runs in response to an interaction, not while React calculates the component's output. Work such as logging a user action or sending a request after a button click belongs in the handler for that action:

~~~jsx
function SaveButton({ onSave }) {
  function handleClick() {
    onSave();
  }

  return <button onClick={handleClick}>Save</button>;
}
~~~

Defining `handleClick` during render is fine. Its body runs only when the user clicks. Do not call it while rendering, such as `onClick={handleClick()}`, because that would execute the work immediately instead of registering the handler.

## Use an Effect only to synchronize with an external system

Sometimes a component must synchronize with something outside React, such as a browser event listener, a timer, or an external widget. An Effect is one place for that work because it runs after rendering. Use an Effect when there is an external system to synchronize with; do not use one just to calculate a value that can be derived from props or state during render.

This lesson focuses on the boundary: render calculates UI, while event handlers and Effects perform side effects at the appropriate time. The details of Effects are covered later in the course.

## Strict Mode can reveal impure rendering

When Strict Mode is enabled in development, React intentionally calls render-related code more than once in certain situations. A component that increments a shared variable, mutates props, or creates side effects during rendering may then show duplicated or inconsistent results.

Treat this as a clue to look for render-time changes. Do not fix it by relying on a specific number of calls or by disabling Strict Mode. Make the render calculation pure so repeated calls with the same inputs remain safe. Strict Mode's extra development checks do not run the same way in production.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Incrementing a module-level variable during render | Output depends on prior calls or other components. | Pass the value as a prop or keep changing information in state. |
| Mutating a prop or an object from props | The parent and other consumers can observe unexpected changes. | Calculate a display value or create a copy. |
| Calling `Math.random()` or reading the current time for render output | The same inputs can produce a different result each time React renders. | Pass the value in as an input or update it outside render when needed. |
| Changing the DOM during render | React and the direct DOM change can disagree. | Describe the UI in JSX; synchronize with external systems outside render. |
| Starting a request or timer during render | Extra renders can start duplicate work. | Start interaction-driven work in an event handler or external synchronization in an Effect. |
| Calling an event handler while assigning it | The handler runs during render rather than after an interaction. | Pass the function itself, for example `onClick={handleClick}`. |
| Mutating a temporary array created during the same render and assuming all mutation is forbidden | This adds needless complexity and misunderstands purity. | Local mutation of fresh values is safe; avoid changing shared or preexisting values. |
| Using an Effect to calculate a value from props | It adds an extra update and duplicates data unnecessarily. | Derive the value directly during render. |

## Key takeaways

- A pure render returns UI based on its current inputs without changing outside values.
- The same props, state, and context should produce the same JSX.
- Treat props, state, and context as read-only snapshots.
- Local temporary objects can be mutated safely when they are created during that render and not shared.
- Keep side effects out of render; handle user-triggered work in event handlers.
- Use Effects to synchronize with external systems when needed, not as a default calculation step.
- Strict Mode can expose code that breaks when rendering happens more than once.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [020 - Component boundaries and responsibility](020-component-boundaries-and-responsibility.md) | [Open Q&A](../qnas/021-pure-components-and-predictable-rendering.md) | [Open Example](../examples/021-pure-components-and-predictable-rendering.md) | [022 - Designing reusable component APIs](022-designing-reusable-component-apis.md) |