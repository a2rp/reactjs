# React quick reference and glossary: examples

These examples accompany [the quick reference lesson](../lessons/120-react-quick-reference-and-glossary.md). They put the main React ideas into usable patterns. Start with the task-list app, then use the focused recipes as needed.

The examples use function components, JSX, and public React APIs. They assume a React project with a JSX-aware build tool such as Vite. The form action is a client-side demonstration; connect it to a real service before relying on it to store or protect data.

## 1. A small task-list app

This app brings together components, props, state, a reducer, controlled inputs, stable keys, derived values, accessible labels, and an Effect that synchronizes the browser tab title.

Create a project with Vite and install its dependencies:

~~~sh
npm create vite@latest react-quick-reference -- --template react
cd react-quick-reference
npm install
npm run dev
~~~

Replace the generated files with the following.

### src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./App.css";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

The HTML entry page needs a root element:

~~~html
<div id="root"></div>
~~~

### src/App.jsx

~~~jsx
import { useEffect, useId, useReducer, useState } from "react";

const APP_TITLE = "Tasks | React quick reference";

const startingTasks = [
  { id: "task-read", text: "Read the lesson", done: true },
  { id: "task-practice", text: "Try the examples", done: false },
];

function taskReducer(tasks, action) {
  switch (action.type) {
    case "added":
      return [...tasks, action.task];

    case "toggled":
      return tasks.map((task) =>
        task.id === action.id ? { ...task, done: !task.done } : task,
      );

    case "removed":
      return tasks.filter((task) => task.id !== action.id);

    case "completed-cleared":
      return tasks.filter((task) => !task.done);

    default:
      throw new Error("Unknown task action.");
  }
}

function TaskItem({ task, onToggle, onRemove }) {
  return (
    <li className={task.done ? "task task--done" : "task"}>
      <label className="task__label">
        <input
          checked={task.done}
          onChange={() => onToggle(task.id)}
          type="checkbox"
        />
        <span>{task.text}</span>
      </label>

      <button
        aria-label={"Remove " + task.text}
        className="button button--quiet"
        onClick={() => onRemove(task.id)}
        type="button"
      >
        Remove
      </button>
    </li>
  );
}

export default function App() {
  const [tasks, dispatch] = useReducer(taskReducer, startingTasks);
  const [draft, setDraft] = useState("");
  const [query, setQuery] = useState("");
  const [filter, setFilter] = useState("all");
  const inputId = useId();

  const remainingCount = tasks.filter((task) => !task.done).length;
  const normalizedQuery = query.trim().toLocaleLowerCase();

  // Derive these values during render instead of duplicating them in state.
  const visibleTasks = tasks.filter((task) => {
    const matchesFilter =
      filter === "all" ||
      (filter === "active" && !task.done) ||
      (filter === "completed" && task.done);
    const matchesQuery = task.text
      .toLocaleLowerCase()
      .includes(normalizedQuery);

    return matchesFilter && matchesQuery;
  });

  // document.title belongs to the browser, so synchronize it with an Effect.
  useEffect(() => {
    document.title =
      remainingCount === 0
        ? APP_TITLE
        : "(" + remainingCount + ") " + APP_TITLE;

    return () => {
      document.title = APP_TITLE;
    };
  }, [remainingCount]);

  function handleSubmit(event) {
    event.preventDefault();

    const text = draft.trim();
    if (!text) return;

    dispatch({
      type: "added",
      task: { id: crypto.randomUUID(), text, done: false },
    });
    setDraft("");
  }

  return (
    <main className="app">
      <header className="app__header">
        <p className="eyebrow">React quick reference</p>
        <h1>My tasks</h1>
        <p className="intro">
          A small example of state, events, lists, and accessible controls.
        </p>
      </header>

      <form className="task-form" onSubmit={handleSubmit}>
        <label className="field-label" htmlFor={inputId}>
          New task
        </label>
        <div className="task-form__row">
          <input
            autoComplete="off"
            id={inputId}
            onChange={(event) => setDraft(event.target.value)}
            placeholder="For example, review the glossary"
            value={draft}
          />
          <button className="button" type="submit">
            Add task
          </button>
        </div>
      </form>

      <section aria-label="Task controls" className="controls">
        <div className="search">
          <label className="field-label" htmlFor="task-search">
            Search tasks
          </label>
          <input
            id="task-search"
            onChange={(event) => setQuery(event.target.value)}
            type="search"
            value={query}
          />
        </div>

        <div aria-label="Show tasks" className="filters">
          {["all", "active", "completed"].map((option) => (
            <button
              aria-pressed={filter === option}
              className="filter-button"
              key={option}
              onClick={() => setFilter(option)}
              type="button"
            >
              {option[0].toUpperCase() + option.slice(1)}
            </button>
          ))}
        </div>
      </section>

      <p aria-live="polite" className="status" role="status">
        {remainingCount} {remainingCount === 1 ? "task" : "tasks"} remaining
      </p>

      {visibleTasks.length > 0 ? (
        <ul className="task-list">
          {visibleTasks.map((task) => (
            <TaskItem
              key={task.id}
              onRemove={(id) => dispatch({ type: "removed", id })}
              onToggle={(id) => dispatch({ type: "toggled", id })}
              task={task}
            />
          ))}
        </ul>
      ) : (
        <p className="empty-state">No tasks match these controls.</p>
      )}

      <button
        className="button button--quiet"
        disabled={!tasks.some((task) => task.done)}
        onClick={() => dispatch({ type: "completed-cleared" })}
        type="button"
      >
        Clear completed
      </button>
    </main>
  );
}
~~~

### src/App.css

~~~css
:root {
  color-scheme: light;
  font-family: system-ui, sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  color: #172033;
  background: #f4f6fa;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  min-width: 20rem;
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

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.app {
  width: min(100% - 2rem, 46rem);
  margin: 4rem auto;
  padding: clamp(1.25rem, 5vw, 2.5rem);
  border: 1px solid #dce2ec;
  border-radius: 1.25rem;
  background: #fff;
  box-shadow: 0 1rem 3rem rgb(25 41 70 / 8%);
}

.app__header {
  margin-bottom: 2rem;
}

.eyebrow {
  margin: 0 0 0.5rem;
  color: #4d5fbd;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1 {
  margin: 0;
  font-size: clamp(2rem, 7vw, 3rem);
}

.intro {
  max-width: 38rem;
  color: #566176;
  line-height: 1.6;
}

.field-label {
  display: block;
  margin-bottom: 0.45rem;
  font-weight: 650;
}

.task-form__row {
  display: flex;
  gap: 0.65rem;
}

.task-form input,
.search input {
  min-width: 0;
  width: 100%;
  padding: 0.7rem 0.8rem;
  border: 1px solid #aeb8c9;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
}

input:focus-visible,
button:focus-visible {
  outline: 3px solid #7084f5;
  outline-offset: 2px;
}

.button,
.filter-button {
  padding: 0.65rem 0.9rem;
  border: 1px solid #3448a5;
  border-radius: 0.55rem;
  color: #fff;
  background: #3448a5;
  font-weight: 650;
}

.button--quiet,
.filter-button {
  border-color: #d4dae5;
  color: #26324a;
  background: #fff;
}

.controls {
  display: flex;
  align-items: end;
  justify-content: space-between;
  gap: 1rem;
  margin-top: 1.75rem;
}

.search {
  flex: 1;
  min-width: 10rem;
}

.filters {
  display: flex;
  flex-wrap: wrap;
  gap: 0.35rem;
}

.filter-button[aria-pressed="true"] {
  border-color: #3448a5;
  color: #253989;
  background: #edf0ff;
}

.status {
  margin: 1.5rem 0 0.65rem;
  color: #566176;
  font-size: 0.95rem;
}

.task-list {
  display: grid;
  gap: 0.55rem;
  margin: 0 0 1rem;
  padding: 0;
  list-style: none;
}

.task {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.8rem;
  border: 1px solid #e0e5ed;
  border-radius: 0.65rem;
}

.task__label {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  overflow-wrap: anywhere;
}

.task__label input {
  width: 1.1rem;
  height: 1.1rem;
  accent-color: #3448a5;
}

.task--done .task__label span {
  color: #6b7485;
  text-decoration: line-through;
}

.empty-state {
  padding: 1rem;
  border: 1px dashed #aeb8c9;
  border-radius: 0.65rem;
  color: #566176;
  text-align: center;
}

@media (max-width: 38rem) {
  .app {
    margin: 1rem auto;
  }

  .task-form__row,
  .controls {
    align-items: stretch;
    flex-direction: column;
  }

  .filters {
    justify-content: flex-start;
  }
}
~~~

### What to notice

- **State** is the source of truth for values that change: the task list, draft, query, and selected filter.
- **The reducer** describes state transitions. Each event dispatches an action, and the reducer returns a new array without mutating the old one.
- **Derived values** such as the remaining count and filtered list are calculated during render. Duplicating them in state would create synchronization work.
- **Keys** use stable task IDs. Array indexes are unsafe keys when items can be inserted, removed, or reordered.
- **The Effect** synchronizes React with an external browser value, the document title. It does not calculate the filtered list or copy one state value into another.
- **StrictMode** helps expose impure rendering and missing cleanup during development. Development checks can run some functions more than once; event handlers should still represent one user action.

## 2. Update state from its previous value

When the next value depends on the previous value, use the updater form. React can queue these updates correctly.

~~~jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  function addThree() {
    setCount((current) => current + 1);
    setCount((current) => current + 1);
    setCount((current) => current + 1);
  }

  return (
    <button onClick={addThree} type="button">
      Count: {count}
    </button>
  );
}
~~~

For objects and arrays, create a new value instead of mutating the existing one:

~~~jsx
setProfile((current) => ({ ...current, name: "Ada" }));
setItems((current) => [...current, newItem]);
setItems((current) => current.filter((item) => item.id !== removedId));
~~~

## 3. Subscribe to an external value

For a value maintained outside React, useSyncExternalStore gives React a snapshot and a subscription with cleanup. This example subscribes to the browser's online and offline events.

~~~jsx
import { useSyncExternalStore } from "react";

function subscribeToOnlineStatus(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);

  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function getOnlineSnapshot() {
  return navigator.onLine;
}

function getServerOnlineSnapshot() {
  // Use the same server snapshot during server rendering and hydration.
  return true;
}

function useOnlineStatus() {
  return useSyncExternalStore(
    subscribeToOnlineStatus,
    getOnlineSnapshot,
    getServerOnlineSnapshot,
  );
}

function NetworkNotice() {
  const isOnline = useOnlineStatus();

  return (
    <p role="status">
      {isOnline ? "You are online." : "You are offline. Changes may not sync."}
    </p>
  );
}
~~~

For ordinary component state, prefer useState or useReducer. Use useSyncExternalStore when reading and subscribing to a store or browser API that exists independently of the component.

## 4. Submit a form with React form Hooks

useActionState connects a form action to a state result. useFormStatus reads the pending state from a component rendered inside that form.

~~~jsx
import { useActionState } from "react";
import { useFormStatus } from "react-dom";

async function submitFeedback(previousState, formData) {
  const message = String(formData.get("message") ?? "").trim();

  if (!message) {
    return { message: "Enter feedback before submitting.", ok: false };
  }

  // Replace this demonstration pause with a request to your service.
  await new Promise((resolve) => setTimeout(resolve, 300));

  return { message: "Feedback is ready to send.", ok: true };
}

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button disabled={pending} type="submit">
      {pending ? "Submitting..." : "Send feedback"}
    </button>
  );
}

export function FeedbackForm() {
  const [state, formAction] = useActionState(submitFeedback, {
    message: "",
    ok: false,
  });

  return (
    <form action={formAction}>
      <label htmlFor="feedback-message">Feedback</label>
      <textarea id="feedback-message" name="message" required />
      <SubmitButton />
      {state.message && (
        <p role={state.ok ? "status" : "alert"}>{state.message}</p>
      )}
    </form>
  );
}
~~~

This client-side demonstration does not store or send anything. A production action should send data to a trusted server, validate it there, handle failures, and never treat browser-side checks as a security boundary.

## 5. Keep urgent input responsive with a transition

A transition marks an update as non-urgent. The controlled input updates immediately while React can render the results at lower priority.

~~~jsx
import { useState, useTransition } from "react";

function SearchableList({ items }) {
  const [query, setQuery] = useState("");
  const [deferredQuery, setDeferredQuery] = useState("");
  const [isPending, startTransition] = useTransition();

  const results = items.filter((item) =>
    item.toLocaleLowerCase().includes(deferredQuery.toLocaleLowerCase()),
  );

  function handleChange(event) {
    const nextQuery = event.target.value;

    // Keep the controlled input update urgent.
    setQuery(nextQuery);

    // Let React render the result list as non-urgent work.
    startTransition(() => {
      setDeferredQuery(nextQuery);
    });
  }

  return (
    <section>
      <label htmlFor="large-list-search">Search</label>
      <input
        id="large-list-search"
        onChange={handleChange}
        value={query}
      />
      {isPending && <p role="status">Updating results...</p>}
      <ul aria-busy={isPending}>
        {results.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </section>
  );
}
~~~

Use useTransition when you need to mark a particular state update as non-urgent. useDeferredValue is another fit when a rendered value should lag behind an urgent value. Neither API removes expensive work; profile the UI and keep render calculations pure.

## 6. Load a component on demand

lazy defers loading a component module until it is first rendered. Suspense provides a fallback while that module is loading.

~~~jsx
import { lazy, Suspense, useState } from "react";

const ReportsPanel = lazy(() => import("./ReportsPanel.jsx"));

function Dashboard() {
  const [showReports, setShowReports] = useState(false);

  return (
    <main>
      <button onClick={() => setShowReports(true)} type="button">
        Open reports
      </button>

      {showReports && (
        <Suspense fallback={<p role="status">Loading reports...</p>}>
          <ReportsPanel />
        </Suspense>
      )}
    </main>
  );
}
~~~

Lazy loading is useful when code is not needed for the initial view. A Suspense boundary handles loading for resources that integrate with Suspense, such as lazy component code.

## 7. Catch rendering errors with an Error Boundary

Error Boundaries catch errors thrown while rendering descendant components and show fallback UI. The built-in boundary API is class-based.

~~~jsx
import { Component } from "react";

export class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Send diagnostic details to an approved error-reporting service.
    console.error("A child component failed to render.", error, info);
  }

  render() {
    if (this.state.hasError) {
      return (
        <section role="alert">
          <h2>This section could not be displayed.</h2>
          <p>Try again later or return to the previous page.</p>
        </section>
      );
    }

    return this.props.children;
  }
}
~~~

Place the boundary around a region that can fail independently:

~~~jsx
<ErrorBoundary>
  <ReportsPanel />
</ErrorBoundary>
~~~

An Error Boundary does not catch every kind of error. It does not replace input validation, request error handling, or try/catch for ordinary asynchronous work in event handlers.

## 8. Choose an API by the job

| Need | Start with |
| --- | --- |
| A local value changes after an event | useState |
| Related state changes follow named actions | useReducer |
| A value can be calculated from current props or state | Calculate it during render |
| React must synchronize with a browser API or external system | useEffect |
| Read from and subscribe to an external store | useSyncExternalStore |
| Preserve a value between renders without rendering on change | useRef |
| Cache a pure calculation after measuring a real bottleneck | useMemo |
| Cache a function identity when a consumer needs it | useCallback |
| Keep non-urgent rendering from blocking urgent input | useTransition or useDeferredValue |
| Defer component code until it is needed | lazy and Suspense |
| Display a fallback for descendant render errors | An Error Boundary |
| Coordinate work for a form action | useActionState, useFormStatus, or useOptimistic as appropriate |

Memoization Hooks are performance tools, not requirements for correctness. Start with clear code and measure before adding them.

## 9. Choose the correct DOM root API

For a client-rendered application, create one root and render into the HTML element:

~~~jsx
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The HTML page must contain an element with id=root.");
}

createRoot(container).render(<App />);
~~~

If HTML was already rendered on the server, hydrate that markup instead of replacing it with a fresh client render:

~~~jsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.jsx";

hydrateRoot(document.getElementById("root"), <App />);
~~~

The server and initial client render must agree for hydration. Follow the framework or server-rendering setup that produced the HTML; do not switch between createRoot and hydrateRoot for the same root.

## 10. Safety and correctness checklist

Before reusing an example in an application, check that:

- Every mapped item has a stable, unique key.
- State arrays and objects are updated immutably.
- Inputs that use value or checked also update those values in their change handlers.
- Buttons inside forms declare whether they submit with type=submit or act separately with type=button.
- Labels are associated with their controls; repeated component instances use useId when an ID is needed.
- Loading, empty, error, and success states are visible and understandable without color alone.
- User-provided text is rendered as text. Avoid passing untrusted content to dangerouslySetInnerHTML.
- Authorization and validation happen on the server for protected operations.
- Effects have the right dependencies and clean up subscriptions, timers, or other resources they create.
- Server-rendered markup is hydrated with the matching React DOM API.

## Related lessons

- [React, React DOM, and frameworks](../lessons/003-react-react-dom-and-frameworks.md)
- [Rendering collections and stable keys](../lessons/019-rendering-collections-and-stable-keys.md)
- [Event handlers and user interactions](../lessons/025-event-handlers-and-user-interactions.md)
- [Controlled form elements](../lessons/028-controlled-form-elements.md)
- [State with useState](../lessons/037-state-with-usestate.md)
- [Effect Hooks and synchronization](../lessons/052-effect-hooks-and-synchronization.md)
- [Transitions and useTransition](../lessons/064-transitions-and-usetransition.md)
- [Suspense boundaries and fallbacks](../lessons/066-suspense-boundaries-and-fallbacks.md)
- [Lazy loading components](../lessons/067-lazy-loading-components.md)
- [Error Boundaries and recovery UI](../lessons/068-error-boundaries-and-recovery-ui.md)
- [React DOM roots and client APIs](../lessons/070-react-dom-roots-and-client-apis.md)
- [React form actions](../lessons/078-react-form-actions.md)
- [React quick reference and glossary](../lessons/120-react-quick-reference-and-glossary.md)

The maintained React API documentation is at [react.dev/reference](https://react.dev/reference).
