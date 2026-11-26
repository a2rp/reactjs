# Build and Ship a Complete Accessible React Application

This capstone brings together the React skills from the course by building a small task tracker. The goal is not to use every React API. It is to turn a clear product requirement into a maintainable interface, keep state minimal, handle real interaction states, support keyboard and assistive technology users, and prepare the application for a responsible release.

## What you will build

The application lets a visitor:

- add a task;
- mark a task as complete or active;
- delete a task;
- search tasks by title;
- filter by all, active, or completed tasks;
- clear completed tasks; and
- keep tasks in that browser using local storage.

The capstone uses a task list as a learning project. Browser local storage is not a secure database, a backup, or cross-device synchronization. Do not store secrets or sensitive personal data there. A production multi-user application needs an authenticated backend, authorization, server-side validation, and a deliberate data-retention policy.

## Learning outcomes

By the end, you should be able to:

- translate a small feature brief into a component tree and data model;
- identify the minimum state and derive counts and filtered lists from it;
- use controlled form inputs and immutable state updates;
- keep components focused and pass data and callbacks through props;
- synchronize state with browser storage using an Effect;
- implement labels, fieldsets, keyboard interactions, focus indicators, and status messages;
- handle empty states, invalid input, and unavailable browser storage; and
- review an application before building and deploying it.

## 1. Turn the feature brief into requirements

Before writing JSX, decide what counts as done.

### Functional requirements

1. A task has an ID, a title, and a completion state.
2. A title is required, trimmed before saving, and limited to 120 characters.
3. A visitor can change completion state and delete a task.
4. Search matches the title without regard to letter case.
5. The selected filter is All, Active, or Completed.
6. Clear completed removes only completed tasks.
7. The app derives the visible list and counts from the current tasks, search, and filter.
8. A storage failure does not prevent the core task interactions from working.

### Accessibility and usability requirements

- Every input has a visible, programmatically associated label.
- Filter options use a fieldset and legend.
- Buttons have clear names and can be reached and activated with a keyboard.
- Focus is visible.
- Validation feedback is connected to the form field.
- Dynamic feedback uses appropriate text and does not rely on color alone.
- Empty data and empty search results have different explanations.
- The layout works at narrow and wide viewport sizes.

Writing these requirements first gives you a way to review the finished app beyond “it renders.”

## 2. Design the data and component structure

A task is plain data:

```js
{
  id: "stable-unique-id",
  title: "Read the React lesson",
  completed: false
}
```

One possible component tree is:

```text
App
├── Header
├── TaskForm
├── TaskFilters
├── TaskSummary
└── TaskList
    └── TaskItem
```

Start with a single `App` if that keeps the first version easy to understand. Split a component when it has a clear responsibility, reusable behavior, or enough complexity to obscure the parent. Components should receive task data and event callbacks through props rather than reaching into unrelated state.

## 3. Keep state minimal

The state that changes over time is:

- the array of tasks;
- the current draft title;
- the search text;
- the selected filter; and
- a form or storage message when needed.

The completed count, active count, and visible tasks are derived values. Do not save them as separate state. Compute them from the task array and current controls. Duplicating facts in state creates opportunities for them to disagree.

For example:

```js
const completedCount = tasks.filter((task) => task.completed).length;
const activeCount = tasks.length - completedCount;

const visibleTasks = tasks.filter((task) => {
  const matchesSearch = task.title
    .toLocaleLowerCase()
    .includes(searchText.trim().toLocaleLowerCase());

  const matchesFilter =
    filter === "all" ||
    (filter === "active" && !task.completed) ||
    (filter === "completed" && task.completed);

  return matchesSearch && matchesFilter;
});
```

If profiling later shows that filtering is expensive, measure the application and consider memoization. Do not add memoization before it is needed.

## 4. Build the static interface before interactions

A useful order is:

1. Render a heading, form, filters, summary, and list from sample data.
2. Make the form controlled and add a task on submit.
3. Add completion, deletion, search, and filter interactions.
4. Add local persistence.
5. Review validation, empty states, storage failures, and keyboard use.
6. Build and verify the production output.

This order lets you check the structure and content before introducing every state transition at once.

## 5. Handle state changes immutably

When a task is completed, create a new array and a new object for the changed task:

```js
setTasks((currentTasks) =>
  currentTasks.map((task) =>
    task.id === taskId
      ? { ...task, completed: !task.completed }
      : task
  )
);
```

Delete by filtering into a new array:

```js
setTasks((currentTasks) =>
  currentTasks.filter((task) => task.id !== taskId)
);
```

Use a stable task ID as the React list key. Do not use the array index for a list where items can be inserted, removed, or reordered.

## 6. Build a reliable form

Use a real `form` and handle `onSubmit` so keyboard submission works. Trim the title before storing it. Reject an empty title and explain how to fix it. The HTML `required` and `maxLength` attributes provide useful browser behavior, while application validation lets the interface present a consistent message.

Keep the visitor's draft when validation fails. Clear it only after the task has been added. Use `useId` when a component needs a unique ID to connect a label, hint, and error message. IDs are for accessibility relationships; the task's data ID should come from the data itself.

## 7. Treat browser storage as an external system

Local storage is outside React. Reading it to initialize the app and writing when task data changes are interactions with an external browser API.

- Parse stored JSON inside `try`/`catch`.
- Validate the parsed value before trusting it.
- Fall back to starter data if the entry is missing or invalid.
- Catch failures when reading and writing; browser privacy settings or storage quotas can make storage unavailable.
- Show a concise warning when persistence cannot be used.
- Continue allowing in-memory task updates if storage fails.

The storage Effect should synchronize the task array with storage. Search and filter results are derived during render, so they do not need an Effect.

## 8. Make states understandable

The interface should distinguish:

- **Initial list:** show a few example tasks or explain how to add the first one.
- **No tasks:** explain that the list is empty.
- **No matches:** explain that the current search or filter found nothing and offer a way to adjust it.
- **Invalid form:** keep the draft and say what needs changing.
- **Storage warning:** say that changes may not persist after this page is closed.
- **Normal list:** show each task with completion and deletion actions.

Do not use a red color alone to communicate an error. Keep text readable, controls large enough to activate, and focus indication visible.

## 9. Review the complete user journey

Review the task tracker with these scenarios:

1. Add a task with leading and trailing whitespace; verify stored title is trimmed.
2. Submit an empty title; verify the visitor gets a clear message.
3. Add a title at the maximum length and try a longer value.
4. Toggle a task by keyboard and by pointer.
5. Delete one task and verify the others remain.
6. Search with different letter casing.
7. Change each filter and confirm the derived counts.
8. Clear completed tasks and verify active tasks remain.
9. Refresh the page and confirm saved tasks return.
10. Use a clean browser profile or remove the saved value to check the initial state.
11. Simulate invalid or unavailable storage and confirm the app still works in memory.
12. Navigate the full interface with a keyboard and check focus and labels.
13. Inspect the layout at a narrow viewport.
14. Check the browser console and production build for errors.

A manual review catches important interaction and accessibility problems. For a maintained application, automate durable behavior with component or browser tests too.

## 10. Prepare for a release

Before shipping:

- Install dependencies from the lockfile and review dependency changes.
- Run the project's production build and resolve build errors.
- Test the deployed build with the configured environment, not only the development server.
- Confirm that client configuration contains no secret values.
- Check that all visible actions have useful outcomes.
- Inspect the app with keyboard navigation and a screen reader where possible.
- Review the app's privacy, storage, and retention choices.
- Monitor errors and performance after deployment.
- Keep a rollback path for a faulty release.

The example chapter contains a complete implementation of this capstone and calls out the places where a real multi-user product needs a backend.

## Common mistakes

- **Saving derived values separately:** calculate filtered tasks and counts from source data to avoid state drift.
- **Mutating an existing task or array:** create new arrays and objects for state updates.
- **Using array indexes as keys:** use the task's stable ID.
- **Putting every input into a separate Effect:** update form state in its event handler; derive values during render.
- **Assuming local storage always works:** catch parse and access errors and preserve in-memory behavior.
- **Persisting sensitive information in local storage:** browser storage is accessible to scripts running on the page and is not an authorization boundary.
- **Showing the same empty message for every case:** distinguish an empty list from a search with no matches.
- **Making controls icon-only without names:** provide visible text or an accessible name.
- **Checking only with a mouse:** verify keyboard, focus, and form submission behavior.
- **Treating a successful dev server as release verification:** build and review the deployed configuration too.

## Completion checklist

The capstone is ready for review when:

- [ ] The component tree has clear responsibilities.
- [ ] State contains only changing source data and interaction values.
- [ ] Counts and filtered tasks are derived.
- [ ] Create, toggle, delete, search, filter, and clear-completed work.
- [ ] Form validation preserves the draft and explains corrections.
- [ ] Browser persistence handles missing, invalid, and unavailable storage.
- [ ] Controls have labels, names, and visible focus.
- [ ] Empty, no-match, error, and storage-warning states are distinct.
- [ ] The app builds and the main journey works in the target browsers.
- [ ] No secret or sensitive data is stored or shipped accidentally.

## Key takeaways

A complete React application is a set of deliberate decisions: product requirements, component boundaries, minimal state, data flow, accessible controls, external-system synchronization, and release checks. React helps render the interface from state; the developer is responsible for making state accurate, actions understandable, and failures recoverable.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="115-production-monitoring-and-maintenance.md">115 - Production Errors, Monitoring, and Maintenance</a></td><td><a href="../qnas/116-react-capstone-project.md">Q&amp;A</a></td><td><a href="../examples/116-react-capstone-project.md">Example</a></td><td><a href="117-react-upgrades-and-version-changes.md">117 - Upgrading React and Handling Version Changes</a></td></tr>
  </tbody>
</table>
