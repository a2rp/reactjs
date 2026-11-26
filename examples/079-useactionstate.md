# Managing Submissions with useActionState - Example

This React 19 + Vite demo edits a small profile. The Action returns a state object with field errors, submitted values, pending status, and the saved profile. Validation failures preserve what the user typed; success clears the form and updates the profile summary.

The save is simulated in the browser. This is a learning example, not a Server Function or persistent API.

## Create the project

    npm create vite@latest action-state-demo -- --template react
    cd action-state-demo
    npm install
    npm run dev

Replace the generated files with these.

## File structure

    src/
      App.jsx
      App.css
      main.jsx

## src/App.jsx

    import { useActionState } from 'react';
    import './App.css';

    const initialState = {
      status: 'idle',
      message: '',
      errors: {},
      submitted: { name: '', email: '' },
      profile: null,
    };

    function wait(milliseconds) {
      return new Promise(resolve => {
        window.setTimeout(resolve, milliseconds);
      });
    }

    async function updateProfile(previousState, formData) {
      const nameValue = formData.get('name');
      const emailValue = formData.get('email');

      const submitted = {
        name: typeof nameValue === 'string' ? nameValue.trim() : '',
        email: typeof emailValue === 'string' ? emailValue.trim() : '',
      };

      const errors = {};

      if (submitted.name.length < 3) {
        errors.name = 'Use at least three characters.';
      }

      if (!submitted.email) {
        errors.email = 'Enter an email address.';
      }

      if (Object.keys(errors).length > 0) {
        return {
          status: 'error',
          message: 'Please correct the highlighted fields.',
          errors,
          submitted,
          profile: previousState.profile,
        };
      }

      await wait(700);

      if (submitted.email === 'fail@example.test') {
        return {
          status: 'error',
          message: 'The demo service could not save this address. Try another.',
          errors: {},
          submitted,
          profile: previousState.profile,
        };
      }

      return {
        status: 'success',
        message: 'Profile saved.',
        errors: {},
        submitted: { name: '', email: '' },
        profile: submitted,
      };
    }

    export default function App() {
      const [state, submitAction, isPending] =
        useActionState(updateProfile, initialState);

      return (
        <main className="page">
          <header className="header">
            <p className="eyebrow">React form state</p>
            <h1>Update profile</h1>
            <p>
              Validation results return as state, so the form can explain an
              error and keep the submitted values.
            </p>
          </header>

          <div className="layout">
            <section className="panel" aria-labelledby="form-title">
              <h2 id="form-title">Profile details</h2>

              <form action={submitAction} className="profile-form">
                <label htmlFor="name">Name</label>
                <input
                  id="name"
                  name="name"
                  autoComplete="name"
                  defaultValue={state.submitted.name}
                  disabled={isPending}
                  aria-invalid={Boolean(state.errors.name)}
                  aria-describedby="name-help"
                  required
                />
                <p
                  id="name-help"
                  className={state.errors.name ? 'field-error' : 'hint'}
                >
                  {state.errors.name || 'Use at least three characters.'}
                </p>

                <label htmlFor="email">Email</label>
                <input
                  id="email"
                  name="email"
                  type="email"
                  autoComplete="email"
                  defaultValue={state.submitted.email}
                  disabled={isPending}
                  aria-invalid={Boolean(state.errors.email)}
                  aria-describedby="email-help"
                  required
                />
                <p
                  id="email-help"
                  className={state.errors.email ? 'field-error' : 'hint'}
                >
                  {state.errors.email || 'Use fail@example.test to see a returned service error.'}
                </p>

                <button type="submit" disabled={isPending}>
                  {isPending ? 'Saving…' : 'Save profile'}
                </button>
              </form>

              {state.message && (
                <p
                  className={state.status === 'error' ? 'form-error' : 'success'}
                  role={state.status === 'error' ? 'alert' : 'status'}
                  aria-live="polite"
                >
                  {state.message}
                </p>
              )}
            </section>

            <section className="panel" aria-labelledby="saved-title">
              <h2 id="saved-title">Saved profile</h2>

              {state.profile ? (
                <dl className="profile">
                  <div>
                    <dt>Name</dt>
                    <dd>{state.profile.name}</dd>
                  </div>
                  <div>
                    <dt>Email</dt>
                    <dd>{state.profile.email}</dd>
                  </div>
                </dl>
              ) : (
                <p>No profile has been saved in this session.</p>
              )}
            </section>
          </div>
        </main>
      );
    }

updateProfile receives previousState first and FormData second because useActionState wraps it. Expected validation and simulated service errors return structured state, so React renders them without throwing to an Error Boundary.

The submitted object becomes each input's new defaultValue. When the Action completes with an error, the form reset uses those updated defaults and preserves the values. On success, the returned defaults are empty strings, so the uncontrolled fields clear.

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
    input {
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

    button:disabled,
    input:disabled {
      cursor: wait;
      opacity: 0.7;
    }

    button:focus-visible,
    input:focus-visible {
      outline: 3px solid #f0b429;
      outline-offset: 3px;
    }

    .page {
      width: min(100% - 2rem, 1040px);
      margin: 0 auto;
      padding: 4rem 0;
    }

    .header {
      max-width: 700px;
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
      grid-template-columns: repeat(2, minmax(0, 1fr));
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

    .profile-form {
      display: grid;
      gap: 0.55rem;
    }

    .profile-form label {
      margin-top: 0.6rem;
      font-weight: 700;
    }

    .profile-form input {
      width: 100%;
      border: 1px solid #45536a;
      border-radius: 0.6rem;
      padding: 0.75rem;
      color: #e8edf5;
      background: #0d1117;
    }

    .profile-form input[aria-invalid="true"] {
      border-color: #ff8b8b;
    }

    .hint,
    .field-error {
      margin: 0 0 0.5rem;
      font-size: 0.875rem;
      line-height: 1.45;
    }

    .hint {
      color: #aebbd0;
    }

    .field-error,
    .form-error {
      color: #ffaaaa;
    }

    .success {
      color: #7ee0ae;
    }

    .profile {
      display: grid;
      gap: 1rem;
    }

    .profile div {
      border-top: 1px solid #303b4b;
      padding-top: 0.75rem;
    }

    .profile dt {
      color: #aebbd0;
      font-size: 0.85rem;
    }

    .profile dd {
      margin: 0.25rem 0 0;
      overflow-wrap: anywhere;
    }

    @media (max-width: 700px) {
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

1. Submit a one- or two-character name. The Action returns a validation error and keeps the typed value.
2. Enter a valid name and an ordinary email address. The Action waits, shows a pending label, then saves the profile and clears the fields.
3. Enter fail@example.test as the email. The simulated service returns a recoverable error and preserves both values.
4. Reload the page. The saved profile disappears because this example stores state only in memory.

## What this demonstrates

- The Action receives previous state before FormData.
- useActionState returns state, a form dispatcher, and isPending.
- Expected validation and service failures are normal returned state.
- Updated defaultValue values preserve failed submissions during React's form reset.
- Success can return empty defaults to clear the form.
- The browser demo does not provide server persistence, authentication, or authorization.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="078-react-form-actions.md">078 - React Form Actions and Asynchronous Submissions - Example</a></td><td><a href="../lessons/079-useactionstate.md">Lesson</a></td><td><a href="../qnas/079-useactionstate.md">Q&amp;A</a></td><td><a href="080-useformstatus.md">080 - Example: Pending form state with useFormStatus</a></td></tr>
  </tbody>
</table>
