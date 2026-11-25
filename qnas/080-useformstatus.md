# 080. Q&A: Pending form state with useFormStatus

## Core concepts

### 1. What does useFormStatus do?

It lets a component rendered inside a form read status information about the most recent submission of its parent form. It observes the form; it does not submit data or perform the request.

### 2. Where is useFormStatus imported from?

Import it from react-dom:

    import { useFormStatus } from "react-dom";

It is a React DOM hook because it observes a DOM form.

### 3. Does useFormStatus take arguments?

No. Call useFormStatus() at the top level of a function component or custom Hook that follows the Rules of Hooks.

### 4. What does useFormStatus return?

It returns a status object with pending, data, method, and action properties.

### 5. What does pending mean?

pending is true while the parent form is submitting a function Action or while an associated Transition started from onSubmit is running. It is false when there is no active tracked submission.

### 6. When should a submit button use pending?

Use it to communicate progress and, when appropriate, prevent repeated submissions:

    <button type="submit" disabled={pending}>
      {pending ? "Saving…" : "Save"}
    </button>

Whether to disable the button depends on the interaction. The server must still protect operations that must not be duplicated.

### 7. What does data contain?

While submission is active, data is the FormData being submitted by the parent form. It is null when there is no active submission or no parent form.

### 8. How do I read a field from status.data?

Use FormData.get with the field's name:

    const email = data?.get("email");

The field must have a name. A returned value can be a string or a File, depending on the input.

### 9. Should I rely on data being available after the action finishes?

No. It represents the active submission and becomes null when there is no active submission. Save a value in other state or use the Action result if it must remain visible afterward.

### 10. What does method tell me?

It reports the parent form's method, generally get or post. A form defaults to get. When a function is supplied as the form Action, React submits it with post semantics.

### 11. What does action tell me?

For a parent form using a function as its action prop, action is the function reference. It is null when the parent form uses a URL, has no action, or there is no parent form.

### 12. Does useFormStatus read the form element in the same component?

No. It only observes a parent form. A hook call in a component that returns the form cannot observe that form.

### 13. What is the correct component structure?

Put the hook in a child rendered inside the form. A reusable SubmitButton is a common choice:

    function SubmitButton() {
      const { pending } = useFormStatus();
      return <button disabled={pending}>Submit</button>;
    }

    function Form() {
      return (
        <form action={save}>
          <SubmitButton />
        </form>
      );
    }

### 14. Can I pass a pending prop from the form parent instead?

Yes, if the parent owns pending state through another mechanism. The benefit of useFormStatus is that a descendant can read parent form status directly, which can avoid threading a prop through intermediate components.

## Actions and event handlers

### 15. How does useFormStatus work with a function passed to form action?

React calls the function with the submitted FormData and runs it in a Transition. Descendants can read pending through useFormStatus. If the Action succeeds, React resets uncontrolled form fields.

### 16. Does a function form Action need an onSubmit handler?

Usually not. A function passed to action receives FormData directly. Use onSubmit when the browser submit event itself needs custom handling.

### 17. Can useFormStatus work with onSubmit?

Yes. Call preventDefault, construct FormData from the submitted form, and start the asynchronous work in startTransition from that submit handler. The child can observe the Transition's pending status.

### 18. Why call preventDefault in a manual onSubmit handler?

Without it, the browser's normal form submission can navigate or reload the page. Preventing the default lets the handler control what happens.

### 19. Why wrap the manual work in startTransition?

That marks the asynchronous form work as a Transition so the form's child status can track the pending operation. Starting unrelated asynchronous work outside this flow is not automatically tracked by useFormStatus.

### 20. Does manual onSubmit automatically clear the fields after success?

No. Unlike a function supplied to the form action prop, a manual onSubmit handler does not automatically reset uncontrolled fields. Keep the values or reset them deliberately after a successful submission.

### 21. What if I use a URL as the action?

That is a normal browser form submission. status.action is null for a URL action, and the hook does not provide the same function-Action pending behavior before the browser navigates.

### 22. Can I put both action and onSubmit on the form?

They can be used together, but understand the event flow. If onSubmit calls preventDefault, it prevents the form's Action from running. If it does not, the Action can run after the event handler. Choose one clear submission path unless you intentionally need both.

## Component structure and scope

### 23. Does useFormStatus work if the button is inside a wrapper component?

Yes, provided the component using the hook is rendered anywhere beneath the parent form in the React tree. The wrapper itself does not have to be the immediate child.

### 24. Which form does it observe if there are multiple forms?

It reads the nearest parent form in the rendered tree. Avoid nested form elements because nested forms are invalid HTML and can produce confusing ownership.

### 25. What does it return outside a form?

It reports the inactive default state: pending is false, data and action are null, and method uses its default status value. Place the component under the intended form.

### 26. Can one SubmitButton component be reused in different forms?

Yes. Each rendered instance observes its own nearest parent form, so the same component can show the status for whichever form contains that instance.

### 27. Does useFormStatus work in a component that renders a form as a child?

Not for that child form. The component has no parent form status to read. Move the hook into a descendant rendered inside the form.

## Choosing the right state tool and UX

### 28. How is useFormStatus different from useActionState?

useFormStatus exposes a parent form's submission metadata to a descendant. useActionState stores a value returned by an Action and provides an isPending flag for that Action. Choose based on whether you need form status or the Action's result.

### 29. Should I use useFormStatus or useTransition for a form button?

If the work is a form submission and the button is rendered inside that form, useFormStatus is a direct way to read the form's status. useTransition is appropriate when the component needs to start and track its own Transition. Avoid having two independent pending flags for the same operation without a clear reason.

### 30. Is pending an error state?

No. It only means work is in progress. Show validation or server errors through explicit Action state, error handling, or an error boundary as appropriate.

### 31. Is a disabled submit button enough to stop duplicate writes?

No. Client-side disabling only improves the interface. The server or data layer must validate requests and enforce idempotency or other protections required by the operation.

### 32. How can pending state be announced accessibly?

Keep the button label descriptive and add a short element with role="status" when an additional progress message is useful. Do not rely only on color or an unlabeled spinner.

### 33. Should all form fields be disabled while pending?

Not automatically. Disable fields only if changing them during submission would confuse the interaction. Some applications should keep fields editable while the request runs.

### 34. Can I show the submitted value using data?

Yes, while an active submission exists:

    const { pending, data } = useFormStatus();
    const submittedName = data?.get("name");

    return pending ? <p role="status">Saving {submittedName}…</p> : null;

Do not show sensitive values, and do not treat client-visible FormData as trusted authorization data.

### 35. What happens if the Action rejects?

useFormStatus is not an error store. Handle failures in the Action's state flow, useActionState for returned error information, or use an error boundary where appropriate. Do not assume pending communicates success or failure.

### 36. Can a form Action run on the server?

A framework can support Server Functions for a form action. That requires server-side framework support and a correctly configured server function; a normal client function does not become a server endpoint by being passed to action.

### 37. Does useFormStatus make a form work without JavaScript?

Not by itself. Server Function actions can support submission before client JavaScript loads in frameworks that provide this behavior. A client-side hook alone does not provide progressive enhancement.

### 38. How should I handle file inputs in status.data?

A file field is represented as a File object in FormData. Do not assume every value is a string, and do not display or trust file metadata as proof that the file is safe. Validate uploads on the server.

### 39. Why might pending seem to turn off before my follow-up UI updates?

pending describes the active form operation, not the lifetime of a success message. Store the result or message separately if it needs to remain after the submission completes.

### 40. What is the main mental model?

The form owns the submission; useFormStatus lets a descendant observe that parent form's active submission. The hook is status plumbing, not a request, validation, persistence, or authorization mechanism.
