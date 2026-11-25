# 080. Example: Pending form state with useFormStatus

## Goal

Build a small React 19 form with a reusable submit button and a pending message. The form Action simulates a request with a short delay. It does not send or save data.

## Requirements

Use a React 19 project with react and react-dom installed. The example below can replace App.jsx in a standard React project.

## App.jsx

    import { useActionState } from "react";
    import { useFormStatus } from "react-dom";

    function delay(milliseconds) {
      return new Promise((resolve) => {
        window.setTimeout(resolve, milliseconds);
      });
    }

    async function requestNewsletter(previousMessage, formData) {
      const email = String(formData.get("email") ?? "").trim();

      // Simulate an asynchronous operation. Replace this with your app's
      // actual request or server-supported Action.
      await delay(1000);

      return `Demo complete for ${email}. Nothing was sent or stored.`;
    }

    function PendingStatus() {
      const { pending, data, method } = useFormStatus();

      if (!pending) {
        return null;
      }

      const email = data?.get("email");

      return (
        <p role="status">
          Sending the demo request for {email} using {method.toUpperCase()}…
        </p>
      );
    }

    function SubmitButton() {
      const { pending } = useFormStatus();

      return (
        <button type="submit" disabled={pending}>
          {pending ? "Submitting…" : "Join the demo"}
        </button>
      );
    }

    export default function App() {
      const [message, formAction] = useActionState(requestNewsletter, "");

      return (
        <main>
          <h1>Newsletter form demo</h1>
          <p>
            Submitting waits for one second so you can see the pending state.
            This example does not contact a service.
          </p>

          <form action={formAction}>
            <label htmlFor="email">Email address</label>
            <input
              id="email"
              name="email"
              type="email"
              autoComplete="email"
              required
            />

            <PendingStatus />
            <SubmitButton />
          </form>

          {message && <p role="status">{message}</p>}
        </main>
      );
    }

## What happens

1. The user enters an email address and submits the form.
2. The form Action receives a FormData object. Its email field is read by name.
3. requestNewsletter waits for the simulated operation, then returns a message.
4. While the Action is pending, PendingStatus and SubmitButton read the same parent form status with useFormStatus.
5. The submit button is disabled and its label changes. The status message uses the submitted data and method.
6. When the Action succeeds, React resets the uncontrolled input. useActionState keeps the returned message available after the pending status ends.

The hook calls are in descendants of the form. If either hook were moved into App, useFormStatus would not observe the form that App returns.

## Try these changes

- Remove the delay and observe how quickly the pending message disappears.
- Temporarily remove PendingStatus while keeping SubmitButton to see that the button still independently observes pending.
- Remove the name="email" attribute and inspect the submitted value. FormData includes named controls.
- Replace the simulated delay with your own application Action. Validate input and enforce authorization and duplicate-write protections on the server.
- Return a different message from the Action and observe how useActionState retains it after submission.

## Important boundaries

This is an interface demonstration, not a real newsletter subscription. It has no server endpoint, persistence, email delivery, or server-side validation. A production action must validate submitted data and enforce access rules in the trusted server environment.

The pending state only describes work associated with this form submission. It does not prove that the operation succeeded; the message returned by the Action demonstrates a separate result state.
