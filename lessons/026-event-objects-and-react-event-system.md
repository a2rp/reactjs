# 026. Event objects and React's event system

## Learning goals

By the end of this lesson, you should be able to:

- Describe the React event object passed to an event handler.
- Distinguish `event.target` from `event.currentTarget`.
- Read common and event-specific event properties.
- Explain what `nativeEvent` is and when not to depend on it.
- Distinguish preventing a browser default from stopping event propagation.
- Use event data synchronously inside the handler that receives it.

## React passes an event object to the handler

When React calls an event handler, it passes an event object as the first argument. It contains information about the interaction and methods for controlling browser behavior:

~~~jsx
function handleClick(event) {
  console.log(event.type);
}

<button onClick={handleClick}>Save</button>
~~~

The object is often called a React event or a synthetic event. It follows the familiar DOM event interface while providing consistent values through React's event system. The exact internal listener setup is an implementation detail; write code against the event props and documented event object.

If an inline wrapper is needed, it receives the event too:

~~~jsx
<button onClick={(event) => handleSave(record.id, event)}>
  Save {record.name}
</button>
~~~

The wrapper can pass both an application value and the event to another function. When you do not need the event, you can ignore that argument:

~~~jsx
<button onClick={() => handleSave(record.id)}>
  Save {record.name}
</button>
~~~

## `target` and `currentTarget` are different

`event.target` is the element where the event originated. `event.currentTarget` is the element whose handler is currently running.

~~~jsx
<button onClick={(event) => {
  console.log(event.target);
  console.log(event.currentTarget);
}}>
  <span aria-hidden="true">?</span> Save
</button>
~~~

If the user clicks the star, `target` may be the `<span>`, while `currentTarget` is the `<button>` where `onClick` is attached. If the user clicks the button text, the target may be the button itself. Use `currentTarget` when you specifically need the element that owns the current handler�for example, to read an input's value or a form's submitted data.

For a handler attached directly to an input, both properties will ordinarily refer to that input. Still, `currentTarget` communicates that you are reading from the element whose handler is being handled:

~~~jsx
function handleChange(event) {
  const nextValue = event.currentTarget.value;
  console.log(nextValue);
}
~~~

Do not assume the event target is always the element you attached the handler to. Nested elements can be the origin of an interaction.

## Common and event-specific properties

An event object has common properties such as `type`, `target`, `currentTarget`, `bubbles`, `cancelable`, and `defaultPrevented`. The event type also determines which additional properties are useful.

For a keyboard handler, inspect keyboard-specific values:

~~~jsx
function handleKeyDown(event) {
  console.log(event.key);
  console.log(event.code);
  console.log(event.altKey);
}

<input onKeyDown={handleKeyDown} />
~~~

`key` describes the key value produced by the user's keyboard layout and modifiers, while `code` identifies a physical keyboard key. They serve different needs. For text fields, use input/change events to read the text value; a key event is for behavior that specifically depends on a key press.

For pointer and mouse events, the event may expose coordinates and button information. For focus events, it may expose the element receiving or losing focus. Learn the properties for the specific event you handle rather than trying to use one event shape for every interaction.

## Access the underlying browser event only when needed

The React event object exposes the original browser event through `event.nativeEvent`. Most application code should use the React event properties directly. React's event name and the underlying browser event type do not always correspond one-to-one; for example, a React `onMouseLeave` event can be backed by a native `mouseout` event.

Use `nativeEvent` only when you need browser-specific information that the React event object does not provide. Avoid depending on undocumented mappings between React event props and specific native event types; those mappings are not the component API.

Older examples may tell you to call `event.persist()` before using a React event asynchronously. For React DOM, event pooling is no longer used and `persist()` is not needed. Prefer to read and copy the specific values needed by an asynchronous operation inside the handler:

~~~jsx
async function handleSubmit(event) {
  event.preventDefault();
  const formData = new FormData(event.currentTarget);
  const itemName = String(formData.get('item') ?? '');

  await saveItem(itemName);
}
~~~

The handler reads from the form while `currentTarget` identifies the form. It then passes the ordinary string value to the asynchronous function instead of keeping the event object as application data.

## Preventing default browser behavior

Some browser interactions have a default action. A form submission normally navigates or reloads the page. A link follows its destination. Call `event.preventDefault()` when the application intentionally replaces that default behavior:

~~~jsx
function handleSubmit(event) {
  event.preventDefault();
  // Read the form and handle it in the application.
}

<form onSubmit={handleSubmit}>
  <button type="submit">Add item</button>
</form>
~~~

Returning `false` from a React event handler does not cancel the browser's default action. Call `preventDefault()` explicitly when cancellation is required. Do not cancel a default action without a reason; links, forms, and native controls provide useful browser behavior.

## Stopping propagation is a separate operation

`preventDefault()` and `stopPropagation()` solve different problems:

- `preventDefault()` asks the browser not to perform the event's default action.
- `stopPropagation()` prevents the event from continuing to ancestor handlers.

Stopping propagation does not cancel a link or form's default behavior. Preventing the default does not stop a parent handler from running. Use each only for the behavior it controls.

Event propagation, capture handlers, and common nested-interaction patterns are covered in the next chapter. For now, remember that an event object offers separate methods for the browser default and the propagation path.

## Read the event in the right handler

The event object describes one interaction. Read the values you need in its handler and pass ordinary data onward:

~~~jsx
function handleInputChange(event) {
  const value = event.currentTarget.value;
  updateSearchQuery(value);
}
~~~

Avoid storing the event object in React state or passing it around as if it were the user's data. Store a value such as the input string, selected ID, or submitted form field instead. This keeps application state independent of a particular browser event.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Treating `target` and `currentTarget` as interchangeable | A nested child can be the event target while a parent owns the handler. | Use `currentTarget` for the element whose handler is running. |
| Reading an input value from an unrelated nested target | The target may be a child or another element. | Read from the input's `currentTarget` in its handler. |
| Using `nativeEvent.type` as the React event name | React and browser event mappings are not always one-to-one. | Use `event.type` and documented React event props unless native details are needed. |
| Calling `persist()` in React DOM based on an old tutorial | Modern React DOM does not pool these event objects. | Read the needed values in the handler and pass plain data onward. |
| Returning `false` to cancel a browser action | React does not use that return value to cancel the default. | Call `event.preventDefault()` when cancellation is intended. |
| Calling `preventDefault()` to stop a parent handler | It affects browser behavior, not propagation. | Use `stopPropagation()` only when the event should not reach ancestors. |
| Storing an event object as application state | Application state becomes tied to a transient interaction object. | Store the relevant value, such as text or an ID. |
| Reading event-specific fields from the wrong event kind | Different event types expose different useful information. | Choose a handler for the interaction and use its relevant properties. |

## Key takeaways

- React passes an event object to each handler; its first argument is commonly named `event` or `e`.
- `target` is where the event originated; `currentTarget` is where the running handler is attached.
- Event-specific properties differ; use keyboard, pointer, focus, or input data for the interaction you handle.
- `nativeEvent` is available when browser-level details are needed, but do not rely on internal event mappings.
- React DOM does not require `event.persist()`; read the needed values and pass plain data onward.
- Use `preventDefault()` to cancel a browser default and `stopPropagation()` to stop ancestor handlers.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="025-event-handlers-and-user-interactions.md">025 - Event handlers and user interactions</a></td><td><a href="../qnas/026-event-objects-and-react-event-system.md">Q&amp;A</a></td><td><a href="../examples/026-event-objects-and-react-event-system.md">Example</a></td><td><a href="027-event-propagation-and-default-actions.md">027 - Event propagation, default actions, and cancellation</a></td></tr>
  </tbody>
</table>
