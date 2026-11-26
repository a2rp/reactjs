# React Form Actions and Asynchronous Submissions - Q&A

## Native forms and React Actions

### 1. What does a browser form do?

It groups named controls and submits their values to a URL or to a JavaScript handler. A normal URL submission may navigate or reload the page.

### 2. What is FormData?

A browser object containing the submitted successful controls. Fields need name attributes; repeated names can have multiple values.

### 3. What does FormData.get(name) return?

A string, a File, or null. Check the value's type before using string operations. Use getAll(name) for repeated fields.

### 4. What is a React form Action?

A function passed to a form's action prop. React calls it with the submitted FormData and runs the work as an Action in a Transition.

### 5. Does a form Action need event.preventDefault()?

No. A function passed to action takes over the submission through React. Do not also manually intercept the same submission unless you intentionally choose the onSubmit model.

### 6. What is the difference between action="/search" and action={search}?

A URL uses native browser form submission. A function is called by React with FormData and has React Action behavior.

### 7. Which method does a function Action use?

React function actions use POST semantics, regardless of a form's method prop. A URL action follows native form method behavior.

### 8. How does a submit button override the form Action?

Give the button a formAction function. That function is used for that submission instead of the form's default action.

### 9. Does the submit button's name/value get submitted?

Yes, the activated submitter may contribute its name and value. This can distinguish which button submitted the form.

### 10. What happens to a successful form's uncontrolled fields?

React resets them to their default values after a function Action succeeds.

## Pending state and asynchronous work

### 11. How can a submit button know that its form is pending?

A child component inside the form can call useFormStatus from react-dom and read the pending property.

### 12. Why must useFormStatus be in a child of the form?

It reads status from a parent form. A component cannot call it and expect it to observe a form that the same component is rendering.

### 13. What other data does useFormStatus provide?

It provides status information such as submitted data, method, and the parent form's action reference, in addition to pending.

### 14. Does a function Action have to be synchronous?

No. It can return a Promise or be declared async. React keeps the Action pending while asynchronous work is underway.

### 15. Should the whole form be disabled while pending?

Only if that is appropriate for the interaction. Often disabling the submit control prevents duplicate clicks while leaving navigation or unrelated controls usable.

### 16. What if users submit repeatedly?

Choose a deliberate policy. Show pending feedback and prevent accidental duplicate submissions where useful. For independently ordered operations, the application or data library may need explicit concurrency rules.

### 17. Does an Action block all user interaction?

No. React runs it as a Transition so the interface can remain responsive while the work is pending.

## Form values, validation, and errors

### 18. Which fields appear in FormData?

Successful form controls with name attributes. Disabled or unnamed controls are omitted; unchecked checkboxes are omitted; file controls provide File objects.

### 19. Do HTML constraints replace server validation?

No. required, minLength, type, and pattern improve client usability but can be bypassed. Validate the submitted values again at the trusted server boundary.

### 20. What if FormData.get returns null?

The field may be absent, disabled, unnamed, or otherwise not submitted. Handle the missing value rather than assuming it exists.

### 21. What if a field uses the same name multiple times?

Use FormData.getAll(name) to get every value. get(name) returns only one matching value.

### 22. Does returning a validation message count as an Action failure?

No. If the function resolves normally with a validation result, React considers that a successful Action. Uncontrolled fields are reset after success.

### 23. How can a form preserve values after validation fails?

Preserve them deliberately with React state, controlled fields, refreshed defaults, or a stateful form Action pattern such as useActionState.

### 24. What happens if an Action throws or rejects?

React sends the error to the nearest Error Boundary. Use that for unexpected failures and provide a useful recovery path.

### 25. Should expected validation errors be thrown?

Usually not. Expected validation feedback is better represented as structured state that the user can correct. The next lesson covers managing Action results and pending state.

### 26. Does a Suspense fallback show for a failing form submission?

No. Suspense handles supported pending rendering. An Error Boundary or explicit form state handles submission errors.

## Client and server behavior

### 27. Is every function passed to action a Server Function?

No. A function defined in client code is a client Action. A Server Function requires a framework integration that supports server execution.

### 28. Can a plain Vite app use a Server Function by adding a directive?

No. Server Functions need framework/build and deployment support. A directive alone does not create a server endpoint.

### 29. What is progressive enhancement for a form?

The core form can submit before client JavaScript loads. Framework-supported Server Functions can enable this behavior for function-based Actions.

### 30. Are hidden fields trusted?

No. Hidden inputs are still submitted by the client and can be changed. Treat every submitted value as untrusted.

### 31. What must a server Action verify?

Validate input, authenticate the caller, and check that the current user is authorized to perform the requested operation.

### 32. Can an Action take extra values besides FormData?

A client closure can read props or state. Server Function patterns can pass additional serializable data through framework-supported means such as bound arguments or hidden fields, but the server must still validate and authorize.

## Choosing a submission pattern

### 33. When should I use onSubmit instead?

Use it when direct event access, a non-React integration, or compatibility with older React is required. Call preventDefault when taking over native submission.

### 34. What if both onSubmit and a function action are present?

React runs onSubmit first, then the Action unless onSubmit calls preventDefault. Avoid two handlers that both send the same request.

### 35. When should I use a URL action?

When standard browser navigation/submission is the intended behavior, such as a GET search or a server endpoint designed for native HTML forms.

### 36. How should I choose between controlled and uncontrolled fields?

Use uncontrolled fields when the DOM can own their current values and reset-on-success is useful. Use controlled fields when React must preserve, transform, or coordinate each value.

### 37. Why might fields disappear after a validation response?

A fulfilled Action that returns an error object or message still counts as success for the reset behavior. Preserve field values intentionally.

### 38. What is a reasonable pending label?

A short description of the operation, such as “Saving…”. Keep it clear to sighted and assistive-technology users.

### 39. What should a multiple-action form communicate?

Each submit button should clearly state what it will do, and the server must enforce permissions for every action.

### 40. What should I check before shipping a form Action?

Check names and FormData types, validation, authorization, pending feedback, field-reset behavior, error recovery, duplicate submission behavior, and whether the action is client-side or server-side.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="077-async-ui-and-framework-data-loading.md">077 - Async UI, Suspense, and Framework Data Loading - Q&amp;A</a></td><td><a href="../lessons/078-react-form-actions.md">Lesson</a></td><td><a href="../examples/078-react-form-actions.md">Example</a></td><td><a href="079-useactionstate.md">079 - Managing Submissions with useActionState - Q&amp;A</a></td></tr>
  </tbody>
</table>
