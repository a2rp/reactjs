# Server Functions and form Actions — Q&A

## Definitions and setup

### 1. What is a Server Function?

It is an asynchronous function that runs on the server and can be invoked by client-side code through a framework-supported request.

### 2. What is a Server Action?

A Server Function used as an Action, such as a form submission or mutation. Server Function is the broader term.

### 3. What does the use server directive do?

It marks an asynchronous function or a module's exported functions as Server Functions for a supported React Server Components framework or bundler to transform.

### 4. Does use server automatically create a backend?

No. The framework or bundler must implement Server Function transport and execution. The directive alone does not create an API, server process, database, or deployment setup.

### 5. Does this work in a basic Vite React app?

Not by itself. A plain client-only Vite app does not turn a function into server code from the directive. Use a supported framework or make a normal API endpoint and call it with fetch.

### 6. How is a Server Function different from a Server Component?

A Server Component is a component rendered on the server. A Server Function is an async function callable across a server boundary. They are distinct features.

### 7. Where can the use server directive go?

It can be the first statement in an async function body, or the first statement in a module to mark its exported functions. A module-level directive must appear before imports and other code.

### 8. Can a Client Component define a Server Function locally?

No. A Client Component cannot turn its local function into server code by adding the directive. Import a Server Function from a supported server module or receive a server reference through props.

### 9. Does a Server Function have to be async?

Yes. The call crosses a network boundary, so it must be asynchronous and return a serializable result supported by the framework.

### 10. What may a module marked with use server export?

In frameworks that support this pattern, export asynchronous Server Functions. Do not export ordinary values or React components from a server-action module.

## Form submission and data

### 11. How do I call a Server Function from a form?

Pass its function reference to the form action prop:

    <form action={saveProfile}>
      <input name="displayName" />
      <button type="submit">Save</button>
    </form>

The framework must support the Server Function reference.

### 12. What does the form pass to the function?

React passes a FormData object containing the form's successful named controls. Read fields with FormData methods such as get.

### 13. Why must inputs have name attributes?

Form submission data is keyed by control name. An input without a name is not included under a useful field key in FormData.

### 14. What types can FormData.get return?

It can return a string, a File, or null when the requested key is absent. Check the type before treating a value as text.

### 15. How do I validate a submitted field?

Read it on the server, check its type, normalize it, and validate length and allowed values before using it. Browser required or type attributes only improve client-side usability.

### 16. What happens if I pass a URL string to form action?

It behaves like a normal browser form destination. A URL is not a Server Function and will not be executed as server Action code by React.

### 17. What does React do with a function form Action?

It invokes the function as an Action with POST semantics and makes pending information available. Uncontrolled fields reset when the Action resolves without throwing.

### 18. Can a button run a different action from its form?

Yes. A submit button can use formAction to override the parent form's action for that submission. This is useful for distinct operations such as save draft and publish.

### 19. Can the Server Function be called outside a form?

A framework may support invoking Server Functions from event handlers or other client code. Call it from an Action or Transition and follow the framework's invocation rules.

### 20. Does a function call directly from an onClick automatically have form pending status?

No. The function call must be placed in the supported Action/Transition flow. For form submissions, use useActionState or a child using useFormStatus for pending feedback.

## useActionState and form state

### 21. What does useActionState add?

It stores the value returned by an Action, returns a wrapped form Action, and provides an isPending flag.

### 22. How does useActionState change the server function arguments?

The wrapped function receives previousState first and FormData second:

    async function save(previousState, formData) {
      "use server";
      // ...
    }

Without useActionState, the form Action receives FormData as its first argument.

### 23. Why might my Action be calling get on the wrong value?

If it is wrapped with useActionState, FormData is the second argument. The first one is the previous state.

### 24. How can I show validation errors from a Server Function?

Return a structured state object with an error message and any field-specific errors. Render that state in the Client Component, commonly through useActionState.

### 25. Should expected validation failures be thrown?

Usually no. Return expected validation errors as normal state so the form can show them. Reserve thrown errors for failures that should go to an error boundary or framework error handling.

### 26. What happens to uncontrolled fields when the action returns a validation object?

If the function resolves normally, React resets uncontrolled fields. To preserve the submitted text, return it in state and use it as the fields' defaultValue, or handle the values with controlled state.

### 27. What happens to uncontrolled fields if the Action throws?

React does not treat the Action as a successful completion, so the normal success reset does not occur. The thrown error follows the framework's error handling path.

### 28. What does useFormStatus do in this pattern?

A descendant rendered inside the form can read the parent form's pending status and submission data. It is useful for a reusable button or status message.

### 29. How is useActionState pending different from useFormStatus pending?

useActionState gives the component that owns the wrapped Action its isPending value. useFormStatus lets a nested component inspect the parent form. Choose based on where the UI needs the status.

### 30. Can useOptimistic be combined with a Server Action?

Yes. useOptimistic can show a likely result while the Server Action runs. The UI must still reconcile with confirmed server data and explain failures.

## Server rendering and progressive enhancement

### 31. What does progressive enhancement mean for a Server Function form?

In a framework-supported Server Component form, the submission can work before client JavaScript has loaded. The form's essential operation does not have to wait for hydration.

### 32. Does every Client Component form submit before JavaScript loads?

Do not assume that. The exact behavior depends on the framework and form setup. Progressive enhancement is specifically provided by supported Server Component form patterns.

### 33. Does use server mean the component is a Server Component?

No. The directive marks a function or module exports as Server Functions. Server and Client Components are separate rendering categories.

### 34. Are Server Function implementations fully framework-independent?

No. React defines the concept and APIs, but framework and bundler support is implementation-specific. Pin or follow versions and instructions required by the chosen framework.

## Security and data design

### 35. Are Server Function arguments trusted because they run on the server?

No. The browser controls submitted arguments. Treat them as untrusted input and validate them on the server.

### 36. Do I still need authorization checks?

Yes. Authenticate the caller and verify they can perform the requested action on the requested resource inside every sensitive Server Function.

### 37. Can I trust a hidden field containing a user or record ID?

No. A user can modify it. Treat it as a requested identifier, then check ownership and permission using trusted server-side identity.

### 38. Can I return server secrets to the form state?

No. The function's return value is serialized to the caller. Return only the information the UI needs; keep credentials, private data, and internal error details on the server.

### 39. Are Server Functions a good way to fetch all page data?

They are designed primarily for mutations. Use Server Components, framework loaders, route handlers, or a query/data library for reads and caching when appropriate.

### 40. Does the server action request only come from the visible form?

No. Server Functions can be called through network requests, not only by clicking the rendered form. Security must be enforced inside the function.

### 41. What should happen after a successful mutation?

Persist the change, then update or revalidate the UI data using the framework's supported mechanism. A success message without a real write should say clearly that nothing was saved.

### 42. What is the main mental model?

The form sends serialized user-controlled data to an asynchronous server function through framework support. The function validates and authorizes the request, performs the mutation, and returns only safe UI state.
