# 030. Form submission and browser validation - Example

## Goal

Build a small contact request form that demonstrates:

- Semantic form submission through the form's `onSubmit` event.
- Browser validation with `required`, `type="email"`, and `minLength`.
- Reading the submitted controls through `FormData`.
- Preventing the browser's navigation while the React page displays a receipt.
- Resetting uncontrolled controls and clearing the displayed receipt.

The example uses the manual `onSubmit` approach, which works across React versions. React 19 form Actions are covered later in the course.

## Files

Create or update:

- `src/main.jsx`
- `src/App.jsx`

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
import { useState } from 'react';

const topicLabels = {
  account: 'Account question',
  course: 'Course question',
  feedback: 'Feedback',
};

export default function App() {
  const [receipt, setReceipt] = useState(null);

  function handleSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);

    setReceipt({
      name: String(formData.get('name') ?? ''),
      email: String(formData.get('email') ?? ''),
      topic: String(formData.get('topic') ?? ''),
      message: String(formData.get('message') ?? ''),
    });
  }

  function handleReset() {
    setReceipt(null);
  }

  return (
    <main>
      <h1>Contact the course team</h1>
      <p>
        Complete the fields and submit the form. The browser checks its native
        constraints before React handles a valid submission.
      </p>

      <form onSubmit={handleSubmit} onReset={handleReset}>
        <div>
          <label htmlFor="contact-name">Name</label>
          <input
            id="contact-name"
            name="name"
            type="text"
            autoComplete="name"
            required
          />
        </div>

        <div>
          <label htmlFor="contact-email">Email</label>
          <input
            id="contact-email"
            name="email"
            type="email"
            autoComplete="email"
            required
          />
        </div>

        <div>
          <label htmlFor="contact-topic">Topic</label>
          <select
            id="contact-topic"
            name="topic"
            defaultValue=""
            required
          >
            <option value="">Choose a topic</option>
            <option value="account">Account question</option>
            <option value="course">Course question</option>
            <option value="feedback">Feedback</option>
          </select>
        </div>

        <div>
          <label htmlFor="contact-message">Message</label>
          <textarea
            id="contact-message"
            name="message"
            rows={5}
            minLength={20}
            maxLength={500}
            required
          />
        </div>

        <div>
          <button type="submit">Send request</button>
          <button type="reset">Clear form</button>
        </div>
      </form>

      {receipt !== null && (
        <section aria-labelledby="receipt-heading" role="status">
          <h2 id="receipt-heading">Request recorded</h2>
          <p>This local example did not send data to a server.</p>
          <p>Name: {receipt.name}</p>
          <p>Email: {receipt.email}</p>
          <p>Topic: {topicLabels[receipt.topic]}</p>
          <p>Message: {receipt.message}</p>
        </section>
      )}
    </main>
  );
}
~~~

## How the example works

### Native validation runs before the submit handler

Try submitting without filling the required fields, with a malformed email, or with a message shorter than 20 characters. The browser blocks an invalid attempt and shows its native validation feedback. Since the submit event is not dispatched for that invalid attempt, `handleSubmit` does not run and no receipt is displayed.

The email check only verifies an acceptable format. It does not prove that the address exists or belongs to the user.

### React handles a valid submission

After the constraints pass, `handleSubmit` receives the form's submit event. It calls `preventDefault()` so the browser does not navigate or reload, then constructs `FormData` from the form and reads the named controls.

The receipt is stored in React state so the user can see what the local demo collected. The example does not send a network request.

### The fields are uncontrolled

The form fields use browser-owned values; they do not pass `value` or `checked` props. `FormData` reads their current values after native validation succeeds.

### Reset clears the form and receipt

The reset button has `type="reset"`, so the browser restores the uncontrolled controls to their initial values. The form's `onReset` handler also clears the React receipt. The handler does not cancel the browser reset.

## Experiments

1. Submit with every field empty. Observe that the browser blocks submission before the React handler runs.
2. Enter an invalid email format and submit again.
3. Enter a message shorter than 20 characters, then a message at least 20 characters long.
4. Submit valid values and inspect the receipt.
5. Change a field after submission. The receipt remains a snapshot of the last valid submission.
6. Press “Clear form.” The browser resets the controls and React clears the receipt.
7. Temporarily add `noValidate` to the form. Notice that the browser no longer performs automatic interactive validation; remove the attribute afterward.

## Key takeaways

- Handle submission on the form's `onSubmit` event.
- When React takes over a normal browser submission, call `preventDefault()`.
- Native constraints run before `onSubmit` when interactive validation is enabled.
- `FormData` collects named controls; React state stores the displayed receipt.
- A reset button resets uncontrolled controls, while `onReset` can also reset related React state.
- This demo stores values locally; it does not send them to a server.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [029 - Text inputs, selects, checkboxes, and radio buttons](../lessons/029-inputs-selects-checkboxes-and-radio-buttons.md) | [030 - Form submission and browser validation](../lessons/030-form-submission-and-browser-validation.md) | [Open Q&A](../qnas/030-form-submission-and-browser-validation.md) | [031 - Accessible validation messages and error summaries](../lessons/031-accessible-validation-messages.md) |
