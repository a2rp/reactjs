# 031. Accessible validation messages and error summaries - Q&A

## Writing useful errors

### 1. What should a validation error tell someone?

It should identify the field, describe what is wrong, and give a useful next step. “Email address: enter an address in the format name@example.com” is more helpful than “Invalid input.”

### 2. Should an error message be visible text?

Yes. Visible text helps everyone understand and correct the problem. A red border, icon, tooltip, or aria-invalid attribute alone does not explain what happened.

### 3. Should error messages blame or scold the person?

No. Use neutral language that describes the value or required action. For example, say “Enter a phone number with 10 digits,” not “You entered this incorrectly.”

### 4. Should the message repeat the field label?

It depends on where it appears. Inline text directly beside a clearly labeled field can be concise, but a summary link should make sense on its own. Including the field name in a summary item helps users scan a list of links.

### 5. How much detail should an error include?

Include enough detail to identify the violated rule and correct it. If a password needs 12 characters, state the limit. Avoid unrelated explanations and technical terms the person cannot act on.

### 6. Is a generic “Something went wrong” message enough?

Usually not when the application knows the specific problem. Identify the affected field or operation and explain the correction. A general message can supplement specific field messages for unexpected failures.

## Associating errors with controls

### 7. What does aria-invalid do?

It exposes that a control's current value has failed validation. It does not display an error message or explain how to fix the value, so use it alongside visible error text.

### 8. When should I set aria-invalid to true?

Set it after validation determines that the current value is invalid, such as after a submit attempt or after the user has interacted with the field under your validation policy. Do not set it on every untouched, empty required control at initial render.

### 9. Should I omit aria-invalid or set it to false when there is no error?

Both can communicate a non-invalid state. Many components omit the attribute until a validation error exists. Whichever pattern you choose, make the state consistent and ensure a resolved error is no longer exposed as invalid.

### 10. What does aria-describedby do?

It associates one or more descriptive text elements with a control. A screen reader can use those descriptions when the control is read. It is commonly used for instructions, hints, and field errors.

### 11. Can aria-describedby refer to both help text and an error?

Yes. Put both element ids in its space-separated value, for example aria-describedby="email-help email-error". Include the error id only while that error element exists.

### 12. Does aria-describedby replace a visible label?

No. The label gives the control its name. A description adds supplementary information. Use a properly associated visible label, such as a label with htmlFor="email" and an input whose id is "email".

### 13. Does aria-invalid automatically connect the error text to the field?

No. Give the error element an id and reference it from the control's aria-describedby. The visible message and the invalid state serve different purposes.

### 14. Can I use the same error id for multiple fields?

No. An id must identify one element in the document. Generate or assign a unique, stable id for every field error and reference the matching id from that field.

### 15. Should a field error remain in the DOM when it is not visible?

A simple pattern is to render the error only when it exists and include its id in aria-describedby at the same time. If an element is hidden, make sure the hidden message is not still exposed as an active description.

### 16. Should the input's aria-describedby include the summary item too?

Usually the inline error is the best field description because it stays next to the field and provides the current correction. The summary is a separate navigation aid. Avoid associating multiple duplicate versions of the same error without a reason.

## Error summaries and announcements

### 17. Why show an error summary?

It gives an overview after a failed submission and lets the person go directly to each invalid control. It is particularly useful when a form is long or the invalid fields are spread out.

### 18. Where should the summary appear?

Place it in a predictable location near the start of the form, where it is easy to find after submission. Use a descriptive heading so the section's purpose is clear.

### 19. What should each summary item contain?

Make it a link to the affected control and describe the problem clearly. Include the field name and correction when practical, so the link remains meaningful when read out of context.

### 20. How do I link a summary item to a field?

Give the control a stable id such as email, then use a fragment link such as href="#email". The target id must match exactly. Use link text that describes the problem, not just “Error.”

### 21. Should a dynamically inserted error summary use role="alert"?

It can. An alert announces an important message that appears dynamically. Use it for the newly displayed summary after a failed submission, and avoid stacking several competing live announcements on the same content.

### 22. Should I put role="alert" on every inline error too?

Usually not when a summary is already announced and the inline errors are associated with their fields. Reannouncing every message can be noisy. Ensure errors are available when their controls receive focus.

### 23. Should I move focus to the summary after an invalid submit?

That can be an intentional design choice, but it is not the only approach. A dynamically announced summary may already be conveyed without moving focus. Choose and test one coherent behavior; do not cause duplicate announcements or surprising focus movement.

### 24. Should I move focus to the first invalid field?

It can help some flows, but do it deliberately after submission, not while the person types. Ensure the error is discoverable at that field and that users can still find the summary. Test the actual screen-reader and keyboard behavior.

### 25. What is role="status" useful for?

It exposes a non-urgent update, such as a success message after a form completes. It typically announces the update without moving keyboard focus.

### 26. Should the success message use role="alert"?

Usually not. Success is generally non-urgent, so role="status" is a better fit. Reserve alerts for important, time-sensitive feedback that needs immediate announcement.

## Validation timing and React

### 27. Should I show required-field errors before someone interacts with the form?

Usually no. Show instructions at first, then reveal required-field errors according to a clear policy, often after submission or after the person has interacted with the field. An empty required field is not necessarily an error the person has encountered yet.

### 28. Should validation happen while typing or on blur?

Choose based on the field and the experience. On blur can avoid announcing an error on every keystroke. After a failed submit, revalidate as values change or when a field loses focus so corrected errors do not remain stale.

### 29. How can React know when to show errors?

Keep state that represents whether validation has been attempted and the current errors, or derive errors from values and interaction state. Render the inline text, invalid state, summary, and descriptions from the same current validation result.

### 30. Why can storing values and errors separately cause stale messages?

If an update changes the value but not its corresponding error, the form can show contradictory feedback. Recompute the affected error when values change after submission, and validate all current values again on each submit.

### 31. Should I validate the entire form on every keystroke?

Not necessarily. It can be distracting and can repeatedly change announcements. A common pattern is validate all fields on submit, then update an affected error on blur or after an edit once the person has tried to submit.

### 32. How should I handle React's onInvalid event?

React's onInvalid event bubbles, which can let a form-level handler observe invalid controls. The browser's native interactive validation normally prevents the submit event for an invalid form. Decide whether native popups or your custom error interface owns the feedback rather than accidentally showing two systems at once.

### 33. What does noValidate do on a form?

It disables the browser's automatic interactive validation UI during submission. It lets a custom submit handler render the application's own error messages. It does not make invalid values valid or remove the need to validate them.

### 34. Can I still inspect native constraints when using custom errors?

Yes. Controls expose validity information such as whether a required value is missing or a value does not match its input type. Your custom handler can use that information to produce clearer application messages.

### 35. Should I mix native browser popups and a custom summary?

Avoid showing two competing feedback systems for the same submission. Choose whether the browser's native UI or your custom flow presents errors, and make sure the chosen flow explains every problem accessibly.

## Server and testing

### 36. Do accessible client-side errors replace server validation?

No. Client-side checks improve the interaction, but requests can bypass the page or alter its code. The server must validate input and enforce authorization independently.

### 37. How should I show a server-side field error?

Map the response to the corresponding field, show visible inline text, set that field invalid, associate the error with aria-describedby, and update the summary if the submission failed. Keep server messages specific and safe to display.

### 38. What if the server returns an error that is not about one field?

Show a clear form-level message that explains what happened and what the person can do next. For example, a temporary service problem may offer a retry. Do not attach it to an unrelated field.

### 39. How should I test accessible error feedback?

Submit invalid data using a keyboard, read the summary, activate each link, and confirm the destination field exposes the right label, invalid state, help, and error. Also test corrected values, success feedback, and the screen-reader announcement behavior when available.

### 40. What are the most important rules to remember?

Use specific visible text, connect each inline error to its field, show a useful summary after a failed submit, avoid marking untouched fields invalid too early, announce updates without excessive repetition, and validate again on the server.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [030 - Form submission and browser validation](../lessons/030-form-submission-and-browser-validation.md) | [031 - Accessible validation messages and error summaries](../lessons/031-accessible-validation-messages.md) | [Open Example](../examples/031-accessible-validation-messages.md) | [032 - Managing state for multi-field forms](../lessons/032-multi-field-form-state.md) |
