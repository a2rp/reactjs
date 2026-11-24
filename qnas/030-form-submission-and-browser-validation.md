# 030. Form submission and browser validation - Q&A

## Submission flow

### 1. What submits a form?

A user can activate a submit button or use a supported keyboard action such as pressing Enter in a field. The browser then runs interactive constraint validation if it is enabled, and dispatches a submit event when validation passes.

### 2. What happens when native validation finds an invalid control?

The browser reports the problem and normally does not dispatch the form's submit event. As a result, a React `onSubmit` handler does not run for that invalid attempt unless native interactive validation has been disabled.

### 3. Why should submission logic be attached to the form's `onSubmit`?

The form submit event represents submission regardless of which submit control or keyboard action initiated it. A handler attached only to one button can miss other valid ways to submit.

### 4. What does a form do by default after a valid submission?

The browser sends the named control values to the form's `action` destination using its `method`, then follows the normal navigation behavior. The default method is `GET`.

### 5. What does `event.preventDefault()` do in `onSubmit`?

It cancels the browser's default navigation or reload, allowing the client-side handler to process the values. It does not stop event propagation.

### 6. What is `event.currentTarget` inside a form's `onSubmit` handler?

It is the form element whose handler is running. It can be passed to `new FormData(event.currentTarget)` to collect that form's controls.

### 7. Should I read controlled values from `FormData` or React state?

Either can be appropriate. Read controlled values from React state when that state is already the source used by the component. Use `FormData` when collecting named DOM controls as a group, including uncontrolled fields.

## Form data

### 8. What does a control need to be included usefully in `FormData`?

It needs a `name`. The name becomes the key for its value. A visible label is also important, but labels and form-data names serve different purposes.

### 9. What is the difference between `FormData.get()` and `FormData.getAll()`?

`get(name)` returns one entry for that name. `getAll(name)` returns every entry with that name, which is useful for multiple selects or checked checkbox groups.

### 10. Are all `FormData` entries strings?

No. Most text-like control values are strings, but file inputs contribute `File` values. Use the API appropriate to the control type.

### 11. Do unchecked checkboxes appear in `FormData` as false?

No. An unchecked checkbox is omitted. If the application needs a boolean, use controlled state or interpret the absence of that name as unchecked.

### 12. Are disabled controls included in `FormData`?

No. Disabled controls are omitted from form data and cannot be interacted with. Read-only text controls are different: where supported, they remain focusable and are included.

### 13. Does `FormData` include the submit button's own value?

The activated submit button can contribute its `name` and `value`. Other submit buttons that were not used do not contribute as the submitter.

### 14. Does `new FormData(form)` perform the network request?

No. It collects the form's current successful controls into a `FormData` object. Your code can inspect it, send it with an API request, or leave submission to the browser.

### 15. What is the difference between `GET` and `POST` form methods?

A `GET` submission encodes form values in the destination URL, which is useful for searches and other read-only requests. A `POST` submission sends form data in the request body and is commonly used for changes. `POST` does not encrypt data; use HTTPS for transport security.

### 16. Why should sensitive values not be sent in a `GET` URL?

URLs can appear in browser history, copied links, logs, and other places. Use an appropriate method and HTTPS, and handle the values securely on the server. A method choice alone does not protect data.

## Native validation

### 17. What does `required` do?

It makes a supported control invalid while it has no required value. For a select, an empty prompt option can represent the missing choice. For a checkbox, the checkbox must be checked.

### 18. What does `type="email"` validate?

It checks that a non-empty value has a browser-accepted email-like format. It cannot prove the address exists or that the user controls it.

### 19. What do `min`, `max`, and `step` do?

They describe allowed numeric or date limits and increments for supported input types. The browser uses them during constraint validation.

### 20. What do `minLength`, `maxLength`, and `pattern` do?

They define text length limits or a required pattern for supported controls. The browser can use these constraints when validating a submission.

### 21. Does the browser validate before React's submit handler?

When interactive constraint validation is enabled, yes. If a control is invalid, the browser reports it and the submit event does not reach the handler. A form with `noValidate` or a submitter with `formNoValidate` skips that interactive step.

### 22. What does `noValidate` do on a form?

It disables the browser's automatic interactive constraint validation for that form. The submit event can then run even when fields fail their declared constraints, so application code must take responsibility for checks it needs.

### 23. What does `formNoValidate` do on a submit button?

It skips interactive constraint validation for that particular submission. This can be useful for a separate action such as saving an incomplete draft.

### 24. What is the difference between `checkValidity()` and `reportValidity()`?

Both evaluate the form's or control's constraints and return whether they pass. `reportValidity()` also asks the browser to display its validation feedback for invalid controls.

### 25. What does `setCustomValidity(message)` do?

It sets a custom validation error on a control. A non-empty message makes it invalid; pass an empty string to clear that custom error. Accessible custom error messages and summaries are covered in chapter 031.

### 26. Is client-side validation enough to trust submitted data?

No. Client-side checks can be bypassed or changed. Validate and authorize data on the server before relying on it or performing a protected action.

## Buttons and reset behavior

### 27. What is the default type of a button inside a form?

A plain `<button>` inside a form defaults to submitting the form. Explicitly use `type="submit"` for submission and `type="button"` for an unrelated action.

### 28. What does a `type="reset"` button reset?

It restores uncontrolled controls to their initial values. For controlled controls, update the React state as well; otherwise React will render the current state value back into the field.

### 29. Should every form have a reset button?

No. A reset action can discard user input unexpectedly. Add one only when it is useful, label it clearly, and ensure both controlled state and uncontrolled fields reset as intended.

## React submission approaches

### 30. Does React's `onSubmit` handler work in older React versions?

Yes. `onSubmit` and `event.preventDefault()` are the manual event-handler approach and work across React versions.

### 31. What is a function form `action` in React?

Starting with React 19, a function can be passed to a form's `action` prop. React calls it with the submitted `FormData` and manages the action as a Transition. The later Actions chapters cover pending state and asynchronous workflows.

### 32. Does a function form `action` use the same flow as manual `onSubmit`?

Not exactly. A function `action` is managed by React. Calling `preventDefault()` in `onSubmit` cancels the submission, so the action will not run if the event is prevented.

### 33. What happens to uncontrolled values after a successful function action?

React resets the form's uncontrolled fields to their initial values after the action succeeds. Store values in state or use the later Actions patterns when they need to remain visible.

### 34. What is the difference between `form.submit()` and `form.requestSubmit()`?

`requestSubmit()` behaves like activating a submit button: it runs applicable constraint validation and dispatches a submit event when valid. `submit()` bypasses interactive constraint validation and does not dispatch the submit event.

### 35. Where can I learn the React Actions APIs in more detail?

Continue with [React form Actions and asynchronous submissions](../lessons/078-react-form-actions.md), [Managing submissions with useActionState](../lessons/079-useactionstate.md), and [Pending form state with useFormStatus](../lessons/080-useformstatus.md).

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [029 - Text inputs, selects, checkboxes, and radio buttons](../lessons/029-inputs-selects-checkboxes-and-radio-buttons.md) | [030 - Form submission and browser validation](../lessons/030-form-submission-and-browser-validation.md) | [Open Example](../examples/030-form-submission-and-browser-validation.md) | [031 - Accessible validation messages and error summaries](../lessons/031-accessible-validation-messages.md) |
