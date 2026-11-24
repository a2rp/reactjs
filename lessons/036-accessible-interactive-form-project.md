# 036. Build an accessible interactive form feature

## Goal

This chapter brings the earlier form lessons together in a small course-question form. The form collects a name, email address, topic, and question. It uses controlled React state, clear instructions, validation, inline errors, an error summary, keyboard-accessible navigation, and success feedback.

This is an integration project. The chapters on form controls, submission, validation messages, multi-field state, refs, files, and browser APIs explain the individual techniques. Here, the goal is to make them work together as one understandable feature.

The example performs only local validation. It does not send or store the submitted question. A real application must connect submission to a server and validate the received data there.

## Learning outcomes

After this chapter, you should be able to:

- Turn a form requirement into labeled controls, state, validation rules, and feedback.
- Keep the UI state focused on field values, touched fields, validation errors, and submission feedback.
- Associate visible labels, instructions, and error text with their controls.
- Present a text error summary with links to invalid fields.
- Move keyboard focus to the error summary after an invalid submission.
- Announce successful client-side feedback without relying on color or an icon.
- Explain what belongs in client validation and what the server must check.

## Project brief

Build a form where a learner can send a question about the course. It has four required fields:

| Field | Purpose | Rule |
| --- | --- | --- |
| Name | Identify the learner | Must not be blank after trimming whitespace |
| Email address | Provide a reply address | Must not be blank and must have a browser-valid email format |
| Topic | Route the question | Must select one of the available topics |
| Question | Explain what help is needed | Must contain at least 20 non-space characters and no more than 1,000 characters |

The interface should explain which fields are required and how long the question must be. If submission fails, show actionable field errors and a summary of all errors. After a valid local submission, show a clear message that says the data was not actually sent.

## Plan the state

Keep the values in one object because the fields belong to the same form:

~~~jsx
const initialValues = {
  name: '',
  email: '',
  topic: '',
  question: '',
};
~~~

The form may also need:

- `touched`: which fields the learner has left.
- `errors`: the current validation messages.
- `submitted`: whether a submit attempt has happened.
- a count or identifier for failed submissions, so the error summary can be focused after each failed attempt.
- a status message after a valid local submission.

Do not store values that can be calculated during rendering. For example, the question's character count is `values.question.length`; it does not need its own state variable.

## Use a real form and visible labels

Use a semantic form and native controls. Each label must identify its control, and instructions should appear before the fields that depend on them.

~~~jsx
<p id="form-help">
  All fields are required. The question must contain at least 20 characters.
</p>

<form onSubmit={handleSubmit} noValidate>
  <label htmlFor="name">Name (required)</label>
  <input id="name" name="name" required autoComplete="name" />

  <label htmlFor="email">Email address (required)</label>
  <input
    id="email"
    name="email"
    type="email"
    required
    autoComplete="email"
  />

  <button type="submit">Send question</button>
</form>
~~~

In a controlled React form, give each field a stable value and update it from `onChange`. The control's `name` should match the corresponding property in the values object. Use a select for the topic and a textarea for the question.

The example uses `noValidate` so that it can present all custom errors together rather than stopping at the browser's first interactive validation message. The `required` and `type="email"` attributes still describe the controls and provide browser validity information. Custom messages do not remove the need for server-side checks.

## Decide when errors appear

Avoid showing errors for every untouched empty field as soon as the page loads. A practical flow is:

1. Before interaction, show the labels and instructions, but no error messages.
2. When a field loses focus, mark it touched and show that field's current error if it is invalid.
3. When the form is submitted, validate every field and show every remaining error.
4. After a failed submit, update the relevant errors as the learner edits the fields.
5. When all fields pass, replace the error summary with an accurate status message.

This gives people time to enter information while still making the form's problems clear. Validation timing depends on the task; a password-strength meter or a constrained date may need different feedback.

## Validate with specific messages

Keep validation in a function that returns a map of field names to messages. It should not change React state or the DOM itself.

~~~js
function validate(values, emailHasTypeMismatch) {
  const errors = {};

  if (!values.name.trim()) {
    errors.name = 'Enter your name.';
  }

  if (!values.email.trim()) {
    errors.email = 'Enter an email address.';
  } else if (emailHasTypeMismatch) {
    errors.email = 'Enter an email address in a valid format.';
  }

  if (!values.topic) {
    errors.topic = 'Choose a topic.';
  }

  const question = values.question.trim();

  if (!question) {
    errors.question = 'Enter your question.';
  } else if (question.length < 20) {
    errors.question = 'Enter at least 20 non-space characters.';
  }

  return errors;
}
~~~

The email control can report `validity.typeMismatch`. This avoids writing a homemade email-address regular expression and lets the browser apply its email input syntax rules. The server still needs its own input validation.

Validation errors should name the problem and give a next step. Avoid messages such as “Invalid input” or “Something went wrong” when the specific correction is known.

## Connect help and errors to each control

Give help text and error text stable IDs. Include the relevant IDs in `aria-describedby`. Only set `aria-invalid` after the field has actually failed validation and the message is being shown.

~~~jsx
const emailErrorIsVisible = Boolean(
  errors.email && (touched.email || submitted),
);

<label htmlFor="email">Email address (required)</label>
<p id="email-help">Use an address where we can reply.</p>
<input
  id="email"
  name="email"
  type="email"
  required
  value={values.email}
  aria-invalid={emailErrorIsVisible ? 'true' : undefined}
  aria-describedby={
    emailErrorIsVisible ? 'email-help email-error' : 'email-help'
  }
  onChange={handleChange}
  onBlur={handleBlur}
/>
{emailErrorIsVisible && <p id="email-error">{errors.email}</p>}
~~~

The label, helper, and error do different jobs:

- The label names the control.
- The helper gives instructions that apply before or during entry.
- The error explains what failed and how to correct it.

Do not use placeholder text as the only label or instruction. Do not rely on a red border or an icon without a text explanation.

## Build a useful error summary

After a failed submit, place an error summary before the form fields. It should:

- Say that the form needs attention and state the number of errors.
- List every error using text.
- Identify each field by name.
- Link each summary item to its field's ID.
- Receive keyboard focus after the failed submission.

A summary link can use a normal fragment target:

~~~jsx
<li>
  <a href="#email">Email address: Enter an email address.</a>
</li>
~~~

Give the summary container `tabIndex={-1}` and focus it after React renders the failed-submit state. This lets the user hear the summary and then follow a link to a field. If you focus the summary directly, avoid also making the same container an assertive alert; duplicate announcements can be confusing. Each inline message remains associated with its field through `aria-describedby`.

## Announce the successful result

A status message should be visible text and programmatically exposed when it appears. In React, a conditional status paragraph is a simple option:

~~~jsx
{status && <p role="status">{status}</p>}
~~~

Use a polite status for routine confirmation. Do not claim that a message was sent if the example did not contact a server. For this local exercise, say that the values passed the local checks and no data was sent.

For a real network submission, add pending and failure states, prevent duplicate submission while pending where appropriate, and report the server's actual result. Keep error and success text visible long enough for people to read it.

## Handle submission as a user action

A submit handler can prevent the browser's default navigation, validate the current values, focus the summary on failure, and report the local example's result when the values pass:

~~~jsx
function handleSubmit(event) {
  event.preventDefault();

  const nextErrors = validate(
    values,
    emailRef.current?.validity.typeMismatch ?? false,
  );

  setErrors(nextErrors);
  setSubmitted(true);
  setStatus('');

  if (Object.keys(nextErrors).length > 0) {
    setFailedSubmitCount((count) => count + 1);
    return;
  }

  setStatus('Your entries passed local checks. This example did not send data.');
}
~~~

This example keeps controlled values in React state and validates those values on submission. Another form can build a `FormData` object from the form element. Choose a consistent approach and give every submitted control a `name`.

If a real server endpoint is connected, send only the data the feature needs. Handle rejected requests and server validation responses, and never treat client-side validation as a security boundary.

## Keep the client and server responsibilities clear

Client-side validation improves feedback, but the browser is under the user's control. A request can be changed or sent without using this page. A server must validate the received values, enforce authorization and business rules, and safely store or process data.

Do not place secrets in client code. Avoid collecting information the feature does not need. Decide what happens to personal information before connecting a real submission endpoint.

## Review the feature with real interactions

Check the complete flow with a keyboard:

1. Tab through the page and confirm each field and button can be reached.
2. Confirm that every input has a visible, descriptive label.
3. Submit the empty form and inspect the summary, error text, focus location, and links.
4. Follow a summary link and confirm it moves to the matching control.
5. Correct one field at a time and confirm its message clears when it becomes valid.
6. Submit valid values and confirm that the status accurately describes what happened.
7. Resize or zoom the page and confirm labels, instructions, errors, and controls remain usable.

Check that information is not communicated by color alone. If you use assistive technology, verify that instructions, field errors, and the result are announced in a useful order. A code review or automated checker cannot replace trying the actual keyboard and assistive-technology flow.

## Acceptance criteria

The project is ready when:

- The form has a heading, an introduction, and visible instructions.
- Every field has a visible label and a unique ID.
- Required fields are marked in text and with the native `required` attribute.
- Email uses the email input type, and the question has a clear length limit.
- Errors identify the affected field and explain how to correct it.
- Field help and visible errors are associated using `aria-describedby`.
- `aria-invalid` appears only while a relevant error is visible.
- Failed submission shows a linked summary and moves focus to it.
- A valid local submission produces a polite, truthful status message.
- The code clearly states that there is no server submission in this exercise.
- A real server would repeat all important validation and authorization checks.

## Summary

An accessible form is a complete interaction: people need to understand what each field asks for, correct mistakes, navigate between errors, and know what happened after submission. Combine native HTML controls with React state, clear text, associated instructions and errors, deliberate focus behavior, and truthful status feedback. Treat client validation as user assistance and repeat validation at the server boundary.