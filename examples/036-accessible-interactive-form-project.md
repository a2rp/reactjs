# 036. Build an accessible interactive form feature - Example

This runnable example builds a course-question form with four required fields. It validates the values in the browser, shows errors beside affected controls, focuses a linked error summary after a failed submission, and announces a truthful local result after valid input. It does not send or store data.

## Project files

Create these files in an existing React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

Start the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useEffect, useRef, useState } from 'react';

const initialValues = {
  name: '',
  email: '',
  topic: '',
  question: '',
};

const fieldLabels = {
  name: 'Name',
  email: 'Email address',
  topic: 'Topic',
  question: 'Question',
};

function validate(values, emailHasTypeMismatch) {
  const nextErrors = {};

  if (!values.name.trim()) {
    nextErrors.name = 'Enter your name.';
  }

  if (!values.email.trim()) {
    nextErrors.email = 'Enter an email address.';
  } else if (emailHasTypeMismatch) {
    nextErrors.email = 'Enter an email address in a valid format.';
  }

  if (!values.topic) {
    nextErrors.topic = 'Choose a topic.';
  }

  const question = values.question.trim();

  if (!question) {
    nextErrors.question = 'Enter your question.';
  } else if (question.length < 20) {
    nextErrors.question = 'Enter at least 20 non-space characters.';
  }

  return nextErrors;
}

function App() {
  const [values, setValues] = useState(initialValues);
  const [touched, setTouched] = useState({});
  const [errors, setErrors] = useState({});
  const [submitted, setSubmitted] = useState(false);
  const [failedSubmitCount, setFailedSubmitCount] = useState(0);
  const [status, setStatus] = useState('');

  const emailRef = useRef(null);
  const errorSummaryRef = useRef(null);

  useEffect(() => {
    if (failedSubmitCount > 0) {
      errorSummaryRef.current?.focus();
    }
  }, [failedSubmitCount]);

  function handleChange(event) {
    const { name, value } = event.currentTarget;
    const nextValues = {
      ...values,
      [name]: value,
    };

    setValues(nextValues);
    setStatus('');

    if (submitted || touched[name]) {
      const emailHasTypeMismatch =
        name === 'email'
          ? event.currentTarget.validity.typeMismatch
          : Boolean(emailRef.current?.validity.typeMismatch);

      setErrors(validate(nextValues, emailHasTypeMismatch));
    }
  }

  function handleBlur(event) {
    const { name } = event.currentTarget;

    setTouched((currentTouched) => ({
      ...currentTouched,
      [name]: true,
    }));

    setErrors(
      validate(values, Boolean(emailRef.current?.validity.typeMismatch)),
    );
  }

  function handleSubmit(event) {
    event.preventDefault();

    const nextErrors = validate(
      values,
      Boolean(emailRef.current?.validity.typeMismatch),
    );

    setErrors(nextErrors);
    setSubmitted(true);
    setStatus('');

    if (Object.keys(nextErrors).length > 0) {
      setFailedSubmitCount((count) => count + 1);
      return;
    }

    setStatus(
      'Your entries passed local checks. This example did not send data.',
    );
  }

  function isErrorVisible(fieldName) {
    return Boolean(errors[fieldName] && (touched[fieldName] || submitted));
  }

  const errorEntries = Object.entries(errors);

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React form project</p>
        <h1>Ask a course question</h1>
        <p>
          Share a question about the learning notes. This demonstration checks
          your entries in the browser only; it does not send or save them.
        </p>
      </header>

      <p className="form-instructions" id="form-help">
        All fields are required. Your question must contain at least 20
        non-space characters and no more than 1,000 characters.
      </p>

      {submitted && errorEntries.length > 0 && (
        <section
          className="error-summary"
          ref={errorSummaryRef}
          tabIndex={-1}
          aria-labelledby="error-summary-title"
        >
          <h2 id="error-summary-title">
            Review {errorEntries.length}{' '}
            {errorEntries.length === 1 ? 'issue' : 'issues'}
          </h2>
          <p>Use these links to move to the fields that need attention.</p>
          <ul>
            {errorEntries.map(([fieldName, message]) => (
              <li key={fieldName}>
                <a href={'#' + fieldName}>
                  {fieldLabels[fieldName]}: {message}
                </a>
              </li>
            ))}
          </ul>
        </section>
      )}

      <form className="question-form" onSubmit={handleSubmit} noValidate>
        <div className="field">
          <label htmlFor="name">Name (required)</label>
          <p className="field-help" id="name-help">
            Enter the name you would like us to use.
          </p>
          <input
            id="name"
            name="name"
            type="text"
            autoComplete="name"
            required
            value={values.name}
            aria-invalid={isErrorVisible('name') ? 'true' : undefined}
            aria-describedby={
              isErrorVisible('name') ? 'name-help name-error' : 'name-help'
            }
            onChange={handleChange}
            onBlur={handleBlur}
          />
          {isErrorVisible('name') && (
            <p className="field-error" id="name-error">
              {errors.name}
            </p>
          )}
        </div>

        <div className="field">
          <label htmlFor="email">Email address (required)</label>
          <p className="field-help" id="email-help">
            Use an address where you can receive a reply.
          </p>
          <input
            id="email"
            ref={emailRef}
            name="email"
            type="email"
            autoComplete="email"
            required
            value={values.email}
            aria-invalid={isErrorVisible('email') ? 'true' : undefined}
            aria-describedby={
              isErrorVisible('email')
                ? 'email-help email-error'
                : 'email-help'
            }
            onChange={handleChange}
            onBlur={handleBlur}
          />
          {isErrorVisible('email') && (
            <p className="field-error" id="email-error">
              {errors.email}
            </p>
          )}
        </div>

        <div className="field">
          <label htmlFor="topic">Topic (required)</label>
          <p className="field-help" id="topic-help">
            Choose the area that best matches your question.
          </p>
          <select
            id="topic"
            name="topic"
            required
            value={values.topic}
            aria-invalid={isErrorVisible('topic') ? 'true' : undefined}
            aria-describedby={
              isErrorVisible('topic') ? 'topic-help topic-error' : 'topic-help'
            }
            onChange={handleChange}
            onBlur={handleBlur}
          >
            <option value="">Choose a topic</option>
            <option value="react">React</option>
            <option value="html">HTML</option>
            <option value="css">CSS</option>
            <option value="javascript">JavaScript</option>
            <option value="other">Other</option>
          </select>
          {isErrorVisible('topic') && (
            <p className="field-error" id="topic-error">
              {errors.topic}
            </p>
          )}
        </div>

        <div className="field">
          <label htmlFor="question">Question (required)</label>
          <p className="field-help" id="question-help">
            Include enough detail to explain what you are trying to understand.
          </p>
          <textarea
            id="question"
            name="question"
            rows="6"
            maxLength="1000"
            required
            value={values.question}
            aria-invalid={isErrorVisible('question') ? 'true' : undefined}
            aria-describedby={
              isErrorVisible('question')
                ? 'question-help question-count question-error'
                : 'question-help question-count'
            }
            onChange={handleChange}
            onBlur={handleBlur}
          />
          <p className="character-count" id="question-count">
            {values.question.length} of 1,000 characters
          </p>
          {isErrorVisible('question') && (
            <p className="field-error" id="question-error">
              {errors.question}
            </p>
          )}
        </div>

        <button className="submit-button" type="submit">
          Review question
        </button>

        {status && (
          <p className="success-message" role="status">
            {status}
          </p>
        )}
      </form>

      <footer className="page-footer">
        <p>
          In a real application, connect this form to a server. The server must
          validate the submitted values independently.
        </p>
      </footer>
    </main>
  );
}

export default App;
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

StrictMode helps expose lifecycle and cleanup issues in development. This form does not need an Effect for its submit action; its Effect is only used to focus the error summary after React renders it.

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  color: #1c2434;
  background: #f3f5f8;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 20rem;
  min-height: 100vh;
}

input,
select,
textarea,
button {
  font: inherit;
}

.page {
  width: min(100% - 2rem, 46rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin-bottom: 0.5rem;
  color: #344a9a;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  max-width: 20ch;
  margin-bottom: 0.75rem;
  font-size: clamp(2rem, 6vw, 3.2rem);
}

.page-header > p:last-child,
.form-instructions,
.page-footer {
  color: #465166;
  line-height: 1.6;
}

.form-instructions {
  margin-bottom: 1rem;
}

.question-form,
.error-summary {
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d0d7e2;
  border-radius: 0.9rem;
  background: #fff;
}

.field + .field {
  margin-top: 1.25rem;
}

.field label {
  display: block;
  margin-bottom: 0.35rem;
  font-weight: 700;
}

.field-help,
.character-count {
  margin-bottom: 0.5rem;
  color: #465166;
  font-size: 0.95rem;
  line-height: 1.5;
}

.field input,
.field select,
.field textarea {
  display: block;
  width: 100%;
  min-height: 2.9rem;
  padding: 0.65rem 0.75rem;
  border: 1px solid #727d90;
  border-radius: 0.45rem;
  color: #1c2434;
  background: #fff;
}

.field textarea {
  min-height: 9rem;
  resize: vertical;
}

.field input[aria-invalid="true"],
.field select[aria-invalid="true"],
.field textarea[aria-invalid="true"] {
  border-color: #a12632;
  border-width: 2px;
}

.field-error {
  margin-top: 0.45rem;
  color: #8d1d29;
  font-weight: 700;
  line-height: 1.5;
}

.character-count {
  margin-top: 0.4rem;
  margin-bottom: 0;
  text-align: right;
}

.error-summary {
  margin-bottom: 1rem;
  border-color: #9c2732;
  background: #fff4f3;
}

.error-summary:focus {
  outline: 3px solid #344a9a;
  outline-offset: 3px;
}

.error-summary h2 {
  margin-bottom: 0.6rem;
  color: #761923;
  font-size: 1.2rem;
}

.error-summary p,
.error-summary li {
  line-height: 1.6;
}

.error-summary ul {
  margin-top: 0.5rem;
  padding-left: 1.4rem;
}

.error-summary a {
  color: #761923;
  font-weight: 700;
  text-underline-offset: 0.15em;
}

.submit-button {
  margin-top: 1.5rem;
  padding: 0.75rem 1.1rem;
  border: 0;
  border-radius: 0.5rem;
  color: #fff;
  background: #293f91;
  font-weight: 700;
  cursor: pointer;
}

.submit-button:hover {
  background: #1e3074;
}

:where(a, input, select, textarea, button):focus-visible {
  outline: 3px solid #344a9a;
  outline-offset: 3px;
}

.success-message {
  margin-top: 1rem;
  padding: 0.85rem 1rem;
  border: 1px solid #28683e;
  border-radius: 0.5rem;
  color: #174526;
  background: #edf8f0;
  line-height: 1.5;
}

.page-footer {
  margin-top: 1.25rem;
  font-size: 0.95rem;
}
~~~

## What to try

1. Submit the empty form. The error summary should appear before the fields and receive keyboard focus.
2. Follow each summary link and confirm it moves to the named control.
3. Enter only spaces for the name or question. Those values should still fail validation.
4. Enter an invalid email format, then correct it. The email message should clear after the value becomes valid.
5. Choose a topic and enter a question with at least 20 non-space characters.
6. Submit the valid form. The page should announce that local checks passed and that no data was sent.
7. Navigate only with the keyboard and confirm the focus indicator remains visible.

## How the example works

- React state controls the four field values and stores touched fields, validation errors, submit state, and local status.
- The validation function returns an error object without changing the interface itself.
- Each visible field error is referenced by that control's aria-describedby and paired with aria-invalid.
- After an invalid submission, a failed-submit counter changes. An Effect runs after the error summary is rendered and focuses it.
- The summary links use each control's id as a fragment target.
- The success paragraph uses role="status" for a polite announcement.
- No network request is made. A real server must receive and validate a submitted payload before the interface can report that a question was delivered.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="035-browser-apis-and-event-listeners.md">035 - Working with browser APIs and external event listeners - Example</a></td><td><a href="../lessons/036-accessible-interactive-form-project.md">Lesson</a></td><td><a href="../qnas/036-accessible-interactive-form-project.md">Q&amp;A</a></td><td><a href="037-state-with-usestate.md">037 - State with useState - Example</a></td></tr>
  </tbody>
</table>
