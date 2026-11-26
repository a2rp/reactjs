# Example: Recovering from a Render Error

This runnable React app demonstrates a local Error Boundary around a reports panel:

- The dashboard heading and control stay outside the boundary.
- A button changes parent state so the child throws during rendering.
- The boundary replaces only the failed reports panel with an accessible fallback.
- A recovery button clears the cause and remounts the boundary with a new key.
- The raw error and component stack are logged for development diagnostics, not displayed in the UI.

The example intentionally triggers a render error. It does not throw from the button's event handler, because an Error Boundary does not catch errors thrown directly by event handlers.

## Create the project

~~~sh
npm create vite@latest error-boundary-demo -- --template react
cd error-boundary-demo
npm install
npm run dev
~~~

Replace or add the files below, then open the local address printed by Vite.

## Project files

~~~text
error-boundary-demo/
  index.html
  src/
    main.jsx
    App.jsx
    ErrorBoundary.jsx
    components/
      ReportsPanel.jsx
    styles.css
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f6fb" />
    <title>React Error Boundary Demo</title>
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

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## src/ErrorBoundary.jsx

~~~jsx
import { Component } from "react";

export default class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Development-only diagnostic example. Use an approved reporting service
    // in production, and do not attach private form values or credentials.
    console.error("Reports panel render failed:", error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return (
        <section className="error-panel" role="alert">
          <div className="error-symbol" aria-hidden="true">
            !
          </div>
          <div className="error-copy">
            <p className="eyebrow">Reports unavailable</p>
            <h2>This report could not be displayed</h2>
            <p>
              The rest of the dashboard is still available. Restore the demo
              panel to clear the example error and try again.
            </p>
            {this.props.onReset && (
              <button
                className="button button--primary"
                type="button"
                onClick={this.props.onReset}
              >
                Restore reports panel
              </button>
            )}
          </div>
        </section>
      );
    }

    return this.props.children;
  }
}
~~~

The boundary uses `getDerivedStateFromError` only to switch to fallback UI. `componentDidCatch` logs diagnostic details for this local demonstration. The fallback never prints the raw thrown value.

## src/components/ReportsPanel.jsx

~~~jsx
const reportRows = [
  { project: "Website refresh", owner: "Design", progress: 82 },
  { project: "Billing updates", owner: "Platform", progress: 64 },
  { project: "Onboarding flow", owner: "Product", progress: 91 },
];

export default function ReportsPanel({ shouldFail }) {
  if (shouldFail) {
    throw new Error("Intentional error used to demonstrate the boundary.");
  }

  return (
    <article className="reports-card" aria-labelledby="reports-title">
      <div className="card-heading">
        <div>
          <p className="eyebrow">Project reports</p>
          <h2 id="reports-title">Current work</h2>
        </div>
        <span className="period-label">This week</span>
      </div>

      <p className="card-description">
        A normal render of the reports region. The data is local demo content.
      </p>

      <div className="table-scroll">
        <table>
          <thead>
            <tr>
              <th scope="col">Project</th>
              <th scope="col">Team</th>
              <th scope="col">Progress</th>
            </tr>
          </thead>
          <tbody>
            {reportRows.map((row) => (
              <tr key={row.project}>
                <th scope="row">{row.project}</th>
                <td>{row.owner}</td>
                <td>
                  <div
                    className="progress-cell"
                    aria-label={row.progress + "% complete"}
                  >
                    <span className="progress-track" aria-hidden="true">
                      <span style={{ width: row.progress + "%" }} />
                    </span>
                    <span className="progress-value">{row.progress}%</span>
                  </div>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </article>
  );
}
~~~

The error is thrown while `ReportsPanel` renders. It is not thrown by an event handler. The parent's button only changes a prop; the next render triggers the intentional error inside the boundary.

## src/App.jsx

~~~jsx
import { useState } from "react";
import ErrorBoundary from "./ErrorBoundary.jsx";
import ReportsPanel from "./components/ReportsPanel.jsx";

function SummaryCard({ label, value, note }) {
  return (
    <article className="summary-card">
      <p>{label}</p>
      <strong>{value}</strong>
      <span>{note}</span>
    </article>
  );
}

export default function App() {
  const [shouldFail, setShouldFail] = useState(false);
  const [boundaryVersion, setBoundaryVersion] = useState(0);

  function resetReports() {
    setShouldFail(false);
    setBoundaryVersion((version) => version + 1);
  }

  return (
    <main className="app-shell">
      <header className="topbar">
        <a className="brand" href="#dashboard">
          <span className="brand-mark" aria-hidden="true">
            N
          </span>
          Northstar
        </a>
        <span className="topbar-label">Project workspace</span>
        <span className="avatar" aria-label="Signed in as Ash">
          A
        </span>
      </header>

      <div className="page-content" id="dashboard">
        <section className="page-heading" aria-labelledby="page-title">
          <p className="eyebrow">Team dashboard</p>
          <h1 id="page-title">Project overview</h1>
          <p>
            A problem in one feature should not make the whole page unusable.
          </p>
        </section>

        <section className="summary-grid" aria-label="Project summary">
          <SummaryCard label="Active projects" value="08" note="Across 3 teams" />
          <SummaryCard label="Completed tasks" value="124" note="This month" />
          <SummaryCard label="Next milestone" value="Friday" note="Release review" />
        </section>

        <section className="demo-controls" aria-labelledby="demo-title">
          <div>
            <p className="eyebrow">Boundary test</p>
            <h2 id="demo-title">Simulate a reports failure</h2>
            <p>
              This control is outside the reports boundary, so it remains
              available when the reports panel fails.
            </p>
          </div>
          <button
            className="button button--danger"
            type="button"
            onClick={() => setShouldFail(true)}
            disabled={shouldFail}
          >
            Simulate render error
          </button>
        </section>

        <section className="reports-region" aria-label="Reports region">
          <ErrorBoundary
            key={boundaryVersion}
            onReset={resetReports}
          >
            <ReportsPanel shouldFail={shouldFail} />
          </ErrorBoundary>
        </section>

        <footer className="page-footer">
          <span className="status-dot" aria-hidden="true" />
          Dashboard shell is still available
        </footer>
      </div>
    </main>
  );
}
~~~

`boundaryVersion` is part of the boundary's `key`. Pressing **Restore reports panel** clears `shouldFail` and changes the key, so React mounts a fresh boundary and a fresh reports subtree. The error is recoverable here because the parent can remove the condition that caused it.

## src/styles.css

~~~css
:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #182230;
  background: #f4f6fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  --ink: #182230;
  --muted: #667085;
  --line: #e3e7ef;
  --brand: #514dd4;
  --brand-soft: #efefff;
  --surface: #ffffff;
  --danger: #b42318;
  --danger-soft: #fff4f2;
  --shadow: 0 16px 42px rgb(31 45 74 / 7%);
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button {
  font: inherit;
}

button:focus-visible,
a:focus-visible {
  outline: 3px solid #a5a3ff;
  outline-offset: 3px;
}

.app-shell {
  min-height: 100vh;
}

.topbar {
  display: flex;
  min-height: 70px;
  align-items: center;
  gap: 1rem;
  padding: 0 clamp(1rem, 5vw, 4.5rem);
  border-bottom: 1px solid var(--line);
  background: var(--surface);
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
  color: var(--ink);
  font-weight: 750;
  text-decoration: none;
}

.brand-mark {
  display: grid;
  width: 34px;
  height: 34px;
  place-items: center;
  border-radius: 11px;
  background: var(--brand);
  color: white;
  font-weight: 800;
}

.topbar-label {
  color: var(--muted);
  font-size: 0.88rem;
}

.avatar {
  display: grid;
  width: 34px;
  height: 34px;
  margin-left: auto;
  place-items: center;
  border: 1px solid #d8d7ff;
  border-radius: 50%;
  background: var(--brand-soft);
  color: var(--brand);
  font-size: 0.85rem;
  font-weight: 750;
}

.page-content {
  width: min(100% - 2rem, 1050px);
  margin: 0 auto;
  padding: clamp(2.5rem, 7vw, 5rem) 0 2rem;
}

.page-heading {
  margin-bottom: 1.6rem;
}

.eyebrow {
  margin: 0 0 0.5rem;
  color: var(--brand);
  font-size: 0.72rem;
  font-weight: 800;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.page-heading h1 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(2rem, 5vw, 3.1rem);
  letter-spacing: -0.055em;
}

.page-heading > p:last-child,
.card-description,
.demo-controls p:last-child {
  color: var(--muted);
  line-height: 1.65;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.summary-card {
  display: grid;
  gap: 0.55rem;
  min-height: 130px;
  align-content: start;
  padding: 1.1rem;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.summary-card p,
.summary-card span {
  margin: 0;
  color: var(--muted);
  font-size: 0.82rem;
}

.summary-card strong {
  color: var(--ink);
  font-size: 1.75rem;
  letter-spacing: -0.04em;
}

.demo-controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1.25rem;
  margin: 1.75rem 0 1rem;
  padding: 1.2rem 1.3rem;
  border: 1px solid #e4e3ff;
  border-radius: 14px;
  background: #fafaff;
}

.demo-controls h2 {
  margin: 0;
  color: var(--ink);
  font-size: 1.05rem;
}

.demo-controls p:last-child {
  max-width: 640px;
  margin: 0.4rem 0 0;
  font-size: 0.86rem;
}

.button {
  min-height: 42px;
  flex: 0 0 auto;
  padding: 0.65rem 0.9rem;
  border: 1px solid transparent;
  border-radius: 9px;
  cursor: pointer;
  font-size: 0.86rem;
  font-weight: 700;
}

.button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.button--danger {
  border-color: #f2c9c4;
  background: white;
  color: var(--danger);
}

.button--danger:hover:not(:disabled) {
  background: var(--danger-soft);
}

.button--primary {
  background: var(--brand);
  color: white;
}

.button--primary:hover {
  background: #403bb9;
}

.reports-region {
  min-height: 270px;
}

.reports-card,
.error-panel {
  min-height: 270px;
  padding: clamp(1.2rem, 4vw, 1.8rem);
  border: 1px solid var(--line);
  border-radius: 16px;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.card-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
}

.card-heading h2,
.error-copy h2 {
  margin: 0;
  color: var(--ink);
  font-size: 1.45rem;
  letter-spacing: -0.035em;
}

.period-label {
  padding: 0.45rem 0.7rem;
  border: 1px solid var(--line);
  border-radius: 8px;
  color: #475467;
  font-size: 0.8rem;
  white-space: nowrap;
}

.card-description {
  margin: 0.8rem 0 1.25rem;
  font-size: 0.9rem;
}

.table-scroll {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
}

th,
td {
  padding: 0.85rem 0.75rem;
  border-top: 1px solid var(--line);
  color: #475467;
  font-size: 0.87rem;
}

thead th {
  color: var(--muted);
  font-size: 0.75rem;
  letter-spacing: 0.04em;
  text-transform: uppercase;
}

tbody th {
  color: var(--ink);
  font-weight: 650;
}

.progress-cell {
  display: flex;
  min-width: 130px;
  align-items: center;
  gap: 0.65rem;
}

.progress-track {
  display: block;
  width: min(100%, 100px);
  height: 7px;
  overflow: hidden;
  border-radius: 999px;
  background: #eceef5;
}

.progress-track > span {
  display: block;
  height: 100%;
  border-radius: inherit;
  background: var(--brand);
}

.progress-value {
  color: #475467;
  font-size: 0.8rem;
}

.error-panel {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  border-color: #f0c7c3;
  background: #fffdfd;
}

.error-symbol {
  display: grid;
  width: 34px;
  height: 34px;
  flex: 0 0 34px;
  place-items: center;
  border-radius: 50%;
  background: #ffe6e3;
  color: var(--danger);
  font-size: 1rem;
  font-weight: 800;
}

.error-copy {
  max-width: 650px;
}

.error-copy > p:not(.eyebrow) {
  margin: 0.65rem 0 1rem;
  color: var(--muted);
  line-height: 1.65;
}

.page-footer {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  padding: 1.25rem 0;
  color: var(--muted);
  font-size: 0.82rem;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #20a56a;
}

@media (max-width: 700px) {
  .summary-grid {
    grid-template-columns: 1fr;
  }

  .summary-card {
    min-height: 105px;
  }

  .demo-controls {
    align-items: flex-start;
    flex-direction: column;
  }

  .button {
    width: 100%;
  }

  .card-heading {
    flex-direction: column;
  }

  th,
  td {
    min-width: 105px;
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

## What to observe

1. The dashboard renders with its summary, control, and reports panel.
2. Select **Simulate render error**. The parent state changes, then `ReportsPanel` throws during its render.
3. The Error Boundary replaces the reports region with an alert fallback. The heading, summary, and trigger control remain visible because they are outside the boundary.
4. Select **Restore reports panel**. The parent clears the failure condition and changes the boundary key. React mounts a fresh boundary and reports panel, which now renders successfully.
5. Check the development console for the error and component stack. The visible fallback does not reveal the raw error.
6. Compare the render failure above with a throw placed directly inside an event handler: an Error Boundary does not catch the event-handler exception. Handle that error in the handler instead.

This is a client-rendering example. An Error Boundary is not server-rendering error handling, and ordinary rejected requests or timer callbacks must be handled in their own code paths.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="067-lazy-loading-components.md">067 - Example: On-Demand Dashboard Panels with lazy</a></td><td><a href="../lessons/068-error-boundaries-and-recovery-ui.md">Lesson</a></td><td><a href="../qnas/068-error-boundaries-and-recovery-ui.md">Q&amp;A</a></td><td><a href="069-portals.md">069 - Example: A Portaled Help Panel</a></td></tr>
  </tbody>
</table>
