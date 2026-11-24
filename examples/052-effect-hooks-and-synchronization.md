# Example: Effect Hooks and Synchronization

This runnable Vite example demonstrates three different decisions:

- a stopwatch and browser online/offline subscription use `useEffect` with cleanup;
- a panel uses `useLayoutEffect` to measure its expanded layout before the browser paints;
- a filtered list is derived during render and does not use an Effect.

No network requests are made and no data is persisted. The browser online/offline events only report the current connection state.

## Project files

~~~text
effect-hooks-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

Use these files in a React project created with Vite. From the project root, install dependencies if needed and start the development server:

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
    <title>Effect Hooks Demo</title>
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

Strict Mode is enabled so development performs the extra Effect setup-and-cleanup check. The examples should still leave only one active timer or one copy of each event listener.

## src/App.jsx

~~~jsx
import { useEffect, useLayoutEffect, useRef, useState } from "react";

const products = [
  { id: 1, name: "Notebook", category: "Stationery" },
  { id: 2, name: "Desk lamp", category: "Lighting" },
  { id: 3, name: "Mechanical pencil", category: "Stationery" },
  { id: 4, name: "Reading light", category: "Lighting" },
];

function NetworkStatus() {
  const [online, setOnline] = useState(null);

  useEffect(() => {
    function updateStatus() {
      setOnline(navigator.onLine);
    }

    updateStatus();
    window.addEventListener("online", updateStatus);
    window.addEventListener("offline", updateStatus);

    return () => {
      window.removeEventListener("online", updateStatus);
      window.removeEventListener("offline", updateStatus);
    };
  }, []);

  let message = "Checking browser network status…";
  if (online === true) message = "Browser reports that you are online.";
  if (online === false) message = "Browser reports that you are offline.";

  return (
    <p className="status" aria-live="polite">
      <span
        className={online ? "status-dot is-online" : "status-dot"}
        aria-hidden="true"
      />
      {message}
    </p>
  );
}

function Stopwatch() {
  const [running, setRunning] = useState(false);
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    if (!running) return;

    const timerId = window.setInterval(() => {
      setSeconds((currentSeconds) => currentSeconds + 1);
    }, 1000);

    return () => window.clearInterval(timerId);
  }, [running]);

  return (
    <section className="card">
      <p className="eyebrow">useEffect · timer</p>
      <h2>Stopwatch</h2>
      <p className="timer" aria-live="off">
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
        The Effect starts an interval only while the stopwatch is running.
        Cleanup clears it when paused or when this component is removed.
      </p>
    </section>
  );
}

function MeasuredPanel() {
  const panelRef = useRef(null);
  const [expanded, setExpanded] = useState(false);
  const [height, setHeight] = useState(null);

  useLayoutEffect(() => {
    if (!expanded) return;

    const panel = panelRef.current;
    if (!panel) return;

    setHeight(Math.round(panel.getBoundingClientRect().height));
  }, [expanded]);

  return (
    <section className="card">
      <p className="eyebrow">useLayoutEffect · layout measurement</p>
      <h2>Measure before paint</h2>
      <div className="measurement-panel" ref={panelRef}>
        <p>This panel is in the DOM before its size is measured.</p>
        {expanded && (
          <p>
            This additional content makes the panel taller. The measured value
            updates before the browser paints the expanded result.
          </p>
        )}
      </div>
      <div className="button-row">
        <button
          type="button"
          aria-expanded={expanded}
          aria-controls="measurement-details"
          onClick={() => setExpanded((value) => !value)}
        >
          {expanded ? "Collapse panel" : "Expand panel"}
        </button>
      </div>
      {expanded && (
        <p id="measurement-details" className="measurement-result">
          Measured height: {height === null ? "measuring…" : height + "px"}
        </p>
      )}
      <p className="explanation">
        useLayoutEffect reads the committed panel size and updates the result
        before repaint. It blocks painting, so use it only when that timing is
        needed.
      </p>
    </section>
  );
}

function ProductSearch() {
  const [query, setQuery] = useState("");
  const normalizedQuery = query.trim().toLowerCase();

  const visibleProducts = products.filter((product) =>
    product.name.toLowerCase().includes(normalizedQuery)
  );

  return (
    <section className="card">
      <p className="eyebrow">Render calculation · no Effect</p>
      <h2>Filter products</h2>
      <label className="field-label" htmlFor="product-search">
        Search by product name
      </label>
      <input
        id="product-search"
        type="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
        placeholder="Try “light” or “pencil”"
      />
      {visibleProducts.length > 0 ? (
        <ul className="product-list">
          {visibleProducts.map((product) => (
            <li key={product.id}>
              <span>{product.name}</span>
              <span className="category">{product.category}</span>
            </li>
          ))}
        </ul>
      ) : (
        <p className="empty-state">No matching products.</p>
      )}
      <p className="explanation">
        The visible list is calculated from the current query during render.
        There is no duplicate filtered state and no synchronization Effect.
      </p>
    </section>
  );
}

export default function App() {
  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React Hooks · chapter 052</p>
        <h1>Effects synchronize with the outside world</h1>
        <p className="intro">
          Start and pause the timer, expand the measured panel, and filter the
          list. Each example uses the simplest React pattern that fits.
        </p>
        <NetworkStatus />
      </header>

      <div className="demo-grid">
        <Stopwatch />
        <MeasuredPanel />
        <ProductSearch />
      </div>

      <footer className="page-footer">
        The specialized style insertion Hook is intentionally not used in this
        application demo. It is intended for CSS-in-JS library authors who
        insert style rules before layout Effects.
      </footer>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #172033;
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
input {
  font: inherit;
}

button {
  min-height: 44px;
  border: 0;
  border-radius: 0.65rem;
  padding: 0.65rem 1rem;
  color: #ffffff;
  background: #2459d3;
  cursor: pointer;
}

button:hover {
  background: #1948b5;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #f2a900;
  outline-offset: 3px;
}

.button-secondary {
  color: #24324b;
  background: #e7edf7;
}

.button-secondary:hover {
  background: #d8e1f0;
}

.page {
  width: min(1100px, calc(100% - 2rem));
  margin: 0 auto;
  padding: 3rem 0;
}

.page-header {
  max-width: 760px;
  margin-bottom: 2rem;
}

.eyebrow {
  margin-bottom: 0.65rem;
  color: #2459d3;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  max-width: 700px;
  font-size: clamp(2rem, 5vw, 3.5rem);
  letter-spacing: -0.04em;
}

h2 {
  margin-bottom: 1rem;
  font-size: 1.35rem;
}

.intro {
  max-width: 680px;
  margin-top: 1rem;
  color: #4b5870;
  font-size: 1.05rem;
  line-height: 1.65;
}

.status {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  margin-top: 1.25rem;
  color: #35425a;
}

.status-dot {
  width: 0.7rem;
  height: 0.7rem;
  border-radius: 50%;
  background: #9aa6b8;
}

.status-dot.is-online {
  background: #16834a;
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

.timer {
  margin: 1.25rem 0;
  font-size: 3rem;
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

.measurement-panel {
  display: grid;
  gap: 0.75rem;
  min-height: 5rem;
  border-left: 4px solid #2459d3;
  border-radius: 0.4rem;
  padding: 1rem;
  color: #35425a;
  background: #f2f6ff;
  line-height: 1.55;
}

.measurement-result {
  margin-top: 0.8rem;
  color: #1948b5;
  font-weight: 700;
}

.field-label {
  display: block;
  margin-bottom: 0.45rem;
  color: #35425a;
  font-weight: 650;
}

input[type="search"] {
  width: 100%;
  min-height: 44px;
  border: 1px solid #aebbd0;
  border-radius: 0.65rem;
  padding: 0.65rem 0.8rem;
  color: #172033;
  background: #ffffff;
}

.product-list {
  display: grid;
  gap: 0.55rem;
  margin-top: 1rem;
  list-style: none;
}

.product-list li {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  border-radius: 0.55rem;
  padding: 0.7rem 0.8rem;
  background: #f4f7fb;
}

.category {
  color: #58657a;
  font-size: 0.9rem;
}

.empty-state {
  margin-top: 1rem;
  color: #58657a;
}

.page-footer {
  max-width: 760px;
  margin-top: 1.5rem;
  color: #58657a;
  line-height: 1.6;
}

@media (max-width: 700px) {
  .page {
    padding: 2rem 0;
  }

  .demo-grid {
    grid-template-columns: 1fr;
  }
}
~~~

## What to observe

1. Start the stopwatch. The seconds increase while its Effect is active.
2. Pause it. Cleanup clears the interval. Start it again and there is only one active timer.
3. Expand the panel. The layout measurement runs after the expanded DOM is committed but before the browser paints, and the measured height is shown.
4. Search for “light” or “pencil.” The filtered list updates directly from state during render without an Effect.
5. In development Strict Mode, setup and cleanup may run an extra time. The network listeners and timer still remain correctly paired.

## Why the style-insertion Hook is not used here

The style-insertion Hook is intended primarily for CSS-in-JS library authors who insert generated style rules before layout Effects. A normal application should use stylesheets, extracted CSS, or ordinary style props. It is intentionally omitted from the runnable UI so the example does not present a specialized library API as a standard component pattern.
