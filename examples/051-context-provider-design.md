# Example: Designing a Scoped Context Provider

This runnable Vite example combines a reducer with two contexts for a small task workspace:

- the provider owns the task list and reducer transitions;
- one context provides task state;
- another provides the stable dispatch function;
- custom Hooks hide context access and detect a missing provider;
- a local add-task input stays local instead of being moved into shared context;
- the provider wraps only the feature subtree that needs the task data.

The example changes only local React state. It does not save tasks or send requests.

## Project files

~~~text
scoped-task-context/
├── index.html
└── src/
    ├── App.jsx
    ├── main.jsx
    ├── styles.css
    └── contexts/
        └── tasks-context.jsx
~~~

Use these files in an existing Vite React project. From the project root, install dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

## src/contexts/tasks-context.jsx

~~~jsx
import { createContext, useContext, useReducer } from "react";

const initialTasks = [
  {
    id: "task-1",
    text: "Choose the feature state owner",
    done: true,
  },
  {
    id: "task-2",
    text: "Keep input state close to its form",
    done: false,
  },
  {
    id: "task-3",
    text: "Use actions for shared task updates",
    done: false,
  },
];

const TasksStateContext = createContext(undefined);
const TasksDispatchContext = createContext(undefined);

function tasksReducer(tasks, action) {
  switch (action.type) {
    case "task/added":
      return [...tasks, action.task];

    case "task/toggled":
      return tasks.map((task) =>
        task.id === action.id
          ? { ...task, done: !task.done }
          : task
      );

    case "task/deleted":
      return tasks.filter((task) => task.id !== action.id);

    default:
      throw new Error("Unknown task action: " + action.type);
  }
}

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksStateContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksStateContext.Provider>
  );
}

export function useTasks() {
  const tasks = useContext(TasksStateContext);

  if (tasks === undefined) {
    throw new Error("useTasks must be used inside TasksProvider.");
  }

  return tasks;
}

export function useTasksDispatch() {
  const dispatch = useContext(TasksDispatchContext);

  if (dispatch === undefined) {
    throw new Error("useTasksDispatch must be used inside TasksProvider.");
  }

  return dispatch;
}
~~~

The contexts are created once at module scope. TasksProvider owns the reducer state and supplies state and dispatch separately. The custom Hooks keep consumers from importing the raw context objects and give a useful error if the provider is missing.

## src/App.jsx

~~~jsx
import { useState } from "react";
import {
  TasksProvider,
  useTasks,
  useTasksDispatch,
} from "./contexts/tasks-context.jsx";

function AddTaskForm() {
  const [text, setText] = useState("");
  const dispatch = useTasksDispatch();

  function handleSubmit(event) {
    event.preventDefault();

    const trimmedText = text.trim();
    if (!trimmedText) {
      return;
    }

    dispatch({
      type: "task/added",
      task: {
        id: crypto.randomUUID(),
        text: trimmedText,
        done: false,
      },
    });

    setText("");
  }

  return (
    <form className="add-form" onSubmit={handleSubmit}>
      <label className="field">
        New task
        <input
          value={text}
          onChange={(event) => setText(event.target.value)}
          placeholder="Add a task to the shared list"
        />
      </label>
      <button className="primary-button" type="submit">
        Add task
      </button>
    </form>
  );
}

function TaskSummary() {
  const tasks = useTasks();
  const completedCount = tasks.filter((task) => task.done).length;

  return (
    <p className="summary">
      {completedCount} of {tasks.length} tasks complete
    </p>
  );
}

function TaskItem({ task }) {
  const dispatch = useTasksDispatch();

  return (
    <li className="task-row">
      <label className="task-label">
        <input
          type="checkbox"
          checked={task.done}
          onChange={() =>
            dispatch({
              type: "task/toggled",
              id: task.id,
            })
          }
        />
        <span className={task.done ? "task-text task-done" : "task-text"}>
          {task.text}
        </span>
      </label>

      <button
        className="delete-button"
        type="button"
        aria-label={"Delete task: " + task.text}
        onClick={() =>
          dispatch({
            type: "task/deleted",
            id: task.id,
          })
        }
      >
        Delete
      </button>
    </li>
  );
}

function TaskList() {
  const tasks = useTasks();

  if (tasks.length === 0) {
    return <p className="empty-state">No tasks yet. Add one above.</p>;
  }

  return (
    <ul className="task-list">
      {tasks.map((task) => (
        <TaskItem key={task.id} task={task} />
      ))}
    </ul>
  );
}

function TaskWorkspace() {
  return (
    <section className="workspace" aria-labelledby="workspace-heading">
      <div className="workspace-heading">
        <div>
          <p className="eyebrow">Shared feature state</p>
          <h2 id="workspace-heading">Team task list</h2>
        </div>
        <TaskSummary />
      </div>

      <AddTaskForm />
      <TaskList />

      <p className="explanation">
        The form owns its draft text locally. The task list and its actions are
        shared through the provider for this workspace.
      </p>
    </section>
  );
}

export default function App() {
  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">Provider design</p>
        <h1>Scope shared state to its feature</h1>
        <p className="intro">
          Add, complete, and delete tasks. The workspace shares task state
          without passing it through each intermediate component.
        </p>
      </header>

      <TasksProvider>
        <TaskWorkspace />
      </TasksProvider>

      <footer className="page-footer">
        <p>
          Refreshing the page restores the initial example tasks and removes
          tasks added during this session.
        </p>
      </footer>
    </main>
  );
}
~~~

The ID is created inside the submit event, not during rendering, and is stored with the new task. That stable ID is then used as the row key.

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
  width: min(100% - 2rem, 820px);
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
  font-size: clamp(2rem, 6vw, 3.3rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0.35rem;
  font-size: clamp(1.25rem, 3vw, 1.7rem);
  letter-spacing: -0.025em;
}

.intro,
.explanation,
.page-footer {
  color: #566477;
  line-height: 1.65;
}

.intro {
  margin-bottom: 0;
  font-size: 1.05rem;
}

.workspace {
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d7e0ec;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 32px rgb(32 53 84 / 6%);
}

.workspace-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.2rem;
}

.workspace-heading .eyebrow {
  margin-bottom: 0.35rem;
}

.summary {
  flex: 0 0 auto;
  margin: 0.25rem 0 0;
  padding: 0.4rem 0.65rem;
  border-radius: 999px;
  color: #294775;
  background: #eaf0f8;
  font-size: 0.84rem;
  font-weight: 700;
}

.add-form {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  align-items: end;
  gap: 0.75rem;
  padding-bottom: 1.2rem;
  border-bottom: 1px solid #e4eaf2;
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

.primary-button,
.delete-button {
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

.task-list {
  display: grid;
  gap: 0.55rem;
  margin: 1rem 0 0;
  padding: 0;
  list-style: none;
}

.task-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
  padding: 0.7rem 0.75rem;
  border: 1px solid #e0e6ef;
  border-radius: 0.6rem;
  background: #fbfcfe;
}

.task-label {
  display: flex;
  align-items: flex-start;
  gap: 0.65rem;
  min-width: 0;
  line-height: 1.5;
}

.task-label input {
  flex: 0 0 auto;
  width: 1rem;
  height: 1rem;
  margin-top: 0.2rem;
  accent-color: #315db5;
}

.task-text {
  overflow-wrap: anywhere;
}

.task-done {
  color: #69778a;
  text-decoration: line-through;
}

.delete-button {
  flex: 0 0 auto;
  min-height: 2.25rem;
  border: 1px solid #e2b5b0;
  color: #8c3029;
  background: #fff8f7;
  font-size: 0.85rem;
  font-weight: 650;
}

.delete-button:hover {
  background: #ffebe8;
}

.empty-state {
  margin: 1rem 0 0;
  padding: 1rem;
  border: 1px dashed #bac8da;
  border-radius: 0.65rem;
  color: #566477;
  text-align: center;
}

.explanation {
  margin: 1rem 0 0;
  padding-top: 0.85rem;
  border-top: 1px solid #e4eaf2;
  font-size: 0.9rem;
}

.page-footer {
  margin-top: 1.25rem;
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

  .workspace-heading {
    flex-direction: column;
  }

  .add-form {
    grid-template-columns: 1fr;
  }

  .primary-button {
    width: 100%;
  }

  .task-row {
    align-items: flex-start;
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
      content="A React example of scoped context providers with reducer-managed task state."
    />
    <title>Scoped React Context Provider Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How to explore the example

### Add a task

1. Enter a task in the form.
2. Submit it.
3. AddTaskForm clears its own local text state.
4. The reducer receives a task/added action and returns a new task array.
5. TaskSummary and TaskList read the changed state through useTasks.

The input text stays local to AddTaskForm until it is submitted. It does not need to be placed in the shared task context.

### Change or remove a task

Use the checkbox or Delete button. TaskItem reads only the dispatch context and sends an action. The reducer owns the update rules and returns new arrays and task objects without mutating the existing state.

### Inspect the provider boundary

TasksProvider wraps TaskWorkspace. The page header and footer are outside that provider, so they do not depend on task context.

TaskList and TaskSummary read the state context. AddTaskForm and TaskItem read the dispatch context. This separates their context dependencies; it does not mean those components can never render for other React reasons.

### Try the missing-provider guard

If a component calling useTasks or useTasksDispatch is moved outside TasksProvider, the custom Hook throws a clear error. This is intentional: task consumers require that provider.

## What to notice

- The provider's location expresses the feature's sharing scope.
- The reducer owns task transitions; context distributes the state and dispatch function.
- Separate contexts distinguish state readers from action dispatchers.
- Each consumer imports a feature-specific Hook instead of the raw context.
- The form's temporary draft remains local.
- A context provider is not needed for every component or every prop.
