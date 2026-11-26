# 027. Event propagation, default actions, and cancellation - Q&A

## Propagation and handler order

### 1. What is event propagation?

It is an event traveling through elements associated with the interaction. A bubbling event can run a handler on the target and then handlers on its ancestors.

### 2. What happens when I click a button inside an element with `onClick`?

The button's click handler runs, then the event can bubble to the ancestor's `onClick` handler. If the button has no handler, the ancestor can still receive the click.

### 3. What is the capture phase?

It is the phase where an event travels from ancestors toward the target before the target's regular handler runs. In React, a prop such as `onClickCapture` registers a capture handler.

### 4. What is the usual order for nested capture and bubble handlers?

Capture handlers run from outer ancestors toward the target, then the target's regular handler runs, then bubble handlers run from the target back toward outer ancestors.

### 5. Does `onClickCapture` run before a child calls `stopPropagation()`?

Yes. Capture handlers run before the target's regular handler. A child cannot undo a capture handler that already ran by stopping propagation afterward.

### 6. Does every React event bubble the same way?

No. Many common events such as clicks bubble, but particular events have special behavior. For example, React `onScroll` does not bubble like `onClick`; pointer and mouse enter/leave handlers also have special propagation behavior.

### 7. Why would I use a capture handler?

A capture handler lets an ancestor observe an interaction before target and bubble handlers run. It can be useful for broad logging or monitoring, but most local application behavior belongs on the component that owns the action.

## Stopping propagation

### 8. What does `event.stopPropagation()` do?

It stops the event from continuing to ancestor handlers along its propagation path. The handler that calls it still runs.

### 9. When is stopping propagation useful?

When an action inside a larger region should not also trigger that ancestor's behavior. For example, a delete button inside a selectable row may need to delete without selecting the row.

### 10. Does `stopPropagation()` cancel a link or form's default behavior?

No. It affects the event path, not the browser's default action. Use `preventDefault()` if the default action itself should be canceled.

### 11. Does `stopPropagation()` prevent a handler that already ran?

No. It cannot undo a target handler or capture handler that already executed. It prevents further propagation after the call.

### 12. Should I stop every event from bubbling out of a reusable component?

No. That would block useful parent behavior and make components harder to compose. Stop an event only when a clear interaction boundary requires it.

### 13. If a child should trigger some parent behavior, should I rely on bubbling?

Not always. An explicit callback prop often makes the intended relationship easier to follow. The child can report a specific action while the parent decides what it means.

### 14. Does stopping propagation from a child button stop the button's own callback?

No. The child handler runs, and it can call `stopPropagation()` before or after invoking its own callback. The parent bubble handler is the part that will not receive the event after propagation is stopped.

## Preventing browser default actions

### 15. What does `event.preventDefault()` do?

It asks the browser not to perform the event's default action when that action is cancelable. A common example is preventing a form submit from navigating or reloading the page while the application handles the data.

### 16. Does `preventDefault()` stop ancestor handlers?

No. The event can continue to propagate. Preventing a browser default and stopping ancestor handlers are separate operations.

### 17. Does `stopPropagation()` prevent form submission?

No. It does not cancel the form's default submit action. Call `preventDefault()` in the form's submit handler when the application takes over submission.

### 18. Does returning `false` from a React event handler prevent the default action?

No. React does not interpret a `false` return value as cancellation. Call `event.preventDefault()` explicitly.

### 19. Should I prevent the default for every link or form?

No. Allow the browser to navigate or submit when that is the intended behavior. Prevent the default only when the application replaces it with another behavior.

### 20. Can one handler call both `preventDefault()` and `stopPropagation()`?

Yes, if the interaction requires both. Each method has a different purpose; do not call them together by habit.

### 21. What does `event.defaultPrevented` tell me?

It reports whether a handler has called `preventDefault()` for that event. It does not tell you whether propagation was stopped.

## Semantic controls and nested interactions

### 22. Why avoid a button inside another button?

Interactive controls should not be nested inside one another. Nested buttons create conflicting browser semantics and interaction behavior. Restructure the UI into separate controls or use one clear interactive element.

### 23. How can I make a whole row selectable without confusing child actions?

Choose a structure that makes each action clear. Use a dedicated link or button for selection, keep separate actions as sibling controls, and use propagation control only when a real nested interaction requires it.

### 24. Should I add an `onClick` to a generic `div` to make it act like a button?

Prefer a native button for an action or an anchor for navigation. A generic element does not gain keyboard behavior or native semantics just because it has an event handler.

## React tree and special cases

### 25. Do portal events follow the DOM tree?

They follow the React tree. A portal changes where its content is physically rendered in the DOM, but an event from that content can still propagate to React ancestors that are not DOM ancestors.

### 26. How do I handle an event from a portal that should not reach a React parent?

Stop propagation inside the portal at the interaction boundary, or reorganize the React tree if the parent-child relationship is not intended. Also check which React ancestor owns the handler.

### 27. Does `onScroll` bubble to an ancestor in React?

React's `onScroll` handler does not bubble like `onClick`. Attach it to the element whose scrolling you need to observe, or use the corresponding capture prop where appropriate.

### 28. How should I debug a handler that runs unexpectedly?

Trace the event from its target through capture, target, and bubble handlers in the React tree. Check whether a portal changes the React ancestry, then decide whether the ancestor should observe that interaction.

### 29. How should I choose between `preventDefault()` and `stopPropagation()`?

Ask two separate questions: should the browser perform its default action, and should ancestor handlers receive the event? Use `preventDefault()` for the first and `stopPropagation()` for the second.

### 30. What is the main rule to remember?

Let normal event flow work unless a real interaction needs a boundary. Use semantic controls, cancel browser defaults only when replacing them, and stop propagation only when ancestors should not handle the event.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="026-event-objects-and-react-event-system.md">026 - Event objects and React's event system - Q&amp;A</a></td><td><a href="../lessons/027-event-propagation-and-default-actions.md">Lesson</a></td><td><a href="../examples/027-event-propagation-and-default-actions.md">Example</a></td><td><a href="028-controlled-form-elements.md">028 - Controlled form elements - Q&amp;A</a></td></tr>
  </tbody>
</table>
