# 021. Pure components and predictable rendering - Q&A

## Purity and rendering

### 1. What does it mean for a component to render purely?

The component calculates its JSX from its current inputs and does not change values or perform observable work outside that calculation. When it receives the same props, state, and context, it should describe the same UI.

### 2. Does pure mean the component always returns the exact same JSX?

No. Different inputs can produce different output. Purity means the output is determined by the inputs, not by hidden changes, render order, or an unrelated external event.

### 3. Does a pure component have to return the same JavaScript object reference?

No. JSX can be described again during another render. The important point is that the UI result is determined by the same inputs and that rendering does not cause unrelated changes.

### 4. Why does React expect render code to be predictable?

React may render components again while processing updates or checking work. If rendering changes shared values or starts side effects, repeating or reordering that work can produce inconsistent UI or duplicate actions.

### 5. Does a component have to avoid all side effects forever?

No. Side effects are needed in applications. Keep them out of the render calculation and perform them in response to an event or, when synchronizing with an external system, in an Effect.

## Inputs and mutation

### 6. Which values should I treat as read-only during render?

Treat props, state, and context as read-only inputs for that render. Do not assign to them or mutate objects they reference. Use state setters for state changes and pass new values or copies instead of changing an input object.

### 7. Why is mutating a prop object unsafe?

The object may be shared with its parent or another component. Mutating it creates changes outside the current component's calculation, so other code can observe unexpected values or become out of sync.

### 8. How can I transform a prop for display without mutating it?

Calculate a new value in a local variable:

~~~jsx
function Profile({ person }) {
  const displayName = person.name.trim();
  return <h2>{displayName}</h2>;
}
~~~

This reads the prop and creates a display value without changing `person.name`.

### 9. Is every mutation during render forbidden?

No. It is safe to mutate a fresh local object created during that render, as long as it is not shared with code outside the component. For example, building a new local array with `push()` is fine. Mutating an array received through props or a module-level array is not.

### 10. Can I update a state variable directly inside a component?

No. State is an input snapshot for that render. Update it through the setter returned by `useState`, usually from an event handler. Directly assigning to a state variable does not tell React to render the updated value and can make the UI inconsistent.

### 11. Is a module-level constant always a problem to read during render?

No. Reading an immutable constant is generally predictable. The problem is changing a preexisting variable or object during render, or reading a value that changes independently, such as the current time or a random number, when it is not an input.

## Side effects and timing

### 12. Can I call `Math.random()` or `new Date()` during render?

A call such as `Math.random()` or `new Date()` can return a different result on each render even when the component inputs have not changed. If that value should be part of the UI, supply it as an input or update it outside the render calculation when appropriate.

### 13. Where should work triggered by a button click go?

Put it in the button's event handler. The handler function can be defined inside the component, but its body runs when the event occurs, not when the component returns JSX:

~~~jsx
<button onClick={handleSave}>Save</button>
~~~

### 14. What is wrong with `onClick={handleSave()}`?

That expression calls `handleSave` while rendering and passes its return value as the handler. Pass the function itself as `onClick={handleSave}` so React can call it after the user clicks.

### 15. When should an Effect be used?

Use an Effect when a component needs to synchronize with an external system after rendering, such as a timer, browser event listener, or external widget. Do not use an Effect just to calculate a value that can be derived from props or state during render.

### 16. Can a component change the document title directly in its render function?

No. Changing `document.title` is an observable DOM side effect. Render should describe UI with JSX. If a title must synchronize with the rendered state, do that outside the render calculation using the appropriate synchronization approach.

## Strict Mode and debugging

### 17. What does Strict Mode do to help find impure render logic?

In development, Strict Mode intentionally re-runs certain render-related code to expose logic that changes shared values or otherwise behaves differently when called more than once. It helps reveal bugs; it does not make an impure component pure by itself.

### 18. Does Strict Mode run the same extra checks in production?

No. These extra development checks are for finding problems while building the app. Production behavior does not rely on Strict Mode deliberately repeating render logic in the same way.

### 19. Should I fix a Strict Mode duplicate by disabling Strict Mode?

No. Treat the repeated result as a clue that render has a side effect or depends on hidden state. Make rendering predictable so it is safe when called again.

### 20. Can components depend on the order React renders them?

No. Each component should calculate its own output from its inputs. Do not use a shared counter or mutable global to coordinate what one component renders before another.

### 21. Is logging from render a good way to track user actions?

No. Render can run more than once, so a log in render may be repeated and does not necessarily mean a user action happened. Log an interaction in its event handler when that is what you mean to record.

### 22. Does pure rendering mean I cannot build arrays inside a component?

No. Creating and filling a new local array during render is safe. The component returns that derived result and does not modify data that existed before the render.

### 23. Does `React.memo` make an impure component pure?

No. Memoization can skip some renders as an optimization, but it does not correct render logic that mutates shared data or causes side effects. Components should remain predictable regardless of whether React renders them again.

### 24. What is the main rule to remember?

During render, read the current inputs, calculate the JSX, and avoid changing the outside world. Put user-triggered work in event handlers and external synchronization in Effects when needed.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [020 - Component boundaries and responsibility](../lessons/020-component-boundaries-and-responsibility.md) | [021 - Pure components and predictable rendering](../lessons/021-pure-components-and-predictable-rendering.md) | [Open Example](../examples/021-pure-components-and-predictable-rendering.md) | [022 - Designing reusable component APIs](../lessons/022-designing-reusable-component-apis.md) |