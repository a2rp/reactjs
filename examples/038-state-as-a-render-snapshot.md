# 038. State as a render snapshot - Example

## What this example demonstrates

This small React app has two demonstrations:

1. A counter shows that a click handler still reads the count from its own render after it calls a setter. Its â€œAdd three (direct updates)â€ button makes three calculations from the same snapshot.
2. A delayed message uses the recipient and message captured when the scheduling handler ran. You can edit the form while the timer is running and compare the current form values with the values used by the callback.

The timer cleanup is included so an unfinished timer is cleared if the component unmounts. It is not part of the snapshot rule itself.

## Project files

Add these files to an existing React app created with Vite:

- src/App.jsx
- src/App.css
- src/main.jsx
- src/index.css

Run the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useEffect, useRef, useState } from 'react';
import './App.css';

export default function App() {
  const [count, setCount] = useState(0);
  const [handlerNote, setHandlerNote] = useState(
    'Choose a counter action to inspect its render snapshot.'
  );

  const [recipient, setRecipient] = useState('Mira');
  const [message, setMessage] = useState('Hello from this render.');
  const [preview, setPreview] = useState(
    'No delayed preview has been scheduled.'
  );
  const [isWaiting, setIsWaiting] = useState(false);
  const timeoutRef = useRef(null);

  useEffect(() => {
    return () => {
      if (timeoutRef.current !== null) {
        window.clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  function handleIncrement() {
    const nextCount = count + 1;

    setCount(nextCount);
    setHandlerNote(
      'This handler started with count ' +
        count +
        ' and requested ' +
        nextCount +
        '. It still reads ' +
        count +
        '.'
    );
  }

  function handleAddThreeDirectly() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);

    setHandlerNote(
      'All three direct calls started from count ' +
        count +
        ' and requested ' +
        (count + 1) +
        '. The next count is ' +
        (count + 1) +
        ', not ' +
        (count + 3) +
        '.'
    );
  }

  function handleReset() {
    setCount(0);
    setHandlerNote('This handler requested a count of 0.');
  }

  function handleSchedulePreview() {
    if (timeoutRef.current !== null) {
      window.clearTimeout(timeoutRef.current);
    }

    setPreview(
      'Waiting 3 seconds. The callback will use the values from this click.'
    );
    setIsWaiting(true);

    timeoutRef.current = window.setTimeout(() => {
      setPreview(
        'Callback captured recipient "' +
          recipient +
          '" and message "' +
          message +
          '".'
      );
      setIsWaiting(false);
      timeoutRef.current = null;
    }, 3000);
  }

  function handleCancelPreview() {
    if (timeoutRef.current === null) {
      return;
    }

    window.clearTimeout(timeoutRef.current);
    timeoutRef.current = null;
    setIsWaiting(false);
    setPreview('The scheduled preview was canceled.');
  }

  return (
    <main className="app-shell">
      <header className="page-header">
        <p className="eyebrow">React state</p>
        <h1>State as a render snapshot</h1>
        <p>
          Every render supplies values to the handlers created by that render.
          A setter requests a future render; it does not rewrite a handler
          already running.
        </p>
      </header>

      <section className="demo-card" aria-labelledby="counter-heading">
        <p className="eyebrow">Demo 1</p>
        <h2 id="counter-heading">Counter snapshot</h2>

        <p className="count-value" aria-live="polite">
          Count: {count}
        </p>

        <div className="button-row">
          <button type="button" onClick={handleIncrement}>
            Add one
          </button>
          <button type="button" onClick={handleAddThreeDirectly}>
            Add three (direct updates)
          </button>
          <button className="secondary-button" type="button" onClick={handleReset}>
            Reset
          </button>
        </div>

        <p className="status-box" role="status">
          {handlerNote}
        </p>

        <p className="explanation">
          The direct-update button intentionally calls setCount(count + 1)
          three times. Each call reads count from the same handler snapshot.
          The next chapter shows how to express three queued increments.
        </p>
      </section>

      <section className="demo-card" aria-labelledby="message-heading">
        <p className="eyebrow">Demo 2</p>
        <h2 id="message-heading">Delayed message snapshot</h2>
        <p className="section-intro">
          Schedule a preview, then change the form before the three-second
          timer ends. The callback will show the values captured when you
          scheduled it.
        </p>

        <div className="form-grid">
          <label htmlFor="recipient">Recipient</label>
          <select
            id="recipient"
            value={recipient}
            onChange={(event) => setRecipient(event.target.value)}
          >
            <option value="Mira">Mira</option>
            <option value="Sam">Sam</option>
            <option value="Alex">Alex</option>
          </select>

          <label htmlFor="message">Message</label>
          <textarea
            id="message"
            rows="3"
            value={message}
            onChange={(event) => setMessage(event.target.value)}
          />
        </div>

        <div className="button-row">
          <button type="button" onClick={handleSchedulePreview}>
            {isWaiting
              ? 'Replace with current values'
              : 'Schedule preview'}
          </button>
          {isWaiting && (
            <button
              className="secondary-button"
              type="button"
              onClick={handleCancelPreview}
            >
              Cancel preview
            </button>
          )}
        </div>

        <p className="status-box preview-box" role="status">
          {preview}
        </p>

        <p className="explanation">
          Editing a controlled input requests a new render. It does not change
          the recipient or message captured by an already scheduled callback.
          Choosing â€œReplace with current valuesâ€ creates a new callback from
          the latest render.
        </p>
      </section>

      <footer className="page-footer">
        <p>
          Remember: ask which render created the handler when a value seems
          stale.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/App.css

~~~css
.app-shell {
  width: min(100% - 2rem, 900px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.page-header h1 {
  margin-block: 0.35rem 0.75rem;
  font-size: clamp(2rem, 5vw, 3.25rem);
  line-height: 1.1;
}

.page-header > p:last-child,
.section-intro,
.explanation,
.page-footer {
  max-width: 68ch;
  line-height: 1.65;
  color: #4b5563;
}

.eyebrow {
  color: #5b46d6;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.demo-card {
  margin-block: 1.25rem;
  padding: clamp(1.1rem, 3vw, 1.75rem);
  border: 1px solid #d7dbe3;
  border-radius: 1rem;
  background: #ffffff;
  box-shadow: 0 0.5rem 1.5rem rgb(20 28 50 / 7%);
}

.demo-card h2 {
  margin-block: 0.35rem 1rem;
}

.count-value {
  margin-block: 1rem;
  font-size: 2rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.65rem;
  margin-block: 1rem;
}

button,
select,
textarea {
  font: inherit;
}

button {
  min-height: 2.75rem;
  padding: 0.6rem 0.9rem;
  border: 1px solid #4d3bbd;
  border-radius: 0.55rem;
  background: #5b46d6;
  color: #ffffff;
  cursor: pointer;
}

button:hover {
  background: #4935bd;
}

button:focus-visible,
select:focus-visible,
textarea:focus-visible {
  outline: 3px solid #a99cff;
  outline-offset: 2px;
}

.secondary-button {
  border-color: #aeb5c2;
  background: #ffffff;
  color: #222936;
}

.secondary-button:hover {
  background: #f1f3f7;
}

.status-box {
  min-height: 3.5rem;
  margin-block: 1rem;
  padding: 0.9rem 1rem;
  border-left: 4px solid #5b46d6;
  border-radius: 0.35rem;
  background: #f3f1ff;
  line-height: 1.55;
}

.explanation {
  margin-top: 1rem;
  font-size: 0.95rem;
}

.form-grid {
  display: grid;
  grid-template-columns: 8rem minmax(0, 1fr);
  gap: 0.75rem 1rem;
  align-items: start;
  margin-block: 1.25rem;
}

.form-grid label {
  padding-top: 0.55rem;
  font-weight: 650;
}

.form-grid select,
.form-grid textarea {
  width: 100%;
  padding: 0.65rem 0.75rem;
  border: 1px solid #aeb5c2;
  border-radius: 0.5rem;
  background: #ffffff;
  color: #222936;
}

.form-grid textarea {
  resize: vertical;
}

.preview-box {
  overflow-wrap: anywhere;
}

.page-footer {
  margin-top: 2rem;
  padding-inline: 0.25rem;
}

@media (max-width: 560px) {
  .app-shell {
    width: min(100% - 1.25rem, 900px);
    padding-block: 1.5rem;
  }

  .form-grid {
    grid-template-columns: 1fr;
    gap: 0.4rem;
  }

  .form-grid label:not(:first-child) {
    margin-top: 0.4rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.jsx';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #222936;
  background: #f6f7fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
select,
textarea {
  font: inherit;
}
~~~

## Try it

1. Press Add one. Read the handler status: it identifies the count this click started with and the next count it requested.
2. Press Add three (direct updates). Notice that the three direct expressions all started from the same count snapshot.
3. Reset the counter and try again. A later click after a render starts from the newly displayed count.
4. Leave the recipient as Mira and keep the default message. Select Schedule preview.
5. Before the timer finishes, change the recipient to Sam and edit the message.
6. Wait. The preview shows Mira and the message from the render that scheduled the callback, while the form shows the newer values.
7. Select Replace with current values to schedule a new callback from the current render. The next preview uses the updated recipient and message.
8. Select Cancel preview to clear a pending timer.

## What to notice

- The counter status describes the running handler's snapshot, not a value React has already inserted into that handler.
- A new render gives the counter button a new handler with the new count.
- The delayed callback continues to use the recipient and message captured when it was created.
- Editing the form creates new state and handlers for a later render. It does not rewrite an existing callback.
- In the following chapter, the direct updates are replaced with updater functions so each queued increment can build on the pending value.
