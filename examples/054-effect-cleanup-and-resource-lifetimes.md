# Example: Effect Cleanup and Resource Lifetimes

This runnable Vite example shows four resources with distinct lifetimes:

- a stopwatch interval is cleared when paused or removed;
- a browser resize listener is removed when its component is removed;
- a native modal dialog is closed when its open state changes or its component is removed;
- a mock asynchronous lookup is cancelled when its selected record changes or its component is removed.

The lookup uses a local timer to simulate a request. The app makes no network requests and stores no data.

## Project files

~~~text
effect-cleanup-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

Use these files in a client-rendered React project created with Vite. From the project root, install dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f7fb" />
    <title>Effect Cleanup Demo</title>
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
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

Strict Mode is enabled so development checks that each Effect can clean up and set up again without leaving duplicate resources active.

## src/App.jsx

~~~jsx
import { useEffect, useRef, useState } from "react";

function Stopwatch() {
  const [running, setRunning] = useState(false);
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    if (!running) return;

    const intervalId = window.setInterval(() => {
      setSeconds((currentSeconds) => currentSeconds + 1);
    }, 1000);

    return () => window.clearInterval(intervalId);
  }, [running]);

  return (
    <section className="card">
      <p className="eyebrow">Interval · clearInterval</p>
      <h2>Stopwatch</h2>
      <p className="timer">
        {seconds} <span>seconds</span>
      </p>
      <div className="button-row">
        <button type="button" onClick={() => setRunning((value) => !value)}>
          {running ? "Pause" : "Start"}
        </button>
        <button
          className="button-secondary"
          type="button"
          onClick={() => setSeconds(0)}
        >
          Reset
        </button>
      </div>
      <p className="explanation">
        Pausing runs cleanup and clears this interval. Starting creates a new
        interval for the new running state.
      </p>
    </section>
  );
}

function WindowWidth() {
  const [width, setWidth] = useState(() => window.innerWidth);

  useEffect(() => {
    function updateWidth() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", updateWidth);

    return () => {
      window.removeEventListener("resize", updateWidth);
    };
  }, []);

  return (
    <section className="card">
      <p className="eyebrow">Event listener · removeEventListener</p>
      <h2>Browser width</h2>
      <p className="width-value">{width}px</p>
      <p className="explanation">
        Resize the browser to update the value. The same handler registered by
        setup is removed when this component unmounts.
      </p>
    </section>
  );
}

function DialogExample() {
  const dialogRef = useRef(null);
  const [open, setOpen] = useState(false);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (!dialog || !open) return;

    dialog.showModal();

    return () => {
      if (dialog.open) {
        dialog.close();
      }
    };
  }, [open]);

  return (
    <section className="card">
      <p className="eyebrow">Native dialog · showModal and close</p>
      <h2>Modal lifetime</h2>
      <p className="explanation">
        The Effect opens the dialog while open is true. Cleanup closes that
        same dialog before the next setup or when this component is removed.
      </p>
      <button type="button" onClick={() => setOpen(true)}>
        Open dialog
      </button>

      <dialog
        className="modal"
        ref={dialogRef}
        aria-labelledby="dialog-title"
        onCancel={(event) => {
          event.preventDefault();
          setOpen(false);
        }}
      >
        <h3 id="dialog-title">A modal dialog</h3>
        <p>
          Press Escape or use the button to close it. Closing updates React
          state, and Effect cleanup closes the native dialog.
        </p>
        <button type="button" onClick={() => setOpen(false)}>
          Close dialog
        </button>
      </dialog>
    </section>
  );
}

function loadMockRecord(recordId, signal) {
  return new Promise((resolve, reject) => {
    if (signal.aborted) {
      reject(new DOMException("The request was cancelled.", "AbortError"));
      return;
    }

    const delay = recordId === "slow" ? 2800 : 850;

    const timeoutId = window.setTimeout(() => {
      signal.removeEventListener("abort", handleAbort);
      if (recordId === "error") {
        reject(new Error("The simulated request failed."));
        return;
      }
      resolve({
        id: recordId,
        title: "Details for record " + recordId,
        description:
          "This result came from a local timer that simulates asynchronous work.",
      });
    }, delay);

    function handleAbort() {
      window.clearTimeout(timeoutId);
      reject(new DOMException("The request was cancelled.", "AbortError"));
    }

    signal.addEventListener("abort", handleAbort, { once: true });
  });
}

function RecordLookup() {
  const [recordId, setRecordId] = useState("alpha");
  const [record, setRecord] = useState(null);
  const [failure, setFailure] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    let active = true;

    loadMockRecord(recordId, controller.signal)
      .then((nextRecord) => {
        if (active) {
          setRecord(nextRecord);
        }
      })
      .catch((error) => {
        if (active && !controller.signal.aborted) {
          setFailure({ recordId, message: error.message });
        }
      });

    return () => {
      active = false;
      controller.abort();
    };
  }, [recordId]);

  const currentRecord = record && record.id === recordId ? record : null;
  const currentFailure =
    failure && failure.recordId === recordId ? failure : null;

  return (
    <section className="card">
      <p className="eyebrow">Async work · abort and ignore</p>
      <h2>Record lookup</h2>
      <label className="field-label" htmlFor="record-id">
        Choose a record
      </label>
      <select
        id="record-id"
        value={recordId}
        onChange={(event) => setRecordId(event.target.value)}
      >
        <option value="alpha">Alpha · quick</option>
        <option value="beta">Beta · quick</option>
        <option value="slow">Slow · 2.8 seconds</option>
        <option value="error">Error · simulated failure</option>
      </select>

      {currentRecord ? (
        <div className="record-result">
          <h3>{currentRecord.title}</h3>
          <p>{currentRecord.description}</p>
        </div>
      ) : currentFailure ? (
        <p className="error" role="alert">
          {currentFailure.message}
        </p>
      ) : (
        <p className="empty-state" aria-live="polite">
          Loading record {recordId}…
        </p>
      )}

      <p className="explanation">
        Select Slow and quickly choose another record. Cleanup aborts the old
        timer and prevents its result from replacing the current record.
      </p>
    </section>
  );
}

export default function App() {
  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React Hooks · chapter 054</p>
        <h1>Every Effect resource needs a clear lifetime</h1>
        <p className="intro">
          Start, stop, switch, and remove the examples. Each Effect releases
          the resource created by its own setup.
        </p>
      </header>

      <div className="demo-grid">
        <Stopwatch />
        <WindowWidth />
        <DialogExample />
        <RecordLookup />
      </div>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #182236;
  background: #f4f7fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-weight: 400;
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
select {
  font: inherit;
}

button {
  min-height: 44px;
  border: 0;
  border-radius: 0.65rem;
  padding: 0.65rem 1rem;
  color: #ffffff;
  background: #285bd2;
  cursor: pointer;
}

button:hover {
  background: #1f49ad;
}

button:focus-visible,
select:focus-visible {
  outline: 3px solid #e59c00;
  outline-offset: 3px;
}

.button-secondary {
  color: #25344e;
  background: #e8edf6;
}

.button-secondary:hover {
  background: #d9e2f0;
}

.page {
  width: min(1050px, calc(100% - 2rem));
  margin: 0 auto;
  padding: 3rem 0;
}

.page-header {
  max-width: 760px;
  margin-bottom: 2rem;
}

.eyebrow {
  margin-bottom: 0.6rem;
  color: #285bd2;
  font-size: 0.78rem;
  font-weight: 750;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2,
h3 {
  line-height: 1.25;
}

h1 {
  max-width: 700px;
  font-size: clamp(2rem, 5vw, 3.35rem);
  letter-spacing: -0.04em;
}

h2 {
  margin-bottom: 1rem;
  font-size: 1.3rem;
}

h3 {
  font-size: 1.05rem;
}

.intro {
  max-width: 680px;
  margin-top: 1rem;
  color: #526078;
  font-size: 1.05rem;
  line-height: 1.65;
}

.demo-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
  align-items: start;
}

.card {
  min-width: 0;
  border: 1px solid #dce3ee;
  border-radius: 1rem;
  padding: 1.35rem;
  background: #ffffff;
  box-shadow: 0 10px 30px rgb(30 48 80 / 6%);
}

.timer,
.width-value {
  margin: 1.25rem 0;
  font-size: 2.8rem;
  font-variant-numeric: tabular-nums;
  font-weight: 750;
  letter-spacing: -0.04em;
}

.timer span {
  color: #68758b;
  font-size: 1rem;
  font-weight: 500;
  letter-spacing: 0;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.65rem;
}

.explanation {
  margin-top: 1rem;
  color: #58657a;
  line-height: 1.6;
}

.field-label {
  display: block;
  margin-bottom: 0.45rem;
  color: #35425a;
  font-weight: 650;
}

select {
  width: 100%;
  min-height: 44px;
  border: 1px solid #aebbd0;
  border-radius: 0.65rem;
  padding: 0.65rem 0.8rem;
  color: #182236;
  background: #ffffff;
}

.record-result,
.empty-state,
.error {
  margin-top: 1rem;
  border-radius: 0.65rem;
  padding: 1rem;
  background: #f3f6fb;
}

.record-result {
  display: grid;
  gap: 0.5rem;
  color: #35425a;
  line-height: 1.55;
}

.empty-state {
  color: #58657a;
}

.error {
  color: #8e1e1e;
  background: #fff0f0;
}

.modal {
  width: min(440px, calc(100% - 2rem));
  margin: auto;
  border: 1px solid #dce3ee;
  border-radius: 1rem;
  padding: 1.5rem;
  color: #182236;
  background: #ffffff;
  box-shadow: 0 20px 70px rgb(15 25 45 / 28%);
}

.modal::backdrop {
  background: rgb(15 25 45 / 55%);
}

.modal p {
  margin: 0.8rem 0 1rem;
  color: #526078;
  line-height: 1.6;
}

@media (max-width: 680px) {
  .page {
    padding: 2rem 0;
  }

  .demo-grid {
    grid-template-columns: 1fr;
  }
}
~~~

## How to explore the example

1. Start the stopwatch, then pause it. The interval is cleared when the Effect stops.
2. Resize the browser. The displayed width changes while the listener is active.
3. Open the modal dialog and close it with Escape or its button. Cleanup closes the native dialog.
4. Select Slow, then quickly select Alpha or Beta. Cleanup aborts the old simulated lookup, and only the current selection can display a result.
5. Select Error to see the asynchronous failure message.
6. In development Strict Mode, setup and cleanup may run an extra time. The examples should still have one active timer or listener, one dialog state, and one current lookup.

## Why both abort and an active flag are used

AbortController can cancel this simulated timer and many browser requests. The active flag is an additional guard against stale results from asynchronous work that cannot be cancelled or that completes while cancellation is in progress. Neither technique rolls back an operation already processed by a remote server.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="053-effect-dependencies-and-closures.md">053 - Example: Effect Dependencies, Closures, and Synchronization</a></td><td><a href="../lessons/054-effect-cleanup-and-resource-lifetimes.md">Lesson</a></td><td><a href="../qnas/054-effect-cleanup-and-resource-lifetimes.md">Q&amp;A</a></td><td><a href="055-when-an-effect-is-unnecessary.md">055 - Example: When an Effect Is Unnecessary</a></td></tr>
  </tbody>
</table>
