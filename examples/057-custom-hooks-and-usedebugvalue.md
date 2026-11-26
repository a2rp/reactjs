# Example: Custom Hooks and useDebugValue

This runnable example uses one **useStopwatch** custom Hook in two ways:

- Two independent timer panels each call the Hook and keep separate state.
- A shared timer calls the Hook once in a parent and passes the same state and actions to two display components.

The Hook adds a formatted **useDebugValue** label for React DevTools. The example has no external assets and makes no network requests. The timer displays elapsed interval time for learning purposes; it is not a precision clock.

## Create the project

Create a Vite React project:

~~~sh
npm create vite@latest custom-hooks-demo -- --template react
cd custom-hooks-demo
npm install
~~~

Replace the generated files with the following and run the development server:

~~~sh
npm run dev
~~~

## Project files

~~~text
custom-hooks-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
    useStopwatch.js
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f7fb" />
    <title>Custom Hooks Demo</title>
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

Strict Mode is enabled to help reveal timer Effects that do not clean up correctly.

## src/useStopwatch.js

~~~jsx
import { useDebugValue, useEffect, useState } from "react";

function formatStopwatch({ elapsedMs, running }) {
  const status = running ? "running" : "paused";
  return status + " · " + (elapsedMs / 1000).toFixed(1) + " s";
}

export function useStopwatch(intervalMs = 100) {
  const [elapsedMs, setElapsedMs] = useState(0);
  const [running, setRunning] = useState(false);

  useEffect(() => {
    if (!running) {
      return;
    }

    const intervalId = window.setInterval(() => {
      setElapsedMs((currentElapsedMs) => currentElapsedMs + intervalMs);
    }, intervalMs);

    return () => {
      window.clearInterval(intervalId);
    };
  }, [intervalMs, running]);

  useDebugValue({ elapsedMs, running }, formatStopwatch);

  function start() {
    setRunning(true);
  }

  function pause() {
    setRunning(false);
  }

  function reset() {
    setElapsedMs(0);
  }

  return {
    elapsedMs,
    elapsedSeconds: elapsedMs / 1000,
    running,
    start,
    pause,
    reset,
  };
}
~~~

The Hook owns the elapsed-time and running state, starts an interval while running, and clears that interval when it pauses or unmounts. Its formatter is passed to **useDebugValue** so React DevTools can format the value when the Hook is inspected.

## src/App.jsx

~~~jsx
import { useState } from "react";
import { useStopwatch } from "./useStopwatch.js";

function StopwatchDisplay({ title, stopwatch }) {
  return (
    <article className="timer-card">
      <div className="timer-card-heading">
        <div>
          <p className="eyebrow">{stopwatch.running ? "Running" : "Paused"}</p>
          <h3>{title}</h3>
        </div>
        <span
          className={
            stopwatch.running ? "timer-indicator is-running" : "timer-indicator"
          }
          aria-hidden="true"
        />
      </div>

      <p className="timer-value" aria-live="off">
        {stopwatch.elapsedSeconds.toFixed(1)}
        <span>s</span>
      </p>

      <div className="timer-actions">
        <button
          type="button"
          onClick={stopwatch.start}
          disabled={stopwatch.running}
        >
          Start
        </button>
        <button
          type="button"
          className="button-secondary"
          onClick={stopwatch.pause}
          disabled={!stopwatch.running}
        >
          Pause
        </button>
        <button
          type="button"
          className="button-quiet"
          onClick={stopwatch.reset}
        >
          Reset
        </button>
      </div>
    </article>
  );
}

function IndependentStopwatch({ title, intervalMs }) {
  const stopwatch = useStopwatch(intervalMs);

  return <StopwatchDisplay title={title} stopwatch={stopwatch} />;
}

function SharedStopwatch({ intervalMs }) {
  const stopwatch = useStopwatch(intervalMs);

  return (
    <section className="panel" aria-labelledby="shared-heading">
      <div className="section-heading">
        <div>
          <p className="eyebrow">One Hook call</p>
          <h2 id="shared-heading">Shared timer state</h2>
        </div>
        <span className="state-badge">One owner</span>
      </div>

      <p className="section-description">
        The parent owns one stopwatch. Both displays receive the same current
        value and actions, so either set of buttons controls both displays.
      </p>

      <div className="timer-grid">
        <StopwatchDisplay title="Shared display A" stopwatch={stopwatch} />
        <StopwatchDisplay title="Shared display B" stopwatch={stopwatch} />
      </div>
    </section>
  );
}

export default function App() {
  const [intervalMs, setIntervalMs] = useState(100);

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React Hooks · Chapter 057</p>
        <h1>Custom Hooks reuse behavior</h1>
        <p className="intro">
          Compare separate Hook calls with one Hook call owned by a parent. The
          behavior is reusable; state is shared only when you pass the same
          state owner to multiple views.
        </p>
      </header>

      <section className="panel settings-panel" aria-labelledby="settings-title">
        <div>
          <p className="eyebrow">Reactive input</p>
          <h2 id="settings-title">Timer interval</h2>
        </div>

        <label>
          Update interval
          <select
            value={intervalMs}
            onChange={(event) => setIntervalMs(Number(event.target.value))}
          >
            <option value={100}>100 milliseconds</option>
            <option value={250}>250 milliseconds</option>
            <option value={500}>500 milliseconds</option>
          </select>
        </label>

        <p className="settings-note">
          Changing the interval reconfigures each running timer's Effect.
        </p>
      </section>

      <section className="panel" aria-labelledby="independent-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Two Hook calls</p>
            <h2 id="independent-heading">Independent timer state</h2>
          </div>
          <span className="state-badge state-badge--separate">
            Separate owners
          </span>
        </div>

        <p className="section-description">
          Each timer component calls <strong>useStopwatch</strong> for itself. Starting,
          pausing, or resetting one timer does not change the other.
        </p>

        <div className="timer-grid">
          <IndependentStopwatch title="Timer one" intervalMs={intervalMs} />
          <IndependentStopwatch title="Timer two" intervalMs={intervalMs} />
        </div>
      </section>

      <SharedStopwatch intervalMs={intervalMs} />

      <section className="panel debug-panel" aria-labelledby="debug-heading">
        <p className="eyebrow">React DevTools</p>
        <h2 id="debug-heading">Inspect the custom Hook label</h2>
        <ol>
          <li>Open React DevTools and select a timer component.</li>
          <li>Inspect its Hooks list and find the useStopwatch debug value.</li>
          <li>Start or pause a timer and inspect the formatted label again.</li>
        </ol>
        <p>
          <strong>useDebugValue</strong> is visible in developer tooling, not in the app's
          rendered interface. The Hook still returns its normal values to its
          component.
        </p>
      </section>

      <footer className="page-footer">
        <p>
          This demo keeps all timer state in React. Reloading the page resets
          the timers.
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
  background: #f4f7fb;
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
select {
  font: inherit;
}

button,
select {
  cursor: pointer;
}

button:focus-visible,
select:focus-visible {
  outline: 3px solid #83aaff;
  outline-offset: 2px;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.48;
}

.page-shell {
  display: grid;
  width: min(1000px, calc(100% - 40px));
  gap: 18px;
  margin: 0 auto;
  padding: 48px 0 36px;
}

.page-header {
  margin-bottom: 4px;
}

.eyebrow {
  color: #66758b;
  font-size: 0.73rem;
  font-weight: 760;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2,
h3 {
  color: #1a263a;
  line-height: 1.2;
}

h1 {
  margin-top: 6px;
  font-size: clamp(2rem, 5vw, 3.1rem);
  letter-spacing: -0.045em;
}

h2 {
  font-size: 1.22rem;
  letter-spacing: -0.02em;
}

h3 {
  font-size: 1rem;
}

.intro,
.section-description {
  max-width: 720px;
  color: #5e6a7c;
}

.intro {
  margin-top: 12px;
  font-size: 1.02rem;
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
  gap: 18px;
  margin-bottom: 12px;
}

.section-heading h2 {
  margin-top: 4px;
}

.state-badge {
  border-radius: 999px;
  background: #eaf2ff;
  color: #365b99;
  font-size: 0.8rem;
  font-weight: 720;
  padding: 7px 10px;
  white-space: nowrap;
}

.state-badge--separate {
  background: #f1edfb;
  color: #604b8f;
}

.settings-panel {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(220px, 0.6fr);
  align-items: center;
  gap: 12px 20px;
}

.settings-panel h2 {
  margin-top: 4px;
}

.settings-panel label {
  display: grid;
  gap: 6px;
  color: #354255;
  font-size: 0.87rem;
  font-weight: 650;
}

select {
  width: 100%;
  min-height: 43px;
  border: 1px solid #cbd4e1;
  border-radius: 9px;
  background: #fff;
  color: #1d293b;
  padding: 8px 10px;
}

.settings-note {
  grid-column: 1 / -1;
  color: #677487;
  font-size: 0.85rem;
}

.section-description {
  margin-bottom: 16px;
  font-size: 0.92rem;
}

.timer-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.timer-card {
  display: grid;
  gap: 15px;
  border: 1px solid #e4e9f1;
  border-radius: 12px;
  background: #fbfcfe;
  padding: 16px;
}

.timer-card-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.timer-card-heading h3 {
  margin-top: 3px;
}

.timer-indicator {
  width: 11px;
  height: 11px;
  border-radius: 50%;
  background: #a8b2c1;
}

.timer-indicator.is-running {
  background: #33a368;
  box-shadow: 0 0 0 4px rgb(51 163 104 / 13%);
}

.timer-value {
  color: #18253b;
  font-variant-numeric: tabular-nums;
  font-size: clamp(2.5rem, 6vw, 3.8rem);
  font-weight: 760;
  letter-spacing: -0.055em;
  line-height: 1;
}

.timer-value span {
  margin-left: 5px;
  color: #718096;
  font-size: 1rem;
  font-weight: 650;
  letter-spacing: 0;
}

.timer-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.timer-actions button {
  min-height: 39px;
  border: 1px solid #315b9f;
  border-radius: 8px;
  background: #315b9f;
  color: #fff;
  font-size: 0.86rem;
  font-weight: 700;
  padding: 7px 12px;
}

.timer-actions button:hover:not(:disabled) {
  background: #254b88;
}

.timer-actions .button-secondary {
  border-color: #d3dbe7;
  background: #fff;
  color: #33445d;
}

.timer-actions .button-secondary:hover:not(:disabled) {
  background: #f1f4f8;
}

.timer-actions .button-quiet {
  border-color: transparent;
  background: transparent;
  color: #5e6a7c;
}

.timer-actions .button-quiet:hover {
  background: #eef2f7;
}

.debug-panel h2 {
  margin-top: 4px;
}

.debug-panel ol {
  display: grid;
  gap: 6px;
  margin: 12px 0;
  padding-left: 22px;
  color: #4d5c72;
}

.debug-panel p:last-child {
  color: #5e6a7c;
  font-size: 0.9rem;
}

.page-footer {
  color: #6d798a;
  font-size: 0.84rem;
  text-align: center;
}

@media (max-width: 680px) {
  .page-shell {
    width: calc(100% - 28px);
    padding-top: 30px;
  }

  .panel {
    padding: 17px;
  }

  .settings-panel {
    grid-template-columns: 1fr;
  }

  .settings-note {
    grid-column: auto;
  }

  .timer-grid {
    grid-template-columns: 1fr;
  }

  .section-heading {
    align-items: flex-start;
  }
}
~~~

## What to explore

1. Start Timer one. Timer two remains paused because its Hook call owns separate state.
2. Start both independent timers, then change the interval. Each running timer reconfigures its interval Effect.
3. Start either shared display. Both shared displays update because they receive the same Hook result from one parent-owned call.
4. Pause or reset either shared display. The other display reflects the same state transition.
5. Select a timer component in React DevTools and inspect the formatted **useDebugValue** label.

## Design notes

- **useStopwatch** has a specific purpose and accepts one input: the update interval.
- Each call owns its own **elapsedMs** and **running** state.
- The shared timer example calls the Hook once and passes its result to both children.
- The interval Effect declares **running** and **intervalMs** because they determine whether and how the timer is synchronized.
- The Effect cleanup clears the exact interval created by its setup.
- The formatter returns a development label. It does not change rendered output and is not needed by the user interface.

## Practice changes

1. Add a **lap** action and store lap times in the custom Hook.
2. Add a maximum duration option. Decide whether changing it should alter the current timer's behavior.
3. Create a separate component that displays a formatted duration using values passed from its parent; do not call the Hook again if it should show the parent's timer.
4. Remove **useDebugValue** and compare what the timer's caller sees with what appears in React DevTools.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="056-useeffectevent.md">056 - Example: useEffectEvent and Non-Reactive Event Logic</a></td><td><a href="../lessons/057-custom-hooks-and-usedebugvalue.md">Lesson</a></td><td><a href="../qnas/057-custom-hooks-and-usedebugvalue.md">Q&amp;A</a></td><td><a href="058-useref.md">058 - Example: useRef for DOM Nodes and Mutable Values</a></td></tr>
  </tbody>
</table>
