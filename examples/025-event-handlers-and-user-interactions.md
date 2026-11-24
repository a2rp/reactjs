# 025. Event handlers and user interactions - Example

## Goal

Build a small action panel that demonstrates:

- Passing a named function to a button for a direct callback.
- Wrapping a handler when the caller needs to provide an argument.
- Passing application behavior from a parent through a reusable component.
- Updating rendered feedback in response to a user action.
- Using native button behavior and a disabled state.

## Files

In an existing React project, create or update:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
import { useState } from 'react';

function ActionButton({ children, onAction, disabled = false }) {
  return (
    <button type="button" disabled={disabled} onClick={onAction}>
      {children}
    </button>
  );
}

export default function App() {
  const [lastAction, setLastAction] = useState('No action yet.');
  const [actionCount, setActionCount] = useState(0);

  function handleAction(actionName) {
    setLastAction(`${actionName} was requested.`);
    setActionCount((currentCount) => currentCount + 1);
  }

  function handleReset() {
    setLastAction('Activity was cleared.');
    setActionCount(0);
  }

  return (
    <main>
      <h1>Document actions</h1>
      <p>Choose an action to see how a callback updates the interface.</p>

      <div>
        <ActionButton onAction={() => handleAction('Preview')}>
          Preview
        </ActionButton>
        <ActionButton onAction={() => handleAction('Save draft')}>
          Save draft
        </ActionButton>
        <ActionButton disabled onAction={() => handleAction('Publish')}>
          Publish unavailable
        </ActionButton>
        <ActionButton onAction={handleReset}>Clear activity</ActionButton>
      </div>

      <section aria-labelledby="activity-heading">
        <h2 id="activity-heading">Activity</h2>
        <p role="status">{lastAction}</p>
        <p>Actions requested: {actionCount}</p>
      </section>
    </main>
  );
}
~~~

## Follow a click through the components

1. `ActionButton` renders a native `<button>` and attaches its `onAction` prop to the button's `onClick` event.
2. When Preview is activated, its arrow function calls `handleAction('Preview')`.
3. `handleAction` updates the parent's state with the action name and increments the count.
4. React renders the updated status and count from the new state.

The click handler is passed as a function. `handleAction('Preview')` is inside an arrow function, so it runs after the click. Writing `onAction={handleAction('Preview')}` would call it during rendering and pass its return value instead.

The Preview and Save buttons need to pass different action names, so they use wrapper functions. The Clear activity button needs no argument and passes the named function directly as `onAction={handleReset}`.

`onAction` is a custom prop for `ActionButton`; the native button does not have a built-in `onAction` event. `ActionButton` connects that intent callback to its native `onClick` prop. The parent owns the effects of selecting Preview or Save draft, while the reusable button handles the interactive control.

The Publish button uses the browser's native `disabled` behavior. It cannot be activated while disabled, including through the button's usual keyboard activation.

## Try these changes

1. Add a `Download` button and pass `() => handleAction('Download')`.
2. Replace the wrapper with `onAction={handleAction}`. Observe that the button supplies an event object, not the action name. Restore the wrapper that passes the intended value.
3. Add a named `handlePreview` function that calls `handleAction('Preview')`, then pass it directly. Compare it with the inline arrow version.
4. Change the activity message without changing `ActionButton`. The parent controls the action behavior through the callback prop.
5. Remove `type="button"` and place the panel inside a form. Notice why an action button's form type should be intentional.
6. Try replacing the button with a clickable `<div>`. Then restore the button and use native keyboard behavior for the action.

## What this example does not do

The panel records a local interaction; it does not perform a network request, save data to a server, or navigate to another page. Those behaviors can be placed in the parent handler when the application requires them. Use an anchor for navigation and a form submit handler for form submission.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [024 - Component identity, position, and keys](../lessons/024-component-identity-position-and-keys.md) | [025 - Event handlers and user interactions](../lessons/025-event-handlers-and-user-interactions.md) | [Open Q&A](../qnas/025-event-handlers-and-user-interactions.md) | [026 - Event objects and React's event system](../lessons/026-event-objects-and-react-event-system.md) |