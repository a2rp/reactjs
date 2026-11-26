# 031. Accessible validation messages and error summaries

## Goal

A validation error should tell the person what went wrong, identify the affected field, and explain how to fix it. In React, accessible validation means showing that information in text and programmatically connecting it to the right control.

This chapter builds on native constraints and form submission from Chapter 030. It covers inline errors, error summaries, validation timing, and success feedback. Managing the state of a larger multi-field form is covered in Chapter 032.

## Learning outcomes

After this chapter, you should be able to:

- Write short, specific, actionable validation messages.
- Associate help text and error text with a form control.
- Set `aria-invalid` only when a field has actually failed validation.
- Show an error summary whose links take users to the invalid fields.
- Choose validation timing that gives useful feedback without interrupting typing.
- Announce successful or asynchronous form feedback accessibly.
- Explain why client-side validation does not replace server-side validation.

## What makes an error message useful?

A useful error message answers three questions:

1. **Where is the problem?** Name the field or make its location obvious.
2. **What is wrong?** Describe the problem in plain language.
3. **How can it be fixed?** Give a concrete next step when possible.

For example, “Invalid input” does not say which value needs attention. “Email must include an address after the @ sign” identifies both the issue and a way to correct it.

Prefer concise, calm, specific language. Avoid blaming the user, relying on color alone, or using an error message that disappears before it can be read. If a field has a constraint such as a minimum length, include the relevant limit in the message.

## Connect an inline error to its field

Give the control a visible label. Give the error its own stable id, then include that id in the control's `aria-describedby` when the error is present. If the field also has help text, reference both ids.

~~~jsx
function EmailField({ email, error, onChange }) {
  const describedBy = error
    ? 'email-help email-error'
    : 'email-help';

  return (
    <div>
      <label htmlFor="email">Email address</label>

      <p id="email-help">
        We will use this address to reply to your request.
      </p>

      <input
        id="email"
        name="email"
        type="email"
        value={email}
        onChange={onChange}
        required
        aria-invalid={error ? 'true' : undefined}
        aria-describedby={describedBy}
      />

      {error && <p id="email-error">{error}</p>}
    </div>
  );
}
~~~

The visible label tells everyone what to enter. `aria-describedby` associates the supplementary help and current error with the input for assistive technology. The error is also visible text, so people who do not use a screen reader receive the same information.

Use `aria-invalid="true"` after validation has determined that the current value is invalid. Do not mark every empty required field invalid as soon as the page loads: the person may not have tried to complete or submit it yet. When the error is gone, remove `aria-invalid` (or set it to `"false"` if that better fits the component's consistent rendering).

`aria-invalid` does not create, display, or explain an error by itself. Pair it with visible text and a programmatic association.

## Show an error summary

When submission fails, a summary near the beginning of the form gives users an overview and a direct route to each problem. Use an actual link whose `href` matches the invalid control's `id`.

~~~jsx
function ErrorSummary({ errors }) {
  const entries = Object.entries(errors);

  if (entries.length === 0) {
    return null;
  }

  return (
    <div role="alert" aria-labelledby="error-summary-title">
      <h2 id="error-summary-title">
        There are {entries.length} problems with your submission
      </h2>

      <ul>
        {entries.map(([field, message]) => (
          <li key={field}>
            <a href={'#' + field}>{message}</a>
          </li>
        ))}
      </ul>
    </div>
  );
}
~~~

Keep the summary heading descriptive. Each link should identify the problem well enough to be useful in a list, such as “Email address: enter an address in the format name@example.com.” An anchor to the field lets keyboard users activate it and move to the relevant control.

If a summary is inserted into the page after a failed submit, `role="alert"` can announce the newly displayed feedback. Avoid adding several overlapping live announcements to the same summary and every field error; too many announcements can make the form harder to use. Keep inline errors available through their field descriptions as well.

If your design instead moves focus to the first invalid field, make that behavior deliberate and consistent. Do not move focus unexpectedly while someone is typing.

## Decide when to validate

Validation timing affects how helpful a form feels:

- **Before interaction:** show instructions, but do not announce an error for a required field that has not been attempted.
- **On submit:** validate all fields, show inline errors and the summary, and make it clear that submission did not succeed.
- **After a failed submit:** clear or update a field's error as the person corrects it. Rechecking on blur can avoid announcing a new error on every keystroke.
- **On later submissions:** validate the current values again; never leave an old error summary that no longer matches the form.

A common approach is to keep a `submitted` flag and an `errors` object in React state. Before the first submit, avoid showing validation errors while the user is still entering data. On submit, validate all fields. After the first attempt, revalidate a field when it changes or loses focus so corrected feedback does not stay stale.

Keep error state derived from the current values where practical. If you store both values and errors, update them together so the visible message, `aria-invalid`, and summary cannot disagree.

## Announce successful feedback

When a form is handled without leaving the page, show a clear success message. A status region can announce a non-urgent update without taking focus:

~~~jsx
{message && <p role="status">{message}</p>}
~~~

Use `role="status"` for messages such as “Your request was sent.” Do not report success before the operation has succeeded. For an asynchronous request, also explain when it is still in progress and show an actionable error if it fails.

## Native validation and custom validation

The browser can provide built-in constraint feedback for attributes such as `required`, `type="email"`, `minLength`, and `pattern`. These constraints are useful, but browser messages and presentation vary. They do not give your application a tailored error summary.

If you choose a fully custom submit-and-error flow, `noValidate` on the form disables the browser's automatic popup flow so your React handler can render its messages. You can still inspect controls' validity and validity details in the submit handler. This is a choice about who presents the feedback; it does not make invalid data acceptable.

Do not disable native validation just to replace it with a less clear custom message. Choose one coherent interaction, keep the controls semantically correct, and ensure every error is visible and connected to its field.

Client-side validation is for a better interaction, not a security boundary. Requests can be sent without your interface, so the server must validate and authorize submitted data too. If the server rejects a value, display that feedback using the same accessible pattern.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Showing only a red border or icon | Some people may not perceive the color or know what it means. | Add visible text that describes the problem. |
| Showing “Invalid input” | The message does not name the field or explain the correction. | State the field, issue, and next step. |
| Setting `aria-invalid` on untouched fields at page load | It announces a failure before the person has had a chance to respond. | Mark invalid after an attempted validation finds an error. |
| Rendering an error without connecting it to the control | Assistive technology may not associate the message with the field. | Give the message an id and include it in `aria-describedby`. |
| Putting the same generic message on every summary link | The links are indistinguishable out of context. | Include the field name and the specific correction in each link. |
| Leaving old errors after a value is corrected | The summary and field state no longer match the current form. | Clear or recompute errors when values change and on each submit. |
| Announcing the summary and every error repeatedly | Repeated announcements can interrupt people. | Announce the new summary once and keep field errors associated with their controls. |
| Treating a successful-looking client check as server validation | A request can bypass the browser interface. | Validate and authorize again on the server. |

## Practice

1. Add a visible label, help text, and an email field to a form.
2. Make an inline error appear only after the field has been validated.
3. Connect the error id using `aria-describedby` and set `aria-invalid` while the error exists.
4. Submit the form with two invalid fields and show an error summary.
5. Add a link for each error and confirm it points to the corresponding field id.
6. Correct one value and confirm both the inline error and summary update.
7. Show a `role="status"` message only after the form succeeds.
8. Test the complete flow using only a keyboard and, if available, a screen reader.

## Key takeaways

- An error should identify the field, explain the problem, and help the person fix it.
- Show errors as text; do not rely on color or `aria-invalid` alone.
- Associate each inline error with its control through `aria-describedby`.
- Set `aria-invalid` when validation has found a problem, not merely because an untouched required field is empty.
- An error summary should have a clear heading and links to the fields that need attention.
- Announce dynamic feedback once, and avoid unexpected focus changes.
- Server-side validation remains necessary even when client-side validation is accessible.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="030-form-submission-and-browser-validation.md">030 - Form submission and browser validation</a></td><td><a href="../qnas/031-accessible-validation-messages.md">Q&amp;A</a></td><td><a href="../examples/031-accessible-validation-messages.md">Example</a></td><td><a href="032-multi-field-form-state.md">032 - Managing state for multi-field forms</a></td></tr>
  </tbody>
</table>
