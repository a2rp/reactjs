# Example: Concurrent Rendering and Update Priorities

This runnable React app compares two ways to keep an interaction responsive while slower UI catches up:

- A search field keeps its controlled value urgent and passes a deferred query to a memoized results list.
- A tab switch uses **useTransition** for a non-urgent panel update and reports whether that Transition is pending.

The example makes no network requests. React scheduling depends on the actual render workload and device; a pending indicator may be too brief to paint for fast updates.

## Create the project

~~~sh
npm create vite@latest concurrent-priorities-demo -- --template react
cd concurrent-priorities-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following and start the app:

~~~sh
npm run dev
~~~

## Project files

~~~text
concurrent-priorities-demo/
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
    <meta name="theme-color" content="#f4f7fb" />
    <title>React Update Priorities Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## src/main.jsx

~~~jsx
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
~~~

## src/App.jsx

~~~jsx
import { memo, useDeferredValue, useState, useTransition } from "react";

const CATALOG = [
  { id: "react", title: "React", category: "Library", description: "Build user interfaces from components." },
  { id: "jsx", title: "JSX", category: "Syntax", description: "Describe UI with JavaScript syntax extensions." },
  { id: "props", title: "Props", category: "Components", description: "Pass values from a parent to a child." },
  { id: "state", title: "State", category: "Components", description: "Store values that change over time." },
  { id: "events", title: "Events", category: "Interaction", description: "Respond to user actions with event handlers." },
  { id: "effects", title: "Effects", category: "Components", description: "Synchronize a component with an external system." },
  { id: "memo", title: "Memo", category: "Performance", description: "Skip some renders when props have not changed." },
  { id: "deferred", title: "Deferred values", category: "Performance", description: "Let a slower UI consumer catch up to a value." },
  { id: "transitions", title: "Transitions", category: "Performance", description: "Mark an update as non-urgent rendering work." },
  { id: "suspense", title: "Suspense", category: "Loading", description: "Show a fallback while supported content is unavailable." },
  { id: "context", title: "Context", category: "Components", description: "Make a value available to a subtree." },
  { id: "reducer", title: "Reducer", category: "State", description: "Describe state changes with a reducer function." },
];

const VIEWS = {
  overview: {
    title: "Overview",
    description: "A concise summary of the current workspace and its recent activity.",
    points: ["Review the latest changes", "Check the current project status", "Keep essential actions close"],
  },
  reports: {
    title: "Reports",
    description: "A reporting panel can contain charts, summaries, and a sizeable data view.",
    points: ["Compare the latest reporting period", "Review saved filters", "Export a report when it is ready"],
  },
  activity: {
    title: "Activity",
    description: "An activity panel presents recent work in chronological order.",
    points: ["See recent edits", "Review comments and decisions", "Find the item that needs attention"],
  },
};

const CatalogResults = memo(function CatalogResults({ query }) {
  const normalizedQuery = query.trim().toLowerCase();
  const results = CATALOG.filter((item) => {
    const searchableText = `${item.title} ${item.category} ${item.description}`;
    return searchableText.toLowerCase().includes(normalizedQuery);
  });

  return (
    <ul className="result-list" aria-label="Matching topics">
      {results.map((item) => (
        <li className="result-card" key={item.id}>
          <div className="result-card__heading">
            <h3>{item.title}</h3>
            <span>{item.category}</span>
          </div>
          <p>{item.description}</p>
        </li>
      ))}
      {results.length === 0 && (
        <li className="empty-state">No topics match this search.</li>
      )}
    </ul>
  );
});

function DeferredSearch() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <section className="panel" aria-labelledby="search-title">
      <p className="eyebrow">Urgent input + deferred consumer</p>
      <h2 id="search-title">Search the topic catalog</h2>
      <p className="panel-copy">
        The input uses the latest query immediately. The memoized results list
        receives a deferred query and can catch up during background rendering.
      </p>
      <label className="field-label" htmlFor="catalog-search">Search topics</label>
      <input
        className="search-input"
        id="catalog-search"
        type="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
        placeholder="Try “state” or “performance”"
      />
      <p className="status-line" role="status" aria-live="polite">
        {isStale ? "Updating results…" : `Showing results for ${query.trim() ? `“${query.trim()}”` : "all topics"}.`}
      </p>
      <div className="results-region" aria-busy={isStale}>
        <CatalogResults query={deferredQuery} />
      </div>
      <p className="note">
        The status and busy state belong to DOM elements here, so assistive
        technology can identify when the displayed results are catching up.
      </p>
    </section>
  );
}

function TransitionPanel({ view }) {
  const content = VIEWS[view];

  return (
    <div className="transition-content">
      <div>
        <p className="eyebrow">Selected panel</p>
        <h3>{content.title}</h3>
        <p>{content.description}</p>
      </div>
      <ul>
        {content.points.map((point) => <li key={point}>{point}</li>)}
      </ul>
    </div>
  );
}

function TransitionTabs() {
  const [selectedView, setSelectedView] = useState("overview");
  const [isPending, startTransition] = useTransition();

  function selectView(nextView) {
    startTransition(() => {
      setSelectedView(nextView);
    });
  }

  return (
    <section className="panel" aria-labelledby="tabs-title">
      <p className="eyebrow">Non-urgent state update</p>
      <h2 id="tabs-title">Switch a workspace panel</h2>
      <p className="panel-copy">
        The selected panel state is updated in a Transition. React can keep
        already revealed UI available while preparing the next view.
      </p>
      <div className="tab-list" role="group" aria-label="Workspace panels">
        {Object.entries(VIEWS).map(([id, view]) => (
          <button
            className="tab-button"
            type="button"
            key={id}
            aria-pressed={selectedView === id}
            onClick={() => selectView(id)}
          >
            {view.title}
          </button>
        ))}
      </div>
      <p className="status-line" role="status" aria-live="polite">
        {isPending ? "Preparing the selected panel…" : "Panel is ready."}
      </p>
      <div className="panel-content" aria-busy={isPending}>
        <TransitionPanel view={selectedView} />
      </div>
      <p className="note">
        This small panel may render quickly, so the pending message can be too
        brief to appear on screen. The example does not add an artificial delay.
      </p>
    </section>
  );
}

export default function App() {
  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React rendering priorities</p>
        <h1>Keep immediate interactions responsive</h1>
        <p className="page-intro">
          Try the deferred search and Transition tabs. These APIs let React
          prioritize rendering; they do not make JavaScript run on another
          thread or reduce network requests.
        </p>
      </header>

      <div className="demo-grid">
        <DeferredSearch />
        <TransitionTabs />
      </div>

      <footer className="page-footer">
        Keep controlled input state urgent. Defer a slower consumer with
        <code> useDeferredValue</code>, or mark a non-urgent state update with
        <code> useTransition</code>.
      </footer>
    </main>
  );
}
~~~

## src/styles.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  color: #172033;
  background: #f4f7fb;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input {
  font: inherit;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #6c63ff;
  outline-offset: 3px;
}

.page-shell {
  width: min(1120px, calc(100% - 2rem));
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 760px;
  margin-bottom: 2rem;
}

.eyebrow {
  color: #6256ce;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2,
h3 {
  line-height: 1.2;
}

h1 {
  margin-top: 0.6rem;
  font-size: clamp(2rem, 5vw, 3.4rem);
  letter-spacing: -0.045em;
}

.page-intro,
.panel-copy {
  margin-top: 0.8rem;
  color: #536078;
  line-height: 1.65;
}

.demo-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1.25rem;
  align-items: start;
}

.panel {
  min-width: 0;
  padding: 1.35rem;
  border: 1px solid #d9dfeb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 36px rgb(29 42 70 / 7%);
}

.panel h2 {
  margin-top: 0.45rem;
  font-size: 1.45rem;
}

.field-label {
  display: block;
  margin-top: 1.25rem;
  font-size: 0.9rem;
  font-weight: 700;
}

.search-input {
  width: 100%;
  min-height: 2.8rem;
  margin-top: 0.45rem;
  padding: 0.65rem 0.8rem;
  border: 1px solid #bfc8d8;
  border-radius: 0.55rem;
  color: #172033;
  background: #fff;
}

.search-input::placeholder {
  color: #718096;
}

.status-line {
  min-height: 1.5rem;
  margin-top: 0.85rem;
  color: #6256ce;
  font-size: 0.88rem;
  font-weight: 700;
}

.results-region,
.panel-content {
  margin-top: 0.6rem;
}

.results-region[aria-busy="true"],
.panel-content[aria-busy="true"] {
  opacity: 0.66;
}

.result-list {
  display: grid;
  gap: 0.55rem;
  list-style: none;
}

.result-card,
.empty-state {
  padding: 0.8rem;
  border: 1px solid #e1e6ef;
  border-radius: 0.65rem;
  background: #fbfcfe;
}

.result-card__heading {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 0.5rem;
  align-items: center;
}

.result-card h3,
.transition-content h3 {
  font-size: 1rem;
}

.result-card__heading span {
  padding: 0.22rem 0.5rem;
  border-radius: 999px;
  color: #5145b8;
  background: #efedff;
  font-size: 0.72rem;
  font-weight: 700;
}

.result-card p,
.transition-content p,
.transition-content li {
  margin-top: 0.35rem;
  color: #536078;
  font-size: 0.88rem;
  line-height: 1.5;
}

.empty-state {
  color: #536078;
}

.tab-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 1.25rem;
}

.tab-button {
  min-height: 2.5rem;
  padding: 0.5rem 0.8rem;
  border: 1px solid #c7cfe0;
  border-radius: 0.55rem;
  color: #28334a;
  background: #fff;
  cursor: pointer;
}

.tab-button[aria-pressed="true"] {
  border-color: #6256ce;
  color: #fff;
  background: #6256ce;
}

.panel-content {
  min-height: 12rem;
  padding: 1rem;
  border: 1px solid #e1e6ef;
  border-radius: 0.7rem;
  background: #fbfcfe;
}

.transition-content ul {
  display: grid;
  gap: 0.4rem;
  margin-top: 1rem;
  padding-left: 1.2rem;
}

.note {
  margin-top: 1rem;
  color: #68758b;
  font-size: 0.8rem;
  line-height: 1.55;
}

.page-footer {
  margin-top: 1.5rem;
  padding: 1rem 1.1rem;
  border-left: 4px solid #6256ce;
  color: #536078;
  background: #ebe9ff;
  line-height: 1.6;
}

.page-footer code {
  color: #3f35a0;
  font-weight: 700;
}

@media (max-width: 760px) {
  .page-shell {
    width: min(100% - 1rem, 620px);
    padding-block: 1.5rem;
  }

  .demo-grid {
    grid-template-columns: 1fr;
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

## Run the experiment

1. Type into the search field. The input value remains responsive because its state update is urgent.
2. Notice the results use a deferred query. During the short period when they lag behind, the status and **aria-busy** state indicate that they are catching up.
3. Search for a term such as `performance`, `state`, or text that has no match. Clear the field to show the full catalog again.
4. Select **Reports** or **Activity**. The panel state update is wrapped in **startTransition**, and **isPending** provides local status feedback while work is pending.
5. The panel is intentionally small, so it may update too quickly for the pending message to appear. Do not add an artificial timeout just to force that message.

## Why the search stays responsive

**DeferredSearch** stores the controlled input value in **query** and updates it directly in the input event. **useDeferredValue** produces **deferredQuery** for the results consumer. While that value is behind, the input can show the latest text and the app can keep the existing results visible until React completes the background render.

**CatalogResults** is wrapped with **memo**. That allows it to skip the urgent parent render when its **deferredQuery** prop is still the same value. Once the deferred value changes, React renders the matching result list. The data set is small so the example stays easy to inspect; a very quick render may make the stale interval hard to observe.

The **aria-busy** attribute is placed on a real DOM container around the results. The status uses **role="status"** and **aria-live="polite"** so assistive technology can announce the update without interrupting the user.

## Why the tabs use a Transition

**TransitionTabs** owns the setter for **selectedView**, so it can call **startTransition** around that state update. **useTransition** also provides **isPending** for status text and the panel’s busy state. The callback runs immediately; React assigns Transition priority to the state update scheduled inside it.

This panel is a small illustration of the API rather than a benchmark. On a larger real view, React can pause and restart render work when a more urgent update arrives. A single long synchronous calculation still cannot be interrupted in the middle.

## Key points

- Keep controlled input state urgent.
- Use **useDeferredValue** when a slower consumer should catch up to a value.
- Use **useTransition** when you control a non-urgent state setter and want **isPending**.
- Use a memoized slow child so it can skip an urgent parent render while its deferred prop is unchanged.
- Put accessibility status and **aria-busy** on meaningful DOM elements.
- Neither API adds a fixed delay, runs JavaScript on another thread, speeds up calculations, or reduces network requests by itself.
