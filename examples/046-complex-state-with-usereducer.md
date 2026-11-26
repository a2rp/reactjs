# Complex State with useReducer - Example

This example is a small task manager. It uses one reducer to handle related task-list transitions:

- edit the draft;
- add a task;
- toggle completion;
- start, save, or cancel an edit;
- delete a task;
- clear completed tasks;
- choose a list filter.

The list shown on screen and its counts are derived from the reducer state. They are not kept as duplicate state.

## Project files

In a Vite React project, use these files:

~~~text
index.html
src/
  App.jsx
  main.jsx
  styles.css
~~~

## src/App.jsx

~~~jsx
import { useReducer } from "react";

const initialState = {
  draft: "",
  tasks: [
    { id: "task-1", title: "Write descriptive action types", done: true },
    { id: "task-2", title: "Return new state from the reducer", done: false },
    { id: "task-3", title: "Derive the visible task list", done: false },
  ],
  filter: "all",
  editingId: null,
  editDraft: "",
};

function taskReducer(state, action) {
  switch (action.type) {
    case "task/draftChanged":
      return {
        ...state,
        draft: action.value,
      };

    case "task/added":
      return {
        ...state,
        draft: "",
        tasks: [...state.tasks, action.task],
      };

    case "task/toggled":
      return {
        ...state,
        tasks: state.tasks.map((task) =>
          task.id === action.taskId
            ? { ...task, done: !task.done }
            : task
        ),
      };

    case "task/deleted": {
      const deletesEditingTask = state.editingId === action.taskId;

      return {
        ...state,
        tasks: state.tasks.filter((task) => task.id !== action.taskId),
        editingId: deletesEditingTask ? null : state.editingId,
        editDraft: deletesEditingTask ? "" : state.editDraft,
      };
    }

    case "task/editStarted": {
      const task = state.tasks.find((item) => item.id === action.taskId);

      if (!task) {
        return state;
      }

      return {
        ...state,
        editingId: task.id,
        editDraft: task.title,
      };
    }

    case "task/editDraftChanged":
      return {
        ...state,
        editDraft: action.value,
      };

    case "task/editSaved": {
      const nextTitle = action.title.trim();

      if (!nextTitle) {
        return state;
      }

      return {
        ...state,
        tasks: state.tasks.map((task) =>
          task.id === action.taskId
            ? { ...task, title: nextTitle }
            : task
        ),
        editingId: null,
        editDraft: "",
      };
    }

    case "task/editCancelled":
      return {
        ...state,
        editingId: null,
        editDraft: "",
      };

    case "tasks/completedCleared": {
      const remainingTasks = state.tasks.filter((task) => !task.done);
      const editingTaskRemains = remainingTasks.some(
        (task) => task.id === state.editingId
      );

      return {
        ...state,
        tasks: remainingTasks,
        editingId: editingTaskRemains ? state.editingId : null,
        editDraft: editingTaskRemains ? state.editDraft : "",
      };
    }

    case "filter/changed":
      return {
        ...state,
        filter: action.filter,
      };

    default:
      throw Error("Unknown action: " + action.type);
  }
}

const filters = [
  { value: "all", label: "All" },
  { value: "active", label: "Active" },
  { value: "completed", label: "Completed" },
];

function TaskItem({ task, state, dispatch }) {
  const isEditing = state.editingId === task.id;
  const editInputId = "edit-task-" + task.id;

  function handleEditSubmit(event) {
    event.preventDefault();

    dispatch({
      type: "task/editSaved",
      taskId: task.id,
      title: state.editDraft,
    });
  }

  if (isEditing) {
    return (
      <li className="task-item">
        <form className="edit-form" onSubmit={handleEditSubmit}>
          <label className="visually-hidden" htmlFor={editInputId}>
            Edit task title
          </label>
          <input
            id={editInputId}
            value={state.editDraft}
            onChange={(event) =>
              dispatch({
                type: "task/editDraftChanged",
                value: event.target.value,
              })
            }
            required
          />
          <div className="task-actions">
            <button className="small-button primary-button" type="submit">
              Save
            </button>
            <button
              className="small-button secondary-button"
              type="button"
              onClick={() => dispatch({ type: "task/editCancelled" })}
            >
              Cancel
            </button>
          </div>
        </form>
      </li>
    );
  }

  return (
    <li className={task.done ? "task-item task-item-done" : "task-item"}>
      <label className="task-label">
        <input
          type="checkbox"
          checked={task.done}
          onChange={() =>
            dispatch({ type: "task/toggled", taskId: task.id })
          }
        />
        <span>{task.title}</span>
      </label>

      <div className="task-actions">
        <button
          className="small-button secondary-button"
          type="button"
          onClick={() =>
            dispatch({ type: "task/editStarted", taskId: task.id })
          }
        >
          Edit
        </button>
        <button
          className="small-button danger-button"
          type="button"
          aria-label={"Delete " + task.title}
          onClick={() =>
            dispatch({ type: "task/deleted", taskId: task.id })
          }
        >
          Delete
        </button>
      </div>
    </li>
  );
}

export default function App() {
  const [state, dispatch] = useReducer(taskReducer, initialState);

  const completedCount = state.tasks.filter((task) => task.done).length;
  const activeCount = state.tasks.length - completedCount;

  const visibleTasks = state.tasks.filter((task) => {
    if (state.filter === "active") {
      return !task.done;
    }

    if (state.filter === "completed") {
      return task.done;
    }

    return true;
  });

  function handleAddTask(event) {
    event.preventDefault();

    const title = state.draft.trim();

    if (!title) {
      return;
    }

    dispatch({
      type: "task/added",
      task: {
        id: crypto.randomUUID(),
        title,
        done: false,
      },
    });
  }

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React state transitions</p>
        <h1>Task manager with useReducer</h1>
        <p className="intro">
          Event handlers dispatch actions. One reducer defines how each action
          changes the task state.
        </p>
      </header>

      <section className="task-app" aria-labelledby="task-app-heading">
        <div className="task-app-header">
          <div>
            <h2 id="task-app-heading">My tasks</h2>
            <p className="task-count" aria-live="polite">
              {activeCount} {activeCount === 1 ? "task" : "tasks"} remaining
            </p>
          </div>

          <button
            className="small-button secondary-button"
            type="button"
            disabled={completedCount === 0}
            onClick={() => dispatch({ type: "tasks/completedCleared" })}
          >
            Clear completed ({completedCount})
          </button>
        </div>

        <form className="add-form" onSubmit={handleAddTask}>
          <label className="visually-hidden" htmlFor="new-task">
            New task
          </label>
          <input
            id="new-task"
            value={state.draft}
            onChange={(event) =>
              dispatch({
                type: "task/draftChanged",
                value: event.target.value,
              })
            }
            placeholder="Write a task and press Add"
            required
          />
          <button className="primary-button" type="submit">
            Add task
          </button>
        </form>

        <div className="filter-row" role="group" aria-label="Filter tasks">
          {filters.map((filter) => (
            <button
              key={filter.value}
              className={
                state.filter === filter.value
                  ? "filter-button filter-button-active"
                  : "filter-button"
              }
              type="button"
              aria-pressed={state.filter === filter.value}
              onClick={() =>
                dispatch({
                  type: "filter/changed",
                  filter: filter.value,
                })
              }
            >
              {filter.label}
            </button>
          ))}
        </div>

        {visibleTasks.length > 0 ? (
          <ul className="task-list">
            {visibleTasks.map((task) => (
              <TaskItem
                key={task.id}
                task={task}
                state={state}
                dispatch={dispatch}
              />
            ))}
          </ul>
        ) : (
          <p className="empty-state" role="status">
            {state.tasks.length === 0
              ? "No tasks yet. Add one above."
              : "No tasks match this filter."}
          </p>
        )}

        <footer className="task-app-footer">
          {state.tasks.length} total · {completedCount} completed
        </footer>
      </section>
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
  color: #18212f;
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

button:focus-visible,
input:focus-visible {
  outline: 3px solid #82aaff;
  outline-offset: 3px;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.page {
  width: min(100% - 2rem, 820px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin: 0 0 0.45rem;
  color: #315db5;
  font-size: 0.76rem;
  font-weight: 750;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

h1,
h2,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 6vw, 3.2rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0;
  font-size: 1.25rem;
}

.intro {
  max-width: 64ch;
  margin-bottom: 0;
  color: #526071;
  line-height: 1.65;
}

.task-app {
  padding: clamp(1rem, 4vw, 1.6rem);
  border: 1px solid #d8e0eb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 35px rgb(32 53 84 / 7%);
}

.task-app-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.2rem;
}

.task-count {
  margin: 0.35rem 0 0;
  color: #637185;
  font-size: 0.9rem;
}

.add-form {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  gap: 0.6rem;
}

.add-form input,
.edit-form input {
  min-width: 0;
  padding: 0.7rem 0.8rem;
  border: 1px solid #bac8dc;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
}

.primary-button,
.secondary-button,
.danger-button {
  border-radius: 0.55rem;
  cursor: pointer;
}

.primary-button {
  padding: 0.7rem 0.9rem;
  border: 1px solid #315db5;
  color: #fff;
  background: #315db5;
  font-weight: 700;
}

.primary-button:hover:not(:disabled) {
  background: #244b9a;
}

.secondary-button {
  padding: 0.6rem 0.8rem;
  border: 1px solid #bac8dc;
  color: #244578;
  background: #f7f9fc;
}

.secondary-button:hover:not(:disabled) {
  background: #edf3fb;
}

.danger-button {
  padding: 0.6rem 0.8rem;
  border: 1px solid #e7c2c2;
  color: #9a3030;
  background: #fff8f8;
}

.danger-button:hover {
  background: #ffeded;
}

.filter-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.45rem;
  margin-block: 1.1rem 0.65rem;
}

.filter-button {
  padding: 0.45rem 0.75rem;
  border: 1px solid transparent;
  border-radius: 999px;
  color: #526071;
  background: transparent;
  cursor: pointer;
}

.filter-button:hover {
  background: #f3f6fb;
}

.filter-button-active {
  border-color: #c7d6f4;
  color: #244b9a;
  background: #edf3ff;
  font-weight: 700;
}

.task-list {
  display: grid;
  gap: 0.55rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
  padding: 0.75rem;
  border: 1px solid #e0e6ef;
  border-radius: 0.7rem;
}

.task-item-done {
  background: #f8fafc;
}

.task-label {
  display: flex;
  min-width: 0;
  align-items: flex-start;
  gap: 0.65rem;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.task-label input {
  flex: 0 0 auto;
  width: 1rem;
  height: 1rem;
  margin-top: 0.2rem;
  accent-color: #315db5;
}

.task-item-done .task-label span {
  color: #718096;
  text-decoration: line-through;
}

.task-actions {
  display: flex;
  flex: 0 0 auto;
  flex-wrap: wrap;
  justify-content: flex-end;
  gap: 0.4rem;
}

.small-button {
  font-size: 0.88rem;
}

.edit-form {
  display: grid;
  width: 100%;
  grid-template-columns: minmax(0, 1fr) auto;
  align-items: center;
  gap: 0.6rem;
}

.empty-state {
  margin: 1.2rem 0;
  padding: 1rem;
  border-radius: 0.7rem;
  color: #637185;
  background: #f5f7fb;
  text-align: center;
}

.task-app-footer {
  margin-top: 1rem;
  padding-top: 0.8rem;
  border-top: 1px solid #e4eaf2;
  color: #637185;
  font-size: 0.88rem;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  clip-path: inset(50%);
}

@media (max-width: 600px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .task-app-header {
    align-items: flex-start;
    flex-direction: column;
  }

  .task-item {
    align-items: flex-start;
    flex-direction: column;
  }

  .task-actions {
    width: 100%;
    justify-content: flex-start;
  }
}

@media (max-width: 420px) {
  .add-form {
    grid-template-columns: 1fr;
  }

  .edit-form {
    grid-template-columns: 1fr;
  }

  .edit-form .task-actions {
    justify-content: flex-start;
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
      content="A React task manager example using useReducer for related state transitions."
    />
    <title>useReducer Task Manager - React Example</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How it works

### The reducer

taskReducer is a pure function outside App. It accepts the current state and an action, then returns the complete next state.

- task/draftChanged updates the add form draft.
- task/added appends a task and clears the draft.
- task/toggled changes one task's done value.
- task/editStarted copies the task title into a distinct edit draft.
- task/editDraftChanged updates that draft.
- task/editSaved replaces the task with an updated copy.
- task/editCancelled discards the draft.
- task/deleted removes one task.
- tasks/completedCleared removes completed tasks.
- filter/changed stores the current filter.

Every object and array update returns new objects and arrays. The reducer does not make IDs, access the DOM, or perform a network request.

### Event handlers and actions

App and TaskItem dispatch actions that describe user interactions. The Add handler trims and validates the draft, creates a unique ID in the event handler, then includes the new task in the action. The reducer receives all the information it needs to apply that event.

TaskItem receives state and dispatch as props. It does not own another copy of the task list.

### Derived values

visibleTasks, activeCount, and completedCount are calculated from the reducer state during rendering. They are not separate state variables, so they cannot drift out of sync with tasks or filter.

### An intentional draft

editDraft is separate from the saved task title while the user is editing. That is an intentional distinction: the draft can be changed or cancelled without changing the saved task. Saving commits the draft into the task and clears edit mode.

## Run it

From the Vite project root, install the project's dependencies if needed and start its development server:

~~~sh
npm install
npm run dev
~~~

Open the local address printed by Vite. Add tasks, edit one and cancel, edit and save another, toggle completion, switch filters, delete a task, and clear completed tasks.

## What to notice

- Event handlers describe interactions by dispatching actions.
- The reducer centralizes related state transitions.
- The reducer is pure and returns immutable updates.
- IDs are created in the event handler, not in the reducer.
- The list and counts are derived instead of duplicated in state.
- editDraft is kept separately because it represents an unsaved change.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="045-controlled-and-uncontrolled-component-apis.md">045 - Controlled and Uncontrolled Component APIs - Example</a></td><td><a href="../lessons/046-complex-state-with-usereducer.md">Lesson</a></td><td><a href="../qnas/046-complex-state-with-usereducer.md">Q&amp;A</a></td><td><a href="047-reducer-design-and-state-machines.md">047 - Reducer Design, Transitions, and State Machines - Example</a></td></tr>
  </tbody>
</table>
