# Example: Following the Rules of Hooks

This runnable Vite example puts the Rules of Hooks into practice:

- a custom Hook subscribes to browser online/offline events;
- the Hook is called unconditionally, while its Effect checks whether monitoring is enabled;
- each list row owns its own stateful input and is rendered as a keyed component;
- a conditional child component demonstrates where to place Hooks when an interface section is optional;
- event handlers call state setters, not Hooks.

The app performs no network requests. Its data and notes are held only in local React state.

## Project files

~~~text
rules-of-hooks-demo/
├── index.html
└── src/
    ├── App.jsx
    ├── main.jsx
    └── styles.css
~~~

Use these files in an existing Vite React project. From the project root, install dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

## src/App.jsx

~~~jsx
import { useEffect, useState } from "react";

const initialTasks = [
  { id: "task-state", title: "Review state updates" },
  { id: "task-effects", title: "Check Effect cleanup" },
  { id: "task-components", title: "Render child components" },
];

function useNetworkStatus(isEnabled) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    if (!isEnabled) {
      return;
    }

    function updateStatus() {
      setIsOnline(navigator.onLine);
    }

    updateStatus();
    window.addEventListener("online", updateStatus);
    window.addEventListener("offline", updateStatus);

    return () => {
      window.removeEventListener("online", updateStatus);
      window.removeEventListener("offline", updateStatus);
    };
  }, [isEnabled]);

  if (!isEnabled) {
    return "paused";
  }

  if (isOnline === null) {
    return "checking";
  }

  return isOnline ? "online" : "offline";
}

function TaskRow({ task }) {
  const [note, setNote] = useState("");
  const [isEditing, setIsEditing] = useState(false);

  return (
    <li className="task-card">
      <div className="task-heading">
        <div>
          <h3>{task.title}</h3>
          <p className="small-note">Task ID: {task.id}</p>
        </div>
        <button
          className="secondary-button"
          type="button"
          aria-expanded={isEditing}
          onClick={() => setIsEditing((current) => !current)}
        >
          {isEditing ? "Close note" : "Edit note"}
        </button>
      </div>

      {isEditing && (
        <label className="field">
          Note for {task.title}
          <input
            value={note}
            onChange={(event) => setNote(event.target.value)}
            placeholder="Write a local note"
          />
        </label>
      )}

      <p className="small-note">
        {note ? "Note saved in this row's local state." : "No note yet."}
      </p>
    </li>
  );
}

function OptionalNotePanel() {
  const [note, setNote] = useState("");

  return (
    <div className="optional-panel">
      <label className="field">
        Temporary panel note
        <input
          value={note}
          onChange={(event) => setNote(event.target.value)}
          placeholder="Close the panel to discard this note"
        />
      </label>
      <p className="small-note">
        This component owns the note: {note || "(empty)"}
      </p>
    </div>
  );
}

export default function App() {
  const [isMonitoring, setIsMonitoring] = useState(true);
  const [tasks, setTasks] = useState(initialTasks);
  const [showOptionalPanel, setShowOptionalPanel] = useState(true);
  const networkStatus = useNetworkStatus(isMonitoring);

  function reverseTasks() {
    setTasks((currentTasks) => [...currentTasks].reverse());
  }

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React Hooks practice</p>
        <h1>Call Hooks where React can track them</h1>
        <p className="intro">
          Use the controls to see custom Hook behavior, per-row state, and
          conditional component mounting.
        </p>
      </header>

      <section className="demo-card" aria-labelledby="network-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Custom Hook and Effect</p>
            <h2 id="network-heading">Network status</h2>
          </div>
          <span className={"status status-" + networkStatus}>
            {networkStatus}
          </span>
        </div>

        <p className="muted">
          The useNetworkStatus Hook is called on every App render. Its Effect
          subscribes only while monitoring is enabled.
        </p>

        <button
          className="primary-button"
          type="button"
          aria-pressed={isMonitoring}
          onClick={() => setIsMonitoring((current) => !current)}
        >
          {isMonitoring ? "Pause monitoring" : "Resume monitoring"}
        </button>
      </section>

      <section className="demo-card" aria-labelledby="tasks-heading">
        <div className="section-heading section-heading-row">
          <div>
            <p className="eyebrow">One component per item</p>
            <h2 id="tasks-heading">Notes stay with keyed task rows</h2>
          </div>
          <button className="secondary-button" type="button" onClick={reverseTasks}>
            Reverse tasks
          </button>
        </div>

        <p className="muted">
          Edit notes on multiple rows, then reverse the list. Each TaskRow
          calls its Hooks at its own top level and has a stable task ID key.
        </p>

        <ul className="task-list">
          {tasks.map((task) => (
            <TaskRow key={task.id} task={task} />
          ))}
        </ul>

        <p className="explanation">
          Hooks are not called inside map. The map renders a TaskRow component;
          React renders each row, and that row calls its own Hooks.
        </p>
      </section>

      <section className="demo-card" aria-labelledby="optional-heading">
        <div className="section-heading section-heading-row">
          <div>
            <p className="eyebrow">Conditional UI</p>
            <h2 id="optional-heading">Put optional Hooks in a child component</h2>
          </div>
          <button
            className="secondary-button"
            type="button"
            aria-pressed={showOptionalPanel}
            onClick={() => setShowOptionalPanel((current) => !current)}
          >
            {showOptionalPanel ? "Unmount panel" : "Mount panel"}
          </button>
        </div>

        {showOptionalPanel ? (
          <OptionalNotePanel />
        ) : (
          <p className="empty-state">
            OptionalNotePanel is not currently rendered.
          </p>
        )}

        <p className="explanation">
          The condition chooses whether React renders the child. The child
          calls useState at its top level. Removing the child unmounts it, so
          mounting it again starts with a fresh note.
        </p>
      </section>

      <section className="demo-card" aria-labelledby="setter-heading">
        <p className="eyebrow">Event handlers</p>
        <h2 id="setter-heading">Setters belong in event handlers</h2>
        <p className="muted">
          The buttons above call state setters from their handlers. The Hooks
          that declare that state remain at the top level of App or a child.
        </p>
      </section>

      <footer className="page-footer">
        <p>
          Browser online/offline events may not be available in every preview
          environment. The Hook still demonstrates setup, cleanup, and a
          condition inside an Effect.
        </p>
      </footer>
    </main>
  );
}
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
  </StrictMode>
);
~~~

## src/styles.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  color: #202a3a;
  background: #f3f6fb;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
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

button {
  cursor: pointer;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #86aaff;
  outline-offset: 3px;
}

.page {
  width: min(100% - 2rem, 880px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  max-width: 68ch;
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin: 0 0 0.4rem;
  color: #315db5;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

h1,
h2,
h3,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 6vw, 3.4rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0.35rem;
  font-size: clamp(1.2rem, 3vw, 1.6rem);
  letter-spacing: -0.025em;
}

h3 {
  margin-bottom: 0.2rem;
  font-size: 1rem;
}

.intro,
.muted,
.explanation,
.page-footer {
  color: #566477;
  line-height: 1.65;
}

.intro {
  margin-bottom: 0;
  font-size: 1.05rem;
}

.demo-card {
  margin-top: 1rem;
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d9e1ec;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 10px 28px rgb(32 53 84 / 5%);
}

.section-heading,
.section-heading-row,
.task-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
}

.section-heading {
  margin-bottom: 0.8rem;
}

.section-heading-row {
  flex-wrap: wrap;
}

.muted {
  margin-bottom: 0.9rem;
}

.status {
  padding: 0.35rem 0.65rem;
  border-radius: 999px;
  color: #344257;
  background: #eaf0f8;
  font-size: 0.8rem;
  font-weight: 750;
  text-transform: capitalize;
}

.status-online {
  color: #17653c;
  background: #e4f6ec;
}

.status-offline {
  color: #963b32;
  background: #fff0ed;
}

.status-paused,
.status-checking {
  color: #5e4c13;
  background: #fff5d7;
}

.primary-button,
.secondary-button {
  min-height: 2.6rem;
  padding: 0.6rem 0.85rem;
  border-radius: 0.55rem;
}

.primary-button {
  border: 1px solid #315db5;
  color: #fff;
  background: #315db5;
  font-weight: 750;
}

.primary-button:hover {
  background: #244b9a;
}

.secondary-button {
  border: 1px solid #bac8da;
  color: #294775;
  background: #f6f8fc;
}

.secondary-button:hover {
  background: #eaf0f8;
}

.task-list {
  display: grid;
  gap: 0.7rem;
  margin: 1rem 0;
  padding: 0;
  list-style: none;
}

.task-card,
.optional-panel {
  padding: 0.9rem;
  border: 1px solid #d9e1ec;
  border-radius: 0.7rem;
  background: #f9fbfe;
}

.task-heading {
  align-items: flex-start;
}

.small-note {
  margin: 0.5rem 0 0;
  color: #66758a;
  font-size: 0.86rem;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.field {
  display: grid;
  gap: 0.45rem;
  color: #29374a;
  font-size: 0.93rem;
  font-weight: 700;
}

.field input {
  width: 100%;
  min-height: 2.7rem;
  padding: 0.65rem 0.75rem;
  border: 1px solid #bac8da;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
  font-weight: 450;
}

.task-card .field {
  margin-top: 0.9rem;
}

.optional-panel {
  margin-top: 0.9rem;
}

.optional-panel .small-note {
  margin-top: 0.7rem;
}

.explanation {
  margin: 1rem 0 0;
  padding-top: 0.85rem;
  border-top: 1px solid #e4eaf2;
  font-size: 0.9rem;
}

.empty-state {
  margin: 0;
  padding: 1rem;
  border: 1px dashed #bac8da;
  border-radius: 0.65rem;
  color: #566477;
  background: #f9fbfe;
}

.page-footer {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 0.88rem;
}

.page-footer p {
  margin-bottom: 0;
}

@media (max-width: 560px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .section-heading-row > button {
    width: 100%;
  }

  .task-heading {
    align-items: flex-start;
    flex-direction: column;
  }
}
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="An interactive React example demonstrating the Rules of Hooks."
    />
    <title>React Rules of Hooks Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How to explore the example

### Pause and resume the custom Hook

1. Check the network status shown by the first card.
2. Pause monitoring, then resume it.
3. Observe that App calls useNetworkStatus on every render. The Hook always calls useState and useEffect at its top level.
4. When monitoring is disabled, the Effect callback returns before subscribing. When enabled, it attaches listeners and returns cleanup to remove them.

The event handlers inside the Hook update state with a setter. They do not call Hooks.

### Reorder the rows

1. Open notes on multiple tasks and type different text.
2. Reverse the tasks.
3. Each note stays with the task because TaskRow is rendered with the stable task ID key.

The map callback returns TaskRow elements. It does not call useState itself. React renders each TaskRow as a component, and each row makes its Hook calls at the top level.

### Unmount and mount the optional panel

1. Type text in the temporary panel note.
2. Select Unmount panel.
3. Mount the panel again.
4. The note is empty because OptionalNotePanel was removed from the tree and created again.

The condition controls whether a component is rendered. It does not conditionally call useState inside App.

## Invalid patterns to recognize

Do not copy these patterns into application code:

~~~jsx
// Invalid: a Hook call depends on a condition.
if (isEnabled) {
  const [value, setValue] = useState("");
}

// Invalid: a Hook is called from an event handler.
function handleClick() {
  const theme = useContext(ThemeContext);
}

// Invalid: a Hook call count depends on the array length.
items.map((item) => useState(item.selected));
~~~

The corresponding structures are:

- declare state unconditionally, then use a condition to choose what to render;
- call state setters from event handlers, while keeping the Hook declaration at the component top level;
- render a child component per item and put that item's Hooks inside the child component.

React's special use API has a limited conditional and loop exception. It is different from ordinary Hooks such as useState, useContext, and useEffect; this example uses only ordinary Hooks.

## What to notice

- The custom Hook shares network-listener logic, not one shared state value between all callers.
- Hook declarations stay at each component or custom Hook's top level.
- Conditions may live inside an Effect callback.
- Stateful repeated UI is rendered as child components with stable keys.
- Event handlers update declared state through setters.
- Mounting and unmounting a child affects that child's local state without changing the parent's Hook call order.
