# React Form Actions and Asynchronous Submissions

Forms collect user input and send it somewhere. React 19 adds a function-based Action pattern to the familiar form element: React passes submitted FormData to the function, tracks the work as a Transition, and can expose its pending state to components inside the form.

This lesson covers browser forms, React form Actions, pending feedback, validation, errors, multiple submit actions, and the difference between a client Action and a Server Function.

## Learning goals

- Explain how a normal HTML form submits data.
- Use a function with the form action prop and read its FormData.
- Distinguish a form Action from an onSubmit handler and a URL action.
- Show pending feedback with useFormStatus in a child component.
- Handle validation, errors, successful reset, and multiple submit buttons.
- Explain progressive enhancement and server-side authorization.

## 1. Start with the HTML form model

A form groups controls and submits their named values. The name attribute is the key used in the submitted data:

    <form action="/search" method="get">
      <label>
        Search
        <input name="query" required />
      </label>
      <button type="submit">Search</button>
    </form>

With a string URL in action, the browser performs a standard navigation. GET is useful for searches that should be bookmarkable. POST is generally used to send a change or larger/private payload. The button that submits the form can contribute its own name and value.

Controls without a name are not included in FormData. Checkboxes that are not checked are omitted. Repeated names can have several values, so use FormData.getAll(name) when a field is repeated.

## 2. Function actions in React 19

React 19 lets a form receive a function as its action:

    async function saveProfile(formData) {
      const name = formData.get('name');
      await saveName(name);
    }

    function ProfileForm() {
      return (
        <form action={saveProfile}>
          <input name="name" required />
          <button type="submit">Save</button>
        </form>
      );
    }

React calls the function with one FormData argument. The function may be asynchronous. It runs as a React Action in a Transition, so the interface can stay responsive while the work is pending.

When action is a function, React handles submission without a manual onSubmit handler or preventDefault call. A function supplied to action or formAction uses POST semantics regardless of the method prop. This differs from action="/somewhere", which uses native browser submission rules.

A function may be defined in a Client Component and run in the browser, or it may be a framework-supported Server Function. These are different execution models; an ordinary client function does not become server code just because it is passed to action.

## 3. Read and validate FormData

Only successful controls with name values are included. FormData.get(name) returns a string or File, or null if that field was not submitted. For multiple values, use getAll.

    async function createMessage(formData) {
      const rawTitle = formData.get('title');

      if (typeof rawTitle !== 'string') {
        throw new Error('A text title is required.');
      }

      const title = rawTitle.trim();

      if (title.length < 3) {
        throw new Error('The title must have at least three characters.');
      }

      await sendToApi({ title });
    }

Browser constraints such as required, minLength, type="email", and pattern improve the immediate experience, but they do not replace validation at the trusted data boundary. Users can bypass browser checks. Validate shape, length, allowed values, and authorization again where the data is actually saved.

Do not assume every FormData value is text. File inputs return File objects. Convert and validate values deliberately instead of blindly trusting their type.

## 4. Pending feedback with useFormStatus

A submit button can read the pending state of its parent form using useFormStatus from react-dom:

    import { useFormStatus } from 'react-dom';

    function SubmitButton() {
      const { pending } = useFormStatus();

      return (
        <button type="submit" disabled={pending}>
          {pending ? 'Saving…' : 'Save'}
        </button>
      );
    }

    function ProfileForm({ action }) {
      return (
        <form action={action}>
          <input name="name" required />
          <SubmitButton />
        </form>
      );
    }

The component calling useFormStatus must render inside the form it observes. Calling the Hook in the same component that creates the form does not report that form's status. This is a common component-boundary mistake.

The returned status also contains the submitted data, method, and action reference. Treat submitted data as user input, and only display it where appropriate.

Pending feedback should explain the operation. Disabling the submit control can prevent accidental duplicate clicks, but do not disable unrelated navigation or make the entire page unusable without a reason.

## 5. What happens after the Action

When a function passed to action or formAction succeeds, React resets the form's uncontrolled fields to their default values. This is useful for a successful message or comment submission.

A fulfilled Action that returns a validation message is still a success from React's point of view and also resets uncontrolled fields. If the user must keep their entered values after validation fails, preserve them deliberately—for example, with state, controlled fields, suitable defaultValue values, or the stateful pattern in the next lesson.

Controlled inputs keep their values from React state; the form reset does not replace that state. Choose controlled or uncontrolled fields intentionally instead of mixing assumptions.

## 6. Errors and recovery

If a function Action throws or rejects, React propagates the error to the nearest Error Boundary. Suspense does not handle form submission errors; it is for supported pending rendering work.

Use returned data for expected outcomes such as validation feedback. Reserve thrown errors for failures that should use an Error Boundary, such as an unavailable service or an unexpected server error. Provide a helpful recovery action and do not expose sensitive error details.

The browser may reset uncontrolled fields only after a successful Action. On failure, preserve the user's work and make it possible to retry. If the application needs to preserve or display structured validation results across submissions, a stateful form Action is a better fit; the next lesson covers useActionState.

## 7. Different submit buttons, different Actions

A form can provide a default action and a submit button can override it with formAction:

    <form action={publishArticle}>
      <textarea name="body" required />
      <button type="submit">Publish</button>
      <button type="submit" formAction={saveDraft}>Save draft</button>
    </form>

Both actions receive the submitted FormData. Use this when buttons genuinely represent distinct submissions, such as publish versus save draft. Give the buttons clear labels and ensure each server operation checks that the user is allowed to perform it.

## 8. Client Actions and Server Functions

A client Action can call browser-side code, update local state, or call an API. It runs only where that client code is available.

A Server Function is defined and invoked through a framework that supports React Server Components and Server Functions. It can let a form submit before client JavaScript has loaded, providing progressive enhancement. The framework serializes the request and invokes the server-side function.

Do not add a "use server" directive to an arbitrary Vite client project and expect it to work. Server Functions require framework/build support and deployment configuration.

Never treat the Action function or hidden fields as an authorization boundary. A client can alter submitted values, and a server endpoint can be called independently. Validate input and authenticate and authorize the current user on the server for every protected operation.

## 9. When to use onSubmit

The standard onSubmit event remains useful when you need direct access to the submit event, integrate a non-React system, or support older React versions:

    function handleSubmit(event) {
      event.preventDefault();
      const formData = new FormData(event.currentTarget);
      sendToApi(formData);
    }

Preventing the default browser behavior is important if the handler is taking over the submission. If both onSubmit and a function action are present, React runs the action after onSubmit unless onSubmit calls preventDefault.

Prefer the form Action pattern for React 19 form work when it matches the task. Use onSubmit when you specifically need the event-level behavior. Avoid implementing both paths for the same request.

## 10. Common mistakes

- Leaving out name attributes and wondering why values are absent.
- Treating FormData.get as guaranteed to return a string.
- Using a function action but expecting a GET request.
- Calling useFormStatus in the same component that renders the form.
- Returning validation text and forgetting successful Actions reset uncontrolled fields.
- Assuming a client Action is a Server Function.
- Trusting required or hidden inputs instead of validating on the server.
- Throwing expected field errors into a generic fatal-error screen.
- Allowing rapid duplicate submissions without clear behavior.
- Using an Action where ordinary navigation to a URL is a better fit.

## Practical checklist

1. Does every submitted field have a meaningful name?
2. Is a URL action or a function Action the right model?
3. Are input types and native constraints appropriate?
4. Does the Action validate every value, including File values?
5. Is the operation authorized at the server boundary?
6. Can users see that submission is pending?
7. Does useFormStatus live inside the form?
8. Should successful submission clear or preserve the fields?
9. What should happen on validation failure and network failure?
10. Do multiple submit buttons need distinct formAction functions?

## Summary

A React 19 function form Action receives FormData and runs as asynchronous React work. useFormStatus gives a nested component pending information. Successful function Actions reset uncontrolled fields, while failures should have a deliberate recovery path. Client Actions and framework-supported Server Functions are distinct; server-side validation and authorization remain essential.

## Key terms

- **FormData:** Browser data structure containing named form controls and their submitted values.
- **Action:** A function React runs as part of an asynchronous Transition.
- **Function action:** A function passed to a form's action prop.
- **formAction:** A submitter prop that can override the form's default Action.
- **useFormStatus:** A React DOM Hook that reads the status of a parent form submission.
- **Uncontrolled field:** A field whose current value is held by the DOM rather than React state.
- **Server Function:** A framework-supported function invoked on the server through React's server integration.
- **Progressive enhancement:** A form remains usable before client JavaScript has loaded.
