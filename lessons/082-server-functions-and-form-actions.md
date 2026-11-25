# Server Functions and form Actions

## Learning goals

After this lesson, you should be able to:

- Explain what a Server Function is and how a form Action invokes one.
- Mark an asynchronous function with the use server directive in a supported framework.
- Read, validate, and authorize submitted FormData on the server.
- Use useActionState when the form needs returned state or expected validation errors.
- Understand pending state, form reset behavior, and progressive enhancement.
- Recognize the framework and security boundaries around Server Functions.

## Server Functions and Server Actions

A Server Function is an asynchronous function that runs on the server and can be invoked by client-side code through a framework-supported request. The function's arguments and return value cross a network boundary.

When a Server Function is used to handle a form submission or another mutation, React documentation calls it a Server Action. Server Function is the broader term; Server Action describes its use as an Action.

In a supported React Server Components framework, a function can be marked with the use server directive:

    "use server";

    export async function saveNote(formData) {
      // Server-only work goes here.
    }

A framework or compatible bundler transforms the function reference and provides the network transport. The directive by itself does not create a server, database, API route, or security policy.

## This requires framework support

Server Functions are not a feature that works in every React setup. A plain client-only React app, such as a basic Vite SPA, cannot make a function run on a server merely by adding the use server string.

Use a framework or build tool that explicitly supports React Server Components and Server Functions. Those implementation APIs are framework-dependent and may have version-specific behavior. Follow the chosen framework's setup and deployment requirements.

Do not confuse a Server Function with a Server Component:

- A Server Component is a React component rendered on the server.
- A Server Function is an asynchronous server function exposed to client code.
- A Server Action is a Server Function used in an Action context, such as a form mutation.

They can be used together, but they are separate concepts.

## The use server directive

The directive can appear at the beginning of an asynchronous function body:

    async function saveNote(formData) {
      "use server";

      // Validate and save the note.
    }

It can also appear at the very beginning of a module to mark all exported functions in that file as Server Functions:

    "use server";

    export async function createNote(formData) {
      // ...
    }

    export async function deleteNote(formData) {
      // ...
    }

A module marked this way should export Server Functions, not ordinary configuration values or React components. In many frameworks, a Client Component imports an exported function from this server-only module. The framework turns the import into a callable server reference.

A Server Function must be asynchronous because invoking it crosses the network. Treat the directive as a boundary marker handled by the framework, not as a JavaScript keyword with built-in transport behavior.

## Use a Server Function as a form Action

React's form action prop accepts either a URL string or a function. When it receives a Server Function, submitting the form invokes that function on the server. React sends the named form controls as FormData:

    <form action={createNote}>
      <label>
        Title
        <input name="title" required />
      </label>

      <label>
        Note
        <textarea name="body" required />
      </label>

      <button type="submit">Save note</button>
    </form>

    // In the Server Function:
    export async function createNote(formData) {
      "use server";

      const title = formData.get("title");
      const body = formData.get("body");
      // Validate, authorize, and save.
    }

Controls need name attributes to be included in FormData. FormData.get returns a string for text controls and can return a File for file controls. Check each value's type before using it.

React runs function form Actions in an Action/Transition. You can use useFormStatus in a child component to show pending feedback. An uncontrolled form resets after a function Action completes successfully. If a validation response should keep the user's input, return the submitted values and render them as the fields' default values, or manage the values with state.

A URL string in action is a normal browser form destination; it is not a Server Function. The framework handles the transport only when the action is a supported function reference.

## Return validation state with useActionState

When a form needs to display a success result or expected validation errors, use useActionState:

    import { useActionState } from "react";

    const [state, formAction, isPending] =
      useActionState(createNote, initialState);

    return (
      <form action={formAction}>
        <input name="title" defaultValue={state.title} />
        <button disabled={isPending}>Save</button>
        <p role="status">{state.message}</p>
      </form>
    );

When wrapped by useActionState, the Server Function receives the previous state as its first argument and FormData as its second:

    export async function createNote(previousState, formData) {
      "use server";

      // Validate the submitted values and return the next state.
    }

Return expected problems, such as invalid field values or a title that is too long, as structured state. Include submitted field values when the form should preserve them. Throw unexpected failures when they should be handled as exceptions by the framework's error boundary.

The isPending value from useActionState can drive the form's submit button. A reusable child can instead read the parent's pending status with useFormStatus from react-dom. These are different ways to expose pending state; choose the one that fits the component structure.

## Server-side validation and authorization

Client validation helps users, but it is not a trust boundary. A caller can bypass the rendered form and invoke the server request with different values. Validate all inputs again inside the Server Function.

For a mutation, also check the current user's identity and whether that user may perform this exact operation on this exact record. Never rely on a hidden field, disabled button, client state, or submitted user ID as proof of authorization.

A safe mutation usually follows this order:

1. Read and type-check FormData values.
2. Normalize and validate every field.
3. Authenticate the caller using trusted server-side session data.
4. Authorize the requested operation and resource.
5. Perform the mutation in the data layer.
6. Return only the information the client needs.
7. Revalidate or refresh framework-managed data when required.

Do not return secrets, database credentials, private records, or raw internal errors to the client. Anything returned by the Server Function can be serialized to the caller.

## Progressive enhancement

A Server Component can render a form that uses a Server Function. In a framework that supports this pattern, the form can submit before client JavaScript has loaded. This is progressive enhancement: the basic form operation does not have to wait for client hydration.

A Client Component may add controlled fields, pending indicators, optimistic updates, and other interactivity. The exact pre-hydration behavior depends on the framework and where the form is rendered. Do not claim that a plain client-only app has progressive enhancement just because its form uses an action function.

## Server Functions are for mutations

Server Functions are designed primarily for actions that change server-side data, such as saving a profile or creating a note. They are not a general replacement for data fetching or a query cache.

Use Server Components, framework data loaders, route handlers, or a data library for reads and caching where those fit the app. A Server Function invocation is a network request and has request latency, serialization, error, and authorization concerns.

## Form details to remember

- A function passed to form action receives FormData.
- Each submitted control needs a name.
- A function form Action uses POST semantics.
- A button or submit input can use formAction to select a different Action for that submit control.
- React resets uncontrolled fields when a function form Action completes without throwing.
- When useActionState wraps the function, previousState becomes the first argument and FormData follows it.
- If a form needs to retain values after a validation result, return those values and render them as defaults.
- useFormStatus belongs in a component rendered inside the parent form.

## Common mistakes

### Expecting use server to work in bare Vite

A directive is not a server implementation. Configure a supported framework or use a conventional API endpoint and call it with fetch.

### Defining a Server Function in a Client Component

A Client Component cannot turn its own function into server code with a directive. Define the Server Function in a server-capable module or Server Component using the supported framework pattern.

### Forgetting async

Server Functions cross a network boundary, so mark them async and return values that the framework can serialize.

### Reading the wrong argument after useActionState

Without useActionState, a form Action receives FormData first. With useActionState, it receives previousState first and FormData second.

### Trusting browser validation

required, type, minLength, and similar attributes improve the form, but the server must repeat validation and authorization.

### Assuming the server function is private

Server Functions can be invoked by direct network requests. Treat every argument as untrusted and authorize every mutation inside the function.

### Returning a success result without performing a mutation

A Server Function that only displays a confirmation has not persisted anything. Save through a real data layer before reporting that a change was saved.

## Key takeaways

- Server Functions are asynchronous server code exposed through framework support.
- A Server Function used as a form mutation is a Server Action.
- The use server directive marks functions; a framework provides transport and execution.
- Form submissions pass named controls as FormData.
- useActionState changes the Action signature to previousState, then FormData.
- Validate and authorize on the server because client input is untrusted.
- Keep server-only secrets on the server and return only safe serializable results.
- Choose a framework-supported setup; a directive alone does not add backend behavior.
