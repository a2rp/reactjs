# 029. Text inputs, selects, checkboxes, and radio buttons - Example

## Goal

Build an interactive course-preferences panel that demonstrates:

- Text and email inputs.
- A number input whose value stays a string while edited.
- A single select.
- A multiple select.
- A checkbox group stored as an array.
- An independent checkbox stored as a boolean.
- A radio group stored as one selected string.
- A reset action that updates the state controlling every field.

The example focuses on each control's value shape. Form submission and browser validation are covered in chapter 030.

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

const levelLabels = {
  beginner: 'Beginner',
  intermediate: 'Intermediate',
  advanced: 'Advanced',
};

const topicLabels = {
  components: 'Components',
  state: 'State',
  forms: 'Forms',
  effects: 'Effects',
};

export default function App() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [weeklyHours, setWeeklyHours] = useState('2');
  const [level, setLevel] = useState('beginner');
  const [topics, setTopics] = useState([]);
  const [contactMethod, setContactMethod] = useState('email');
  const [sendUpdates, setSendUpdates] = useState(false);

  function handleTopicChange(event) {
    const { value, checked } = event.currentTarget;

    setTopics((currentTopics) => {
      if (checked) {
        return [...currentTopics, value];
      }

      return currentTopics.filter((topic) => topic !== value);
    });
  }

  function handleTopicsSelectChange(event) {
    const selectedTopics = Array.from(
      event.currentTarget.selectedOptions,
      (option) => option.value,
    );

    setTopics(selectedTopics);
  }

  function resetPreferences() {
    setName('');
    setEmail('');
    setWeeklyHours('2');
    setLevel('beginner');
    setTopics([]);
    setContactMethod('email');
    setSendUpdates(false);
  }

  return (
    <main>
      <h1>Course preferences</h1>
      <p>Choose your learning preferences and see the controlled values below.</p>

      <section aria-labelledby="profile-heading">
        <h2 id="profile-heading">About you</h2>

        <div>
          <label htmlFor="learner-name">Name</label>
          <input
            id="learner-name"
            name="name"
            type="text"
            autoComplete="name"
            value={name}
            onChange={(event) => setName(event.currentTarget.value)}
          />
        </div>

        <div>
          <label htmlFor="learner-email">Email</label>
          <input
            id="learner-email"
            name="email"
            type="email"
            autoComplete="email"
            value={email}
            onChange={(event) => setEmail(event.currentTarget.value)}
          />
        </div>

        <div>
          <label htmlFor="weekly-hours">Study hours per week</label>
          <input
            id="weekly-hours"
            name="weeklyHours"
            type="number"
            min="0"
            max="40"
            step="1"
            value={weeklyHours}
            onChange={(event) => setWeeklyHours(event.currentTarget.value)}
          />
        </div>
      </section>

      <section aria-labelledby="learning-heading">
        <h2 id="learning-heading">Learning choices</h2>

        <div>
          <label htmlFor="current-level">Current level</label>
          <select
            id="current-level"
            name="level"
            value={level}
            onChange={(event) => setLevel(event.currentTarget.value)}
          >
            <option value="beginner">Beginner</option>
            <option value="intermediate">Intermediate</option>
            <option value="advanced">Advanced</option>
          </select>
        </div>

        <div>
          <label htmlFor="topic-picker">Choose several topics</label>
          <select
            id="topic-picker"
            name="selectedTopics"
            multiple
            size={4}
            value={topics}
            onChange={handleTopicsSelectChange}
          >
            <option value="components">Components</option>
            <option value="state">State</option>
            <option value="forms">Forms</option>
            <option value="effects">Effects</option>
          </select>
        </div>

        <fieldset>
          <legend>Or choose topics with checkboxes</legend>

          <label>
            <input
              type="checkbox"
              name="topics"
              value="components"
              checked={topics.includes('components')}
              onChange={handleTopicChange}
            />
            Components
          </label>

          <label>
            <input
              type="checkbox"
              name="topics"
              value="state"
              checked={topics.includes('state')}
              onChange={handleTopicChange}
            />
            State
          </label>

          <label>
            <input
              type="checkbox"
              name="topics"
              value="forms"
              checked={topics.includes('forms')}
              onChange={handleTopicChange}
            />
            Forms
          </label>

          <label>
            <input
              type="checkbox"
              name="topics"
              value="effects"
              checked={topics.includes('effects')}
              onChange={handleTopicChange}
            />
            Effects
          </label>
        </fieldset>

        <label>
          <input
            type="checkbox"
            name="sendUpdates"
            value="yes"
            checked={sendUpdates}
            onChange={(event) => setSendUpdates(event.currentTarget.checked)}
          />
          Send me course updates
        </label>

        <fieldset>
          <legend>Preferred contact method</legend>

          <label>
            <input
              type="radio"
              name="contactMethod"
              value="email"
              checked={contactMethod === 'email'}
              onChange={(event) => setContactMethod(event.currentTarget.value)}
            />
            Email
          </label>

          <label>
            <input
              type="radio"
              name="contactMethod"
              value="phone"
              checked={contactMethod === 'phone'}
              onChange={(event) => setContactMethod(event.currentTarget.value)}
            />
            Phone
          </label>

          <label>
            <input
              type="radio"
              name="contactMethod"
              value="none"
              checked={contactMethod === 'none'}
              onChange={(event) => setContactMethod(event.currentTarget.value)}
            />
            No contact
          </label>
        </fieldset>
      </section>

      <button type="button" onClick={resetPreferences}>
        Reset preferences
      </button>

      <section aria-labelledby="summary-heading">
        <h2 id="summary-heading">Current state preview</h2>
        <p>Name: {name || 'Not entered'}</p>
        <p>Email: {email || 'Not entered'}</p>
        <p>
          Study hours:{' '}
          {weeklyHours === '' ? 'Not entered' : weeklyHours + ' hours per week'}
        </p>
        <p>Current level: {levelLabels[level]}</p>
        <p>Contact method: {contactMethod}</p>
        <p>Send course updates: {sendUpdates ? 'Yes' : 'No'}</p>

        <h3>Selected topics</h3>
        {topics.length === 0 ? (
          <p>No topics selected.</p>
        ) : (
          <ul>
            {topics.map((topic) => (
              <li key={topic}>{topicLabels[topic]}</li>
            ))}
          </ul>
        )}
      </section>
    </main>
  );
}
~~~

## How the controls work

### Text and email inputs

The name and email fields each store a string. Their input types describe the data, while their labels, names, and autocomplete hints provide browser and accessibility information.

### Number input

`weeklyHours` stays a string, including if the user clears the field. The preview checks for an empty string instead of converting it directly to a number.

### Single select

`level` stores one option value. The parent select receives that value, so React keeps the selected option in sync with state.

### Multiple select and checkbox group

Both controls represent the same set of topics as an array of strings. The multiple select reads all selected options from `selectedOptions`. Each checkbox uses `checked` to determine whether its own option is in that array.

The select and checkbox group are two ways to edit the same state. Changing either one updates the other on the next render.

### Independent checkbox

`sendUpdates` is a boolean controlled with `checked`. Its `value="yes"` is the form-data token, not the boolean checked state.

### Radio group

`contactMethod` stores one string. All radio buttons share the same `name`, have distinct values, and compare their value with the state to derive `checked`.

### Reset action

The reset button calls a function that puts every state variable back at its initial value. Because the controls are controlled, they all display the reset state on the next render.

## Experiments

1. Edit the name and email values and observe the preview.
2. Clear the number field. Notice that the preview shows “Not entered” rather than treating the empty field as zero.
3. Select topics in the multiple select, then change them with the checkbox group. Both controls should reflect the same array.
4. Select each radio option and verify only one remains selected.
5. Toggle “Send me course updates.” The preview changes based on a boolean, while the input's `value` remains the form-data string.
6. Press “Reset preferences.” Every control returns to the state set by `resetPreferences`.
7. Change one of the checkbox values without changing the state array and observe that the choice no longer matches the stored value.

## Key takeaways

- Choose `type` based on the kind of data and behavior the field needs.
- Keep number input state as an editable string; convert it only for calculations.
- A single select stores a string, and a multiple select stores an array of strings.
- A checkbox uses a boolean `checked`; checkbox groups can use arrays.
- Radio buttons share a `name` and use one selected string in state.
- State can be shared between controls when they represent the same choices.
- Reset controlled values by updating React state.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="028-controlled-form-elements.md">028 - Controlled form elements - Example</a></td><td><a href="../lessons/029-inputs-selects-checkboxes-and-radio-buttons.md">Lesson</a></td><td><a href="../qnas/029-inputs-selects-checkboxes-and-radio-buttons.md">Q&amp;A</a></td><td><a href="030-form-submission-and-browser-validation.md">030 - Form submission and browser validation - Example</a></td></tr>
  </tbody>
</table>
