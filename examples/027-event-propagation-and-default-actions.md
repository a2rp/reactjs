# 027. Event propagation, default actions, and cancellation - Example

## Goal

Build an interaction lab that demonstrates:

- Capture, target, and bubble handler order.
- A child action that stops propagation to its parent.
- A link whose browser navigation is prevented while its click still bubbles.
- A form submission handled in React with the browser default prevented.
- Why propagation and default cancellation must be considered separately.

## Files

In an existing React project, create or update:

- `src/main.jsx`
- `src/App.jsx`

No extra package or stylesheet is required.

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
  const [eventLog, setEventLog] = useState([]);
  const [submittedQuery, setSubmittedQuery] = useState('Nothing submitted yet.');

  function record(message) {
    setEventLog((currentLog) => [...currentLog, message].slice(-12));
  }

  function handleCardCapture(event) {
    const targetName = event.target.tagName.toLowerCase();
    record(`Capture on card; event target is <${targetName}>.`);
  }

  function handleCardBubble() {
    record('Bubble on card; the parent click handler ran.');
  }

  function handleOpenDetails() {
    record('Open button handler ran.');
  }

  function handleArchive(event) {
    event.stopPropagation();
    record('Archive button stopped propagation.');
  }

  function handleLinkClick(event) {
    event.preventDefault();
    record('Link default navigation was prevented.');
  }

  function handleSubmit(event) {
    event.preventDefault();
    const formData = new FormData(event.currentTarget);
    const query = String(formData.get('query') ?? '').trim();
    setSubmittedQuery(query);
    record('Form submit default was prevented.');
  }

  return (
    <main>
      <h1>Propagation and default actions</h1>
      <p>Try each interaction, then read the event log in order.</p>

      <article
        aria-labelledby="record-heading"
        onClickCapture={handleCardCapture}
        onClick={handleCardBubble}
      >
        <h2 id="record-heading">Record card</h2>
        <p>The article logs clicks that reach its capture and bubble handlers.</p>
        <button type="button" onClick={handleOpenDetails}>
          Open details
        </button>
        <button type="button" onClick={handleArchive}>
          Archive without selecting the card
        </button>
        <p>
          <a href="#record-details" onClick={handleLinkClick}>
            Try this link; its default navigation is canceled.
          </a>
        </p>
      </article>

      <section id="record-details" aria-labelledby="details-heading">
        <h2 id="details-heading">Record details</h2>
        <p>This is the link's normal destination.</p>
      </section>

      <section aria-labelledby="form-heading">
        <h2 id="form-heading">Handle form submission</h2>
        <form onSubmit={handleSubmit}>
          <label htmlFor="search-query">Search query</label>
          <input id="search-query" name="query" required />
          <button type="submit">Search</button>
        </form>
        <p role="status">
          Submitted query: {submittedQuery || 'The query cannot be empty.'}
        </p>
      </section>

      <section aria-labelledby="log-heading">
        <h2 id="log-heading">Event log</h2>
        <button type="button" onClick={() => setEventLog([])}>
          Clear event log
        </button>
        <pre aria-live="polite">
          {eventLog.length === 0 ? 'No events logged.' : eventLog.join('\n')}
        </pre>
      </section>
    </main>
  );
}
~~~

## Observe propagation

Click **Open details**. The log shows the article's capture handler, the button handler, and then the article's bubble handler. The event moves through the capture phase toward the button and then bubbles back to the article.

Click **Archive without selecting the card**. The article capture handler runs first, then the button calls `stopPropagation()` and logs its own action. The article's bubble handler does not run. Stopping propagation cannot undo the capture handler that already ran.

## Observe default cancellation

Click the link inside the article. Its handler calls `preventDefault()`, so the browser does not navigate to the details section. The article's bubble handler still runs because preventing a default action does not stop propagation.

Submit a search query. The form handler prevents the browser's normal submission, reads the query from `event.currentTarget`, and updates the message without a page navigation. This form is only a demonstration; it does not fetch search results.

## Compare the methods

| Interaction | Method used | Result |
| --- | --- | --- |
| Open details button | None | The button and article handlers both run. |
| Archive button | `stopPropagation()` | The button handler runs; the article bubble handler does not. |
| Link | `preventDefault()` | The browser does not navigate; the article bubble handler still runs. |
| Search form | `preventDefault()` | The browser does not perform its normal submit navigation; React handles the value. |

## Try these changes

1. Remove `event.stopPropagation()` from `handleArchive`. The article's bubble handler should now run after the button handler.
2. Remove `event.preventDefault()` from `handleLinkClick`. The browser should follow the link to the details section.
3. Add `onClickCapture` to the button and compare its order with the article capture handler.
4. Move the form inside the article. Observe that preventing the `submit` default does not by itself cancel the separate click event from the submit button.
5. Add a nested button inside the article and give it a callback. Prefer the native button and avoid nesting it inside another interactive control.
6. If you test a portal, remember that its events propagate according to the React tree, even when its DOM nodes are elsewhere.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [026 - Event objects and React's event system](../lessons/026-event-objects-and-react-event-system.md) | [027 - Event propagation, default actions, and cancellation](../lessons/027-event-propagation-and-default-actions.md) | [Open Q&A](../qnas/027-event-propagation-and-default-actions.md) | [028 - Controlled form elements](../lessons/028-controlled-form-elements.md) |