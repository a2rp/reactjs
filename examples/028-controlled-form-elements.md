# 028. Controlled form elements - Example

## Goal

Build a small profile form that demonstrates how React state controls:

- Two text inputs.
- A textarea.
- A select element.
- A checkbox.
- A live preview and a submitted snapshot.

The example uses only React and browser APIs. Add the files to an existing React project.

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

const planLabels = {
  starter: 'Starter',
  complete: 'Complete',
};

export default function App() {
  const [profile, setProfile] = useState({
    name: '',
    email: '',
    bio: '',
    plan: 'starter',
    updates: false,
  });
  const [submittedProfile, setSubmittedProfile] = useState(null);

  function handleChange(event) {
    const { name, type, value, checked } = event.currentTarget;

    setProfile((currentProfile) => ({
      ...currentProfile,
      [name]: type === 'checkbox' ? checked : value,
    }));
  }

  function handleSubmit(event) {
    event.preventDefault();
    setSubmittedProfile(profile);
  }

  return (
    <main>
      <h1>Controlled profile form</h1>
      <p>
        Edit a field to see React state update the preview. Submit the form to
        save a snapshot of the current values.
      </p>

      <form onSubmit={handleSubmit}>
        <div>
          <label htmlFor="profile-name">Name</label>
          <input
            id="profile-name"
            name="name"
            type="text"
            autoComplete="name"
            value={profile.name}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label htmlFor="profile-email">Email</label>
          <input
            id="profile-email"
            name="email"
            type="email"
            autoComplete="email"
            value={profile.email}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label htmlFor="profile-bio">Short bio</label>
          <textarea
            id="profile-bio"
            name="bio"
            rows={4}
            maxLength={160}
            value={profile.bio}
            onChange={handleChange}
          />
        </div>

        <div>
          <label htmlFor="profile-plan">Plan</label>
          <select
            id="profile-plan"
            name="plan"
            value={profile.plan}
            onChange={handleChange}
          >
            <option value="starter">Starter</option>
            <option value="complete">Complete</option>
          </select>
        </div>

        <div>
          <input
            id="profile-updates"
            name="updates"
            type="checkbox"
            value="yes"
            checked={profile.updates}
            onChange={handleChange}
          />
          <label htmlFor="profile-updates">Email me course updates</label>
        </div>

        <button type="submit">Save profile</button>
      </form>

      <section aria-labelledby="preview-heading">
        <h2 id="preview-heading">Live preview</h2>
        <p>
          <strong>Name:</strong> {profile.name || 'Your name will appear here.'}
        </p>
        <p>
          <strong>Email:</strong>{' '}
          {profile.email || 'Your email will appear here.'}
        </p>
        <p>
          <strong>Plan:</strong> {planLabels[profile.plan]}
        </p>
        <p>
          <strong>Bio:</strong> {profile.bio || 'Your bio will appear here.'}
        </p>
        <p>
          <strong>Course updates:</strong> {profile.updates ? 'Yes' : 'No'}
        </p>
      </section>

      {submittedProfile !== null && (
        <section aria-labelledby="submitted-heading" role="status">
          <h2 id="submitted-heading">Submitted snapshot</h2>
          <p>Name: {submittedProfile.name}</p>
          <p>Email: {submittedProfile.email}</p>
          <p>Plan: {planLabels[submittedProfile.plan]}</p>
          <p>Bio: {submittedProfile.bio || 'No bio provided.'}</p>
          <p>
            Course updates: {submittedProfile.updates ? 'Yes' : 'No'}
          </p>
        </section>
      )}
    </main>
  );
}
~~~

## How the example works

### State owns the displayed values

The initial `profile` object contains every field before the first render. The text controls and select receive strings, and the checkbox receives a boolean. None of the controls starts with an undefined value and later becomes controlled.

### One handler updates the fields

Each control has a `name` that matches a property in `profile`. The shared `handleChange` reads the control that fired the event and uses a functional state update so it preserves the other fields.

For a checkbox, the handler stores `checked`, a boolean. For the other controls, it stores `value`, a string.

### The preview follows state

The preview renders directly from `profile`, so it changes as React state changes. The select displays the option matching `profile.plan`. The checkbox displays its boolean selection through `checked`.

### The submitted section is a snapshot

Submitting the form prevents a page reload and stores the current profile object in `submittedProfile`. Later edits still update the live preview, while the submitted section keeps the values that were present when the form was submitted.

### Form names and labels remain useful

The controls have meaningful `name` attributes, so their values can also be inspected through `FormData` in the submit handler. Each input has an associated label, including the checkbox.

## Experiments

1. Type into the name field and watch the preview update.
2. Change the plan and verify that the preview follows the selected option.
3. Toggle the checkbox and confirm that its state is boolean.
4. Enter a bio, submit the form, then edit the bio again. Compare the live preview with the submitted snapshot.
5. Temporarily remove `onChange={handleChange}` from one controlled field. Notice that React keeps rendering the old value and the field stops accepting edits.
6. Change the text field's initial state from `''` to `undefined`, observe the controlled/uncontrolled warning, and restore the stable string value.
7. Inside a submit handler, inspect `new FormData(event.currentTarget)`. The checkbox entry is absent when unchecked; when checked, its value is `yes`.

## Key takeaways

- `value` controls text inputs, textareas, and selects.
- `checked` controls whether a checkbox is selected.
- A change handler updates the backing state from the element that changed.
- State can drive other UI, such as a preview, while the user edits.
- A submitted snapshot can be separate from the live field state.
- Labels and `name` attributes are still important in a controlled form.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="027-event-propagation-and-default-actions.md">027 - Event propagation, default actions, and cancellation - Example</a></td><td><a href="../lessons/028-controlled-form-elements.md">Lesson</a></td><td><a href="../qnas/028-controlled-form-elements.md">Q&amp;A</a></td><td><a href="029-inputs-selects-checkboxes-and-radio-buttons.md">029 - Text inputs, selects, checkboxes, and radio buttons - Example</a></td></tr>
  </tbody>
</table>
