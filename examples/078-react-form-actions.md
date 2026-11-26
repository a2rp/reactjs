# React Form Actions and Asynchronous Submissions - Example

This React 19 + Vite example uses a function form Action to add a note. It reads FormData, simulates an asynchronous save, shows pending state with useFormStatus, and relies on React to reset uncontrolled fields after success.

This is a client-side demonstration. It is not a Server Function, does not persist data, and does not replace server validation or authorization.

## Create the project

    npm create vite@latest form-actions-demo -- --template react
    cd form-actions-demo
    npm install
    npm run dev

Replace the generated files with these.

## File structure

    src/
      App.jsx
      App.css
      ErrorBoundary.jsx
      main.jsx

## src/ErrorBoundary.jsx

    import { Component } from 'react';

    export class ErrorBoundary extends Component {
      state = { error: null };

      static getDerivedStateFromError(error) {
        return { error };
      }

      render() {
        if (this.state.error) {
          return (
            <section className="error" role="alert">
              <h2>Could not save the note</h2>
              <p>{this.state.error.message}</p>
              <button type="button" onClick={() => window.location.reload()}>
                Reload the demo
              </button>
            </section>
          );
        }

        return this.props.children;
      }
    }

The form Action throws only if its submitted data fails the example's defensive checks. React routes that error to this boundary. A production form would normally preserve submitted fields and show structured validation feedback; the next lesson covers stateful Action results.

## src/App.jsx

    import { useState } from 'react';
    import { useFormStatus } from 'react-dom';
    import { ErrorBoundary } from './ErrorBoundary.jsx';
    import './App.css';

    let nextNoteId = 3;

    function SubmitButton() {
      const { pending } = useFormStatus();

      return (
        <button type="submit" disabled={pending}>
          {pending ? 'Saving note…' : 'Add note'}
        </button>
      );
    }

    function NoteList({ notes }) {
      return (
        <section className="panel" aria-labelledby="notes-heading">
          <h2 id="notes-heading">Saved notes</h2>

          {notes.length === 0 ? (
            <p>No notes yet.</p>
          ) : (
            <ul className="notes">
              {notes.map(note => (
                <li key={note.id}>
                  <h3>{note.title}</h3>
                  {note.details && <p>{note.details}</p>}
                </li>
              ))}
            </ul>
          )}
        </section>
      );
    }

    export default function App() {
      const [notes, setNotes] = useState([
        {
          id: 1,
          title: 'Use semantic forms',
          details: 'Give every submitted control a useful name.',
        },
        {
          id: 2,
          title: 'Show clear progress',
          details: 'Tell the user when an asynchronous save is running.',
        },
      ]);
      const [notice, setNotice] = useState('');

      async function addNote(formData) {
        const rawTitle = formData.get('title');
        const rawDetails = formData.get('details');

        if (typeof rawTitle !== 'string') {
          throw new Error('A text title is required.');
        }

        const title = rawTitle.trim();
        const details =
          typeof rawDetails === 'string' ? rawDetails.trim() : '';

        if (title.length < 3) {
          throw new Error('The title must contain at least three characters.');
        }

        await new Promise(resolve => window.setTimeout(resolve, 800));

        setNotes(currentNotes => [
          { id: nextNoteId++, title, details },
          ...currentNotes,
        ]);
        setNotice('Note saved. The form has been cleared.');
      }

      return (
        <main className="page">
          <header className="header">
            <p className="eyebrow">React form Actions</p>
            <h1>Quick notes</h1>
            <p>
              Submit a note and watch React show pending feedback while the
              asynchronous Action runs.
            </p>
          </header>

          <div className="layout">
            <section className="panel" aria-labelledby="form-heading">
              <h2 id="form-heading">Add a note</h2>

              <ErrorBoundary>
                <form action={addNote} className="note-form">
                  <label htmlFor="title">Title</label>
                  <input
                    id="title"
                    name="title"
                    required
                    minLength={3}
                    maxLength={80}
                    placeholder="For example, FormData basics"
                  />

                  <label htmlFor="details">Details</label>
                  <textarea
                    id="details"
                    name="details"
                    rows={4}
                    maxLength={240}
                    placeholder="What do you want to remember?"
                  />

                  <SubmitButton />
                </form>
              </ErrorBoundary>

              <p className="notice" role="status" aria-live="polite">
                {notice}
              </p>
            </section>

            <NoteList notes={notes} />
          </div>
        </main>
      );
    }

SubmitButton is a child rendered inside the form, so useFormStatus observes the correct form. The inputs are uncontrolled: after addNote resolves successfully, React resets them to their defaults. The saved notes remain in component state until the page reloads.

## src/App.css

    :root {
      font-family: Inter, ui-sans-serif, system-ui, sans-serif;
      color: #e8edf5;
      background: #0d1117;
      font-synthesis: none;
      text-rendering: optimizeLegibility;
    }

    * {
      box-sizing: border-box;
    }

    body {
      min-width: 320px;
      min-height: 100vh;
      margin: 0;
    }

    button,
    input,
    textarea {
      font: inherit;
    }

    button {
      width: fit-content;
      border: 0;
      border-radius: 0.65rem;
      padding: 0.75rem 1rem;
      color: #08111f;
      background: #a5d6ff;
      font-weight: 700;
      cursor: pointer;
    }

    button:hover:not(:disabled) {
      background: #c2e3ff;
    }

    button:disabled {
      cursor: wait;
      opacity: 0.7;
    }

    button:focus-visible,
    input:focus-visible,
    textarea:focus-visible {
      outline: 3px solid #f0b429;
      outline-offset: 3px;
    }

    .page {
      width: min(100% - 2rem, 1040px);
      margin: 0 auto;
      padding: 4rem 0;
    }

    .header {
      max-width: 680px;
      margin-bottom: 2rem;
    }

    .header h1 {
      margin: 0.25rem 0 0.5rem;
      font-size: clamp(2rem, 5vw, 3rem);
    }

    .header p {
      color: #aebbd0;
      line-height: 1.6;
    }

    .eyebrow {
      color: #8bceff !important;
      font-size: 0.8rem;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
    }

    .layout {
      display: grid;
      grid-template-columns: minmax(0, 0.9fr) minmax(0, 1.1fr);
      align-items: start;
      gap: 1rem;
    }

    .panel {
      border: 1px solid #303b4b;
      border-radius: 1rem;
      padding: 1.5rem;
      background: #151c27;
    }

    .panel h2 {
      margin-top: 0;
    }

    .note-form {
      display: grid;
      gap: 0.65rem;
    }

    .note-form label {
      margin-top: 0.5rem;
      font-weight: 700;
    }

    .note-form input,
    .note-form textarea {
      width: 100%;
      border: 1px solid #45536a;
      border-radius: 0.6rem;
      padding: 0.75rem;
      color: #e8edf5;
      background: #0d1117;
    }

    .note-form textarea {
      resize: vertical;
    }

    .notice {
      min-height: 1.5em;
      color: #7ee0ae;
    }

    .notes {
      display: grid;
      gap: 0.75rem;
      margin: 0;
      padding: 0;
      list-style: none;
    }

    .notes li {
      border-top: 1px solid #303b4b;
      padding-top: 0.75rem;
    }

    .notes h3 {
      margin: 0 0 0.4rem;
      font-size: 1rem;
    }

    .notes p,
    .error p {
      margin-bottom: 0;
      color: #aebbd0;
      line-height: 1.5;
    }

    .error {
      border: 1px solid #a64545;
      border-radius: 0.75rem;
      padding: 1rem;
      background: #291719;
    }

    .error h2 {
      color: #ffc5c5;
    }

    @media (max-width: 720px) {
      .page {
        padding: 2rem 0;
      }

      .layout {
        grid-template-columns: 1fr;
      }
    }

## src/main.jsx

    import { StrictMode } from 'react';
    import { createRoot } from 'react-dom/client';
    import App from './App.jsx';

    createRoot(document.getElementById('root')).render(
      <StrictMode>
        <App />
      </StrictMode>,
    );

## Try the example

1. Enter a title with at least three characters and optional details.
2. Submit the form. The button changes to “Saving note…” and is disabled during the delay.
3. After the Action succeeds, the note appears in the list and the uncontrolled form fields reset.
4. Try an empty or too-short title. Native constraints block the normal submission; the Action also checks its input defensively and the Error Boundary handles a thrown failure.

## What this demonstrates

- A function passed to form action receives FormData.
- An async Action can wait for work while useFormStatus exposes pending state to a nested submit button.
- Successful submission resets uncontrolled fields.
- Errors from an Action need a recovery boundary.
- A client Action is not a Server Function and does not persist data across reloads.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="077-async-ui-and-framework-data-loading.md">077 - Async UI, Suspense, and Framework Data Loading - Example</a></td><td><a href="../lessons/078-react-form-actions.md">Lesson</a></td><td><a href="../qnas/078-react-form-actions.md">Q&amp;A</a></td><td><a href="079-useactionstate.md">079 - Managing Submissions with useActionState - Example</a></td></tr>
  </tbody>
</table>
