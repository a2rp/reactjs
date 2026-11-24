# 026. Event objects and React's event system - Example

## Goal

Build a small event inspector that demonstrates:

- Reading `type`, `target`, and `currentTarget` from a click event.
- Reading keyboard-specific properties from a keyboard event.
- Reading submitted form data from the form that owns the submit handler.
- Preventing the browser's normal form navigation when React handles the submission.
- Saving ordinary values in state instead of saving event objects.

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

export default function App() {
  const [clickDetails, setClickDetails] = useState('Activate the button to inspect a click.');
  const [keyDetails, setKeyDetails] = useState('Type in the field to inspect a key event.');
  const [submittedName, setSubmittedName] = useState('Nothing submitted yet.');

  function handleButtonClick(event) {
    const targetName =
      event.target instanceof Element
        ? event.target.tagName.toLowerCase()
        : 'unknown';
    const handlerElement = event.currentTarget.tagName.toLowerCase();

    setClickDetails(
      `React event: ${event.type}. Target: <${targetName}>. Handler element: <${handlerElement}>.`,
    );
  }

  function handleKeyDown(event) {
    setKeyDetails(`key: ${event.key}; code: ${event.code}`);
  }

  function handleSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);
    const name = String(formData.get('displayName') ?? '').trim();
    setSubmittedName(name);
  }

  return (
    <main>
      <h1>Inspecting React events</h1>

      <section aria-labelledby="click-heading">
        <h2 id="click-heading">Click event</h2>
        <p>Click the star or the button text and compare the event target.</p>
        <button type="button" onClick={handleButtonClick}>
          <span aria-hidden="true">?</span> Save
        </button>
        <p role="status">{clickDetails}</p>
      </section>

      <section aria-labelledby="keyboard-heading">
        <h2 id="keyboard-heading">Keyboard event</h2>
        <label htmlFor="keyboard-input">Try a few keys</label>
        <input id="keyboard-input" onKeyDown={handleKeyDown} />
        <p role="status">{keyDetails}</p>
      </section>

      <section aria-labelledby="form-heading">
        <h2 id="form-heading">Form event</h2>
        <form onSubmit={handleSubmit}>
          <label htmlFor="display-name">Display name</label>
          <input id="display-name" name="displayName" required />
          <button type="submit">Submit name</button>
        </form>
        <p role="status">
          Submitted name: {submittedName || 'The name cannot be empty.'}
        </p>
      </section>
    </main>
  );
}
~~~

## Observe the event values

### Click event

Click the star inside the button. The event's `target` will be the nested `<span>`, while `currentTarget` will be the `<button>` that owns `onClick`. Clicking the word Save may make the button itself the target. The status message reads the handler element through `currentTarget`.

The component stores a formatted string in state. It does not store the React event object.

### Keyboard event

Focus the text field and type different keys. `event.key` describes the produced key value; `event.code` identifies a physical keyboard key. This input is observing key presses for demonstration. For normal text input behavior, use its change/input event to read the updated text.

### Form event

Enter a display name and submit the form. The handler calls `preventDefault()` so the browser does not navigate or reload the page. It then creates `FormData` from `event.currentTarget`, which is the form with the submit handler, and saves the resulting string in React state.

If the field is empty, the browser's required-field validation prevents a normal submit, so the handler does not run until the field has a value.

## Event object reminders

| Value or method | What this example uses it for |
| --- | --- |
| `event.type` | Identifies the React event handled by the button. |
| `event.target` | Shows which nested element the click originated on. |
| `event.currentTarget` | Reads the element with the handler: the button or form. |
| `event.key` and `event.code` | Show keyboard-specific event data. |
| `event.preventDefault()` | Cancels the browser's default form submission behavior. |
| `event.nativeEvent` | Not needed for this application behavior; most code should use the React event object. |

## Try these changes

1. Remove the nested `<span>` and see how `target` changes.
2. Change the button handler to read `event.currentTarget.textContent` and compare it with the target tag name.
3. Add `event.shiftKey` to the keyboard message and hold Shift while typing.
4. Change the form handler to use `event.currentTarget.elements.displayName.value` instead of `FormData`.
5. Temporarily remove `event.preventDefault()` and observe the browser's normal form behavior.
6. Add an `onMouseLeave` handler and inspect both `event.type` and `event.nativeEvent.type` for learning. Do not make application logic depend on a particular native-event mapping.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [025 - Event handlers and user interactions](../lessons/025-event-handlers-and-user-interactions.md) | [026 - Event objects and React's event system](../lessons/026-event-objects-and-react-event-system.md) | [Open Q&A](../qnas/026-event-objects-and-react-event-system.md) | [027 - Event propagation, default actions, and cancellation](../lessons/027-event-propagation-and-default-actions.md) |