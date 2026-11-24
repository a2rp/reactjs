# Reducer Design, Transitions, and State Machines — Example

This example is a local event-registration wizard with four modes:

~~~text
contact → session → review → complete
    ↑          ↑
    └── Back ──┘
~~~

A reducer owns the form data, current step, validation errors, and completion value. It allows only the transitions defined by the workflow. No information is sent to a server; completing the demo only changes local UI state.

## Project files

In a Vite React project, use these files:

~~~text
index.html
src/
  App.jsx
  main.jsx
  styles.css
~~~

## src/App.jsx

~~~jsx
import { useReducer } from "react";

const initialState = {
  step: "contact",
  values: {
    name: "",
    email: "",
    session: "",
    needsAccessSupport: false,
  },
  errors: {},
  confirmationCode: null,
};

const steps = [
  { id: "contact", label: "Contact" },
  { id: "session", label: "Session" },
  { id: "review", label: "Review" },
];

function validateContact(values) {
  const errors = {};

  if (!values.name.trim()) {
    errors.name = "Enter your name.";
  }

  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email)) {
    errors.email = "Enter a valid email address.";
  }

  return errors;
}

function validateSession(values) {
  if (!values.session) {
    return { session: "Choose a session." };
  }

  return {};
}

function registrationReducer(state, action) {
  switch (action.type) {
    case "registration/fieldChanged": {
      const contactFields = ["name", "email"];
      const sessionFields = ["session", "needsAccessSupport"];

      const fieldIsAllowed =
        (state.step === "contact" && contactFields.includes(action.field)) ||
        (state.step === "session" && sessionFields.includes(action.field));

      if (!fieldIsAllowed) {
        return state;
      }

      const nextErrors = { ...state.errors };
      delete nextErrors[action.field];

      return {
        ...state,
        values: {
          ...state.values,
          [action.field]: action.value,
        },
        errors: nextErrors,
      };
    }

    case "registration/nextRequested": {
      if (state.step === "contact") {
        const errors = validateContact(state.values);

        if (Object.keys(errors).length > 0) {
          return { ...state, errors };
        }

        return {
          ...state,
          step: "session",
          errors: {},
        };
      }

      if (state.step === "session") {
        const errors = validateSession(state.values);

        if (Object.keys(errors).length > 0) {
          return { ...state, errors };
        }

        return {
          ...state,
          step: "review",
          errors: {},
        };
      }

      return state;
    }

    case "registration/backRequested":
      if (state.step === "session") {
        return {
          ...state,
          step: "contact",
          errors: {},
        };
      }

      if (state.step === "review") {
        return {
          ...state,
          step: "session",
          errors: {},
        };
      }

      return state;

    case "registration/completed":
      if (state.step !== "review") {
        return state;
      }

      return {
        ...state,
        step: "complete",
        confirmationCode: action.confirmationCode,
        errors: {},
      };

    case "registration/reset":
      return initialState;

    default:
      throw Error("Unknown action: " + action.type);
  }
}

function StepProgress({ currentStep }) {
  const currentIndex = steps.findIndex((step) => step.id === currentStep);

  return (
    <ol className="step-progress" aria-label="Registration progress">
      {steps.map((step, index) => {
        const isComplete = currentStep === "complete" || index < currentIndex;
        const isCurrent = step.id === currentStep;

        return (
          <li
            className={isCurrent ? "step step-current" : "step"}
            key={step.id}
            aria-current={isCurrent ? "step" : undefined}
          >
            <span className="step-number" aria-hidden="true">
              {isComplete ? "✓" : index + 1}
            </span>
            <span>{step.label}</span>
          </li>
        );
      })}
    </ol>
  );
}

function FieldError({ id, message }) {
  if (!message) {
    return null;
  }

  return (
    <p className="field-error" id={id} role="alert">
      {message}
    </p>
  );
}

function ContactStep({ values, errors, dispatch, onNext }) {
  return (
    <form className="step-form" onSubmit={onNext} noValidate>
      <div className="step-heading">
        <p className="eyebrow">Step 1 of 3</p>
        <h2>Contact details</h2>
        <p>Enter the name and email address for this demo registration.</p>
      </div>

      <label className="field" htmlFor="registration-name">
        <span>Name</span>
        <input
          id="registration-name"
          autoComplete="name"
          value={values.name}
          aria-invalid={Boolean(errors.name)}
          aria-describedby={errors.name ? "registration-name-error" : undefined}
          onChange={(event) =>
            dispatch({
              type: "registration/fieldChanged",
              field: "name",
              value: event.target.value,
            })
          }
        />
        <FieldError
          id="registration-name-error"
          message={errors.name}
        />
      </label>

      <label className="field" htmlFor="registration-email">
        <span>Email</span>
        <input
          id="registration-email"
          type="email"
          autoComplete="email"
          value={values.email}
          aria-invalid={Boolean(errors.email)}
          aria-describedby={
            errors.email ? "registration-email-error" : undefined
          }
          onChange={(event) =>
            dispatch({
              type: "registration/fieldChanged",
              field: "email",
              value: event.target.value,
            })
          }
        />
        <FieldError
          id="registration-email-error"
          message={errors.email}
        />
      </label>

      <div className="step-actions step-actions-end">
        <button className="primary-button" type="submit">
          Continue to session
        </button>
      </div>
    </form>
  );
}

function SessionStep({ values, errors, dispatch, onBack, onNext }) {
  return (
    <form className="step-form" onSubmit={onNext} noValidate>
      <div className="step-heading">
        <p className="eyebrow">Step 2 of 3</p>
        <h2>Choose a session</h2>
        <p>Select one session and optionally request access support.</p>
      </div>

      <label className="field" htmlFor="registration-session">
        <span>Session</span>
        <select
          id="registration-session"
          value={values.session}
          aria-invalid={Boolean(errors.session)}
          aria-describedby={
            errors.session ? "registration-session-error" : undefined
          }
          onChange={(event) =>
            dispatch({
              type: "registration/fieldChanged",
              field: "session",
              value: event.target.value,
            })
          }
        >
          <option value="">Choose a session</option>
          <option value="morning">Morning session</option>
          <option value="afternoon">Afternoon session</option>
        </select>
        <FieldError
          id="registration-session-error"
          message={errors.session}
        />
      </label>

      <label className="checkbox-field">
        <input
          type="checkbox"
          checked={values.needsAccessSupport}
          onChange={(event) =>
            dispatch({
              type: "registration/fieldChanged",
              field: "needsAccessSupport",
              value: event.target.checked,
            })
          }
        />
        <span>I would like the organizer to contact me about access support.</span>
      </label>

      <div className="step-actions">
        <button
          className="secondary-button"
          type="button"
          onClick={onBack}
        >
          Back
        </button>
        <button className="primary-button" type="submit">
          Review registration
        </button>
      </div>
    </form>
  );
}

function ReviewStep({ values, onBack, onComplete }) {
  const sessionLabel =
    values.session === "morning" ? "Morning session" : "Afternoon session";

  return (
    <section className="step-form" aria-labelledby="review-heading">
      <div className="step-heading">
        <p className="eyebrow">Step 3 of 3</p>
        <h2 id="review-heading">Review your details</h2>
        <p>Go back to change a value, or complete this local demo flow.</p>
      </div>

      <dl className="review-list">
        <div>
          <dt>Name</dt>
          <dd>{values.name}</dd>
        </div>
        <div>
          <dt>Email</dt>
          <dd>{values.email}</dd>
        </div>
        <div>
          <dt>Session</dt>
          <dd>{sessionLabel}</dd>
        </div>
        <div>
          <dt>Access support</dt>
          <dd>{values.needsAccessSupport ? "Requested" : "Not requested"}</dd>
        </div>
      </dl>

      <div className="step-actions">
        <button className="secondary-button" type="button" onClick={onBack}>
          Back
        </button>
        <button className="primary-button" type="button" onClick={onComplete}>
          Complete demo registration
        </button>
      </div>
    </section>
  );
}

function CompleteStep({ confirmationCode, onReset }) {
  return (
    <section className="complete-panel" aria-labelledby="complete-heading">
      <p className="complete-mark" aria-hidden="true">
        ✓
      </p>
      <p className="eyebrow">Flow complete</p>
      <h2 id="complete-heading">You reached the final state.</h2>
      <p>
        Demo confirmation code: <strong>{confirmationCode}</strong>
      </p>
      <p className="notice">
        This example only changes local UI state. It does not send or store a
        registration.
      </p>
      <button className="primary-button" type="button" onClick={onReset}>
        Start again
      </button>
    </section>
  );
}

export default function App() {
  const [state, dispatch] = useReducer(registrationReducer, initialState);

  function handleNext(event) {
    event.preventDefault();
    dispatch({ type: "registration/nextRequested" });
  }

  function handleComplete() {
    const confirmationCode =
      "DEMO-" + crypto.randomUUID().slice(0, 8).toUpperCase();

    dispatch({
      type: "registration/completed",
      confirmationCode,
    });
  }

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">Reducer transitions</p>
        <h1>Event registration flow</h1>
        <p className="intro">
          Move through the steps, try invalid input, and observe how the reducer
          guards each transition.
        </p>
      </header>

      <section className="registration-card" aria-label="Demo registration">
        {state.step !== "complete" && (
          <StepProgress currentStep={state.step} />
        )}

        {state.step === "contact" && (
          <ContactStep
            values={state.values}
            errors={state.errors}
            dispatch={dispatch}
            onNext={handleNext}
          />
        )}

        {state.step === "session" && (
          <SessionStep
            values={state.values}
            errors={state.errors}
            dispatch={dispatch}
            onBack={() => dispatch({ type: "registration/backRequested" })}
            onNext={handleNext}
          />
        )}

        {state.step === "review" && (
          <ReviewStep
            values={state.values}
            onBack={() => dispatch({ type: "registration/backRequested" })}
            onComplete={handleComplete}
          />
        )}

        {state.step === "complete" && (
          <CompleteStep
            confirmationCode={state.confirmationCode}
            onReset={() => dispatch({ type: "registration/reset" })}
          />
        )}
      </section>

      <p className="page-note">
        The reducer owns one current step. Validation errors keep the flow in
        that step until its guard passes.
      </p>
    </main>
  );
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/styles.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  color: #18212f;
  background: #f3f6fb;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button,
input,
select {
  font: inherit;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #82aaff;
  outline-offset: 3px;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.page {
  width: min(100% - 2rem, 760px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin: 0 0 0.45rem;
  color: #315db5;
  font-size: 0.76rem;
  font-weight: 750;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

h1,
h2,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 6vw, 3.2rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0.5rem;
  font-size: 1.35rem;
}

.intro,
.step-heading > p:last-child {
  color: #526071;
  line-height: 1.65;
}

.intro {
  max-width: 64ch;
  margin-bottom: 0;
}

.registration-card {
  overflow: hidden;
  border: 1px solid #d8e0eb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 35px rgb(32 53 84 / 7%);
}

.step-progress {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.5rem;
  margin: 0;
  padding: 1rem 1.25rem;
  border-bottom: 1px solid #e4eaf2;
  background: #fbfcfe;
  list-style: none;
}

.step {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  color: #718096;
  font-size: 0.9rem;
}

.step-current {
  color: #244b9a;
  font-weight: 750;
}

.step-number {
  display: grid;
  width: 1.7rem;
  height: 1.7rem;
  flex: 0 0 auto;
  place-items: center;
  border: 1px solid #c9d3e2;
  border-radius: 50%;
  background: #fff;
  font-size: 0.78rem;
}

.step-current .step-number {
  border-color: #315db5;
  color: #fff;
  background: #315db5;
}

.step-form,
.complete-panel {
  padding: clamp(1rem, 4vw, 1.6rem);
}

.step-heading {
  margin-bottom: 1.3rem;
}

.step-heading > p:last-child {
  margin-bottom: 0;
}

.field {
  display: grid;
  gap: 0.45rem;
  margin-top: 1rem;
  font-size: 0.94rem;
  font-weight: 650;
}

.field input,
.field select {
  width: 100%;
  padding: 0.72rem 0.8rem;
  border: 1px solid #bac8dc;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
}

.field input[aria-invalid="true"],
.field select[aria-invalid="true"] {
  border-color: #ba4545;
}

.field-error {
  margin: 0;
  color: #a22e2e;
  font-size: 0.88rem;
  font-weight: 500;
}

.checkbox-field {
  display: flex;
  align-items: flex-start;
  gap: 0.65rem;
  margin-top: 1.2rem;
  color: #344257;
  line-height: 1.5;
}

.checkbox-field input {
  flex: 0 0 auto;
  width: 1rem;
  height: 1rem;
  margin-top: 0.2rem;
  accent-color: #315db5;
}

.step-actions {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 0.7rem;
  margin-top: 1.5rem;
}

.step-actions-end {
  justify-content: flex-end;
}

.primary-button,
.secondary-button {
  padding: 0.7rem 0.9rem;
  border-radius: 0.55rem;
  cursor: pointer;
}

.primary-button {
  border: 1px solid #315db5;
  color: #fff;
  background: #315db5;
  font-weight: 700;
}

.primary-button:hover:not(:disabled) {
  background: #244b9a;
}

.secondary-button {
  border: 1px solid #bac8dc;
  color: #244578;
  background: #f7f9fc;
}

.secondary-button:hover:not(:disabled) {
  background: #edf3fb;
}

.review-list {
  margin: 0;
}

.review-list div {
  display: grid;
  grid-template-columns: minmax(8rem, 0.4fr) minmax(0, 1fr);
  gap: 1rem;
  padding-block: 0.8rem;
  border-top: 1px solid #e4eaf2;
}

.review-list dt {
  color: #637185;
}

.review-list dd {
  margin: 0;
  overflow-wrap: anywhere;
  font-weight: 650;
}

.complete-panel {
  text-align: center;
}

.complete-panel h2 {
  margin-bottom: 0.75rem;
}

.complete-mark {
  display: grid;
  width: 3rem;
  height: 3rem;
  place-items: center;
  margin: 0 auto 1rem;
  border-radius: 50%;
  color: #17653c;
  background: #e4f6ec;
  font-size: 1.4rem;
  font-weight: 800;
}

.notice {
  max-width: 48ch;
  margin: 1rem auto 1.3rem;
  color: #637185;
  font-size: 0.9rem;
  line-height: 1.6;
}

.page-note {
  margin: 1rem 0 0;
  color: #637185;
  font-size: 0.9rem;
  line-height: 1.6;
}

@media (max-width: 520px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .step-progress {
    padding-inline: 0.8rem;
  }

  .step {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.35rem;
    font-size: 0.8rem;
  }

  .review-list div {
    grid-template-columns: 1fr;
    gap: 0.25rem;
  }
}
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="A React reducer example with guarded state-machine transitions."
    />
    <title>Reducer State Machine — React Example</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How it works

### State model

The reducer state has one step value, shared form values, validation errors, and an optional confirmation code. The step is always one of contact, session, review, or complete.

### Transitions and guards

- Continue from contact runs validateContact. Invalid values keep the contact step active and populate errors. Valid values move to session.
- Continue from session runs validateSession. Without a session choice, the flow stays on session and displays an error. Otherwise it moves to review.
- Back moves session to contact or review to session.
- Complete is accepted only from review.
- Reset returns the flow to its initial state.

These rules live in registrationReducer rather than being spread across the UI components.

### Side effects stay outside the reducer

The completion handler creates the demo confirmation code and includes it in an action. The reducer only stores the provided value and changes the step. There is no network request in this example.

### Controlled form fields

The inputs render values from reducer state and dispatch fieldChanged actions. A field can only change in the step that owns it. Validation errors are part of the state, so the UI can associate them with the corresponding input.

## Run it

From the Vite project root, install the project's dependencies if needed and start its development server:

~~~sh
npm install
npm run dev
~~~

Open the local address printed by Vite. Try continuing with empty contact fields, enter valid values, try continuing without a session, then finish and start again.

## What to notice

- The step field models mutually exclusive workflow modes.
- The transition table becomes guarded reducer cases.
- Validation errors keep the user in the current mode rather than throwing.
- One action changes related fields together.
- The confirmation code is created outside the reducer.
- The complete screen explicitly says this local example does not submit data.
