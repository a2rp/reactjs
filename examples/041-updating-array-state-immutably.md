# 041. Updating array state immutably - Example

## What this example demonstrates

This task list shows common immutable array updates:

- Add an item at the beginning or end with spread syntax.
- Remove items with filter.
- Update or rename one item with map and object spread.
- Update a nested item property by copying each object along the path.
- Reorder or sort a copied array without changing the old state array.
- Render each item with its stable ID as the key.

## Project files

Add these files to an existing React app created with Vite:

- src/App.jsx
- src/App.css
- src/main.jsx
- src/index.css

Run the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useState } from 'react';
import './App.css';

const initialTasks = [
  {
    id: 1,
    title: 'Read the array update lesson',
    done: true,
    details: {
      priority: 'normal',
    },
  },
  {
    id: 2,
    title: 'Practice map and filter',
    done: false,
    details: {
      priority: 'high',
    },
  },
  {
    id: 3,
    title: 'Build an immutable task list',
    done: false,
    details: {
      priority: 'normal',
    },
  },
];

let nextTaskId = 4;

function TaskItem({
  task,
  onToggle,
  onRename,
  onPriorityChange,
  onRemove,
  onMoveToFront,
}) {
  const titleId = 'task-title-' + task.id;
  const doneId = 'task-done-' + task.id;
  const priorityId = 'task-priority-' + task.id;

  return (
    <li className={task.done ? 'task-item is-done' : 'task-item'}>
      <div className="task-main">
        <input
          id={doneId}
          type="checkbox"
          checked={task.done}
          aria-label={'Mark ' + task.title + ' complete'}
          onChange={(event) => onToggle(task.id, event.target.checked)}
        />

        <div className="task-fields">
          <label className="visually-hidden" htmlFor={titleId}>
            Task title
          </label>
          <input
            id={titleId}
            className="task-title-input"
            value={task.title}
            onChange={(event) => onRename(task.id, event.target.value)}
          />

          <label className="priority-label" htmlFor={priorityId}>
            Priority
          </label>
          <select
            id={priorityId}
            value={task.details.priority}
            onChange={(event) =>
              onPriorityChange(task.id, event.target.value)
            }
          >
            <option value="low">Low</option>
            <option value="normal">Normal</option>
            <option value="high">High</option>
          </select>
        </div>
      </div>

      <div className="task-actions">
        <button
          className="small-button secondary-button"
          type="button"
          onClick={() => onMoveToFront(task.id)}
        >
          Move to top
        </button>
        <button
          className="small-button danger-button"
          type="button"
          onClick={() => onRemove(task.id)}
          aria-label={'Remove ' + task.title}
        >
          Remove
        </button>
      </div>
    </li>
  );
}

export default function App() {
  const [tasks, setTasks] = useState(initialTasks);
  const [draft, setDraft] = useState('');
  const [status, setStatus] = useState(
    'Try the list controls to see immutable array updates.'
  );

  function createTaskFromDraft() {
    const title = draft.trim();

    if (title === '') {
      setStatus('Enter a task title before adding a task.');
      return null;
    }

    return {
      id: nextTaskId,
      title,
      done: false,
      details: {
        priority: 'normal',
      },
    };
  }

  function handleAddToEnd(event) {
    event.preventDefault();

    const newTask = createTaskFromDraft();
    if (newTask === null) {
      return;
    }

    nextTaskId += 1;
    setTasks((currentTasks) => [...currentTasks, newTask]);
    setDraft('');
    setStatus('Added a task at the end with a new array.');
  }

  function handleAddToFront() {
    const newTask = createTaskFromDraft();
    if (newTask === null) {
      return;
    }

    nextTaskId += 1;
    setTasks((currentTasks) => [newTask, ...currentTasks]);
    setDraft('');
    setStatus('Added a task at the beginning with a new array.');
  }

  function handleToggle(taskId, nextDone) {
    setTasks((currentTasks) =>
      currentTasks.map((task) =>
        task.id === taskId
          ? { ...task, done: nextDone }
          : task
      )
    );
    setStatus('Updated one task and returned a new array.');
  }

  function handleRename(taskId, nextTitle) {
    setTasks((currentTasks) =>
      currentTasks.map((task) =>
        task.id === taskId
          ? { ...task, title: nextTitle }
          : task
      )
    );
  }

  function handlePriorityChange(taskId, nextPriority) {
    setTasks((currentTasks) =>
      currentTasks.map((task) =>
        task.id === taskId
          ? {
              ...task,
              details: {
                ...task.details,
                priority: nextPriority,
              },
            }
          : task
      )
    );
    setStatus('Updated a nested task property with new objects.');
  }

  function handleRemove(taskId) {
    setTasks((currentTasks) =>
      currentTasks.filter((task) => task.id !== taskId)
    );
    setStatus('Removed one task with filter, leaving the old array unchanged.');
  }

  function handleClearCompleted() {
    setTasks((currentTasks) => currentTasks.filter((task) => !task.done));
    setStatus('Removed completed tasks with filter.');
  }

  function handleSortByTitle() {
    setTasks((currentTasks) =>
      [...currentTasks].sort((firstTask, secondTask) =>
        firstTask.title.localeCompare(secondTask.title)
      )
    );
    setStatus('Sorted a copied array; the previous array was not sorted in place.');
  }

  function handleReverseOrder() {
    setTasks((currentTasks) => [...currentTasks].reverse());
    setStatus('Reversed a copied array; the previous array was not reversed in place.');
  }

  function handleMoveToFront(taskId) {
    setTasks((currentTasks) => {
      const taskToMove = currentTasks.find((task) => task.id === taskId);

      if (taskToMove === undefined) {
        return currentTasks;
      }

      return [
        taskToMove,
        ...currentTasks.filter((task) => task.id !== taskId),
      ];
    });
    setStatus('Moved one item by creating a new array.');
  }

  function handleReset() {
    setTasks(initialTasks);
    setDraft('');
    setStatus('Restored the initial tasks.');
  }

  const completedCount = tasks.filter((task) => task.done).length;

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React state</p>
        <h1>Updating array state immutably</h1>
        <p>
          Add, edit, remove, sort, and reorder tasks. Each operation creates a
          replacement array instead of changing the current state array.
        </p>
      </header>

      <section className="task-app" aria-labelledby="tasks-heading">
        <div className="task-header">
          <div>
            <p className="eyebrow">Task list</p>
            <h2 id="tasks-heading">Practice tasks</h2>
          </div>
          <p className="task-count" aria-live="polite">
            {completedCount} of {tasks.length} complete
          </p>
        </div>

        <form className="add-form" onSubmit={handleAddToEnd}>
          <label className="visually-hidden" htmlFor="new-task">
            New task title
          </label>
          <input
            id="new-task"
            value={draft}
            onChange={(event) => setDraft(event.target.value)}
            placeholder="Write a task title"
          />
          <button type="submit">Add to end</button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleAddToFront}
          >
            Add to top
          </button>
        </form>

        <div className="toolbar" role="group" aria-label="Task list actions">
          <button
            className="secondary-button"
            type="button"
            onClick={handleSortByTitle}
          >
            Sort by title
          </button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleReverseOrder}
          >
            Reverse order
          </button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleClearCompleted}
          >
            Clear completed
          </button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleReset}
          >
            Reset list
          </button>
        </div>

        <p className="status-box" role="status">
          {status}
        </p>

        {tasks.length === 0 ? (
          <p className="empty-state">No tasks yet. Add one above.</p>
        ) : (
          <ul className="task-list">
            {tasks.map((task) => (
              <TaskItem
                key={task.id}
                task={task}
                onToggle={handleToggle}
                onRename={handleRename}
                onPriorityChange={handlePriorityChange}
                onRemove={handleRemove}
                onMoveToFront={handleMoveToFront}
              />
            ))}
          </ul>
        )}
      </section>

      <footer className="page-footer">
        <p>
          An array copy is shallow. When an object item changes, create a new
          item object too.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/App.css

~~~css
.page-shell {
  width: min(100% - 2rem, 1000px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 72ch;
  margin-bottom: 1.75rem;
}

.page-header h1 {
  margin-block: 0.35rem 0.75rem;
  font-size: clamp(2rem, 5vw, 3.2rem);
  line-height: 1.1;
}

.page-header > p:last-child,
.page-footer {
  line-height: 1.65;
  color: #4b5563;
}

.eyebrow {
  color: #4d3ab8;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.task-app {
  padding: clamp(1rem, 3vw, 1.75rem);
  border: 1px solid #d5dae4;
  border-radius: 1rem;
  background: #ffffff;
  box-shadow: 0 0.5rem 1.5rem rgb(24 33 54 / 7%);
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: end;
  gap: 1rem;
}

.task-header h2 {
  margin-top: 0.35rem;
}

.task-count {
  padding: 0.45rem 0.7rem;
  border-radius: 999px;
  background: #f0edff;
  color: #4431a8;
  font-weight: 700;
  white-space: nowrap;
}

.add-form,
.toolbar,
.task-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
}

.add-form {
  margin-block: 1.25rem;
}

.add-form input {
  flex: 1 1 14rem;
  min-height: 2.7rem;
  padding: 0.55rem 0.75rem;
  border: 1px solid #abb3c0;
  border-radius: 0.5rem;
}

button,
input,
select {
  font: inherit;
}

button {
  min-height: 2.7rem;
  padding: 0.55rem 0.8rem;
  border: 1px solid #4d3ab8;
  border-radius: 0.5rem;
  background: #5b46d6;
  color: #ffffff;
  cursor: pointer;
}

button:hover {
  background: #4835b5;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #a99cff;
  outline-offset: 2px;
}

.secondary-button {
  border-color: #abb3c0;
  background: #ffffff;
  color: #202735;
}

.secondary-button:hover {
  background: #f0f2f6;
}

.danger-button {
  border-color: #b42318;
  background: #ffffff;
  color: #9d1c13;
}

.danger-button:hover {
  background: #fff1f0;
}

.toolbar {
  padding-block: 0.9rem;
  border-top: 1px solid #e2e5eb;
  border-bottom: 1px solid #e2e5eb;
}

.status-box {
  min-height: 3rem;
  margin-block: 1rem;
  padding: 0.8rem 0.95rem;
  border-left: 4px solid #5b46d6;
  border-radius: 0.35rem;
  background: #f3f1ff;
  line-height: 1.5;
}

.task-list {
  list-style: none;
}

.task-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  padding: 1rem 0;
  border-bottom: 1px solid #e2e5eb;
}

.task-main {
  display: flex;
  flex: 1;
  align-items: start;
  gap: 0.8rem;
  min-width: 0;
}

.task-main > input[type="checkbox"] {
  width: 1.1rem;
  height: 1.1rem;
  margin-top: 0.45rem;
  accent-color: #5b46d6;
}

.task-fields {
  display: grid;
  grid-template-columns: minmax(8rem, 1fr) auto 7rem;
  align-items: center;
  gap: 0.65rem;
  flex: 1;
  min-width: 0;
}

.task-title-input,
.task-fields select {
  min-width: 0;
  min-height: 2.4rem;
  padding: 0.4rem 0.6rem;
  border: 1px solid #d1d6e0;
  border-radius: 0.4rem;
  background: #ffffff;
  color: #202735;
}

.task-title-input {
  font-weight: 650;
}

.is-done .task-title-input {
  color: #657083;
  text-decoration: line-through;
}

.priority-label {
  color: #596273;
  font-size: 0.9rem;
}

.task-actions {
  justify-content: end;
}

.small-button {
  min-height: 2.35rem;
  padding: 0.4rem 0.65rem;
  font-size: 0.9rem;
}

.empty-state {
  padding-block: 2rem;
  text-align: center;
  color: #596273;
}

.page-footer {
  margin-top: 1.5rem;
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

@media (max-width: 720px) {
  .page-shell {
    width: min(100% - 1.25rem, 1000px);
    padding-block: 1.5rem;
  }

  .task-item {
    align-items: stretch;
    flex-direction: column;
  }

  .task-actions {
    padding-left: 1.9rem;
  }
}

@media (max-width: 520px) {
  .task-header {
    align-items: start;
    flex-direction: column;
  }

  .task-fields {
    grid-template-columns: 1fr;
    gap: 0.4rem;
  }
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.jsx';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #202735;
  background: #f6f7fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input,
select {
  font: inherit;
}
~~~

## Try it

1. Enter a task and select Add to end.
2. Enter another task and select Add to top. Notice its position changes without mutating the previous array.
3. Edit a task title and change its priority. The matching item is replaced with a new object; the nested priority update also copies details.
4. Check a task, then select Clear completed. filter returns a new array without completed tasks.
5. Select Sort by title or Reverse order. Both handlers copy before reordering.
6. Select Move to top on a task. The item keeps its ID while the list gets a new order.
7. Remove a task and notice the remaining items retain their stable keys.
8. Select Reset list to restore the initial data.

## What to notice

- Every task-list update returns a new array.
- map creates a new object only for the item being changed.
- Nested item updates copy each changed object along the path.
- Sorting and reversing operate on a copy.
- The task ID is created when the task is added and reused as its React key.
