# 024. Component identity, position, and keys - Example

## Goal

Build a task list and a profile editor to observe component identity in action. The example demonstrates:

- Stable keys for list rows whose order can change.
- Row-local UI state staying with the same task when rows reorder.
- State disappearing when a task row is removed.
- A key outside a list intentionally resetting an editor for a different profile.
- Passing an ordinary prop when a component needs an ID; `key` itself is not a prop.

## Files

In an existing React project, create or update:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
import { useState } from 'react';

const INITIAL_TASKS = [
  {
    id: 'outline',
    title: 'Write the outline',
    description: 'Group the main ideas before drafting the lesson.',
    done: false,
  },
  {
    id: 'examples',
    title: 'Prepare examples',
    description: 'Create a small example for each important concept.',
    done: true,
  },
  {
    id: 'review',
    title: 'Review the draft',
    description: 'Check the explanation, links, and navigation.',
    done: false,
  },
];

const PROFILES = [
  { id: 'ari', name: 'Ari' },
  { id: 'bea', name: 'Bea' },
];

function TaskRow({ task, onToggle, onRemove }) {
  const [detailsOpen, setDetailsOpen] = useState(false);

  return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={task.done}
          onChange={() => onToggle(task.id)}
        />
        {task.title}
      </label>
      <button
        type="button"
        aria-expanded={detailsOpen}
        onClick={() => setDetailsOpen(!detailsOpen)}
      >
        {detailsOpen ? 'Hide details' : 'Show details'}
      </button>
      {detailsOpen && <p>{task.description}</p>}
      <button type="button" onClick={() => onRemove(task.id)}>
        Remove {task.title}
      </button>
    </li>
  );
}

function TaskList({ tasks, onToggle, onRemove }) {
  if (tasks.length === 0) {
    return <p role="status">There are no tasks left.</p>;
  }

  return (
    <ul>
      {tasks.map((task) => (
        <TaskRow
          key={task.id}
          task={task}
          onToggle={onToggle}
          onRemove={onRemove}
        />
      ))}
    </ul>
  );
}

function ProfileEditor({ profile }) {
  const [draftName, setDraftName] = useState(profile.name);

  return (
    <form onSubmit={(event) => event.preventDefault()}>
      <label htmlFor="profile-draft">
        Unsaved name for {profile.name}
      </label>
      <input
        id="profile-draft"
        value={draftName}
        onChange={(event) => setDraftName(event.target.value)}
      />
      <p>This draft is local to the current editor instance.</p>
    </form>
  );
}

export default function App() {
  const [tasks, setTasks] = useState(INITIAL_TASKS);
  const [reverseOrder, setReverseOrder] = useState(false);
  const [selectedProfileId, setSelectedProfileId] = useState(PROFILES[0].id);
  const selectedProfile = PROFILES.find(
    (profile) => profile.id === selectedProfileId,
  );
  const visibleTasks = reverseOrder ? [...tasks].reverse() : tasks;

  function handleToggleTask(taskId) {
    setTasks((currentTasks) =>
      currentTasks.map((task) =>
        task.id === taskId ? { ...task, done: !task.done } : task,
      ),
    );
  }

  function handleRemoveTask(taskId) {
    setTasks((currentTasks) =>
      currentTasks.filter((task) => task.id !== taskId),
    );
  }

  return (
    <main>
      <h1>Component identity and keys</h1>

      <section aria-labelledby="task-heading">
        <h2 id="task-heading">Reorderable tasks</h2>
        <p>
          Open a task's details, then reverse the list. The open details should
          remain with that task.
        </p>
        <button
          type="button"
          aria-pressed={reverseOrder}
          onClick={() => setReverseOrder(!reverseOrder)}
        >
          {reverseOrder ? 'Restore original order' : 'Reverse task order'}
        </button>
        <TaskList
          tasks={visibleTasks}
          onToggle={handleToggleTask}
          onRemove={handleRemoveTask}
        />
      </section>

      <section aria-labelledby="profile-heading">
        <h2 id="profile-heading">Editor for the selected profile</h2>
        <div>
          {PROFILES.map((profile) => (
            <button
              key={profile.id}
              type="button"
              aria-pressed={profile.id === selectedProfileId}
              onClick={() => setSelectedProfileId(profile.id)}
            >
              Edit {profile.name}
            </button>
          ))}
        </div>
        <ProfileEditor key={selectedProfile.id} profile={selectedProfile} />
      </section>
    </main>
  );
}
~~~

## Observe list identity

Each `TaskRow` has local `detailsOpen` state. Open the details for �Prepare examples,� then reverse the task list. Because each row uses `key={task.id}`, React can match each task with the same row after its position changes. The open details remain associated with that task.

Remove a task after opening its details. The row leaves the tree, so its local state is destroyed. Adding the same record again later would create a new row instance unless the state had been stored elsewhere.

For a quick experiment, temporarily replace `key={task.id}` with `key={index}` by mapping with an index. After changing the order, notice that row-local state can follow the position instead of the task. Restore the stable ID key after the experiment.

## Observe an intentional reset

Type a draft name for Ari, then select Bea. The `ProfileEditor` has a different key, so React creates a fresh editor and initializes its draft from Bea's name. Selecting Ari again starts Ari's editor from the initial value too; this example intentionally does not save drafts per profile.

Remove `key={selectedProfile.id}` as an experiment. The editor remains the same component at the same position, so its local draft is preserved even though the `profile` prop changes. That may be wrong for a new editing task, which is why this example uses a key.

If drafts should survive switching between profiles, store them in `App` by profile ID and pass the selected draft and change callback into the editor. Avoid using a changing key when preserving a user's work is the desired behavior.

## What each key means

| Location | Key | Purpose |
| --- | --- | --- |
| Each task row | `task.id` | Keeps local row state matched to the task when list order changes. |
| Each profile button | `profile.id` | Identifies each button among its siblings in the mapped list. |
| Profile editor | `selectedProfile.id` | Creates a fresh editor instance when the selected profile changes. |

The editor still receives `profile` as an ordinary prop. It cannot read its `key` value from props.

## Try these changes

1. Remove one task while its details are open, then add it back by restoring `INITIAL_TASKS`. Observe that a removed row starts with its details closed.
2. Change the profile editor to keep drafts in an object in `App`, keyed by profile ID. Confirm that switching away and back restores each draft.
3. Give one task a different title but keep its ID. Its row identity remains stable because the ID represents the task.
4. Change a task's ID during rendering and observe that the row is recreated. Do not use this pattern in the finished application.
5. Add a component to the mapped task rows and keep the stable key on the outermost element returned by the map.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="023-thinking-in-react-and-component-trees.md">023 - Thinking in React: from a design to a component tree - Example</a></td><td><a href="../lessons/024-component-identity-position-and-keys.md">Lesson</a></td><td><a href="../qnas/024-component-identity-position-and-keys.md">Q&amp;A</a></td><td><a href="025-event-handlers-and-user-interactions.md">025 - Event handlers and user interactions - Example</a></td></tr>
  </tbody>
</table>
