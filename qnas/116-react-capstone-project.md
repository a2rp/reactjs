# Q&A: Build and Ship a Complete Accessible React Application

## 1. What application does this capstone build?

A small task tracker. A visitor can add tasks, mark them active or complete, delete them, search titles, filter by status, clear completed tasks, and keep the list in the current browser with local storage.

## 2. Why start with requirements before writing components?

Requirements clarify what the app must do and how people should use it. They give you concrete checks for functionality, accessibility, empty states, storage failure, and release readiness instead of relying only on whether the page renders.

## 3. What is the task data shape?

Each task has a stable ID, a title, and a completion boolean, for example `{ id: "task-123", title: "Read the lesson", completed: false }`. Avoid storing values that can be derived from those fields.

## 4. What state does this task tracker need?

The task array, current title draft, search text, selected filter, and any transient form or storage messages. Counts and filtered results can be computed from that state and do not need separate state variables.

## 5. Why should the app not store the active count and completed count separately?

They can be calculated from the task array. Keeping duplicate facts in state creates an extra update obligation and can make the displayed counts disagree with the actual tasks.

## 6. Should the filtered task list be saved in state?

Usually not. It is derived from the current tasks, search text, and selected filter. Compute it during rendering. Add memoization only if measurement shows the calculation is expensive.

## 7. Where should shared task state live?

Keep it in the closest common parent that needs to coordinate the list and its controls-often the top-level app for this small example. Pass task values and callbacks to child components through props. If a larger app develops distant consumers, consider a suitable shared-state design then.

## 8. Why use a stable task ID as a React key?

A stable key lets React match each rendered item to the same data item when tasks are inserted or removed. An array index can refer to a different task after the list changes and cause state or DOM mismatches.

## 9. Why update a task array immutably?

Create a new array and a new object for the changed task. This keeps state transitions explicit and lets React compare state updates predictably. Do not mutate an existing task and then pass the same array back.

## 10. What is a controlled input?

A controlled input gets its current value from React state and updates that state in its `onChange` handler. It is useful when the app needs the current draft for validation, submission, or other UI behavior.

## 11. Why submit through a form's `onSubmit` handler?

A real form supports keyboard submission and integrates with browser form behavior. Use a submit button and prevent the page's default navigation when handling the submission in a client-side React app.

## 12. Why trim a task title before saving it?

Trimming prevents a title made only of spaces from being accepted and avoids storing accidental leading or trailing whitespace. Keep the draft intact when validation fails so the visitor can correct it.

## 13. Are `required` and `maxLength` enough validation?

They provide useful browser validation, but the application should also check the trimmed value before adding it. Client-side checks improve the interface; a real server must independently validate submitted data.

## 14. What is `useId` useful for in this project?

It can create stable unique IDs for relationships between a form control, its visible label, hint, and error text. It is for accessibility relationships, not for generating task IDs or React list keys.

## 15. Why distinguish an empty list from no search matches?

An empty list means there are no tasks yet and suggests adding one. No matches means tasks exist but the current query or filter hides them; the useful action is to change or clear those controls.

## 16. Why is local storage treated as an external system?

It is a browser API outside React. Reading and writing it are interactions with that external system. A React Effect can keep the stored task data synchronized when tasks change.

## 17. Which data needs an Effect in this app?

Synchronizing tasks with local storage needs an Effect because storage is external to React. Search results and task counts are calculations from current state and should be derived during rendering rather than copied into state by an Effect.

## 18. Why catch errors while reading local storage?

Stored data may be malformed, JSON parsing can fail, or browser policies can block access. Catch the error, validate the parsed value, and start with safe in-memory data rather than making the whole app unusable.

## 19. Why validate data loaded from storage?

Browser storage can contain old, malformed, or manually modified values. Check that the value is an array and that each item has the expected fields and types before trusting it.

## 20. What should happen if local storage is unavailable?

The app should keep working in memory and display a concise warning that changes may not persist after the page closes. A storage failure should not disable adding, completing, or deleting tasks during the current visit.

## 21. Is local storage an appropriate place for passwords or private data?

No. Local storage is available to JavaScript running on the same origin and is not a secure storage or authorization boundary. Avoid saving secrets and sensitive personal data there.

## 22. Does local storage synchronize tasks across devices?

No. It is specific to a browser profile and origin. A multi-device or multi-user product needs a backend and a deliberate synchronization and conflict-resolution design.

## 23. Should the app call the backend directly from each task row?

Keep data operations at a clear layer. For this local-only capstone, callbacks update parent-owned state. When adding a backend, use a service or data layer with defined request and error behavior rather than embedding ad hoc fetch logic in every presentational child.

## 24. What must a real backend validate?

It must authenticate and authorize the caller, validate task fields and lengths, handle concurrent updates as needed, and enforce data-retention rules. Browser validation is not a security control.

## 25. Why include a warning when data cannot persist?

Without the warning, the visitor can reasonably assume their tasks will still exist later. State the consequence in plain language while keeping the current in-memory session usable.

## 26. How should “clear completed” work?

Filter out completed tasks and preserve active ones. If there are no completed tasks, hide or disable the action so the interface does not offer work that cannot change anything.

## 27. What makes task controls accessible?

Each input has an associated visible label; grouped radio or checkbox choices use a fieldset and legend; buttons have understandable names; keyboard focus is visible; errors are linked to the relevant field; and status changes are communicated in text.

## 28. Why should a delete button's name include the task title?

A list may contain many delete buttons. A name such as “Delete Read the lesson” gives assistive technology users the context of which item the button affects. If the title can be very long or private, use a concise accessible name that still identifies the task within the app.

## 29. Is color alone enough to show completion or an error?

No. Use text, checkbox state, icons with accessible names, or other non-color cues. Color can reinforce meaning but should not be the only way to perceive it.

## 30. Should every component use `memo` or `useMemo`?

No. Start with straightforward code and measure. Memoization adds complexity and is most useful when profiling shows expensive work or avoidable renders that matter to the user.

## 31. Why test storage failure as well as normal storage?

A user may browse privately, have storage disabled, exceed a quota, or have invalid saved data. Reviewing those paths confirms that the app can still function and that its warning accurately describes persistence.

## 32. What should be tested manually with a keyboard?

Move through the heading, form, filters, list actions, and clear-completed control without a pointer. Confirm focus is visible, the form can be submitted, labels make sense, and the current state can be understood.

## 33. What should the production build review include?

Build errors, environment configuration, browser console errors, responsive layout, keyboard and screen-reader behavior, dependency changes, and a smoke test of the main journey in the deployed environment. A working development server alone does not prove the deployed application is healthy.

## 34. Why must a `VITE_*` environment value not contain a secret?

Vite exposes variables with that prefix to browser code. Visitors can inspect the built JavaScript. Use these values only for public configuration and keep secrets on the server.

## 35. What makes this capstone “complete”?

It meets the stated scope: task creation and management, search and filters, derived summary values, persistence with a safe fallback, accessible controls, distinct empty/error states, and a release review. It is not a complete production service for multiple users; that requires backend storage, authentication, authorization, server validation, monitoring, and operational ownership.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="115-production-monitoring-and-maintenance.md">115 - Q&amp;A: Production Errors, Monitoring, and Maintenance</a></td><td><a href="../lessons/116-react-capstone-project.md">Lesson</a></td><td><a href="../examples/116-react-capstone-project.md">Example</a></td><td><a href="117-react-upgrades-and-version-changes.md">117 - Q&amp;A: Upgrading React and Handling Version Changes</a></td></tr>
  </tbody>
</table>
