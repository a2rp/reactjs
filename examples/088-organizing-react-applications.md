# Organizing Components, Features, and Application Code — Examples

This example builds a small task feature and organizes it by ownership. The application shell composes the feature. The task feature owns its page, form, filters, list, reducer, starter data, and styles.

## 1. File structure

```text
src/
  main.jsx
  app/
    App.jsx
    app.css
  features/
    tasks/
      components/
        TaskFilter.jsx
        TaskForm.jsx
        TaskItem.jsx
        TaskList.jsx
      initialTasks.js
      taskReducer.js
      TaskPage.jsx
      tasks.css
```

There is no `shared/` folder yet. None of these components has a real cross-feature use case, so keeping them inside `features/tasks/` makes their ownership clear.

## 2. Mount the application

Create `src/main.jsx`:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./app/App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

The entry point only mounts the app. It does not contain task behavior.

## 3. Compose the app shell

Create `src/app/App.jsx`:

```jsx
import TaskPage from "../features/tasks/TaskPage.jsx";
import "./app.css";

export default function App() {
  return (
    <div className="app-shell">
      <header className="app-header">
        <p className="app-kicker">React application structure</p>
        <h1>My task board</h1>
        <p>
          A small feature organized around the behavior it owns.
        </p>
      </header>

      <main>
        <TaskPage />
      </main>
    </div>
  );
}
```

Create `src/app/app.css`:

```css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #192235;
  background: #f3f5fa;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
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
input,
select {
  font: inherit;
}

button,
select {
  cursor: pointer;
}

.app-shell {
  width: min(100% - 2rem, 760px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.app-header {
  margin-bottom: 2rem;
}

.app-kicker {
  margin-bottom: 0.5rem;
  color: #4b5ec7;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.app-header h1 {
  margin: 0 0 0.5rem;
  font-size: clamp(2rem, 5vw, 3rem);
}

.app-header > p:last-child {
  margin: 0;
  color: #586174;
  line-height: 1.6;
}
```

The app shell owns the page heading and global layout. The task implementation stays within its feature.

## 4. Define the task state transitions

Create `src/features/tasks/taskReducer.js`:

```js
export default function taskReducer(tasks, action) {
  switch (action.type) {
    case "task/added":
      return [action.task, ...tasks];

    case "task/toggled":
      return tasks.map((task) =>
        task.id === action.taskId
          ? { ...task, completed: !task.completed }
          : task,
      );

    case "task/deleted":
      return tasks.filter((task) => task.id !== action.taskId);

    default:
      throw new Error(`Unknown task action: ${action.type}`);
  }
}
```

The reducer contains task transition rules. It does not render UI or access the DOM.

Create `src/features/tasks/initialTasks.js`:

```js
const initialTasks = [
  {
    id: "task-read",
    title: "Read the lesson on component boundaries",
    completed: false,
  },
  {
    id: "task-outline",
    title: "Sketch a feature folder for the project",
    completed: true,
  },
];

export default initialTasks;
```

This module provides the feature's starter data. In a real application, a data layer could load tasks from a server or persistent store instead.

## 5. Build focused feature components

Create `src/features/tasks/components/TaskForm.jsx`:

```jsx
import { useState } from "react";

export default function TaskForm({ onAdd }) {
  const [title, setTitle] = useState("");

  function handleSubmit(event) {
    event.preventDefault();

    const nextTitle = title.trim();

    if (!nextTitle) {
      return;
    }

    onAdd(nextTitle);
    setTitle("");
  }

  return (
    <form className="task-form" onSubmit={handleSubmit}>
      <label htmlFor="new-task">New task</label>
      <div className="task-form-row">
        <input
          id="new-task"
          type="text"
          value={title}
          onChange={(event) => setTitle(event.target.value)}
          maxLength={100}
          placeholder="What needs to be done?"
        />
        <button type="submit" disabled={!title.trim()}>
          Add task
        </button>
      </div>
    </form>
  );
}
```

The form owns its unsubmitted input draft. It reports a complete title to its parent instead of owning the task collection.

Create `src/features/tasks/components/TaskFilter.jsx`:

```jsx
const filterOptions = [
  { value: "all", label: "All tasks" },
  { value: "open", label: "Open tasks" },
  { value: "completed", label: "Completed tasks" },
];

export default function TaskFilter({ value, onChange }) {
  return (
    <label className="task-filter">
      Show
      <select value={value} onChange={(event) => onChange(event.target.value)}>
        {filterOptions.map((option) => (
          <option key={option.value} value={option.value}>
            {option.label}
          </option>
        ))}
      </select>
    </label>
  );
}
```

The filter is controlled by the page, which owns the selected view. Its interface is only the current value and a callback.

Create `src/features/tasks/components/TaskItem.jsx`:

```jsx
export default function TaskItem({ task, onToggle, onDelete }) {
  return (
    <li className={task.completed ? "task-item is-complete" : "task-item"}>
      <label className="task-item-label">
        <input
          type="checkbox"
          checked={task.completed}
          onChange={() => onToggle(task.id)}
        />
        <span>{task.title}</span>
      </label>

      <button
        className="delete-button"
        type="button"
        aria-label={`Delete task: ${task.title}`}
        onClick={() => onDelete(task.id)}
      >
        Delete
      </button>
    </li>
  );
}
```

The item renders one task and reports the requested action. React renders the title as text, so user-entered text is not interpreted as HTML.

Create `src/features/tasks/components/TaskList.jsx`:

```jsx
import TaskItem from "./TaskItem.jsx";

export default function TaskList({ tasks, onToggle, onDelete }) {
  if (tasks.length === 0) {
    return (
      <p className="empty-state" role="status">
        No tasks match this filter.
      </p>
    );
  }

  return (
    <ul className="task-list" aria-label="Tasks">
      {tasks.map((task) => (
        <TaskItem
          key={task.id}
          task={task}
          onToggle={onToggle}
          onDelete={onDelete}
        />
      ))}
    </ul>
  );
}
```

The list owns list rendering and the empty state, while the parent remains the owner of the task data.

## 6. Assemble the feature and own its state

Create `src/features/tasks/TaskPage.jsx`:

```jsx
import { useReducer, useState } from "react";
import TaskFilter from "./components/TaskFilter.jsx";
import TaskForm from "./components/TaskForm.jsx";
import TaskList from "./components/TaskList.jsx";
import initialTasks from "./initialTasks.js";
import taskReducer from "./taskReducer.js";
import "./tasks.css";

function createTaskId() {
  return globalThis.crypto.randomUUID();
}

export default function TaskPage() {
  const [tasks, dispatch] = useReducer(taskReducer, initialTasks);
  const [filter, setFilter] = useState("all");

  const visibleTasks = tasks.filter((task) => {
    if (filter === "open") {
      return !task.completed;
    }

    if (filter === "completed") {
      return task.completed;
    }

    return true;
  });

  const completedCount = tasks.filter((task) => task.completed).length;

  function handleAdd(title) {
    dispatch({
      type: "task/added",
      task: {
        id: createTaskId(),
        title,
        completed: false,
      },
    });
  }

  function handleToggle(taskId) {
    dispatch({ type: "task/toggled", taskId });
  }

  function handleDelete(taskId) {
    dispatch({ type: "task/deleted", taskId });
  }

  return (
    <section className="task-feature" aria-labelledby="tasks-title">
      <div className="task-heading">
        <div>
          <h2 id="tasks-title">Tasks</h2>
          <p>
            {completedCount} of {tasks.length} completed
          </p>
        </div>

        <TaskFilter value={filter} onChange={setFilter} />
      </div>

      <TaskForm onAdd={handleAdd} />

      <TaskList
        tasks={visibleTasks}
        onToggle={handleToggle}
        onDelete={handleDelete}
      />
    </section>
  );
}
```

The page owns the task collection and filter because they coordinate the form, filter, and list. The form owns only its temporary text draft. Each component receives just the props and callbacks it needs.

## 7. Style the feature beside its owner

Create `src/features/tasks/tasks.css`:

```css
.task-feature {
  border: 1px solid #dce1ec;
  border-radius: 16px;
  padding: clamp(1rem, 4vw, 1.5rem);
  background: #fff;
  box-shadow: 0 10px 30px rgb(25 39 74 / 6%);
}

.task-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.task-heading h2 {
  margin: 0 0 0.3rem;
}

.task-heading p {
  margin: 0;
  color: #626d81;
  font-size: 0.9rem;
}

.task-filter,
.task-form {
  display: grid;
  gap: 0.5rem;
  color: #35405a;
  font-size: 0.9rem;
  font-weight: 650;
}

.task-filter select,
.task-form input {
  min-height: 44px;
  border: 1px solid #bac3d4;
  border-radius: 8px;
  padding: 0.6rem 0.75rem;
  background: #fff;
  color: #192235;
}

.task-form {
  margin-bottom: 1.5rem;
}

.task-form-row {
  display: flex;
  gap: 0.65rem;
}

.task-form-row input {
  min-width: 0;
  flex: 1;
}

.task-form-row button,
.delete-button {
  border: 0;
  border-radius: 8px;
  padding: 0.6rem 0.9rem;
  font-weight: 700;
}

.task-form-row button {
  background: #3448b5;
  color: #fff;
}

.task-form-row button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.task-list {
  display: grid;
  gap: 0.65rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  border: 1px solid #e0e4ed;
  border-radius: 10px;
  padding: 0.8rem;
}

.task-item-label {
  display: flex;
  align-items: flex-start;
  gap: 0.7rem;
  line-height: 1.5;
}

.task-item-label input {
  width: 1.1rem;
  height: 1.1rem;
  margin-top: 0.2rem;
  accent-color: #3448b5;
}

.is-complete .task-item-label span {
  color: #6c7587;
  text-decoration: line-through;
}

.delete-button {
  flex: none;
  background: #f1f3f8;
  color: #35405a;
}

.empty-state {
  border: 1px dashed #bac3d4;
  border-radius: 10px;
  padding: 1rem;
  color: #626d81;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #8897ed;
  outline-offset: 2px;
}

@media (max-width: 520px) {
  .task-heading {
    align-items: stretch;
    flex-direction: column;
  }

  .task-filter select {
    width: 100%;
  }

  .task-form-row {
    align-items: stretch;
    flex-direction: column;
  }

  .task-item {
    align-items: flex-start;
  }
}
```

## 8. Follow the data flow

The task feature's data flow is:

```text
TaskForm --submits title--> TaskPage --dispatches action--> taskReducer
TaskPage --passes filtered tasks and callbacks--> TaskList --> TaskItem
TaskFilter --reports selected filter--> TaskPage
```

The reducer is the only module that defines how task actions change the task collection. The page coordinates the feature. The smaller components render props and report interactions.

## 9. Extend the structure only when there is a reason

- If task requests become complex, add a feature-owned `tasksApi.js` or a focused `useTasks()` Hook.
- If several deeply nested task components need the same state, consider a task-level context with a clear provider boundary.
- If another unrelated feature needs a general button, extract a stable button interface into `shared/components/`.
- If task-specific tests are added, keep them near their owner or follow the repository's established test layout.
- If the app adds routing or global providers, compose them in `app/App.jsx` or the framework's designated app entry point.

Do not create empty folders for hypothetical future code. Add structure when it makes current ownership and changes clearer.
