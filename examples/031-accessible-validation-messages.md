# 031. Accessible validation messages and error summaries - Example

This small React form demonstrates visible inline errors, an error summary with links to the affected fields, validation after submission, and a non-urgent status message. It runs locally and does not send or save any data.

## Project files

Create these files in a React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

## src/App.jsx

~~~jsx
import { useState } from 'react';

const initialValues = {
  fullName: '',
  email: '',
  topic: '',
  message: '',
};

const fieldLabels = {
  fullName: 'Full name',
  email: 'Email address',
  topic: 'Topic',
  message: 'Message',
};

function validateField(field, value, control) {
  if (field === 'fullName') {
    if (!value.trim()) {
      return 'Enter your name.';
    }

    if (value.trim().length < 2) {
      return 'Enter at least 2 characters.';
    }
  }

  if (field === 'email') {
    if (!value.trim()) {
      return 'Enter your email address.';
    }

    if (control?.validity.typeMismatch) {
      return 'Enter an email address in the format name@example.com.';
    }
  }

  if (field === 'topic' && !value) {
    return 'Choose a topic.';
  }

  if (field === 'message') {
    if (!value.trim()) {
      return 'Enter a message.';
    }

    if (value.trim().length < 20) {
      return 'Enter at least 20 characters so we have enough detail.';
    }
  }

  return '';
}

function validateValues(values, form) {
  const nextErrors = {};

  for (const field of Object.keys(fieldLabels)) {
    const control = form.elements.namedItem(field);
    const error = validateField(field, values[field], control);

    if (error) {
      nextErrors[field] = error;
    }
  }

  return nextErrors;
}

function ErrorSummary({ errors }) {
  const entries = Object.entries(errors);

  if (entries.length === 0) {
    return null;
  }

  return (
    <section
      className="error-summary"
      role="alert"
      aria-labelledby="error-summary-title"
    >
      <h2 id="error-summary-title">
        There are {entries.length} problems with your request
      </h2>

      <p>Review these fields and correct the information:</p>

      <ul>
        {entries.map(([field, message]) => (
          <li key={field}>
            <a href={'#' + field}>
              {fieldLabels[field]}: {message}
            </a>
          </li>
        ))}
      </ul>
    </section>
  );
}

function App() {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [hasSubmitted, setHasSubmitted] = useState(false);
  const [status, setStatus] = useState('');

  function handleChange(event) {
    const control = event.currentTarget;
    const { name, value } = control;

    setValues((currentValues) => ({
      ...currentValues,
      [name]: value,
    }));

    if (hasSubmitted) {
      const error = validateField(name, value, control);

      setErrors((currentErrors) => {
        const nextErrors = { ...currentErrors };

        if (error) {
          nextErrors[name] = error;
        } else {
          delete nextErrors[name];
        }

        return nextErrors;
      });
    }

    setStatus('');
  }

  function handleSubmit(event) {
    event.preventDefault();

    const form = event.currentTarget;
    const nextErrors = validateValues(values, form);

    setHasSubmitted(true);
    setErrors(nextErrors);
    setStatus('');

    if (Object.keys(nextErrors).length > 0) {
      return;
    }

    setStatus(
      'All fields are valid. This demo does not send or save your message.',
    );
    setHasSubmitted(false);
  }

  const errorCount = Object.keys(errors).length;

  return (
    <main className="page">
      <h1>Request support</h1>
      <p>
        Complete the fields below. All fields are required. This example checks
        your entries locally and does not send them anywhere.
      </p>

      <ErrorSummary errors={errors} />

      <form noValidate onSubmit={handleSubmit}>
        <div className="field">
          <label htmlFor="fullName">Full name</label>
          <p id="fullName-help">Enter at least 2 characters.</p>
          <input
            id="fullName"
            name="fullName"
            type="text"
            autoComplete="name"
            required
            value={values.fullName}
            onChange={handleChange}
            aria-invalid={errors.fullName ? 'true' : undefined}
            aria-describedby={
              errors.fullName ? 'fullName-help fullName-error' : 'fullName-help'
            }
          />
          {errors.fullName && (
            <p className="field-error" id="fullName-error">
              <strong>Error:</strong> {errors.fullName}
            </p>
          )}
        </div>

        <div className="field">
          <label htmlFor="email">Email address</label>
          <p id="email-help">Use an address where we can contact you.</p>
          <input
            id="email"
            name="email"
            type="email"
            autoComplete="email"
            required
            value={values.email}
            onChange={handleChange}
            aria-invalid={errors.email ? 'true' : undefined}
            aria-describedby={
              errors.email ? 'email-help email-error' : 'email-help'
            }
          />
          {errors.email && (
            <p className="field-error" id="email-error">
              <strong>Error:</strong> {errors.email}
            </p>
          )}
        </div>

        <div className="field">
          <label htmlFor="topic">Topic</label>
          <p id="topic-help">Choose the topic closest to your question.</p>
          <select
            id="topic"
            name="topic"
            required
            value={values.topic}
            onChange={handleChange}
            aria-invalid={errors.topic ? 'true' : undefined}
            aria-describedby={
              errors.topic ? 'topic-help topic-error' : 'topic-help'
            }
          >
            <option value="">Choose a topic</option>
            <option value="account">Account</option>
            <option value="billing">Billing</option>
            <option value="technical">Technical issue</option>
          </select>
          {errors.topic && (
            <p className="field-error" id="topic-error">
              <strong>Error:</strong> {errors.topic}
            </p>
          )}
        </div>

        <div className="field">
          <label htmlFor="message">Message</label>
          <p id="message-help">Enter at least 20 characters.</p>
          <textarea
            id="message"
            name="message"
            rows="5"
            required
            minLength="20"
            value={values.message}
            onChange={handleChange}
            aria-invalid={errors.message ? 'true' : undefined}
            aria-describedby={
              errors.message ? 'message-help message-error' : 'message-help'
            }
          />
          {errors.message && (
            <p className="field-error" id="message-error">
              <strong>Error:</strong> {errors.message}
            </p>
          )}
        </div>

        <button type="submit">Check my request</button>
      </form>

      {status && <p className="status-message" role="status">{status}</p>}

      {errorCount > 0 && (
        <p className="form-hint">
          Correct the listed fields, then submit the form again.
        </p>
      )}
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

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  color: #172033;
  background: #f4f6fa;
  font-family: system-ui, sans-serif;
  line-height: 1.5;
}

button,
input,
select,
textarea {
  font: inherit;
}

.page {
  width: min(100% - 2rem, 42rem);
  margin: 3rem auto;
  padding: 2rem;
  background: #fff;
  border: 1px solid #c7cfdb;
  border-radius: 0.75rem;
}

h1,
h2 {
  line-height: 1.2;
}

.field {
  display: grid;
  gap: 0.4rem;
  margin-block: 1.25rem;
}

.field label {
  font-weight: 700;
}

.field p {
  margin: 0;
}

input,
select,
textarea {
  width: 100%;
  padding: 0.65rem 0.75rem;
  color: inherit;
  background: #fff;
  border: 1px solid #667085;
  border-radius: 0.35rem;
}

input:focus-visible,
select:focus-visible,
textarea:focus-visible,
button:focus-visible,
a:focus-visible {
  outline: 3px solid #2459d3;
  outline-offset: 2px;
}

button {
  padding: 0.7rem 1rem;
  color: #fff;
  background: #1747a6;
  border: 0;
  border-radius: 0.35rem;
  cursor: pointer;
}

button:hover {
  background: #103778;
}

.error-summary {
  margin-block: 1.5rem;
  padding: 1rem 1.25rem;
  color: #551313;
  background: #fff1f0;
  border: 2px solid #9c1c1c;
  border-radius: 0.35rem;
}

.error-summary h2 {
  margin-top: 0;
}

.error-summary p {
  margin-bottom: 0.5rem;
}

.error-summary li + li {
  margin-top: 0.35rem;
}

.error-summary a {
  color: #651515;
}

.field-error {
  color: #7b1717;
}

.status-message {
  margin-top: 1.5rem;
  padding: 1rem;
  background: #e9f7ed;
  border: 1px solid #2b6a3f;
  border-radius: 0.35rem;
}

.form-hint {
  margin-top: 1rem;
}
~~~

## How the example works

- The form uses noValidate so a failed submit reaches the React handler and can display the custom summary and inline messages.
- The handler validates every field, stores current errors, and keeps the entered values available for correction.
- Before the first submit, fields are not marked invalid. After a failed submit, changing a field rechecks that field and removes its error when corrected.
- Every input has a visible label and help text. When an error exists, its id is added to that control's aria-describedby value and aria-invalid is set to true.
- The summary is inserted with role="alert" and each item links to the matching field id. The example does not also force focus, avoiding two competing announcement mechanisms.
- The status uses role="status" only after every local check passes. It clearly states that the demo does not send or save the form.
- The message text and border provide error cues in addition to color.

## Experiments

1. Submit the empty form. Confirm that the form handler displays the summary and inline errors instead of a browser validation popup.
2. Activate a summary link. Confirm that the browser moves to the matching field.
3. Correct the name and email after the failed submit. Confirm each error clears when its value becomes valid.
4. Choose a topic and enter fewer than 20 message characters. Confirm the remaining errors identify what to fix.
5. Submit valid values. Confirm the status message says the example did not transmit or save anything.
6. Use only Tab, Shift+Tab, and Enter to navigate and submit. Check that focus remains visible.
7. With a screen reader, inspect a field before and after validation and listen for the inserted summary and final status.

## Important limitation

This is a local teaching example, not a production contact form. A real application must send values to a server, handle network and server validation failures, protect sensitive data, and apply the same visible and programmatic error associations to returned field errors.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="030-form-submission-and-browser-validation.md">030 - Form submission and browser validation - Example</a></td><td><a href="../lessons/031-accessible-validation-messages.md">Lesson</a></td><td><a href="../qnas/031-accessible-validation-messages.md">Q&amp;A</a></td><td><a href="032-multi-field-form-state.md">032 - Managing state for multi-field forms - Example</a></td></tr>
  </tbody>
</table>
