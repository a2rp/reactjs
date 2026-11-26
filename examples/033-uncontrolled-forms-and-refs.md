# 033. Uncontrolled forms and refs - Example

This React form leaves its field values in the browser DOM until submission. It reads the submitted values with FormData, stores a receipt snapshot in React state, and uses refs for two focused DOM tasks: focusing an input and resetting the form. The example does not send or save data.

## Project files

Create these files in a React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

## src/App.jsx

~~~jsx
import { useRef, useState } from 'react';

function App() {
  const formRef = useRef(null);
  const fullNameRef = useRef(null);
  const [receipt, setReceipt] = useState(null);
  const [status, setStatus] = useState('');

  function handleSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);

    setReceipt({
      fullName: String(formData.get('fullName') ?? ''),
      email: String(formData.get('email') ?? ''),
      topic: String(formData.get('topic') ?? ''),
      message: String(formData.get('message') ?? ''),
      receiveUpdates: formData.has('receiveUpdates'),
    });

    setStatus('Values were read from the form. Nothing was sent or saved.');
  }

  function focusNameField() {
    fullNameRef.current?.focus();
  }

  function handleReset() {
    formRef.current?.reset();
    setReceipt(null);
    setStatus('');
  }

  return (
    <main className="page">
      <h1>Contact request</h1>
      <p>
        The browser keeps these field values until submission. This teaching
        example does not transmit or store your information.
      </p>

      <form ref={formRef} onSubmit={handleSubmit}>
        <div className="field">
          <label htmlFor="fullName">Full name</label>
          <input
            ref={fullNameRef}
            id="fullName"
            name="fullName"
            type="text"
            autoComplete="name"
            defaultValue=""
            required
          />
        </div>

        <div className="field">
          <label htmlFor="email">Email address</label>
          <input
            id="email"
            name="email"
            type="email"
            autoComplete="email"
            defaultValue=""
            required
          />
        </div>

        <div className="field">
          <label htmlFor="topic">Topic</label>
          <select id="topic" name="topic" defaultValue="">
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
            defaultValue=""
          />
        </div>

        <label className="choice">
          <input
            type="checkbox"
            name="receiveUpdates"
            value="yes"
            defaultChecked={false}
          />
          Send me product updates
        </label>

        <div className="actions">
          <button type="submit">Read form values</button>
          <button type="button" className="secondary" onClick={focusNameField}>
            Focus name field
          </button>
          <button type="button" className="secondary" onClick={handleReset}>
            Reset form and receipt
          </button>
        </div>
      </form>

      {status && <p className="status" role="status">{status}</p>}

      {receipt && (
        <section className="receipt" aria-labelledby="receipt-title">
          <h2 id="receipt-title">Last submitted snapshot</h2>
          <p>
            This is the set of values read when you last submitted the form.
            Editing the fields does not change this snapshot.
          </p>
          <dl>
            <div>
              <dt>Full name</dt>
              <dd>{receipt.fullName}</dd>
            </div>
            <div>
              <dt>Email address</dt>
              <dd>{receipt.email}</dd>
            </div>
            <div>
              <dt>Topic</dt>
              <dd>{receipt.topic || 'No topic selected'}</dd>
            </div>
            <div>
              <dt>Message</dt>
              <dd>{receipt.message || 'No message entered'}</dd>
            </div>
            <div>
              <dt>Product updates</dt>
              <dd>{receipt.receiveUpdates ? 'Yes' : 'No'}</dd>
            </div>
          </dl>

          <h3>Receipt object</h3>
          <pre>{JSON.stringify(receipt, null, 2)}</pre>
        </section>
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
  width: min(100% - 2rem, 46rem);
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

.field label {
  font-weight: 700;
}

input:not([type='checkbox']),
select,
textarea {
  width: 100%;
  padding: 0.65rem 0.75rem;
  color: inherit;
  background: #fff;
  border: 1px solid #667085;
  border-radius: 0.35rem;
}

input[type='checkbox'] {
  width: 1.1rem;
  height: 1.1rem;
  margin: 0;
}

.choice {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  margin-block: 1rem;
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

.receipt {
  margin-top: 2rem;
  padding: 1rem;
  background: #f8f9fc;
  border: 1px solid #c7cfdb;
  border-radius: 0.5rem;
}

.receipt dl {
  display: grid;
  gap: 0.75rem;
}

.receipt dl div {
  display: grid;
  grid-template-columns: 10rem minmax(0, 1fr);
  gap: 1rem;
}

.receipt dt {
  font-weight: 700;
}

.receipt dd {
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

  .receipt dl div {
    grid-template-columns: 1fr;
    gap: 0.2rem;
  }
}
~~~

## How the example works

- Text inputs, the select, and the textarea use defaultValue. The checkbox uses defaultChecked. The browser owns their current values while the person edits.
- Every field has a name, so the submit handler can construct FormData from the form and read the values as a group.
- FormData.has returns a boolean for the optional checkbox. An unchecked checkbox is absent from the submitted data.
- The submit handler stores a receipt snapshot in React state. That state update renders the receipt, but does not control or overwrite the form fields.
- The name input ref is used only to focus that input after the focus button is activated.
- The form ref is used to call the browser's reset method. The handler also clears the separate receipt and status state.
- Required and email constraints are native browser validation. When they fail, the browser reports them before the submit handler runs.
- The status clearly says that the values were only read locally and were not sent or saved.

## Experiments

1. Activate “Focus name field” and confirm that the name input receives focus.
2. Submit the empty form. Confirm that native validation prevents the submit handler from running.
3. Enter a name and valid email, then submit. Inspect the receipt and its displayed object.
4. Change the name after submitting. Confirm that the uncontrolled field changes but the receipt remains the last submitted snapshot.
5. Submit with the checkbox unchecked and confirm the receipt says “No.”
6. Check the box, submit again, and confirm the receipt says “Yes.”
7. Change the select and message, then activate “Reset form and receipt.” Confirm that the fields return to their initial values and the receipt disappears.
8. Use only a keyboard to reach and activate all buttons.

## Important limitation

This example demonstrates browser-owned form values and DOM refs. It does not send data to a server. A production form needs an actual submission path, server-side validation, and appropriate handling for personal data.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="032-multi-field-form-state.md">032 - Managing state for multi-field forms - Example</a></td><td><a href="../lessons/033-uncontrolled-forms-and-refs.md">Lesson</a></td><td><a href="../qnas/033-uncontrolled-forms-and-refs.md">Q&amp;A</a></td><td><a href="034-file-inputs-and-upload-flows.md">034 - File inputs and upload flows - Example</a></td></tr>
  </tbody>
</table>
