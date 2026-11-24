# 032. Managing state for multi-field forms - Example

This React example keeps several related values in one state object. A shared change handler updates text fields, a select, a checkbox, and radio buttons. A live preview is derived from the current values; nothing is submitted to a server.

## Project files

Create these files in a React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

## src/App.jsx

~~~jsx
import { useState } from 'react';

function createInitialForm() {
  return {
    fullName: '',
    email: '',
    topic: '',
    message: '',
    receiveUpdates: false,
    contactMethod: 'email',
  };
}

function App() {
  const [form, setForm] = useState(createInitialForm);
  const [status, setStatus] = useState('');

  function handleChange(event) {
    const control = event.currentTarget;
    const { name, type, value, checked } = control;
    const nextValue = type === 'checkbox' ? checked : value;

    setForm((currentForm) => ({
      ...currentForm,
      [name]: nextValue,
    }));

    setStatus('');
  }

  function handleSubmit(event) {
    event.preventDefault();
    setStatus('Local preview prepared. No data was sent or saved.');
  }

  function handleReset() {
    setForm(createInitialForm());
    setStatus('');
  }

  const contactSummary = [
    form.fullName.trim() || 'Your name',
    form.topic || 'No topic selected',
  ].join(' — ');

  const completedContactFields = [form.fullName, form.email].filter(
    (value) => value.trim() !== '',
  ).length;

  return (
    <main className="page">
      <h1>Support request</h1>
      <p>
        Enter some details to see how related form values stay together in
        React state. This example does not send a request.
      </p>

      <form onSubmit={handleSubmit}>
        <div className="field">
          <label htmlFor="fullName">Full name</label>
          <input
            id="fullName"
            name="fullName"
            type="text"
            autoComplete="name"
            value={form.fullName}
            onChange={handleChange}
          />
        </div>

        <div className="field">
          <label htmlFor="email">Email address</label>
          <input
            id="email"
            name="email"
            type="email"
            autoComplete="email"
            value={form.email}
            onChange={handleChange}
          />
        </div>

        <div className="field">
          <label htmlFor="topic">Topic</label>
          <select
            id="topic"
            name="topic"
            value={form.topic}
            onChange={handleChange}
          >
            <option value="">Choose a topic</option>
            <option value="account">Account</option>
            <option value="billing">Billing</option>
            <option value="technical">Technical issue</option>
          </select>
        </div>

        <div className="field">
          <label htmlFor="message">Message</label>
          <textarea
            id="message"
            name="message"
            rows="5"
            value={form.message}
            onChange={handleChange}
          />
        </div>

        <fieldset className="choice-group">
          <legend>How should we contact you?</legend>

          <label className="choice">
            <input
              type="radio"
              name="contactMethod"
              value="email"
              checked={form.contactMethod === 'email'}
              onChange={handleChange}
            />
            Email
          </label>

          <label className="choice">
            <input
              type="radio"
              name="contactMethod"
              value="phone"
              checked={form.contactMethod === 'phone'}
              onChange={handleChange}
            />
            Phone
          </label>
        </fieldset>

        <label className="choice">
          <input
            type="checkbox"
            name="receiveUpdates"
            checked={form.receiveUpdates}
            onChange={handleChange}
          />
          Send me product updates
        </label>

        <div className="actions">
          <button type="submit">Prepare local preview</button>
          <button type="button" className="secondary" onClick={handleReset}>
            Clear form
          </button>
        </div>
      </form>

      {status && <p className="status" role="status">{status}</p>}

      <section className="preview" aria-labelledby="preview-title">
        <h2 id="preview-title">Live preview</h2>
        <p>{contactSummary}</p>
        <p>
          Contact fields completed: {completedContactFields} of 2
        </p>

        <dl>
          <div>
            <dt>Email</dt>
            <dd>{form.email || 'Not entered'}</dd>
          </div>
          <div>
            <dt>Message</dt>
            <dd>{form.message || 'Not entered'}</dd>
          </div>
          <div>
            <dt>Contact method</dt>
            <dd>{form.contactMethod}</dd>
          </div>
          <div>
            <dt>Product updates</dt>
            <dd>{form.receiveUpdates ? 'Yes' : 'No'}</dd>
          </div>
        </dl>

        <h3>Current form state</h3>
        <pre>{JSON.stringify(form, null, 2)}</pre>
      </section>
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
  width: min(100% - 2rem, 48rem);
  margin: 3rem auto;
  padding: 2rem;
  background: #fff;
  border: 1px solid #c7cfdb;
  border-radius: 0.75rem;
}

h1,
h2,
h3 {
  line-height: 1.2;
}

.field {
  display: grid;
  gap: 0.4rem;
  margin-block: 1.25rem;
}

.field label,
.choice-group legend {
  font-weight: 700;
}

input:not([type='checkbox']):not([type='radio']),
select,
textarea {
  width: 100%;
  padding: 0.65rem 0.75rem;
  color: inherit;
  background: #fff;
  border: 1px solid #667085;
  border-radius: 0.35rem;
}

.choice-group {
  display: grid;
  gap: 0.65rem;
  margin: 1.5rem 0;
  padding: 1rem;
  border: 1px solid #98a2b3;
  border-radius: 0.35rem;
}

.choice {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  margin-block: 0.75rem;
}

input[type='checkbox'],
input[type='radio'] {
  width: 1.1rem;
  height: 1.1rem;
  margin: 0;
}

input:focus-visible,
select:focus-visible,
textarea:focus-visible,
button:focus-visible {
  outline: 3px solid #2459d3;
  outline-offset: 2px;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-top: 1.5rem;
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

button.secondary {
  color: #172033;
  background: #e4e8ef;
  border: 1px solid #8a94a6;
}

.status {
  margin-top: 1.5rem;
  padding: 1rem;
  background: #e9f7ed;
  border: 1px solid #2b6a3f;
  border-radius: 0.35rem;
}

.preview {
  margin-top: 2rem;
  padding: 1rem;
  background: #f8f9fc;
  border: 1px solid #c7cfdb;
  border-radius: 0.5rem;
}

.preview dl {
  display: grid;
  gap: 0.75rem;
}

.preview dl div {
  display: grid;
  grid-template-columns: 10rem minmax(0, 1fr);
  gap: 1rem;
}

.preview dt {
  font-weight: 700;
}

.preview dd {
  margin: 0;
  overflow-wrap: anywhere;
}

pre {
  overflow-x: auto;
  padding: 1rem;
  background: #e9edf4;
  border-radius: 0.35rem;
}

@media (max-width: 36rem) {
  .page {
    margin-block: 1rem;
    padding: 1.25rem;
  }

  .preview dl div {
    grid-template-columns: 1fr;
    gap: 0.2rem;
  }
}
~~~

## How the example works

- The form's values live together in the form state object. The status message is separate because it is independent UI feedback.
- Every control name matches one property in the state object.
- The shared handler uses the control name to select a property. It stores checked for checkboxes and value for text controls, selects, and radios.
- The updater copies the existing object and replaces one property, preserving all the other fields.
- The inputs receive their current values from React state, so they are controlled.
- The contact summary, completion count, and JSON display are calculated from current form state during rendering. They are not duplicated in state.
- The reset handler creates a fresh object with the initial values.
- The submit handler prevents navigation and displays a local status only. It does not contact a server.

## Experiments

1. Type into the name field and inspect the live preview and current form state.
2. Change the topic and confirm that the name and email remain intact.
3. Select Phone. Confirm that the radio choice updates the contactMethod property.
4. Toggle the updates checkbox. Confirm that receiveUpdates is a boolean in the displayed object.
5. Clear the form and confirm that every field returns to its initial value.
6. Temporarily remove the object spread from the shared handler. Change one field and observe the other state properties disappear; then restore the spread.
7. Temporarily use value instead of checked on the checkbox. Notice that the control's selected state is no longer represented correctly; then restore checked.
8. Edit a field after preparing the preview. Notice that the preview remains current because it is derived from the latest form object.

## Important limitation

This example teaches form state, not validation or server submission. Use the accessible validation patterns from Chapter 031 for errors, and validate data on the server in a real application.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [031 - Accessible validation messages and error summaries](../lessons/031-accessible-validation-messages.md) | [032 - Managing state for multi-field forms](../lessons/032-multi-field-form-state.md) | [Open Q&A](../qnas/032-multi-field-form-state.md) | [033 - Uncontrolled forms and refs](../lessons/033-uncontrolled-forms-and-refs.md) |
