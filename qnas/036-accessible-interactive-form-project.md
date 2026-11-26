# 036. Build an accessible interactive form feature - Q&A

## Project scope and state

### 1. What is the goal of this project?

Build a course-question form that combines labeled controls, React-managed values, validation, inline feedback, an error summary, keyboard focus, and an accurate submission status.

### 2. Does the example send the learner's question to a server?

No. It checks the values locally and says that no data was sent. A real application needs a server endpoint and server-side validation.

### 3. Why keep the field values in one state object?

The name, email, topic, and question belong to one form. One object makes it straightforward to update a field while preserving the others and to validate or submit the set of values together.

### 4. What other state may the form need?

It may track which fields were touched, the current validation errors, whether submission was attempted, how many invalid submissions occurred, and the current status message. Keep state limited to information needed to change the interface.

### 5. Should the character count have its own state?

Usually no. Derive it from the question value, for example values.question.length. Storing a second copy can make the count and text disagree.

### 6. What does “touched” mean?

A field is touched after the person has interacted with it and moved focus away, commonly handled by the onBlur event. The form can then show that field's error without showing every empty-field error on the initial page.

### 7. Should all errors be visible as soon as the page loads?

Usually no. Start with instructions and no errors. Show a field's error after it is touched, and show all errors after a submit attempt.

## Labels and field instructions

### 8. Why use a visible label for every control?

A visible label identifies the purpose of the control, remains available while someone enters a value, and provides a reliable programmatic name. Placeholder text is not a replacement.

### 9. How do I associate a label with an input in React?

Give the input an id and set the label's htmlFor to the same value. The values must match.

### 10. Why add both visible “required” wording and the required attribute?

The visible wording helps people understand which fields must be completed. The native attribute programmatically communicates the requirement and exposes it to browser validation APIs and assistive technology.

### 11. What is aria-describedby for?

It connects a control to additional descriptive text, such as instructions or a currently visible error. The referenced IDs must exist in the rendered document and be unique.

### 12. Should aria-describedby include the error before the error element exists?

No. Include the error's ID only while the associated error text is rendered. Keep the helper-text ID when the error is absent.

### 13. What does aria-invalid communicate?

It indicates that the current field value has failed validation. Set it when the field has a visible error, and remove it after the error is corrected or no longer shown.

### 14. Should an error be indicated only with color or an icon?

No. Show a text message that identifies the problem and provides a correction when one is known. Color or an icon may supplement the text.

### 15. Should the label be replaced by a placeholder?

No. A placeholder disappears when someone enters text and may be difficult to read. Keep a visible label and provide persistent instructions separately.

## Validation

### 16. What should a useful field error say?

It should make clear which field has a problem, what is wrong, and how to fix it when the correction is known. “Enter at least 20 non-space characters” is more useful than “Invalid.”

### 17. Why trim the name and question before validating them?

Whitespace alone should not satisfy a requirement for a name or a meaningful question. Calling trim checks whether there is non-space content.

### 18. Why use type="email"?

It gives the control email-specific semantics, can present a suitable input method on mobile, and exposes browser validity information such as typeMismatch.

### 19. Does type="email" prove that the address exists?

No. It checks a browser-defined format, not whether the mailbox exists, belongs to the learner, or can receive messages.

### 20. Why use noValidate in the example?

It prevents the browser's automatic interactive validation UI from stopping submission before React's handler can display the project's custom error summary. The example still uses native control attributes and validity information.

### 21. Does noValidate remove required or email validity information?

No. It suppresses the form's automatic validation UI during submission. The attributes remain on the controls, and the control's validity state can still be read by application code.

### 22. Should I write a regular expression to validate email addresses?

Usually not for a basic form. Use an email input and its validity state for browser-level format feedback. The server must independently validate and process the value.

### 23. Why put validation in a separate function?

A validation function that returns an error map is easier to review and reuse from submit, blur, and change handlers. It can calculate errors without changing the DOM or React state itself.

### 24. Should validation happen only when the form is submitted?

Not necessarily. Submit validation is essential. Showing a field's error after blur and updating it as the person corrects the value can make recovery easier without interrupting every keystroke.

### 25. Why should the form show errors after a failed submission as values change?

Once the person asks to submit, they need feedback about whether corrections have worked. Revalidate the edited value and clear its error when it passes.

### 26. Does client-side validation secure a form?

No. Users can change client code or send requests without the page. The server must validate all received values and enforce authorization and business rules.

## Error summary and focus

### 27. What belongs in an error summary?

State that the form needs attention, give the number of issues, and list each affected field with its error text. Each item should link to the corresponding control.

### 28. Why move focus to the summary after an invalid submit?

It brings keyboard users to the newly displayed feedback, so they do not need to search the page to discover that submission failed.

### 29. Why give an error summary tabIndex={-1}?

A negative tab index allows code to focus the summary without inserting it into the normal Tab sequence. The summary links can then lead to individual fields.

### 30. Why should focus happen after React renders the summary?

If the summary is conditionally rendered, it does not exist yet at the instant the submit handler calls a state setter. Focus it after React commits the new error state, for example in an Effect that responds to a failed-submit counter.

### 31. Why can a failed-submit counter help?

If the form is submitted invalid more than once, a boolean may already be true, so setting it to true again does not create a changed dependency. A counter changes for each failed attempt and can trigger focus again.

### 32. Should the focused error summary also use role="alert"?

Avoid adding an assertive alert to the same summary when code already moves focus there. The focus change and alert can announce the same content twice. Choose one clear notification strategy and test it with assistive technology.

### 33. How do summary links move to fields?

Use fragment links whose href matches the invalid control's id, such as href="#email". The target id must be unique and present in the page.

### 34. Should inline errors also be connected to the fields?

Yes. The summary helps locate issues; aria-describedby lets a person learn the specific error while focused on the field.

## Submission and feedback

### 35. Why call preventDefault in the submit handler?

The example handles submission locally and does not navigate to a server endpoint. Preventing the default action keeps the browser from performing its usual form navigation.

### 36. What should a successful demo submission say?

It should accurately say that the values passed local checks and no data was sent. Do not claim that a question was delivered or saved when the example has no server.

### 37. What is role="status" useful for?

It exposes a routine, dynamically added status message so assistive technology can be notified politely. It is appropriate for a confirmation such as a local demo result.

### 38. Should a status message be shown only visually?

No. Keep it as visible text and expose it programmatically. Do not rely on a color change, icon, or animation alone to communicate the result.

### 39. Should the form clear its fields after a valid local submission?

Not necessarily. The example can keep the values visible because nothing was transmitted. A production form should decide deliberately whether to preserve or clear values based on the actual submission result and user needs.

### 40. Should an actual submission use FormData or controlled state?

Either can be appropriate. Controlled state is useful when the UI needs to update React with each value; FormData can read named controls at submit time. Choose one coherent submission path and include name attributes.

## Review and production boundary

### 41. What should I test with a keyboard?

Tab through every control, submit with missing values, inspect where focus moves, follow error-summary links, correct fields, and confirm the status after valid input. Ensure no action requires a pointer.

### 42. Is an automated accessibility checker enough to approve the form?

No. Automated checks can catch some structural issues but cannot fully evaluate whether instructions, error timing, focus, and announcements make sense to people. Manually inspect keyboard and assistive-technology behavior.

### 43. What must change before connecting a real endpoint?

Add a real submission action, handle pending and failure results, prevent unintended duplicate submissions where appropriate, and validate all data on the server. Explain what information is collected and how it is handled.

### 44. Why is a truthful status especially important?

People need to know whether their action succeeded. Saying that data was sent when the demo only checked it locally can cause someone to believe a request was received when it was not.

### 45. What is the overall form review checklist?

Confirm the labels and instructions are visible, errors are connected to their fields, the summary is actionable and focused, the result is announced, keyboard use works, and the interface accurately distinguishes local checks from server submission.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="035-browser-apis-and-event-listeners.md">035 - Working with browser APIs and external event listeners - Q&amp;A</a></td><td><a href="../lessons/036-accessible-interactive-form-project.md">Lesson</a></td><td><a href="../examples/036-accessible-interactive-form-project.md">Example</a></td><td><a href="037-state-with-usestate.md">037 - State with useState - Q&amp;A</a></td></tr>
  </tbody>
</table>
