# Managing Submissions with useActionState

A form Action often needs to return more than “done”: it may return validation errors, preserve what the user typed, or provide the saved result. React's useActionState Hook connects an Action's return value to component state and exposes whether that Action is pending.

This lesson explains the Hook's signature, how it changes a form Action's arguments, how to preserve fields, and how to choose between returned errors and thrown errors.

## Learning goals

- Use useActionState to store the latest result from an Action.
- Understand its state, action dispatcher, and pending return values.
- Read FormData from the correct argument position.
- Return expected validation errors and preserve submitted values.
- Distinguish expected errors from unexpected failures.
- Understand queued submissions, manual dispatch, and reset choices.
- Know when useActionState is a good fit and when it is not.

## 1. What useActionState provides

Call useActionState at the top level of a component:

    const [state, formAction, isPending] =
      useActionState(actionFunction, initialState);

It returns:

1. The latest state returned by the Action. On the first render this is initialState.
2. A wrapped Action dispatcher. Pass it to a form's action prop or a submit button's formAction prop.
3. A boolean that is true while the Action is pending.

The initial state is used before the Action first runs. Later calls do not reinitialize the Hook from a changed initialState argument. To start a new form flow from scratch, design an explicit reset or remount the component with a new key.

## 2. The Action receives previous state first

When useActionState wraps an Action, React passes the previous state as the first argument. For a form Action, the submitted FormData is the second argument:

    async function saveName(previousState, formData) {
      const name = formData.get('name');
      // validate and save
      return { message: 'Name saved', name };
    }

    function NameForm() {
      const [state, formAction, isPending] =
        useActionState(saveName, { message: '', name: '' });

      return (
        <form action={formAction}>
          <input name="name" defaultValue={state.name} />
          <button type="submit" disabled={isPending}>Save</button>
          <p role="status">{state.message}</p>
        </form>
      );
    }

Without useActionState, a form Action usually receives FormData as its first argument. Remember the changed signature or the Action may mistakenly call FormData methods on the previous state.

The Action returns the next state. Treat this like a state transition: inspect the previous value and submitted payload, perform the side effect, then return a new value for the UI. Do not mutate the previous state object.

## 3. Return expected validation results

Expected outcomes belong in returned state. For example, a server can return field errors and the submitted values:

    async function updateName(previousState, formData) {
      const submitted = {
        name: String(formData.get('name') ?? '').trim(),
      };

      if (submitted.name.length < 3) {
        return {
          status: 'error',
          message: 'Please correct the name.',
          errors: { name: 'Use at least three characters.' },
          submitted,
        };
      }

      await saveName(submitted.name);

      return {
        status: 'success',
        message: 'Name saved.',
        errors: {},
        submitted: null,
      };
    }

The UI can display the message and use state.submitted.name as an input defaultValue. After a failed validation, React's form reset uses the updated default value and the user keeps the entered text. After success, submitted is null, so the field resets to an empty value.

An ordinary function Action that resolves with a validation object has completed successfully from React's perspective. Returning an error does not throw and does not create an Error Boundary. If preserving fields matters, include the submitted values in the returned state and render them deliberately.

## 4. Native constraints and trusted validation

HTML constraints such as required and type="email" prevent common mistakes early. They improve usability but are not a security boundary. A caller can bypass client-side validation and send arbitrary data directly to a server.

Validate again where data is trusted: check type, length, allowed values, and authorization. Return a structured error for expected invalid input. Do not expose raw server exceptions or secret details in the UI.

For file controls, FormData values can be File objects rather than strings. Check their type, size, content, and authorization before storing or processing them.

## 5. Pending feedback

The third value from useActionState isPending describes the wrapped Action:

    const [state, formAction, isPending] =
      useActionState(updateProfile, initialState);

Use it to disable a submit button, show a message, or prevent an interaction that should not be repeated:

    <button type="submit" disabled={isPending}>
      {isPending ? 'Saving…' : 'Save'}
    </button>

You can also use useFormStatus from react-dom in a component nested inside the form. That Hook reports the parent form's pending state and submission data. useActionState is convenient when the component that owns the Action also needs its state; useFormStatus is useful for a reusable nested submit button.

Avoid replacing the whole form with a spinner if users can safely continue reading or editing. A pending indicator should match the operation and keep the interface understandable.

## 6. Form integration and progressive enhancement

Pass the wrapped dispatcher to the form:

    const [state, submitAction, isPending] =
      useActionState(updateName, initialState);

    return (
      <form action={submitAction}>
        <input name="name" defaultValue={state.submitted?.name ?? ''} />
        <button type="submit" disabled={isPending}>Save</button>
        {state.message && <p role="status">{state.message}</p>}
      </form>
    );

When used as a form Action, React runs the dispatcher in a Transition. You do not need to call startTransition around that form submission.

A third optional permalink argument can support progressive enhancement for a Server Function form on a page with dynamic content. It identifies the destination page to render if submission happens before hydration. The destination must render the same form component with the same Server Function and permalink. This is framework-specific infrastructure; a client-only Vite component does not gain server submission merely by passing a permalink.

## 7. Dispatching outside a form

The returned dispatcher can be called outside a form, but the call must happen in an Action/Transition so React can track pending work:

    import { startTransition, useActionState } from 'react';

    function SaveButton() {
      const [state, dispatchSave, isPending] =
        useActionState(saveCurrentDraft, initialState);

      function handleClick() {
        startTransition(() => {
          dispatchSave({ type: 'save-draft' });
        });
      }

      return (
        <button onClick={handleClick} disabled={isPending}>
          Save draft
        </button>
      );
    }

When dispatching directly, the Action receives the supplied payload after previousState. For form submission, that payload is FormData.

## 8. Actions are queued in order

useActionState processes calls sequentially because the next Action needs the previous Action's returned state. Several quick submissions may therefore wait in a queue. This is useful when each result depends on the last one, such as incrementing a server-confirmed quantity.

It may be the wrong model for independent requests that should run in parallel. In that case, use state and transitions or the data library's concurrency tools, and define how out-of-order results are handled. Do not assume a queued Action makes a server mutation reversible.

## 9. Expected errors and unexpected errors

Return expected problems such as invalid fields, unavailable inventory, or a rejected business rule as normal state:

    return {
      status: 'error',
      message: 'That username is already taken.',
      submitted,
    };

Throw unexpected failures when they should cancel the queued Actions and be handled by an Error Boundary. Network and service failures may be expected in your application, so choose whether to return a recoverable error state or throw based on the recovery experience.

Do not catch an error merely to return a success-shaped state. Keep success, validation failure, and unexpected failure distinct.

## 10. Resetting state deliberately

useActionState does not expose a built-in reset function. Common options include:

- Return the initial shape from a dedicated reset Action.
- Dispatch a reset payload and handle it in the Action.
- Remount the form component with a new key when a new form session begins.
- Clear only the related fields in controlled state while preserving other result state.

A reset should not accidentally discard a successful result that users still need to see. Decide whether resetting fields, clearing the message, and clearing saved data are separate operations.

## 11. useActionState and other state tools

- Use useState for independent local UI state, such as whether a panel is expanded.
- Use useReducer for pure UI-state transitions that do not perform side effects.
- Use useActionState for state that is produced by a user Action, especially async form submissions.
- Use useFormStatus when a nested component needs information about the parent form.
- Use a framework or query library for server data caching and synchronization.

The Action passed to useActionState can perform side effects, unlike a pure reducer. Since calls are ordered, consider whether a serial queue matches the user operation.

## 12. Common mistakes

- Reading FormData from the first argument instead of the second.
- Forgetting to return the next state from every expected path.
- Mutating previousState rather than returning a new object.
- Returning validation errors without preserving submitted values.
- Assuming initialState resets after every submission.
- Calling the dispatcher directly outside an Action/Transition.
- Throwing expected validation errors into an Error Boundary.
- Assuming queued Actions execute in parallel.
- Treating client validation as server authorization.
- Using useActionState for unrelated generic UI state.

## Practical checklist

1. Does the Action accept previousState first?
2. Is the second argument FormData for this form submission?
3. Is each result a fresh state value?
4. Are expected validation problems returned as state?
5. Are submitted values preserved on failure?
6. Are successful values cleared or retained intentionally?
7. Is pending feedback clear and accessible?
8. Does an unexpected error have a useful boundary?
9. Does sequential queuing fit the operation?
10. Is server validation and authorization performed where the data is saved?

## Summary

useActionState turns an Action's returned value into state and provides its wrapped dispatcher and pending flag. A form-wrapped Action receives previous state first and FormData second. Return expected validation results as structured state, including submitted values when the user should keep them. Unexpected errors can reach an Error Boundary. Calls are queued in order, so choose the Hook when sequential stateful submissions match the interaction.

## Key terms

- **useActionState:** A Hook that stores an Action's latest returned state and exposes a wrapped dispatcher and pending flag.
- **Previous state:** The first argument passed to the Action wrapped by useActionState.
- **Action dispatcher:** The wrapped function returned by useActionState and passed to a form or called in a Transition.
- **Returned validation state:** A normal Action result describing field errors or other expected outcomes.
- **Queued Action:** A call that waits for the prior call to return state.
- **Permalink:** Optional destination information used by framework-supported progressive enhancement.
