# Example: On-Demand Dashboard Panels with lazy

This runnable React app keeps its overview in the initial application code and loads three larger panels only when selected:

- **Usage chart** loads a chart panel.
- **Release notes** loads a notes panel.
- **Preferences** loads a settings panel.

The lazy component declarations live at module scope. Each panel module has a default export. A single Suspense boundary shows a local loading skeleton while the selected panel's code is pending, and an Error Boundary displays a separate message if the import fails.

The loading indicator may appear only briefly on a fast local connection. The code does not add an artificial delay. The dynamically imported module is cached by React after it is loaded, but switching away from a panel unmounts its component instance; any local component state would be reset if it is mounted again.

## Create the project

~~~sh
npm create vite@latest lazy-components-demo -- --template react
cd lazy-components-demo
npm install
npm run dev
~~~

Replace or add the files below. Open the local address printed by Vite.

## Project files

~~~text
lazy-components-demo/
  index.html
  src/
    main.jsx
    App.jsx
    styles.css
    components/
      ChartPanel.jsx
      NotesPanel.jsx
      PreferencesPanel.jsx
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f6fb" />
    <title>React Lazy Loading Dashboard</title>
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

## src/App.jsx

~~~jsx
import { Component, lazy, Suspense, useState } from "react";

const ChartPanel = lazy(() => import("./components/ChartPanel.jsx"));
const NotesPanel = lazy(() => import("./components/NotesPanel.jsx"));
const PreferencesPanel = lazy(() =>
  import("./components/PreferencesPanel.jsx"),
);

const panels = {
  chart: {
    component: ChartPanel,
    label: "Usage chart",
  },
  notes: {
    component: NotesPanel,
    label: "Release notes",
  },
  preferences: {
    component: PreferencesPanel,
    label: "Preferences",
  },
};

class PanelErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-card" role="alert">
          <span className="status-icon status-icon--error" aria-hidden="true">
            !
          </span>
          <div>
            <h2>This panel could not be loaded</h2>
            <p>
              Check your connection. If the problem continues, refresh the
              application and try again.
            </p>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}

function OverviewPanel() {
  return (
    <article className="panel overview-panel">
      <div className="panel-heading">
        <div>
          <p className="eyebrow">Workspace overview</p>
          <h2>Your project at a glance</h2>
        </div>
        <span className="live-badge">
          <span aria-hidden="true" />
          All systems ready
        </span>
      </div>

      <p className="panel-intro">
        The overview is available immediately. Open a section below to load its
        component code on demand.
      </p>

      <div className="summary-grid">
        <section className="summary-card">
          <span className="summary-label">Active projects</span>
          <strong>08</strong>
          <span className="summary-note">Across 3 teams</span>
        </section>
        <section className="summary-card">
          <span className="summary-label">Tasks completed</span>
          <strong>124</strong>
          <span className="summary-note">This month</span>
        </section>
        <section className="summary-card">
          <span className="summary-label">Next milestone</span>
          <strong>Friday</strong>
          <span className="summary-note">Release review</span>
        </section>
      </div>

      <div className="tip-card">
        <span className="tip-mark" aria-hidden="true">i</span>
        <p>
          Choose <strong>Usage chart</strong>, <strong>Release notes</strong>,
          or <strong>Preferences</strong>. The first time you open one, its
          module is requested and the loading placeholder can appear.
        </p>
      </div>
    </article>
  );
}

function PanelLoading({ label }) {
  return (
    <div className="panel loading-panel" role="status" aria-live="polite">
      <div className="loading-heading">
        <span className="loading-pulse" aria-hidden="true" />
        <div>
          <p className="eyebrow">Loading section</p>
          <h2>{label}</h2>
        </div>
      </div>
      <div className="skeleton-line skeleton-line--wide" aria-hidden="true" />
      <div className="skeleton-line" aria-hidden="true" />
      <div className="skeleton-block" aria-hidden="true" />
      <span className="sr-only">Loading {label}...</span>
    </div>
  );
}

export default function App() {
  const [activePanel, setActivePanel] = useState(null);
  const selectedPanel = activePanel ? panels[activePanel] : null;
  const ActiveComponent = selectedPanel?.component;

  return (
    <main className="app-shell">
      <header className="topbar">
        <a className="brand" href="#overview" onClick={() => setActivePanel(null)}>
          <span className="brand-mark" aria-hidden="true">N</span>
          <span>Northstar</span>
        </a>
        <p className="topbar-caption">Project workspace</p>
        <div className="avatar" aria-label="Signed in as Ash">A</div>
      </header>

      <div className="page-content" id="overview">
        <section className="page-heading">
          <p className="eyebrow">Team snapshot</p>
          <h1>Good morning, Ash</h1>
          <p>Here is what is happening across your workspace.</p>
        </section>

        <nav className="panel-navigation" aria-label="Dashboard sections">
          <button
            type="button"
            className="nav-button"
            aria-pressed={activePanel === null}
            onClick={() => setActivePanel(null)}
          >
            Overview
          </button>
          <button
            type="button"
            className="nav-button"
            aria-pressed={activePanel === "chart"}
            onClick={() => setActivePanel("chart")}
          >
            Usage chart
          </button>
          <button
            type="button"
            className="nav-button"
            aria-pressed={activePanel === "notes"}
            onClick={() => setActivePanel("notes")}
          >
            Release notes
          </button>
          <button
            type="button"
            className="nav-button"
            aria-pressed={activePanel === "preferences"}
            onClick={() => setActivePanel("preferences")}
          >
            Preferences
          </button>
        </nav>

        <section className="workspace" aria-label="Selected dashboard content">
          {ActiveComponent ? (
            <PanelErrorBoundary key={activePanel}>
              <Suspense fallback={<PanelLoading label={selectedPanel.label} />}>
                <ActiveComponent />
              </Suspense>
            </PanelErrorBoundary>
          ) : (
            <OverviewPanel />
          )}
        </section>

        <footer className="page-footer">
          <span className="footer-dot" aria-hidden="true" />
          Your workspace is up to date
        </footer>
      </div>
    </main>
  );
}
~~~

The navigation and overview are eager. The panel modules are not rendered until selected, so their dynamic import loaders are not called at the initial render. The lazy component declarations remain stable because they are outside `App`.

The Error Boundary is outside Suspense. Pending code uses `PanelLoading`; a rejected import uses the error UI. Changing the selected panel resets the boundary's display state because its key changes. React still caches each lazy loader Promise, so this is not a guaranteed retry of a rejected import.

## src/components/ChartPanel.jsx

~~~jsx
const bars = [
  { day: "Mon", value: 42 },
  { day: "Tue", value: 68 },
  { day: "Wed", value: 54 },
  { day: "Thu", value: 86 },
  { day: "Fri", value: 72 },
  { day: "Sat", value: 38 },
  { day: "Sun", value: 61 },
];

export default function ChartPanel() {
  const points = bars
    .map((bar, index) => {
      const x = 32 + index * 56;
      const y = 202 - bar.value * 1.6;
      return index === 0 ? "M " + x + " " + y : "L " + x + " " + y;
    })
    .join(" ");

  return (
    <article className="panel">
      <div className="panel-heading">
        <div>
          <p className="eyebrow">Usage chart</p>
          <h2>Weekly activity</h2>
        </div>
        <span className="chart-period">Last 7 days</span>
      </div>

      <p className="panel-intro">
        A sample chart panel loaded from its own component module.
      </p>

      <div className="chart-card">
        <div className="chart-total">
          <div>
            <span className="summary-label">Total actions</span>
            <strong>1,842</strong>
          </div>
          <span className="trend-badge">+12.8%</span>
        </div>

        <svg
          className="activity-chart"
          viewBox="0 0 420 250"
          role="img"
          aria-labelledby="activity-chart-title activity-chart-description"
        >
          <title id="activity-chart-title">Weekly workspace activity</title>
          <desc id="activity-chart-description">
            A line chart showing activity from Monday through Sunday, highest
            on Thursday.
          </desc>
          {[50, 100, 150, 200].map((y) => (
            <line
              key={y}
              x1="24"
              x2="396"
              y1={y}
              y2={y}
              className="chart-gridline"
            />
          ))}
          <path d={points} className="chart-line" />
          {bars.map((bar, index) => {
            const x = 32 + index * 56;
            const y = 202 - bar.value * 1.6;
            return (
              <g key={bar.day}>
                <circle cx={x} cy={y} r="5" className="chart-point" />
                <text x={x} y="232" textAnchor="middle" className="chart-label">
                  {bar.day}
                </text>
              </g>
            );
          })}
        </svg>
      </div>
    </article>
  );
}
~~~

## src/components/NotesPanel.jsx

~~~jsx
const notes = [
  {
    version: "2.4.0",
    date: "November 18",
    title: "Workspace filters",
    description:
      "Save a useful filter and return to the same project view later.",
    tag: "New",
  },
  {
    version: "2.3.2",
    date: "November 12",
    title: "Keyboard navigation",
    description:
      "Move through dashboard sections with a keyboard and clear focus states.",
    tag: "Improved",
  },
  {
    version: "2.3.0",
    date: "November 04",
    title: "Compact activity cards",
    description:
      "Review more project activity at a glance on narrow screens.",
    tag: "New",
  },
];

export default function NotesPanel() {
  return (
    <article className="panel">
      <div className="panel-heading">
        <div>
          <p className="eyebrow">Release notes</p>
          <h2>What is new</h2>
        </div>
        <span className="chart-period">3 updates</span>
      </div>

      <p className="panel-intro">
        Recent changes for the project workspace.
      </p>

      <ol className="release-list">
        {notes.map((note) => (
          <li className="release-item" key={note.version}>
            <span className="release-marker" aria-hidden="true" />
            <div className="release-content">
              <div className="release-meta">
                <strong>{note.version}</strong>
                <span>{note.date}</span>
                <span className="release-tag">{note.tag}</span>
              </div>
              <h3>{note.title}</h3>
              <p>{note.description}</p>
            </div>
          </li>
        ))}
      </ol>
    </article>
  );
}
~~~

## src/components/PreferencesPanel.jsx

~~~jsx
import { useState } from "react";

export default function PreferencesPanel() {
  const [weeklySummary, setWeeklySummary] = useState(true);
  const [compactLayout, setCompactLayout] = useState(false);

  return (
    <article className="panel">
      <div className="panel-heading">
        <div>
          <p className="eyebrow">Preferences</p>
          <h2>Workspace settings</h2>
        </div>
      </div>

      <p className="panel-intro">
        These controls demonstrate state owned by a lazy-loaded component.
      </p>

      <div className="preferences-list">
        <label className="preference-row">
          <span className="preference-copy">
            <strong>Weekly summary</strong>
            <span>Show a short summary at the start of each week.</span>
          </span>
          <input
            type="checkbox"
            checked={weeklySummary}
            onChange={(event) => setWeeklySummary(event.target.checked)}
          />
        </label>

        <label className="preference-row">
          <span className="preference-copy">
            <strong>Compact layout</strong>
            <span>Use tighter spacing in the project workspace.</span>
          </span>
          <input
            type="checkbox"
            checked={compactLayout}
            onChange={(event) => setCompactLayout(event.target.checked)}
          />
        </label>
      </div>

      <p className="state-note">
        These preferences are local component state for this demonstration.
        Select another section and return to see that this panel unmounts and
        its local state starts over.
      </p>
    </article>
  );
}
~~~

## src/styles.css

~~~css
:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #172033;
  background: #f4f6fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  font-weight: 400;
  --ink: #172033;
  --muted: #667085;
  --line: #e5e9f1;
  --brand: #5a57d6;
  --brand-soft: #efefff;
  --surface: #ffffff;
  --shadow: 0 16px 45px rgb(32 45 74 / 7%);
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

button,
input {
  font: inherit;
}

button:focus-visible,
a:focus-visible,
input:focus-visible {
  outline: 3px solid #9b9af2;
  outline-offset: 3px;
}

.app-shell {
  min-height: 100vh;
}

.topbar {
  display: flex;
  align-items: center;
  gap: 1rem;
  min-height: 72px;
  padding: 0 clamp(1rem, 5vw, 4.5rem);
  border-bottom: 1px solid var(--line);
  background: var(--surface);
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
  color: var(--ink);
  font-size: 1rem;
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

.topbar-caption {
  margin: 0;
  color: var(--muted);
  font-size: 0.875rem;
}

.avatar {
  display: grid;
  width: 34px;
  height: 34px;
  margin-left: auto;
  place-items: center;
  border: 1px solid #d9d8ff;
  border-radius: 50%;
  background: var(--brand-soft);
  color: var(--brand);
  font-size: 0.85rem;
  font-weight: 750;
}

.page-content {
  width: min(100% - 2rem, 1040px);
  margin: 0 auto;
  padding: clamp(2.5rem, 7vw, 5.5rem) 0 2rem;
}

.page-heading {
  margin-bottom: 2rem;
}

.eyebrow {
  margin: 0 0 0.55rem;
  color: var(--brand);
  font-size: 0.72rem;
  font-weight: 800;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.page-heading h1 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(2rem, 5vw, 3.25rem);
  letter-spacing: -0.055em;
}

.page-heading > p:last-child,
.panel-intro {
  color: var(--muted);
  line-height: 1.65;
}

.panel-navigation {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.nav-button {
  min-height: 42px;
  padding: 0.65rem 1rem;
  border: 1px solid transparent;
  border-radius: 10px;
  background: transparent;
  color: #475467;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 650;
}

.nav-button:hover {
  background: #eaecf4;
}

.nav-button[aria-pressed="true"] {
  border-color: #dadaff;
  background: var(--brand-soft);
  color: #3936a8;
}

.workspace {
  min-height: 390px;
}

.panel {
  min-height: 390px;
  padding: clamp(1.25rem, 4vw, 2rem);
  border: 1px solid var(--line);
  border-radius: 18px;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.panel-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
}

.panel-heading h2,
.error-card h2 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(1.35rem, 3vw, 1.75rem);
  letter-spacing: -0.035em;
}

.panel-intro {
  max-width: 680px;
  margin: 1rem 0 1.5rem;
}

.live-badge,
.trend-badge,
.release-tag {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  white-space: nowrap;
  border-radius: 999px;
  background: #ecfdf3;
  color: #137547;
  font-size: 0.78rem;
  font-weight: 700;
}

.live-badge {
  padding: 0.5rem 0.75rem;
}

.live-badge > span,
.footer-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #25a66a;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
  margin: 1.75rem 0;
}

.summary-card {
  display: grid;
  gap: 0.6rem;
  min-height: 142px;
  align-content: start;
  padding: 1.15rem;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: #fcfcfe;
}

.summary-label,
.summary-note {
  color: var(--muted);
  font-size: 0.82rem;
}

.summary-card strong,
.chart-total strong {
  color: var(--ink);
  font-size: 1.8rem;
  letter-spacing: -0.04em;
}

.tip-card {
  display: flex;
  gap: 0.75rem;
  align-items: flex-start;
  padding: 1rem;
  border: 1px solid #e4e3ff;
  border-radius: 12px;
  background: #f8f7ff;
}

.tip-card p {
  margin: 0;
  color: #55536f;
  font-size: 0.9rem;
  line-height: 1.6;
}

.tip-mark {
  display: grid;
  flex: 0 0 22px;
  width: 22px;
  height: 22px;
  place-items: center;
  border-radius: 50%;
  background: #dedcff;
  color: #3936a8;
  font-size: 0.8rem;
  font-weight: 800;
}

.chart-period {
  padding: 0.45rem 0.7rem;
  border: 1px solid var(--line);
  border-radius: 8px;
  color: #475467;
  font-size: 0.8rem;
}

.chart-card {
  padding: 1.25rem;
  border: 1px solid var(--line);
  border-radius: 14px;
}

.chart-total {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.chart-total > div {
  display: grid;
  gap: 0.4rem;
}

.trend-badge {
  padding: 0.45rem 0.65rem;
}

.activity-chart {
  display: block;
  width: 100%;
  margin-top: 1rem;
  overflow: visible;
}

.chart-gridline {
  stroke: #edf0f5;
  stroke-dasharray: 4 6;
}

.chart-line {
  fill: none;
  stroke: var(--brand);
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 4;
}

.chart-point {
  fill: white;
  stroke: var(--brand);
  stroke-width: 3;
}

.chart-label {
  fill: #7b8496;
  font-size: 12px;
}

.release-list {
  display: grid;
  gap: 0;
  margin: 2rem 0 0;
  padding: 0;
  list-style: none;
}

.release-item {
  display: grid;
  grid-template-columns: 18px 1fr;
  gap: 1rem;
  padding: 0 0 1.75rem;
}

.release-marker {
  position: relative;
  width: 12px;
  height: 12px;
  margin-top: 4px;
  border: 3px solid #dadaff;
  border-radius: 50%;
  background: var(--brand);
}

.release-item:not(:last-child) .release-marker::after {
  position: absolute;
  top: 9px;
  left: 1px;
  width: 2px;
  height: calc(100% + 1.25rem);
  background: #e7e7f5;
  content: "";
}

.release-meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.6rem;
  color: var(--muted);
  font-size: 0.78rem;
}

.release-meta strong {
  color: var(--ink);
}

.release-tag {
  padding: 0.25rem 0.55rem;
  background: var(--brand-soft);
  color: #3936a8;
}

.release-content h3 {
  margin: 0.55rem 0 0.3rem;
  color: var(--ink);
  font-size: 1rem;
}

.release-content p,
.state-note {
  margin: 0;
  color: var(--muted);
  font-size: 0.9rem;
  line-height: 1.6;
}

.preferences-list {
  display: grid;
  border-top: 1px solid var(--line);
}

.preference-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1.15rem 0;
  border-bottom: 1px solid var(--line);
  cursor: pointer;
}

.preference-copy {
  display: grid;
  gap: 0.35rem;
}

.preference-copy strong {
  color: var(--ink);
  font-size: 0.92rem;
}

.preference-copy span {
  color: var(--muted);
  font-size: 0.84rem;
}

.preference-row input {
  width: 18px;
  height: 18px;
  accent-color: var(--brand);
}

.state-note {
  margin-top: 1rem;
  font-size: 0.82rem;
}

.loading-panel {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.loading-heading {
  display: flex;
  align-items: center;
  gap: 0.9rem;
  margin-bottom: 0.5rem;
}

.loading-heading h2 {
  margin: 0;
  font-size: 1.35rem;
}

.loading-pulse {
  width: 42px;
  height: 42px;
  border-radius: 13px;
  background: linear-gradient(110deg, #ececf6 25%, #f7f7fc 50%, #ececf6 75%);
  background-size: 200% 100%;
  animation: loading-shimmer 1.4s ease-in-out infinite;
}

.skeleton-line,
.skeleton-block {
  border-radius: 9px;
  background: #eff1f6;
}

.skeleton-line {
  width: min(60%, 420px);
  height: 13px;
}

.skeleton-line--wide {
  width: min(82%, 620px);
}

.skeleton-block {
  width: 100%;
  height: 205px;
  margin-top: 0.5rem;
  border: 1px solid #e9ebf2;
  background:
    linear-gradient(#f5f6fa, #f5f6fa) padding-box,
    repeating-linear-gradient(
      to bottom,
      transparent 0,
      transparent 48px,
      #e8eaf1 49px,
      transparent 50px
    );
}

.error-card {
  display: flex;
  gap: 1rem;
  align-items: flex-start;
  min-height: 220px;
  padding: 2rem;
  border: 1px solid #f2caca;
  border-radius: 18px;
  background: #fffafa;
  box-shadow: var(--shadow);
}

.error-card p {
  max-width: 580px;
  color: var(--muted);
  line-height: 1.65;
}

.status-icon {
  display: grid;
  flex: 0 0 32px;
  width: 32px;
  height: 32px;
  place-items: center;
  border-radius: 50%;
  font-weight: 800;
}

.status-icon--error {
  background: #ffe5e5;
  color: #b42318;
}

.page-footer {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  padding: 1.25rem 0;
  color: var(--muted);
  font-size: 0.82rem;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  clip-path: inset(50%);
}

@keyframes loading-shimmer {
  to {
    background-position: -200% 0;
  }
}

@media (max-width: 680px) {
  .topbar {
    min-height: 62px;
  }

  .topbar-caption {
    display: none;
  }

  .summary-grid {
    grid-template-columns: 1fr;
  }

  .summary-card {
    min-height: auto;
  }

  .panel-heading {
    flex-direction: column;
  }

  .live-badge {
    align-self: flex-start;
  }

  .chart-card {
    padding: 0.75rem;
  }

  .error-card {
    padding: 1.25rem;
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

1. On the initial view, the overview renders without any of the three panel modules being rendered.
2. Open **Usage chart**. The chart's dynamic import runs when React first renders that lazy component; Suspense can show its fallback while the module is pending.
3. Open **Release notes** and **Preferences**. Each panel has its own module and first-load point.
4. Return to **Overview**. The overview and navigation remain outside the Suspense boundary.
5. Change a preference, open another section, and return. The Preferences component was unmounted, so its local state starts with its initial values again even though its module remains cached.
6. The Error Boundary is separate from the loading fallback. To see a real rejected-import state, use browser developer tools to block a panel's emitted module request, then select that panel. Restore normal network behavior afterward.

This example demonstrates client-side component code loading. It does not use Suspense for data fetching, and the local fallback may be too quick to notice without network throttling.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="066-suspense-boundaries-and-fallbacks.md">066 - Example: Suspense Boundaries and Progressive Fallbacks</a></td><td><a href="../lessons/067-lazy-loading-components.md">Lesson</a></td><td><a href="../qnas/067-lazy-loading-components.md">Q&amp;A</a></td><td><a href="068-error-boundaries-and-recovery-ui.md">068 - Example: Recovering from a Render Error</a></td></tr>
  </tbody>
</table>
