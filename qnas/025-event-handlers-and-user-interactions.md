# 025. Event handlers and user interactions - Q&A

## Event handler basics

### 1. What is an event handler in React?

It is a function passed to a JSX event prop. React calls it when the matching interaction occurs, such as a click, focus change, or form submission.

### 2. How do I add a click handler?

Define a function and pass it to `onClick`:

~~~jsx
function SaveButton() {
  function handleClick() {
    console.log('Save requested');
  }

  return <button type="button" onClick={handleClick}>Save</button>;
}
~~~

### 3. Why are React event prop names camelCase?

JSX uses JavaScript prop names, so event props are written like `onClick`, `onChange`, and `onSubmit`, with a capital letter after `on`.

### 4. What is the difference between `onClick={handleClick}` and `onClick={handleClick()}`?

`onClick={handleClick}` passes the function for React to call after a click. `onClick={handleClick()}` calls it immediately during render and passes its return value instead.

### 5. Does creating an inline arrow function run its body during render?

No. The arrow function value is created while rendering, but its body runs when React invokes it after the event. For example, `onClick={() => handleSelect(id)}` waits for a click.

### 6. When should I use a named handler?

Use one when the work has several steps, has a meaningful name, or is reused. Names such as `handleSubmit` and `handleSelect` make the component's behavior easier to follow.

### 7. When is an inline handler appropriate?

For a short action that is easiest to understand at the call site, such as `onClick={() => setIsOpen(true)}`. If it becomes complex, give it a named function.

### 8. How do I pass an argument to a handler after an interaction?

Wrap the call in a function:

~~~jsx
<button onClick={() => handleSelect(product.id)}>
  Select {product.name}
</button>
~~~

Do not write `onClick={handleSelect(product.id)}` when the call should wait for the click.

### 9. Can I pass a handler directly if it expects an event argument?

Yes. When the event occurs, React supplies the event object to the function. For example, `onClick={handleClick}` lets `handleClick` receive that event. Event object details are covered in the next chapter.

### 10. What naming convention is common for handler functions?

Names often start with `handle` and describe the event or action: `handleClick`, `handleSave`, or `handleQueryChange`. Use names that make the action clear.

## Custom components and callbacks

### 11. Can a custom component receive an event handler prop?

Yes. A parent can pass a callback to a custom component, and the component can attach it to a native element. This lets the parent choose what happens while the child provides the control.

### 12. Is `onSave` a built-in React DOM event prop?

No. It can be an ordinary application-defined prop on a custom component. That component may call it in response to an interaction and pass another built-in event prop such as `onClick` to a native button.

### 13. Should a reusable button decide what “save” or “delete” means?

Usually not. A reusable button should expose a callback prop; the calling component decides the application behavior. A more specialized component can use an intent-named callback when that matches its API.

### 14. Why pass a callback down instead of having a child change parent state directly?

The callback is the explicit boundary between the child interaction and parent behavior. The child reports the action, while the parent keeps ownership of its state and application decisions.

### 15. Can an event handler read props from its component?

Yes. A function defined inside a component can use the props available in that render. If you need to bind a specific item ID, an arrow function can call the handler with that ID when the interaction occurs.

## Elements and behavior

### 16. Should I use a `<div>` with `onClick` to make an action button?

Prefer a native `<button>`. A button already supports keyboard activation, focus, and browser semantics. A generic `<div>` does not become a complete accessible button just because it has a click handler.

### 17. When should I use an anchor instead of a button?

Use an anchor with an `href` for navigation to a destination. Use a button for an action that changes something or performs work on the current page.

### 18. Why should I set a button's `type` explicitly?

A button inside a form submits the form by default. Use `type="button"` for a separate action, and `type="submit"` when the button should submit the form.

### 19. Should I attach `onChange` to a text input or listen for every key press?

Use the event that matches the desired behavior. For a controlled text input, `onChange` is the usual way to respond to edits. Keyboard events are for keyboard-specific behavior, not a substitute for observing the input's changed value.

### 20. Can event handlers update state?

Yes. A handler can call a state setter in response to an interaction. React then renders the UI from the updated state. State mechanics are covered in more depth later in the course.

## Render timing and side effects

### 21. Why should user-triggered work stay out of the component body?

React can render a component more than once. Work in the body runs as part of rendering, not only after the user acts. Put interaction-specific work in the event handler so it runs in response to that interaction.

### 22. Should I directly change the DOM in a click handler to update React UI?

Usually no. For UI that React owns, update state and let the next render describe the change. Direct DOM changes can become inconsistent with what React renders.

### 23. Are event handlers allowed to perform side effects?

Yes. A handler is a suitable place for work directly caused by that user action, such as starting a download or submitting a request. Effects serve a different purpose: synchronizing with an external system based on rendered state.

### 24. What is wrong with `onClick={alert('Saved')}`?

`alert('Saved')` executes during render. Pass a function instead, such as `onClick={() => alert('Saved')}`, or use a named handler.

### 25. Why might a button click reload the page unexpectedly?

If the button is inside a form and is a submit button, the browser submits the form. Give non-submit actions `type="button"`; handle the form's `onSubmit` for actual submission behavior.

### 26. Does this chapter fully explain event bubbling and `preventDefault()`?

No. Event objects, propagation, and default browser actions have dedicated later chapters. This chapter focuses on attaching and passing handlers at the right time.

### 27. What is the core rule to remember?

Pass functions to event props so they run after interactions. Use a wrapper when you need arguments, keep behavior at the right component owner, and use native elements that match the user's action.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [024 - Component identity, position, and keys](../lessons/024-component-identity-position-and-keys.md) | [025 - Event handlers and user interactions](../lessons/025-event-handlers-and-user-interactions.md) | [Open Example](../examples/025-event-handlers-and-user-interactions.md) | [026 - Event objects and React's event system](../lessons/026-event-objects-and-react-event-system.md) |