# Server Functions and form Actions - Example

## Goal

Create a small Next.js App Router form that submits to a Server Function. The function validates the submitted fields on the server and returns state through useActionState.

This example demonstrates the server request and validation flow. It intentionally does not connect to a database, so it does not persist a note.

## Create the project

Create a recent Next.js project and choose JavaScript, the App Router, and no src directory when prompted:

    npx create-next-app@latest server-form-demo
    cd server-form-demo
    npm run dev

Keep the generated app/layout.js and app/globals.css. Add or replace the following files.

## app/actions.js

The file-level directive marks its exported asynchronous function as a Server Function. Only export async Server Functions from this module.

    "use server";

    export async function createNote(previousState, formData) {
      const titleValue = formData.get("title");
      const bodyValue = formData.get("body");

      const title =
        typeof titleValue === "string" ? titleValue.trim() : "";
      const body =
        typeof bodyValue === "string" ? bodyValue.trim() : "";

      if (title.length < 3 || title.length > 80) {
        return {
          status: "error",
          message: "Use a title between 3 and 80 characters.",
          title,
          body,
        };
      }

      if (body.length < 10 || body.length > 1000) {
        return {
          status: "error",
          message: "Use note text between 10 and 1,000 characters.",
          title,
          body,
        };
      }

      // A short delay makes the pending UI visible in this demo.
      await new Promise((resolve) => {
        setTimeout(resolve, 500);
      });

      return {
        status: "demo",
        message:
          'The Server Function validated "' +
          title +
          '". No database is configured, so the note was not saved.',
        title,
        body,
      };
    }

The previousState argument is present because the Client Component wraps this function with useActionState. The submitted FormData is the second argument.

## app/note-form.jsx

    "use client";

    import { useActionState } from "react";
    import { createNote } from "./actions";

    const initialState = {
      status: "idle",
      message: "",
      title: "",
      body: "",
    };

    export default function NoteForm() {
      const [state, formAction, isPending] =
        useActionState(createNote, initialState);

      return (
        <form action={formAction}>
          <p>
            <label htmlFor="title">Title</label>
            <br />
            <input
              id="title"
              name="title"
              type="text"
              minLength={3}
              maxLength={80}
              required
              defaultValue={state.title}
            />
          </p>

          <p>
            <label htmlFor="body">Note</label>
            <br />
            <textarea
              id="body"
              name="body"
              minLength={10}
              maxLength={1000}
              rows={5}
              required
              defaultValue={state.body}
            />
          </p>

          <button type="submit" disabled={isPending}>
            {isPending ? "Checking on the server…" : "Submit note"}
          </button>

          {state.message &&
            (state.status === "error" ? (
              <p role="alert">{state.message}</p>
            ) : (
              <p role="status">{state.message}</p>
            ))}
        </form>
      );
    }

The returned title and body become the fields' new default values. This preserves a user's text when the Action returns a validation message and when this demo returns its final message.

## app/page.js

    import NoteForm from "./note-form";

    export default function HomePage() {
      return (
        <main>
          <h1>Server Function form</h1>
          <p>
            Submit a note to a server-side function. The demo validates it
            but does not store it.
          </p>
          <NoteForm />
        </main>
      );
    }

The page is a Server Component by default. NoteForm is a Client Component because it uses useActionState and interactive pending UI.

## Try it

1. Enter a title shorter than 3 characters. The browser may block submission because of minLength; the Server Function also checks the value.
2. Enter a valid title but note text shorter than 10 characters to see a returned validation error.
3. Enter valid values and submit. The button shows a pending label while the Server Function waits.
4. Read the final message. It confirms that the server function ran and validated the note, and clearly says no database write occurred.
5. To turn this into a real feature, authenticate the user, authorize the mutation, write to a database, and return only safe UI state.

## Request flow

1. The browser submits the named fields as FormData.
2. useActionState wraps createNote and supplies the previous state as the first argument.
3. The framework sends the Server Function request to the server.
4. The Server Function type-checks and validates the user-controlled values.
5. The returned serializable object becomes the form's new state.
6. isPending drives the submit button while the Action is running.

## Important boundaries

- This requires a Next.js App Router setup or another framework that supports Server Functions. A plain React plus Vite client app will not execute this function on a server.
- The HTML constraints are for usability. The server-side checks still run and remain necessary.
- This function is callable through a network request, so real code must authenticate and authorize the requested operation inside the function.
- A returned message does not mean data was persisted. Connect a real data layer before claiming that a note was saved.
- Do not return secrets or raw internal errors in state sent back to the browser.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="081-useoptimistic.md">081 - Optimistic UI with useOptimistic - Example</a></td><td><a href="../lessons/082-server-functions-and-form-actions.md">Lesson</a></td><td><a href="../qnas/082-server-functions-and-form-actions.md">Q&amp;A</a></td><td><a href="083-react-server-components.md">083 - React Server Components: lesson list with one interactive island</a></td></tr>
  </tbody>
</table>
