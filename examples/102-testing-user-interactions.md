# React Example: Testing a Complete User Interaction Flow

This example builds a small task board and tests two user paths. The first adds a task, marks it complete, removes it, and checks each visible checkpoint. The second uses keyboard focus and Enter to submit a task. The tests operate the rendered controls rather than calling component handlers.

The example uses TypeScript, React, Vitest, jsdom, React Testing Library, and user-event. It assumes the test environment and cleanup setup from the previous component-test example. If this is a new project, install the test dependencies and configure Vitest to use jsdom with the test setup file.

## What this example demonstrates

- A coherent user goal that crosses several UI states.
- A fresh user-event session per test.
- Typing, clicking, checkbox interaction, keyboard focus, and Enter submission.
- Scoping queries to a named list item with within.
- Assertions at meaningful visible checkpoints.
- No direct calls to React event handlers or state setters.

## File structure

    src/
    ├── TaskBoard.tsx
    └── test/
        └── TaskBoard.test.tsx

## 1. Build the task board

Create src/TaskBoard.tsx:

    import { useState, type FormEvent } from 'react';

    type Task = {
      id: number;
      title: string;
      completed: boolean;
    };

    export function TaskBoard() {
      const [tasks, setTasks] = useState<Task[]>([]);
      const [draft, setDraft] = useState('');
      const [nextId, setNextId] = useState(1);

      function addTask(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        const title = draft.trim();

        if (title.length === 0) {
          return;
        }

        const task: Task = {
          id: nextId,
          title,
          completed: false,
        };

        setTasks(currentTasks => [...currentTasks, task]);
        setNextId(currentId => currentId + 1);
        setDraft('');
      }

      function toggleTask(taskId: number) {
        setTasks(currentTasks =>
          currentTasks.map(task =>
            task.id === taskId
              ? { ...task, completed: !task.completed }
              : task,
          ),
        );
      }

      function removeTask(taskId: number) {
        setTasks(currentTasks =>
          currentTasks.filter(task => task.id !== taskId),
        );
      }

      return (
        <main>
          <h1>Task board</h1>

          <form onSubmit={addTask}>
            <label htmlFor="new-task">New task</label>
            <input
              id="new-task"
              value={draft}
              onChange={event => setDraft(event.target.value)}
              required
            />
            <button type="submit">Add task</button>
          </form>

          <ul aria-label="Task list">
            {tasks.map(task => (
              <li key={task.id}>
                <label>
                  <input
                    type="checkbox"
                    checked={task.completed}
                    onChange={() => toggleTask(task.id)}
                  />
                  <span>{task.title}</span>
                </label>

                {task.completed && <span>Completed</span>}

                <button
                  type="button"
                  aria-label={`Remove ${task.title}`}
                  onClick={() => removeTask(task.id)}
                >
                  Remove
                </button>
              </li>
            ))}
          </ul>

          <p role="status">
            {tasks.length === 0
              ? 'No tasks yet.'
              : `${tasks.length} ${tasks.length === 1 ? 'task' : 'tasks'}`}
          </p>
        </main>
      );
    }

The form has an associated label, the checkbox gets its accessible name from the task title, and the remove button has a name that identifies its task. The status text tells users whether the list is empty or how many tasks it contains.

## 2. Test the complete pointer flow

Create src/test/TaskBoard.test.tsx:

    import { render, screen, within } from '@testing-library/react';
    import userEvent from '@testing-library/user-event';
    import { describe, expect, it } from 'vitest';
    import { TaskBoard } from '../TaskBoard';

    describe('TaskBoard', () => {
      it('adds, completes, and removes a task', async () => {
        const user = userEvent.setup();
        render(<TaskBoard />);

        const title = 'Review interaction tests';
        const taskField = screen.getByRole('textbox', {
          name: 'New task',
        });

        await user.type(taskField, title);
        await user.click(screen.getByRole('button', { name: 'Add task' }));

        const taskList = screen.getByRole('list', { name: 'Task list' });
        const taskItem = within(taskList).getByRole('listitem');
        const checkbox = within(taskItem).getByRole('checkbox', {
          name: title,
        });

        expect(within(taskItem).getByText(title)).toBeTruthy();
        expect(screen.getByRole('status').textContent).toBe('1 task');

        await user.click(checkbox);

        expect(checkbox.checked).toBe(true);
        expect(within(taskItem).getByText('Completed')).toBeTruthy();

        await user.click(
          within(taskItem).getByRole('button', {
            name: `Remove ${title}`,
          }),
        );

        expect(within(taskList).queryByText(title)).toBeNull();
        expect(screen.getByRole('status').textContent).toBe('No tasks yet.');
      });
    });

The test checks the task after creation, its checked and completed state, and its removal. Each assertion names an observable checkpoint, so a failure points to a particular part of the flow.

## 3. Test keyboard form submission

Add this test inside the same describe block:

    it('adds a task using keyboard navigation and Enter', async () => {
      const user = userEvent.setup();
      render(<TaskBoard />);

      const taskField = screen.getByRole('textbox', {
        name: 'New task',
      });

      await user.tab();
      expect(document.activeElement).toBe(taskField);

      await user.type(taskField, 'Practice keyboard submission');
      await user.tab();

      const addButton = screen.getByRole('button', { name: 'Add task' });
      expect(document.activeElement).toBe(addButton);

      await user.keyboard('{Enter}');

      const taskList = screen.getByRole('list', { name: 'Task list' });
      expect(
        within(taskList).getByText('Practice keyboard submission'),
      ).toBeTruthy();
      expect(screen.getByRole('status').textContent).toBe('1 task');
    });

The test follows the actual tab order from the page, checks focus before submission, and presses Enter on the submit button. It verifies the resulting list and status instead of calling addTask directly.

## 4. Run the tests

From the project root, run:

    npm test

If only the keyboard test fails, inspect whether the rendered controls have the expected tab order and whether focus is being moved or restored by application code. If a test needs to wait for a later result in a future asynchronous version of this flow, await a findBy query or an appropriate waitFor assertion instead of adding a fixed sleep.

## 5. Extend the flow without coupling it to internals

Useful next cases could include:

- Editing a task and confirming its new visible title.
- Confirming a task cannot be added when the field is blank.
- Checking that completing one item leaves another item unchanged.
- Testing a keyboard route for completing or removing a task.

Keep each test focused on one goal and preserve meaningful checkpoints. Do not assert hook state, call the addTask function, or select the checkbox through a private class name.

A DOM-based test does not prove that list items have correct visual spacing, that a coordinate hits the intended button, or that a real browser's native form behavior is identical. Add a browser-level or manual check if those details are part of the requirement.

## Review checklist

- A new user-event session is created inside each test.
- Every user-event operation is awaited.
- The form field and buttons are found by accessible names.
- The task is checked through the actual checkbox.
- The remove action is scoped to the matching list item.
- Assertions mark creation, completion, and removal checkpoints.
- Keyboard navigation and Enter submission are exercised separately.
- Tests check rendered behavior rather than component handlers or private state.
- Browser layout and assistive-technology requirements are checked in suitable environments.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="101-component-tests-and-test-utilities.md">101 - React Example: Component Tests and React Test Utilities</a></td><td><a href="../lessons/102-testing-user-interactions.md">Lesson</a></td><td><a href="../qnas/102-testing-user-interactions.md">Q&amp;A</a></td><td><a href="103-testing-forms-and-async-ui.md">103 - React Example: Testing an Async Username Form and API Boundary</a></td></tr>
  </tbody>
</table>
