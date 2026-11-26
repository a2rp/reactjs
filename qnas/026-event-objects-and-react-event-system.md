# 026. Event objects and React's event system - Q&A

## The React event object

### 1. What does React pass to an event handler?

React passes an event object as the handler's first argument. It describes the interaction and provides properties and methods such as `type`, `target`, `currentTarget`, `preventDefault()`, and `stopPropagation()`.

### 2. What is a React or synthetic event?

It is the event object React provides to event handlers. It follows the familiar DOM event interface while presenting event values through React's event system. Most application code can use it without accessing the underlying browser event.

### 3. What does `event.type` contain?

It identifies the React event being handled, such as `click`, `change`, or `keydown`. Use the event prop and event type relevant to the interaction.

### 4. How do I receive the event in a handler?

Pass the function directly and let React supply the event when it calls the handler:

~~~jsx
function handleClick(event) {
  console.log(event.type);
}

<button onClick={handleClick}>Open</button>
~~~

### 5. How do I pass both an application value and the event to a handler?

Use a wrapper function:

~~~jsx
<button onClick={(event) => handleSelect(item.id, event)}>
  Select
</button>
~~~

The wrapper receives the event and can pass the ID and event to another function.

## `target` and `currentTarget`

### 6. What is `event.target`?

It is the element where the event originated. If a user clicks a nested icon inside a button, the icon can be the target.

### 7. What is `event.currentTarget`?

It is the element whose handler is currently running. In a nested button example, the icon may be `target` while the button is `currentTarget`.

### 8. Which property should I use to read a value from the input whose handler is running?

Use `event.currentTarget.value` in that input's handler. It explicitly refers to the element the handler is attached to.

### 9. Are `target` and `currentTarget` always different?

No. If the interaction starts on the element that owns the handler, they can be the same. They differ when the event originates on a descendant or is handled by an ancestor.

### 10. Why can `target` be an unexpected element?

The event can begin on a nested child, such as a `span` inside a button. If the handler needs the button that owns it, use `currentTarget` instead of assuming the target is the button.

## Event-specific properties and native events

### 11. Do all event types expose the same additional properties?

No. Keyboard events expose values such as `key` and `code`; pointer or mouse events expose coordinates and button information; focus events expose focus-related information. Read properties that belong to the event you are handling.

### 12. What is the difference between keyboard `key` and `code`?

`key` describes the value produced by the user's keyboard layout and modifiers. `code` identifies a physical key location. Choose based on whether the behavior should follow the produced character or a physical key.

### 13. What is `event.nativeEvent`?

It is the underlying browser DOM event. Most code should use the React event object. Access `nativeEvent` only when a browser-specific property is needed.

### 14. Does a React event prop always map to a native event with the same name?

No. The mapping can differ. For example, React's `onMouseLeave` can be backed by a native `mouseout` event. Avoid relying on the exact native type mapping as part of your component API.

### 15. What is `event.defaultPrevented`?

It indicates whether the event's default browser action has been prevented. It is useful when code needs to check whether a handler or another handler already called `preventDefault()`.

### 16. Should I store the whole event object in state?

No. Store the application value you need, such as the input string, selected ID, or form field. That keeps React state independent of a particular interaction object.

### 17. Do I still need to call `event.persist()` in React DOM?

No. Event pooling is not used by modern React DOM, so `persist()` is unnecessary there. Read the needed event values in the handler and pass ordinary values to asynchronous work.

### 18. How should I use event data in an asynchronous function?

Read the needed value synchronously in the handler, then pass that value to the asynchronous function:

~~~jsx
function handleChange(event) {
  const value = event.currentTarget.value;
  saveLater(value);
}
~~~

This avoids making the event object itself part of application data.

## Browser defaults and propagation

### 19. What does `event.preventDefault()` do?

It asks the browser not to perform the event's default action, such as the normal page navigation for a link or a normal form submission, when cancellation is allowed.

### 20. What does `event.stopPropagation()` do?

It stops the event from continuing to ancestor handlers along its propagation path. It does not cancel the browser's default action.

### 21. Does `preventDefault()` stop a parent click handler?

No. It affects the browser's default action, not propagation. Use `stopPropagation()` only when an event should not reach ancestor handlers.

### 22. Does `stopPropagation()` prevent a link from navigating?

No. It stops propagation to ancestors. Call `preventDefault()` if the application intentionally needs to cancel the link's default navigation.

### 23. Does returning `false` from a React handler cancel the default action?

No. Call `event.preventDefault()` explicitly when you intend to cancel a browser default.

### 24. Should I call `preventDefault()` on every form submit or link click?

No. Cancel the default only when the application intentionally replaces it. Native forms and links provide useful browser behavior when they are meant to submit or navigate normally.

### 25. What does `event.cancelable` tell me?

It indicates whether the event's default action can be canceled. `preventDefault()` only has an effect when cancellation is supported and the event has a default action to cancel.

### 26. Does this chapter cover event bubbling and capture fully?

No. It introduces the event object's propagation method, but the next lesson covers event propagation, capture, default actions, and cancellation patterns in more depth.

### 27. What should I remember about the event system?

Use React's documented event props and event object. Read from `currentTarget` when you need the element whose handler is running, and use `nativeEvent` only for browser-specific details rather than relying on internal mappings.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="025-event-handlers-and-user-interactions.md">025 - Event handlers and user interactions - Q&amp;A</a></td><td><a href="../lessons/026-event-objects-and-react-event-system.md">Lesson</a></td><td><a href="../examples/026-event-objects-and-react-event-system.md">Example</a></td><td><a href="027-event-propagation-and-default-actions.md">027 - Event propagation, default actions, and cancellation - Q&amp;A</a></td></tr>
  </tbody>
</table>
