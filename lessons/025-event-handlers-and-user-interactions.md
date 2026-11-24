# 025. Event handlers and user interactions

## Learning goals

By the end of this lesson, you should be able to:

- Attach a function to a JSX event prop such as `onClick`.
- Explain the difference between passing a handler and calling it during render.
- Write named and inline handlers, including handlers that need an argument.
- Pass interaction callbacks through custom components.
- Choose semantic HTML elements that already support the intended interaction.
- Keep user-triggered work in event handlers and UI output in React rendering.

## What an event handler does

A user interaction is something that happens after the interface is rendered: a click, a key press, a pointer entering an element, or a form submission. An event handler is a function React calls in response to one of these events.

Define the function, then pass it to the matching JSX prop. React event props use camelCase names:

~~~jsx
function SaveButton() {
  function handleClick() {
    console.log('Save was requested');
  }

  return (
    <button type="button" onClick={handleClick}>
      Save
    </button>
  );
}
~~~

The function body does not run when `SaveButton` renders. React keeps the function as the button's click handler and calls it when the user activates the button.

Common event props include `onClick`, `onChange`, `onSubmit`, `onFocus`, `onBlur`, `onPointerEnter`, and `onKeyDown`. Use the event that matches the interaction and the element's semantics. The event object and the details of event propagation are covered in later chapters.

## Pass a function; do not call it during render

These expressions look similar but behave differently:

~~~jsx
<button onClick={handleClick}>Save</button>
~~~

This passes the function. It runs after the click.

~~~jsx
<button onClick={handleClick()}>Save</button>
~~~

This calls `handleClick` immediately while React is rendering the JSX. Its return value, not the function, is assigned to `onClick`.

If a handler needs an argument, pass a new function that calls it later:

~~~jsx
<button onClick={() => handleSelect(product.id)}>
  Select {product.name}
</button>
~~~

The arrow function is created during rendering, but its body waits until the event. Avoid writing `onClick={handleSelect(product.id)}` when the intent is to wait for a click.

## Use named or inline handlers

A named handler is useful when the work needs several lines, has a meaningful name, or is reused:

~~~jsx
function DownloadButton({ fileName }) {
  function handleDownload() {
    console.log(`Download requested for ${fileName}`);
  }

  return (
    <button type="button" onClick={handleDownload}>
      Download {fileName}
    </button>
  );
}
~~~

A short inline function is convenient when it keeps a simple one-off action clear:

~~~jsx
<button type="button" onClick={() => setIsOpen(true)}>
  Open details
</button>
~~~

By convention, handler names often start with `handle`, followed by the event or action, such as `handleClick`, `handleSubmit`, or `handleSelect`. Good names describe what the function does. Avoid calling the handler while assigning it.

## Pass interactions through custom components

A parent can decide what should happen, then pass a callback through a custom component to the native element that receives the interaction:

~~~jsx
function ActionButton({ children, onAction }) {
  return (
    <button type="button" onClick={onAction}>
      {children}
    </button>
  );
}

function Toolbar() {
  function handleExport() {
    console.log('Export requested');
  }

  return <ActionButton onAction={handleExport}>Export</ActionButton>;
}
~~~

`onAction` is an application-defined prop; React does not treat it as a built-in DOM event. `ActionButton` connects that callback to the native button's `onClick`. The parent owns the behavior, and the reusable component provides the interactive control.

Use callback names that communicate intent, such as `onSave`, `onDelete`, or `onSelect`. A component can expose a more specific callback than the browser event itself. This lets callers respond to the user's intent without depending on the component's internal markup.

## Choose an element that matches the interaction

Use a native `<button>` for an action and an `<a href="...">` for navigation. Native elements already provide expected keyboard behavior, focus behavior, and browser semantics. A clickable `<div>` does not automatically behave like a button for keyboard or assistive technology users.

Buttons inside forms submit the form by default unless their `type` says otherwise. Set `type="button"` for a button that performs a separate action, and use `type="submit"` when it should submit the form. Form submission and validation are covered later.

Use an element's meaningful event prop. For example, use `onChange` with a control to respond when its value changes, and `onSubmit` on a form for its submission. Do not implement a familiar control by recreating its behavior on a generic container unless there is a strong reason and the full accessibility behavior is supplied.

## Keep rendering separate from interaction work

A component renders in order to describe what the UI should look like for its current props and state. React can render more than once. Do not perform user-triggered work as a side effect of rendering:

~~~jsx
function DeleteButton({ onDelete }) {
  onDelete(); // Incorrect: this runs whenever the component renders.
  return <button type="button">Delete</button>;
}
~~~

Connect the work to the interaction instead:

~~~jsx
function DeleteButton({ onDelete }) {
  return (
    <button type="button" onClick={onDelete}>
      Delete
    </button>
  );
}
~~~

Updating state from an event handler is a common way to change what the next render displays. An event handler can also perform work tied directly to that user action, such as starting a download or submitting a request. Effects are for synchronizing with an external system because of rendered state, not for replaying a specific user action after every render.

Do not directly change the DOM to show or hide React-managed UI. Use state to describe the desired output and let React render it. The course will cover state updates and event objects in more detail in later chapters.

## Understand what runs and when

| Code | When it runs | Typical use |
| --- | --- | --- |
| `handleSave` | Not by naming it; only when called | A handler function passed as a callback. |
| `onClick={handleSave}` | React invokes it after click activation | Register a named click handler. |
| `onClick={() => handleSave(id)}` | The arrow body runs after click activation | Call a handler with a specific argument. |
| `onClick={handleSave(id)}` | `handleSave(id)` runs during render | Usually a mistake when waiting for a click. |
| `setLabel('Saved')` in a render body | During render | Usually a mistake; event-triggered updates belong in a handler. |

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Writing `onClick={handleClick()}` | It invokes the function during rendering. | Pass `onClick={handleClick}`. |
| Calling a handler in the component body | It runs every time that component renders. | Attach it to the relevant event prop. |
| Using a clickable `<div>` for a button action | Keyboard and browser button behavior are missing. | Use a native `<button>`. |
| Using a button for navigation | It does not express a destination as a link. | Use an anchor with an `href` or the app's router link. |
| Forgetting `type="button"` inside a form | The button may submit the form unexpectedly. | Set the correct button type explicitly. |
| Passing an inline call when an argument is needed | The call happens immediately instead of after interaction. | Wrap the call in a function, such as `() => handleSelect(id)`. |
| Making every behavior part of a reusable button | The component becomes tied to one application's action. | Accept a callback prop and let the caller decide the behavior. |
| Changing the DOM directly to reflect a click | The DOM and React's rendered output can disagree. | Update the appropriate state and render from it. |

## Key takeaways

- React calls event handler functions in response to user interactions.
- Pass a function to an event prop; do not call it during rendering.
- Wrap a function in an arrow when you need to provide arguments later.
- Pass callbacks through custom components to keep behavior at the right owner.
- Use native buttons for actions and links for navigation.
- Keep interaction work in event handlers and render the resulting UI from props and state.
- Use the event object and propagation deliberately; those details have their own chapters.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [024 - Component identity, position, and keys](024-component-identity-position-and-keys.md) | [Open Q&A](../qnas/025-event-handlers-and-user-interactions.md) | [Open Example](../examples/025-event-handlers-and-user-interactions.md) | [026 - Event objects and React's event system](026-event-objects-and-react-event-system.md) |