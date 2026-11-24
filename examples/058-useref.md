# Example: useRef for DOM Nodes and Mutable Values

This runnable React example demonstrates three ref uses:

- a DOM ref focuses the search field when the user asks;
- a DOM ref scrolls a saved-entry list after React renders a new entry;
- refs retain a timer ID and the next entry ID between event handlers without rendering them.

The displayed elapsed time and timer status use state because they are visible. The timer ID does not affect the UI, so it stays in a ref. The example makes no network requests.

## Create the project

~~~sh
npm create vite@latest use-ref-demo -- --template react
cd use-ref-demo
npm install
~~~

Replace the generated files with the following and start the development server:

~~~sh
npm run dev
~~~

## Project files

~~~text
use-ref-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f5f7fb" />
    <title>useRef Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

const rootElement = document.getElementById("root");

if (!rootElement) {
  throw new Error("The React root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/App.jsx

~~~jsx
import { useEffect, useRef, useState } from "react";

const INITIAL_ENTRIES = [
  { id: 1, text: "A ref keeps a value between renders." },
  { id: 2, text: "Changing ref.current does not render the screen." },
  { id: 3, text: "Use state for values that appear in the UI." },
];

function formatElapsed(ticks) {
  return (ticks / 10).toFixed(1);
}

export default function App() {
  const [entries, setEntries] = useState(INITIAL_ENTRIES);
  const [search, setSearch] = useState("");
  const [draft, setDraft] = useState("");
  const [message, setMessage] = useState("");
  const [elapsedTicks, setElapsedTicks] = useState(0);
  const [timerRunning, setTimerRunning] = useState(false);

  const searchInputRef = useRef(null);
  const listEndRef = useRef(null);
  const intervalIdRef = useRef(null);
  const nextEntryIdRef = useRef(4);

  const normalizedSearch = search.trim().toLowerCase();
  const visibleEntries = entries.filter((entry) =>
    entry.text.toLowerCase().includes(normalizedSearch)
  );

  useEffect(() => {
    listEndRef.current?.scrollIntoView({
      block: "nearest",
      behavior: "smooth",
    });
  }, [entries]);

  useEffect(() => {
    return () => {
      if (intervalIdRef.current !== null) {
        window.clearInterval(intervalIdRef.current);
      }
    };
  }, []);

  function handleFocusSearch() {
    searchInputRef.current?.focus();
  }

  function handleAddEntry(event) {
    event.preventDefault();

    const text = draft.trim();

    if (!text) {
      setMessage("Write an entry before saving.");
      return;
    }

    const entry = {
      id: nextEntryIdRef.current,
      text,
    };

    nextEntryIdRef.current += 1;
    setEntries((currentEntries) => [...currentEntries, entry]);
    setDraft("");
    setMessage("Entry saved. The list moved to its end.");
  }

  function handleToggleTimer() {
    if (intervalIdRef.current === null) {
      intervalIdRef.current = window.setInterval(() => {
        setElapsedTicks((currentTicks) => currentTicks + 1);
      }, 100);
      setTimerRunning(true);
      return;
    }

    window.clearInterval(intervalIdRef.current);
    intervalIdRef.current = null;
    setTimerRunning(false);
  }

  function handleResetTimer() {
    if (intervalIdRef.current !== null) {
      window.clearInterval(intervalIdRef.current);
      intervalIdRef.current = null;
    }

    setTimerRunning(false);
    setElapsedTicks(0);
  }

  function handleClearEntries() {
    setEntries([]);
    setMessage("All entries were cleared.");
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React Hooks · Chapter 058</p>
        <h1>Refs hold values outside rendering</h1>
        <p className="intro">
          Focus a DOM node, scroll to updated content, and keep a timer ID
          between events. Visible values still come from React state.
        </p>
      </header>

      <section className="panel" aria-labelledby="search-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">DOM ref</p>
            <h2 id="search-heading">Find a saved entry</h2>
          </div>
          <button
            className="button-secondary"
            type="button"
            onClick={handleFocusSearch}
          >
            Focus search
          </button>
        </div>

        <label htmlFor="entry-search">Search entries</label>
        <input
          ref={searchInputRef}
          id="entry-search"
          type="search"
          value={search}
          onChange={(event) => setSearch(event.target.value)}
          placeholder="Try searching for state or render"
        />

        <p className="result-count">
          {visibleEntries.length}{" "}
          {visibleEntries.length === 1 ? "entry" : "entries"} shown
        </p>

        <div className="entry-list" aria-label="Saved entries">
          {visibleEntries.length === 0 ? (
            <p className="empty-state">No entries match that search.</p>
          ) : (
            <ul>
              {visibleEntries.map((entry) => (
                <li key={entry.id}>{entry.text}</li>
              ))}
            </ul>
          )}
          <div ref={listEndRef} aria-hidden="true" />
        </div>
      </section>

      <div className="two-column">
        <section className="panel" aria-labelledby="add-heading">
          <div className="section-heading">
            <div>
              <p className="eyebrow">Commit then scroll</p>
              <h2 id="add-heading">Add an entry</h2>
            </div>
          </div>

          <form className="entry-form" onSubmit={handleAddEntry}>
            <label htmlFor="entry-draft">Entry text</label>
            <textarea
              id="entry-draft"
              rows="3"
              value={draft}
              onChange={(event) => setDraft(event.target.value)}
              placeholder="Write one short note"
            />
            <div className="button-row">
              <button type="submit">Save entry</button>
              <button
                className="button-secondary"
                type="button"
                onClick={handleClearEntries}
              >
                Clear list
              </button>
            </div>
            <p className="message" aria-live="polite">
              {message}
            </p>
          </form>

          <p className="explanation">
            The list-end ref points to a DOM node. An Effect scrolls it after
            the entries have rendered, so the new content exists before the
            browser scrolls to it.
          </p>
        </section>

        <section className="panel timer-panel" aria-labelledby="timer-heading">
          <div className="section-heading">
            <div>
              <p className="eyebrow">Mutable value ref</p>
              <h2 id="timer-heading">Simple timer</h2>
            </div>
            <span
              className={
                timerRunning ? "timer-badge is-running" : "timer-badge"
              }
            >
              {timerRunning ? "Running" : "Paused"}
            </span>
          </div>

          <output className="timer-value" aria-live="off">
            {formatElapsed(elapsedTicks)}
            <span>s</span>
          </output>

          <div className="button-row">
            <button type="button" onClick={handleToggleTimer}>
              {timerRunning ? "Pause" : "Start"}
            </button>
            <button
              className="button-secondary"
              type="button"
              onClick={handleResetTimer}
            >
              Reset
            </button>
          </div>

          <p className="explanation">
            The interval ID lives in a ref so Start, Pause, Reset, and unmount
            cleanup can access the same timer. The elapsed ticks and running
            label use state because they are visible.
          </p>
        </section>
      </div>

      <section className="panel ref-map" aria-labelledby="ref-map-heading">
        <p className="eyebrow">What each value does</p>
        <h2 id="ref-map-heading">Choose state or ref by render needs</h2>
        <dl>
          <div>
            <dt>Search text</dt>
            <dd>State: it controls which entries are rendered.</dd>
          </div>
          <div>
            <dt>Timer status and elapsed ticks</dt>
            <dd>State: the interface displays both values.</dd>
          </div>
          <div>
            <dt>Search input DOM node</dt>
            <dd>Ref: a button uses its focus method.</dd>
          </div>
          <div>
            <dt>Timer ID and next entry ID</dt>
            <dd>Refs: event handlers use them, but the UI does not render them.</dd>
          </div>
        </dl>
      </section>

      <footer className="page-footer">
        <p>
          The timer counts 100 ms intervals for demonstration; browser scheduling
          means it is not a precision clock. Everything stays in this page.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  color: #1d293b;
  background: #f5f7fb;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  line-height: 1.5;
}

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input,
textarea {
  font: inherit;
}

button {
  cursor: pointer;
}

button:focus-visible,
input:focus-visible,
textarea:focus-visible {
  outline: 3px solid #80a8f5;
  outline-offset: 2px;
}

button {
  min-height: 42px;
  border: 1px solid #315b9f;
  border-radius: 9px;
  background: #315b9f;
  color: #fff;
  font-weight: 700;
  padding: 8px 13px;
}

button:hover {
  background: #254b88;
}

.button-secondary {
  border-color: #d1dae7;
  background: #fff;
  color: #34445b;
}

.button-secondary:hover {
  background: #f0f4f9;
}

.page-shell {
  display: grid;
  width: min(1040px, calc(100% - 40px));
  gap: 18px;
  margin: 0 auto;
  padding: 46px 0 34px;
}

.page-header {
  margin-bottom: 2px;
}

.eyebrow {
  color: #68768b;
  font-size: 0.74rem;
  font-weight: 760;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2 {
  color: #1a263a;
  line-height: 1.2;
}

h1 {
  margin-top: 6px;
  font-size: clamp(2rem, 5vw, 3rem);
  letter-spacing: -0.045em;
}

h2 {
  font-size: 1.2rem;
  letter-spacing: -0.02em;
}

.intro {
  max-width: 700px;
  margin-top: 12px;
  color: #5d697c;
}

.panel {
  border: 1px solid #dfe5ef;
  border-radius: 16px;
  background: #fff;
  box-shadow: 0 12px 34px rgb(33 50 82 / 6%);
  padding: 22px;
}

.section-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 15px;
}

.section-heading h2,
.ref-map h2 {
  margin-top: 4px;
}

label {
  display: grid;
  gap: 7px;
  color: #354255;
  font-size: 0.88rem;
  font-weight: 650;
}

input,
textarea {
  width: 100%;
  border: 1px solid #cbd4e1;
  border-radius: 9px;
  background: #fff;
  color: #1d293b;
  padding: 10px 12px;
}

input {
  min-height: 44px;
}

textarea {
  min-height: 92px;
  resize: vertical;
}

input::placeholder,
textarea::placeholder {
  color: #8994a3;
}

.result-count {
  margin-top: 9px;
  color: #69768a;
  font-size: 0.85rem;
}

.entry-list {
  max-height: 240px;
  overflow: auto;
  margin-top: 12px;
  border: 1px solid #e4e9f1;
  border-radius: 10px;
  background: #fbfcfe;
  padding: 5px 14px;
}

.entry-list ul {
  list-style: none;
}

.entry-list li {
  border-bottom: 1px solid #e8ecf2;
  color: #465469;
  padding: 12px 0;
}

.entry-list li:last-child {
  border-bottom: 0;
}

.empty-state {
  color: #69768a;
  padding: 12px 0;
}

.two-column {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  align-items: start;
  gap: 18px;
}

.entry-form {
  display: grid;
  gap: 10px;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.message {
  min-height: 1.5em;
  color: #53647d;
  font-size: 0.86rem;
}

.explanation {
  margin-top: 14px;
  color: #5d697c;
  font-size: 0.88rem;
}

.timer-badge {
  border-radius: 999px;
  background: #eef1f5;
  color: #5e6a7b;
  font-size: 0.8rem;
  font-weight: 700;
  padding: 6px 10px;
}

.timer-badge.is-running {
  background: #e8f5ed;
  color: #34714d;
}

.timer-value {
  display: block;
  margin: 5px 0 16px;
  color: #1a263a;
  font-variant-numeric: tabular-nums;
  font-size: clamp(2.7rem, 7vw, 4rem);
  font-weight: 760;
  letter-spacing: -0.055em;
  line-height: 1;
}

.timer-value span {
  margin-left: 5px;
  color: #748094;
  font-size: 1rem;
  font-weight: 650;
  letter-spacing: 0;
}

.ref-map h2 {
  margin-bottom: 14px;
}

.ref-map dl {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.ref-map dl > div {
  border: 1px solid #e7ebf1;
  border-radius: 10px;
  background: #fbfcfe;
  padding: 12px;
}

.ref-map dt {
  color: #2c3c55;
  font-weight: 720;
}

.ref-map dd {
  margin-top: 4px;
  color: #637085;
  font-size: 0.88rem;
}

.page-footer {
  color: #6c788b;
  font-size: 0.84rem;
  text-align: center;
}

@media (max-width: 720px) {
  .page-shell {
    width: calc(100% - 28px);
    padding-top: 30px;
  }

  .panel {
    padding: 17px;
  }

  .two-column,
  .ref-map dl {
    grid-template-columns: 1fr;
  }
}
~~~

## Explore the behavior

1. Click **Focus search**. The search input receives browser focus through its DOM ref.
2. Type a filter. The visible entries update because **search** is state.
3. Add an entry. React renders the new list, then the Effect reads the list-end DOM ref and scrolls it into view.
4. Start and pause the timer. The interval ID stays in **intervalIdRef**, while elapsed ticks and the visible status use state.
5. Start the timer and refresh or navigate away. The unmount cleanup clears the active interval.
6. Clear the list or add several entries. The next-entry ID stays in a ref and does not cause an extra render by itself.

## Ref review

- **searchInputRef** points to a DOM node and is used from a click handler.
- **listEndRef** points to a DOM node and is used by an Effect after the entries render.
- **intervalIdRef** stores the browser timer ID so handlers and cleanup can access it.
- **nextEntryIdRef** stores non-visual bookkeeping and is read or updated in the submit handler.
- **search**, **entries**, **elapsedTicks**, and **timerRunning** use state because they affect rendered output.
- No ref's **current** value is read to calculate JSX.

## Practice changes

1. Add a button that scrolls to the first visible entry using an additional DOM ref.
2. Add a browser media element and use its DOM ref to call **play()** and **pause()** from button handlers.
3. Add a status value to state if the timer needs to display an error. Do not put that visible error only in a ref.
4. Replace the list-end object ref with callback refs keyed by entry ID, then remove each map entry when its node detaches.
