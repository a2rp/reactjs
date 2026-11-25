# 080. Pending form state with useFormStatus

## Learning goals

After this lesson, you should be able to:

- Explain what problem useFormStatus solves.
- Read a parent form's pending, data, method, and action status.
- Place the hook in the correct child component.
- Build a reusable submit button that reflects an in-progress form Action.
- Distinguish form submission status from general component state and useActionState.

## Why a form needs a pending state

A submission may take time. It can wait for validation, a network request, or other asynchronous work. If the interface gives no feedback, a user may press Submit repeatedly or wonder whether anything happened.

A pending state can make the interaction clear. For example, the submit button can say “Saving…” and become disabled while the current submission is being processed. React DOM provides useFormStatus for child components that need to read the status of the form around them.

This hook is especially useful when the form and its submit button are separate components. The form owns the submission; a child can display its status without the parent manually passing a pending prop through every layer.

## Import and basic shape

Import useFormStatus from react-dom:

    import { useFormStatus } from "react-dom";

Call it at the top level of a React component. It takes no arguments and returns an object:

    const { pending, data, method, action } = useFormStatus();

The hook reads the status of a parent form submission. It does not submit the form and does not perform validation or a network request by itself.

## The placement rule

The component that calls useFormStatus must be rendered inside the form it should observe. A useful design is to make the button a child component:

    import { useFormStatus } from "react-dom";

    function SubmitButton() {
      const { pending } = useFormStatus();

      return (
        <button type="submit" disabled={pending}>
          {pending ? "Submitting…" : "Submit"}
        </button>
      );
    }

    function ContactForm({ action }) {
      return (
        <form action={action}>
          <label>
            Message
            <textarea name="message" required />
          </label>
          <SubmitButton />
        </form>
      );
    }

Calling the hook in ContactForm itself does not make it observe the form that ContactForm returns. At the moment the hook runs, that form is a child, not a parent. The same issue applies if a component calls the hook and then renders both the form and the button. Move the hook into a component rendered beneath the form.

The hook observes the nearest parent form. Do not nest form elements: nested forms are invalid HTML and make form ownership confusing. A child outside any form receives the default inactive status.

## Status properties

The returned status object contains four properties:

| Property | Meaning |
| --- | --- |
| pending | True while the parent form is submitting a function Action or running a Transition started from its submit handler; otherwise false. |
| data | The FormData being submitted while a submission is active; otherwise null. |
| method | The parent form's submission method, usually get or post. |
| action | The function passed to the parent form's action prop. It is null when there is no parent form, when action is a URL, or when no action prop is set. |

Most buttons only need pending. The other properties are useful for status messages and reusable form controls. Treat data as temporary submission data: do not assume it is available after completion. Read values with data.get("fieldName"), and remember that FormData values may be strings or files.

## Use it with a function form Action

In React, a function passed to a form's action prop receives FormData. React runs the Action in a Transition, exposes pending status to descendants, and resets uncontrolled fields after the Action succeeds.

    async function saveMessage(formData) {
      const message = formData.get("message");
      await saveSomewhere(message);
    }

    function MessageForm() {
      return (
        <form action={saveMessage}>
          <label>
            Message
            <textarea name="message" required />
          </label>
          <SubmitButton />
        </form>
      );
    }

The example saveSomewhere is application-specific. A client-side Action alone does not create a server endpoint or persist data securely. Use your application's actual data layer and server-side validation where appropriate.

## Use it with onSubmit

Sometimes a form needs a traditional submit event handler. Prevent the browser's default navigation, create FormData, and start the asynchronous work in a Transition. A descendant can then read the pending state.

    import { startTransition } from "react";

    function handleSubmit(event) {
      event.preventDefault();
      const formData = new FormData(event.currentTarget);

      startTransition(async () => {
        await saveSomewhere(formData);
      });
    }

    function ManualForm() {
      return (
        <form onSubmit={handleSubmit}>
          <input name="title" required />
          <SubmitButton />
        </form>
      );
    }

Unlike a function passed directly to form action, this manual onSubmit pattern does not automatically clear uncontrolled fields after success. Decide explicitly whether to keep or reset values. Avoid starting asynchronous work outside a Transition if you expect useFormStatus to report that work as pending.

## Make the pending UI useful

A good pending state should communicate what is happening and avoid accidental duplicate submissions.

- Change the button label to a short action phrase such as “Saving…”.
- Disable the submit button while pending when another submission should not be started.
- Keep labels and instructions visible; do not replace the entire form with an unexplained spinner.
- Use an accessible status message when users need more detail. A paragraph with role="status" can announce a polite update to assistive technology.
- Preserve useful input values when a submission fails. Handle errors in the action or form state rather than treating pending as the error state.
- Avoid putting sensitive form data into a visible status message. The data property is for UI coordination, not security.

A button being disabled is not a substitute for server-side validation, authorization, or protection against duplicate operations. A user can bypass client UI, so the server must enforce the rules that matter.

## useFormStatus and useActionState

These hooks answer different questions:

- useFormStatus lets a descendant inspect the status of its parent form.
- useActionState stores the result or state returned by a form Action and also exposes an isPending value for that Action.

Use useFormStatus when a reusable child such as SubmitButton needs to respond to the form around it. Use useActionState when the form needs to render a returned result, validation errors, or other state produced by the Action. They can be used together when each has a clear role; do not duplicate state without a reason.

## Common mistakes

### The pending value never becomes true

Check that the hook is in a component rendered inside the form, and that the form is submitting a function Action or a Transition started from onSubmit. A URL action follows normal browser navigation and does not provide the same function-Action status.

### Calling the hook in the form-rendering component

Move the hook to a child rendered inside the returned form. The hook only reports a parent form, not a form rendered by the component that calls it.

### Expecting data to remain after submission

data describes the active submission and is null when there is no active submission. Store a value separately if the interface needs to show it after the Action finishes.

### Expecting useFormStatus to perform the request

The hook only observes form status. The form Action or submit handler must do the actual work.

## Key takeaways

- Import useFormStatus from react-dom.
- Call it in a component rendered inside the form being observed.
- Use pending for immediate feedback during a function Action or tracked Transition.
- data, method, and action provide additional information about the active parent form.
- A form Action and a manual onSubmit handler have different reset behavior.
- Pending UI improves feedback but does not replace validation or security checks.
