# Example: Rendering, Committing, and Updating the UI

This runnable React app lets you observe the difference between React calling components and React changing the DOM:

- the entry file creates one React root and renders the app;
- a counter uses local state to request updates;
- an uncontrolled input keeps its typed value when unrelated UI changes;
- a conditional detail panel mounts and unmounts through state;
- React’s **Profiler** reports when the counter subtree commits an update;
- a three-step counter shows state snapshots and queued updater functions.

The app makes no external requests. Run it in development and open the browser console to see Profiler commit information. Strict Mode may repeat render-related work during development; the Profiler callback reports committed work, not each component function call.

## Create the project

~~~sh
npm create vite@latest render-commit-demo -- --template react
cd render-commit-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following and run:

~~~sh
npm run dev
~~~

## Project files

~~~text
render-commit-demo/
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
    <meta name="theme-color" content="#f4f6fb" />
    <title>React Render and Commit Demo</title>
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

const container = document.getElementById("root");
const root = createRoot(container);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
~~~

## src/App.jsx

~~~jsx
import { Profiler, useState } from "react";

function reportCommit(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  console.log("React Profiler commit", {
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime,
  });
}

function CounterPanel() {
  const [count, setCount] = useState(0);

  function addOne() {
    setCount(count + 1);
  }

  function addThreeUsingSnapshot() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  function addThreeUsingUpdaters() {
    setCount((current) => current + 1);
    setCount((current) => current + 1);
    setCount((current) => current + 1);
  }

  return (
    <section className="panel" aria-labelledby="counter-title">
      <p className="eyebrow">State update</p>
      <h2 id="counter-title">Counter: {count}</h2>
      <p className="panel-copy">
        Each click requests a render. React then commits the text change needed
        for this counter.
      </p>
      <div className="button-row">
        <button className="button button--primary" type="button" onClick={addOne}>
          Add one
        </button>
        <button className="button" type="button" onClick={addThreeUsingSnapshot}>
          Add three with the same snapshot
        </button>
        <button className="button" type="button" onClick={addThreeUsingUpdaters}>
          Add three with updater functions
        </button>
      </div>
      <p className="hint">
        The second button usually adds one: all three calls read this render’s
        count. The third queues three transformations and adds three.
      </p>
    </section>
  );
}

export default function App() {
  const [showDetails, setShowDetails] = useState(false);

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React DOM lifecycle</p>
        <h1>Render first, commit what changed</h1>
        <p className="intro">
          React calculates a UI result from the current state, compares it with
          the previous tree, and applies the DOM changes that are needed.
        </p>
      </header>

      <section className="phase-strip" aria-label="React update phases">
        <div className="phase-step">
          <span className="phase-number">1</span>
          <div><strong>Trigger</strong><span>Initial render or update request</span></div>
        </div>
        <div className="phase-arrow" aria-hidden="true">→</div>
        <div className="phase-step">
          <span className="phase-number">2</span>
          <div><strong>Render</strong><span>Components calculate the next UI</span></div>
        </div>
        <div className="phase-arrow" aria-hidden="true">→</div>
        <div className="phase-step">
          <span className="phase-number">3</span>
          <div><strong>Commit</strong><span>React applies needed DOM changes</span></div>
        </div>
        <div className="phase-arrow" aria-hidden="true">→</div>
        <div className="phase-step">
          <span className="phase-number">4</span>
          <div><strong>Paint</strong><span>The browser draws the page</span></div>
        </div>
      </section>

      <section className="panel" aria-labelledby="input-title">
        <p className="eyebrow">DOM reuse</p>
        <h2 id="input-title">Type, then update another part</h2>
        <label className="input-label" htmlFor="uncontrolled-note">
          Uncontrolled input
        </label>
        <input
          className="text-input"
          id="uncontrolled-note"
          defaultValue="This value lives in the DOM"
        />
        <p className="panel-copy">
          Type into this field, then toggle the detail panel below. The parent
          renders again, but React can reuse this same input node without
          replacing its user-entered value.
        </p>
      </section>

      <section className="panel" aria-labelledby="detail-title">
        <div className="panel-heading">
          <div>
            <p className="eyebrow">Parent state update</p>
            <h2 id="detail-title">Conditional content</h2>
          </div>
          <button
            className="button button--primary"
            type="button"
            aria-expanded={showDetails}
            aria-controls="detail-panel"
            onClick={() => setShowDetails((visible) => !visible)}
          >
            {showDetails ? "Hide details" : "Show details"}
          </button>
        </div>
        <div className="detail-panel" id="detail-panel" hidden={!showDetails}>
          <p>
            This section is present in the DOM only while its state is true.
            React adds or removes it during commit because the JSX describes
            that structure.
          </p>
        </div>
      </section>

      <Profiler id="CounterPanel" onRender={reportCommit}>
        <CounterPanel />
      </Profiler>

      <aside className="panel profiler-note" aria-labelledby="profiler-title">
        <p className="eyebrow">Profiler</p>
        <h2 id="profiler-title">Observe committed work</h2>
        <p className="panel-copy">
          Open the developer console and use the counter buttons. The Profiler
          callback reports the phase and timing for commits in this subtree. It
          does not log every component call or list exactly which DOM nodes
          changed.
        </p>
      </aside>

      <footer className="footer-note">
        This demo is client-rendered. Use hydration instead when the root already
        contains HTML produced by React on the server.
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
  color: #1c2738;
  background: #f4f6fb;
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
  cursor: pointer;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #91b7f5;
  outline-offset: 3px;
}

.page-shell {
  width: min(100% - 30px, 900px);
  margin: 0 auto;
  padding: 48px 0 40px;
}

.page-header {
  margin-bottom: 24px;
}

.eyebrow {
  margin-bottom: 7px;
  color: #61718a;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2,
p {
  overflow-wrap: anywhere;
}

h1 {
  max-width: 720px;
  font-size: clamp(2rem, 6vw, 3.2rem);
  line-height: 1.08;
  letter-spacing: -0.045em;
}

h2 {
  font-size: 1.22rem;
  line-height: 1.3;
}

.intro {
  max-width: 680px;
  margin-top: 12px;
  color: #58677e;
  font-size: 1rem;
  line-height: 1.6;
}

.phase-strip {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 16px;
  border: 1px solid #dce3ed;
  border-radius: 15px;
  background: #eaf1fc;
}

.phase-step {
  display: flex;
  min-width: 0;
  align-items: center;
  gap: 9px;
}

.phase-number {
  display: grid;
  width: 30px;
  height: 30px;
  flex: 0 0 auto;
  place-items: center;
  border-radius: 50%;
  color: #fff;
  background: #3265b8;
  font-size: 0.82rem;
  font-weight: 800;
}

.phase-step strong,
.phase-step span:not(.phase-number) {
  display: block;
}

.phase-step strong {
  font-size: 0.88rem;
}

.phase-step div span {
  margin-top: 3px;
  color: #5d6d85;
  font-size: 0.72rem;
  line-height: 1.35;
}

.phase-arrow {
  flex: 0 0 auto;
  color: #7d8ca3;
  font-size: 1.15rem;
}

.panel {
  margin-top: 16px;
  padding: 21px;
  border: 1px solid #dce3ed;
  border-radius: 15px;
  background: #fff;
  box-shadow: 0 8px 24px rgb(36 51 75 / 5%);
}

.panel-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 14px;
}

.panel-heading .eyebrow {
  margin-bottom: 5px;
}

.input-label {
  display: block;
  margin-top: 17px;
  margin-bottom: 7px;
  color: #344158;
  font-size: 0.9rem;
  font-weight: 750;
}

.text-input {
  width: 100%;
  min-height: 44px;
  padding: 10px 12px;
  border: 1px solid #c8d2e0;
  border-radius: 10px;
  color: #1c2738;
  background: #fff;
}

.panel-copy {
  margin-top: 12px;
  color: #5c6a80;
  font-size: 0.93rem;
  line-height: 1.55;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 9px;
  margin-top: 16px;
}

.button {
  min-height: 41px;
  padding: 9px 12px;
  border: 1px solid #c6d0de;
  border-radius: 9px;
  color: #28364d;
  background: #fff;
  font-size: 0.88rem;
  font-weight: 700;
}

.button:hover {
  border-color: #95a7c0;
  background: #f4f7fb;
}

.button--primary {
  border-color: #2b5eaf;
  color: #fff;
  background: #2b5eaf;
}

.button--primary:hover {
  border-color: #1f4c93;
  background: #1f4c93;
}

.hint {
  margin-top: 13px;
  color: #63718a;
  font-size: 0.87rem;
  line-height: 1.5;
}

.detail-panel {
  margin-top: 17px;
  padding: 14px 15px;
  border-left: 4px solid #5380d0;
  border-radius: 8px;
  color: #394961;
  background: #edf4ff;
  line-height: 1.55;
}

.detail-panel[hidden] {
  display: none;
}

.profiler-note {
  border-style: dashed;
  background: #fbfcff;
}

.profiler-note .eyebrow {
  margin-bottom: 5px;
}

.footer-note {
  margin-top: 18px;
  padding: 0 3px;
  color: #64728a;
  font-size: 0.86rem;
  line-height: 1.5;
}

@media (max-width: 780px) {
  .phase-strip {
    align-items: stretch;
    flex-direction: column;
  }

  .phase-arrow {
    padding-left: 8px;
    transform: rotate(90deg);
    transform-origin: left center;
  }
}

@media (max-width: 560px) {
  .page-shell {
    width: min(100% - 22px, 900px);
    padding-top: 30px;
  }

  .panel {
    padding: 17px;
  }

  .button-row {
    align-items: stretch;
    flex-direction: column;
  }

  .button-row .button {
    width: 100%;
  }
}
~~~

## How to observe the phases

1. When the app starts, **createRoot** and **root.render** request the initial client render. React calls the app components, commits the initial DOM, and the browser paints it.
2. Type into the uncontrolled field. The browser changes that input’s value. This edit does not update React state.
3. Select **Show details**. The event handler updates **showDetails**. React renders the app again, then commits the conditional panel. The same input remains in the same position with the same type, so React can reuse its DOM node and keep the typed value.
4. Press a counter button. The counter’s local state requests an update. The **Profiler** callback logs a committed update for that subtree in the developer console.
5. Compare the two “add three” buttons. The first uses the same state snapshot three times and usually adds one. The updater-function version queues three transformations and adds three.

## What Profiler logs mean

The **Profiler** callback runs when the wrapped subtree commits. Its **phase** is usually **mount** for the initial mount and **update** for later updates. The duration fields describe React rendering work for that subtree; they do not list changed DOM nodes or measure browser paint by themselves.

Profiling adds overhead and is disabled in the regular production build by default. Use the React Developer Tools Profiler for interactive inspection, and a profiling-enabled production build when production measurements are needed.

Strict Mode may invoke render-related code extra times in development to expose impure components. Do not infer an exact render count from console output. The Profiler callback is a commit observation, not a log of every component function call.

## Client rendering versus server hydration

This project is client-rendered, so **main.jsx** uses **createRoot**. If the root already contains HTML produced by React on the server, hydrate that markup instead:

~~~jsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

hydrateRoot(document.getElementById("root"), <App />);
~~~

The initial client output must match the server-rendered tree. Do not call **createRoot** over React-generated server markup because it clears the container and recreates the content.

## Try these changes

1. Change the uncontrolled input to a controlled input with **value** and **onChange** state. Compare how React now supplies its value on every render.
2. Add a new paragraph to **CounterPanel** and update the counter. Observe that a component render does not mean every node in the panel is replaced.
3. Change the key on a child component intentionally and observe that React treats the changed key as a different component identity.
4. Move the detail panel into a child component and keep its open state in the parent. Describe which component’s state setter triggers the update.
5. Temporarily add a render-time mutation to a module-level variable, observe Strict Mode’s development behavior, then remove the impurity.
6. Use the React Developer Tools Profiler to inspect the component tree while toggling details and updating the counter.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="060-memoization-useid-usememo-usecallback.md">060 - Example: Memoization and useId</a></td><td><a href="../lessons/061-rendering-and-committing.md">Lesson</a></td><td><a href="../qnas/061-rendering-and-committing.md">Q&amp;A</a></td><td><a href="062-reconciliation-and-component-identity.md">062 - Example: Reconciliation, Component Identity, and Keys</a></td></tr>
  </tbody>
</table>
