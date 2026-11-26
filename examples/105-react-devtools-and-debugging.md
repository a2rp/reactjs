# React Example: Debug and Profile a Task Board with React DevTools

This example creates a small task board with a component tree that is useful to inspect. Use the Components panel to follow props and state, toggle tasks and add a new one, then record those interactions in the Profiler. Short debugging exercises introduce common mistakes and show how to correct them.

## What this example demonstrates

- Inspecting parent state and child props in React DevTools.
- Profiling real interactions and measuring React render work.
- Rendering a changing list with stable keys and immutable state updates.
- Keeping a controlled text input controlled for its full lifetime.
- Recognizing Strict Mode development checks and cleaning up an Effect.
- Using the browser console and component tree together to find a cause.

## Prerequisites

Start from a React project with a development command such as `npm run dev`. Install the React Developer Tools browser extension in the browser you use for the app. This example uses React 18 or later APIs.

## File structure

```text
src/
  App.jsx
  App.css
  main.jsx
```

## 1. Create an inspectable component tree

Create `src/App.jsx`:

```jsx
import { Profiler, useRef, useState } from 'react';

const startingTasks = [
  { id: 'task-1', title: 'Inspect TaskBoard state', done: false },
  { id: 'task-2', title: 'Inspect TaskRow props', done: true },
];

function reportTaskBoardRender(id, phase, actualDuration, baseDuration) {
  console.table({
    component: id,
    phase,
    actualDuration: `${actualDuration.toFixed(2)} ms`,
    baseDuration: `${baseDuration.toFixed(2)} ms`,
  });
}

function TaskRow({ task, onToggle }) {
  return (
    <li className={task.done ? 'task task--done' : 'task'}>
      <label>
        <input
          type="checkbox"
          checked={task.done}
          onChange={() => onToggle(task.id)}
        />
        <span>{task.title}</span>
      </label>
    </li>
  );
}

function TaskList({ tasks, onToggle }) {
  if (tasks.length === 0) {
    return <p>No tasks yet. Add one above.</p>;
  }

  return (
    <ul className="task-list">
      {tasks.map((task) => (
        <TaskRow key={task.id} task={task} onToggle={onToggle} />
      ))}
    </ul>
  );
}

export default function App() {
  const [tasks, setTasks] = useState(startingTasks);
  const [draft, setDraft] = useState('');
  const nextId = useRef(3);

  function addTask(event) {
    event.preventDefault();
    const title = draft.trim();

    if (!title) {
      return;
    }

    const task = {
      id: `task-${nextId.current++}`,
      title,
      done: false,
    };

    setTasks((currentTasks) => [...currentTasks, task]);
    setDraft('');
  }

  function toggleTask(taskId) {
    setTasks((currentTasks) =>
      currentTasks.map((task) =>
        task.id === taskId ? { ...task, done: !task.done } : task,
      ),
    );
  }

  const completedCount = tasks.filter((task) => task.done).length;

  return (
    <main className="page">
      <h1>Task board</h1>
      <p>
        Complete: {completedCount} of {tasks.length}
      </p>

      <Profiler id="TaskBoard" onRender={reportTaskBoardRender}>
        <section className="panel" aria-labelledby="add-title">
          <h2 id="add-title">Add a task</h2>
          <form onSubmit={addTask}>
            <label htmlFor="task-title">Task title</label>
            <input
              id="task-title"
              value={draft}
              onChange={(event) => setDraft(event.target.value)}
              placeholder="For example, inspect component props"
            />
            <button type="submit" disabled={!draft.trim()}>
              Add task
            </button>
          </form>
        </section>

        <section className="panel" aria-labelledby="list-title">
          <h2 id="list-title">Tasks</h2>
          <TaskList tasks={tasks} onToggle={toggleTask} />
        </section>
      </Profiler>
    </main>
  );
}
```

Create `src/main.jsx`:

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './App.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

Create `src/App.css`:

```css
:root {
  font-family: system-ui, sans-serif;
  color: #1b2833;
  background: #f3f6f8;
  font-synthesis: none;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  margin: 0;
}

button,
input {
  font: inherit;
}

button {
  min-height: 2.5rem;
  padding: 0.5rem 0.85rem;
  border: 0;
  border-radius: 0.4rem;
  color: white;
  background: #145c47;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

:focus-visible {
  outline: 3px solid #d47b00;
  outline-offset: 3px;
}

.page {
  width: min(100% - 2rem, 42rem);
  margin: 2rem auto;
}

.panel {
  display: grid;
  gap: 0.85rem;
  margin-block: 1rem;
  padding: 1.25rem;
  border: 1px solid #d3dce2;
  border-radius: 0.75rem;
  background: white;
}

.panel h2,
.panel p {
  margin: 0;
}

form {
  display: grid;
  gap: 0.6rem;
}

input:not([type='checkbox']) {
  min-height: 2.5rem;
  padding: 0.5rem 0.7rem;
  border: 1px solid #74818a;
  border-radius: 0.35rem;
}

.task-list {
  display: grid;
  gap: 0.65rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.task label {
  display: flex;
  align-items: center;
  gap: 0.65rem;
}

.task--done span {
  color: #52616b;
  text-decoration: line-through;
}
```

## 2. Inspect state and props

Run the app with its development command, then open browser developer tools and select **Components**.

1. Find `App` in the component tree. Its `tasks` and `draft` state belong to this component.
2. Select `TaskList`. Its `tasks` and `onToggle` values are props passed from `App`.
3. Select a `TaskRow`. Its `task` object identifies that row, and `onToggle` is the callback from its parent.
4. Toggle a task in the page. Watch `App` state and the selected row's props change.
5. Add a task. Inspect its new stable ID and verify the input clears after the event.

The browser Elements panel will show the HTML input and list items. Compare it with Components to see how React components produce DOM.

## 3. Record an interaction in Profiler

The page wraps the interactive area in React's `<Profiler>` and logs basic measurements for its commits. This is for learning and has measurement overhead; remove it or keep it only where programmatic measurements are useful.

Also use the **Profiler** panel in React DevTools:

1. Start a recording.
2. Toggle one task, then add one task.
3. Stop recording and select the commits caused by those interactions.
4. Inspect which components rendered and compare their render work.
5. Check whether the work explains a measured slowdown before attempting an optimization.

For this small board, the important result is learning to record a controlled interaction and read the tree. The sample is too small to justify memoization based on guesswork.

## 4. Debugging exercises

Try one change at a time, observe the warning or behavior, then restore the correct version.

### Exercise A: remove a list key

Temporarily change the `TaskRow` line in `TaskList` to omit its key:

```jsx
<TaskRow task={task} onToggle={onToggle} />
```

React reports that mapped children need keys. Restore `key={task.id}`. Then try `key={Math.random()}`: it removes the missing-key warning, but creates a different identity every render. The data ID is the stable solution.

### Exercise B: make the controlled input undefined initially

Change the `draft` initialization to:

```jsx
const [draft, setDraft] = useState(undefined);
```

The input starts without a controlled string value, then receives one after typing. Restore `useState('')` so the field stays controlled from its first render.

### Exercise C: mutate task state

Replace the immutable toggle update with this incorrect version:

```jsx
function toggleTask(taskId) {
  const task = tasks.find((item) => item.id === taskId);
  task.done = !task.done;
  setTasks(tasks);
}
```

This mutates existing state and passes React the same array reference. Restore the immutable `map()` update from the complete example so React receives a new array and a new changed task object.

### Exercise D: call a Hook conditionally

This is incorrect because the Hook call order changes depending on the prop:

```jsx
function BadPanel({ visible }) {
  if (visible) {
    const [open, setOpen] = useState(false);
    return <button onClick={() => setOpen(!open)}>Toggle</button>;
  }

  return null;
}
```

Move the Hook to the top level before the condition, or make the conditional UI a separate component whose Hooks always run whenever that component renders.

### Exercise E: observe Strict Mode and cleanup

A render or Effect may appear to run an extra time during development because the root is wrapped in `StrictMode`. To see the cleanup pattern, try this component:

```jsx
import { useEffect, useState } from 'react';

function Clock() {
  const [now, setNow] = useState(() => new Date());

  useEffect(() => {
    const timerId = window.setInterval(() => setNow(new Date()), 1000);
    return () => window.clearInterval(timerId);
  }, []);

  return <p>Updated at {now.toLocaleTimeString()}</p>;
}
```

The interval cleanup stops the previous timer before a new setup and when the component unmounts. Strict Mode is checking that setup can be safely cleaned up; keep the cleanup and leave Strict Mode enabled.

## 5. Use the browser console to investigate a warning

When a warning appears:

1. Read the full first message and any component stack.
2. Find the named component in **Components**.
3. Inspect the relevant props, state, or Hook values.
4. Trace the value back to where it is created or changed.
5. Correct the source and repeat the same interaction.

For performance, make a Profiler recording around the exact interaction that feels slow. For a request problem, inspect the browser Network panel. For a rendering problem, inspect the DOM and computed styles. React DevTools, Console, Elements, Network, and Performance each show a different part of the same behavior.

## Expected outcomes

- The component tree makes the relationship between `App`, `TaskList`, and `TaskRow` visible.
- Toggling and adding tasks update state immutably and render the expected output.
- List children have stable keys and the text field remains controlled.
- Profiler records the actual interactions instead of relying on a guess about re-renders.
- The debugging exercises reveal why the warnings occur and how to restore the correct implementation.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="104-browser-level-and-end-to-end-tests.md">104 - React Example: Browser-Test a Complete Checkout Journey</a></td><td><a href="../lessons/105-react-devtools-and-debugging.md">Lesson</a></td><td><a href="../qnas/105-react-devtools-and-debugging.md">Q&amp;A</a></td><td><a href="106-profiling-during-development.md">106 - React Example: Profile an Effect-Driven Update</a></td></tr>
  </tbody>
</table>
