# 040. Updating object state immutably - Example

## What this example demonstrates

This React app edits one profile object containing flat fields and nested objects. Every update returns a new object instead of mutating the current state.

It also lets you save the current object as a snapshot. After you edit the profile, the saved snapshot remains unchanged because the update handlers create new objects.

## Project files

Add these files to an existing React app created with Vite:

- src/App.jsx
- src/App.css
- src/main.jsx
- src/index.css

Run the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useState } from 'react';
import './App.css';

const initialPerson = {
  firstName: 'Asha',
  lastName: 'Rao',
  email: 'asha@example.test',
  artwork: {
    title: 'Blue Window',
    city: 'Lima',
    year: 2024,
  },
  settings: {
    notifications: {
      email: true,
    },
  },
};

export default function App() {
  const [person, setPerson] = useState(initialPerson);
  const [savedPerson, setSavedPerson] = useState(null);

  function handleFlatFieldChange(event) {
    const { name, value } = event.target;

    setPerson((currentPerson) => ({
      ...currentPerson,
      [name]: value,
    }));
  }

  function handleArtworkFieldChange(event) {
    const { name, value } = event.target;

    setPerson((currentPerson) => ({
      ...currentPerson,
      artwork: {
        ...currentPerson.artwork,
        [name]: name === 'year' ? Number(value) : value,
      },
    }));
  }

  function handleNotificationChange(event) {
    const isChecked = event.target.checked;

    setPerson((currentPerson) => ({
      ...currentPerson,
      settings: {
        ...currentPerson.settings,
        notifications: {
          ...currentPerson.settings.notifications,
          email: isChecked,
        },
      },
    }));
  }

  function handleAddNickname() {
    setPerson((currentPerson) => ({
      ...currentPerson,
      nickname: 'Mimi',
    }));
  }

  function handleRemoveNickname() {
    setPerson((currentPerson) => {
      const { nickname, ...personWithoutNickname } = currentPerson;
      return personWithoutNickname;
    });
  }

  function handleSaveSnapshot() {
    setSavedPerson(person);
  }

  function handleReset() {
    setPerson(initialPerson);
    setSavedPerson(null);
  }

  const fullName = person.firstName + ' ' + person.lastName;

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React state</p>
        <h1>Updating object state immutably</h1>
        <p>
          Edit the fields, save a snapshot, and edit again. The saved object
          keeps its earlier values because each update creates replacement
          objects.
        </p>
      </header>

      <div className="content-grid">
        <section className="card" aria-labelledby="profile-heading">
          <p className="eyebrow">Flat object fields</p>
          <h2 id="profile-heading">Profile</h2>

          <div className="form-grid">
            <label htmlFor="firstName">First name</label>
            <input
              id="firstName"
              name="firstName"
              autoComplete="given-name"
              value={person.firstName}
              onChange={handleFlatFieldChange}
            />

            <label htmlFor="lastName">Last name</label>
            <input
              id="lastName"
              name="lastName"
              autoComplete="family-name"
              value={person.lastName}
              onChange={handleFlatFieldChange}
            />

            <label htmlFor="email">Email</label>
            <input
              id="email"
              name="email"
              type="email"
              autoComplete="email"
              value={person.email}
              onChange={handleFlatFieldChange}
            />

            {person.nickname !== undefined && (
              <>
                <label htmlFor="nickname">Nickname</label>
                <input
                  id="nickname"
                  name="nickname"
                  value={person.nickname}
                  onChange={handleFlatFieldChange}
                />
              </>
            )}
          </div>

          <div className="button-row">
            {person.nickname === undefined ? (
              <button type="button" onClick={handleAddNickname}>
                Add nickname
              </button>
            ) : (
              <button
                className="secondary-button"
                type="button"
                onClick={handleRemoveNickname}
              >
                Remove nickname
              </button>
            )}
          </div>
        </section>

        <section className="card" aria-labelledby="artwork-heading">
          <p className="eyebrow">Nested object</p>
          <h2 id="artwork-heading">Artwork</h2>

          <div className="form-grid">
            <label htmlFor="artwork-title">Title</label>
            <input
              id="artwork-title"
              name="title"
              value={person.artwork.title}
              onChange={handleArtworkFieldChange}
            />

            <label htmlFor="artwork-city">City</label>
            <input
              id="artwork-city"
              name="city"
              value={person.artwork.city}
              onChange={handleArtworkFieldChange}
            />

            <label htmlFor="artwork-year">Year</label>
            <input
              id="artwork-year"
              name="year"
              type="number"
              value={person.artwork.year}
              onChange={handleArtworkFieldChange}
            />
          </div>
        </section>

        <section className="card" aria-labelledby="settings-heading">
          <p className="eyebrow">Nested settings</p>
          <h2 id="settings-heading">Notifications</h2>

          <label className="checkbox-row" htmlFor="email-notifications">
            <input
              id="email-notifications"
              type="checkbox"
              checked={person.settings.notifications.email}
              onChange={handleNotificationChange}
            />
            Receive email notifications
          </label>

          <p className="help-text">
            This update copies the settings object and the notifications object
            before changing the email property.
          </p>
        </section>

        <section className="card snapshot-card" aria-labelledby="preview-heading">
          <p className="eyebrow">Current render</p>
          <h2 id="preview-heading">Profile preview</h2>

          <p className="profile-name" aria-live="polite">
            {fullName}
            {person.nickname !== undefined && (
              <span className="nickname"> ({person.nickname})</span>
            )}
          </p>
          <p>{person.email}</p>
          <p>
            Artwork: {person.artwork.title}, {person.artwork.city} (
            {person.artwork.year})
          </p>
          <p>
            Email notifications:{' '}
            {person.settings.notifications.email ? 'On' : 'Off'}
          </p>

          <div className="button-row">
            <button type="button" onClick={handleSaveSnapshot}>
              Save current snapshot
            </button>
            <button
              className="secondary-button"
              type="button"
              onClick={handleReset}
            >
              Reset profile
            </button>
          </div>

          <h3>Saved snapshot</h3>
          {savedPerson === null ? (
            <p className="help-text">No snapshot saved yet.</p>
          ) : (
            <div className="saved-preview" role="status">
              <p>
                {savedPerson.firstName} {savedPerson.lastName}
                {savedPerson.nickname !== undefined &&
                  ' (' + savedPerson.nickname + ')'}
              </p>
              <p>{savedPerson.email}</p>
              <p>
                Artwork: {savedPerson.artwork.title},{' '}
                {savedPerson.artwork.city} ({savedPerson.artwork.year})
              </p>
              <p>
                Email notifications:{' '}
                {savedPerson.settings.notifications.email ? 'On' : 'Off'}
              </p>
            </div>
          )}
        </section>
      </div>

      <section className="card state-inspector" aria-labelledby="state-heading">
        <p className="eyebrow">For learning</p>
        <h2 id="state-heading">Current state object</h2>
        <p className="help-text">
          This JSON is displayed for inspection only. It is not used to copy or
          update the state.
        </p>
        <pre>{JSON.stringify(person, null, 2)}</pre>
      </section>

      <footer className="page-footer">
        <p>
          The nested updates create new objects along the changed path. Previous
          objects, including the saved snapshot, stay unchanged.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/App.css

~~~css
.page-shell {
  width: min(100% - 2rem, 1100px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 72ch;
  margin-bottom: 1.75rem;
}

.page-header h1 {
  margin-block: 0.35rem 0.75rem;
  font-size: clamp(2rem, 5vw, 3.2rem);
  line-height: 1.1;
}

.page-header > p:last-child,
.help-text,
.page-footer {
  line-height: 1.65;
  color: #4b5563;
}

.eyebrow {
  color: #4d3ab8;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.content-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
}

.card {
  min-width: 0;
  padding: clamp(1rem, 3vw, 1.5rem);
  border: 1px solid #d5dae4;
  border-radius: 1rem;
  background: #ffffff;
  box-shadow: 0 0.5rem 1.5rem rgb(24 33 54 / 6%);
}

.card h2 {
  margin-block: 0.35rem 1rem;
}

.card h3 {
  margin-block: 1.25rem 0.65rem;
}

.form-grid {
  display: grid;
  grid-template-columns: 7rem minmax(0, 1fr);
  gap: 0.7rem 0.85rem;
  align-items: center;
}

.form-grid label {
  font-weight: 650;
}

input:not([type="checkbox"]) {
  width: 100%;
  min-height: 2.65rem;
  padding: 0.55rem 0.7rem;
  border: 1px solid #abb3c0;
  border-radius: 0.5rem;
  background: #ffffff;
  color: #202735;
}

input[type="checkbox"] {
  width: 1.1rem;
  height: 1.1rem;
  accent-color: #5b46d6;
}

input:focus-visible,
button:focus-visible {
  outline: 3px solid #a99cff;
  outline-offset: 2px;
}

.checkbox-row {
  display: flex;
  align-items: center;
  gap: 0.65rem;
  line-height: 1.5;
}

.help-text {
  margin-top: 0.8rem;
  font-size: 0.93rem;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-top: 1rem;
}

button {
  min-height: 2.65rem;
  padding: 0.55rem 0.8rem;
  border: 1px solid #4d3ab8;
  border-radius: 0.5rem;
  background: #5b46d6;
  color: #ffffff;
  cursor: pointer;
  font: inherit;
}

button:hover {
  background: #4835b5;
}

.secondary-button {
  border-color: #abb3c0;
  background: #ffffff;
  color: #202735;
}

.secondary-button:hover {
  background: #f0f2f6;
}

.snapshot-card > p {
  margin-block: 0.55rem;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.profile-name {
  font-size: 1.3rem;
  font-weight: 750;
}

.nickname {
  font-size: 1rem;
  font-weight: 500;
  color: #596273;
}

.saved-preview {
  padding: 0.85rem;
  border-left: 4px solid #27804a;
  border-radius: 0.4rem;
  background: #edf8f0;
}

.saved-preview p {
  margin-block: 0.25rem;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.state-inspector {
  margin-top: 1rem;
}

.state-inspector pre {
  max-height: 28rem;
  overflow: auto;
  margin-top: 0.8rem;
  padding: 1rem;
  border-radius: 0.6rem;
  background: #202735;
  color: #f5f7fb;
  font-size: 0.9rem;
  line-height: 1.5;
  white-space: pre-wrap;
  overflow-wrap: anywhere;
}

.page-footer {
  margin-top: 1.5rem;
}

@media (max-width: 720px) {
  .page-shell {
    width: min(100% - 1.25rem, 1100px);
    padding-block: 1.5rem;
  }

  .content-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 430px) {
  .form-grid {
    grid-template-columns: 1fr;
    gap: 0.35rem;
  }

  .form-grid label:not(:first-child) {
    margin-top: 0.45rem;
  }
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.jsx';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

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
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #202735;
  background: #f6f7fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input {
  font: inherit;
}
~~~

## Try it

1. Change the first name or email. The flat-field handler copies the current object and replaces only the named property.
2. Change the artwork city. The handler creates a new artwork object and a new outer person object.
3. Toggle email notifications. The handler creates new objects at the settings and notifications levels too.
4. Select Save current snapshot.
5. Change several fields and toggle the notification setting. The saved snapshot keeps the values from the moment you saved it.
6. Select Add nickname, edit it, and then select Remove nickname to see property addition and removal.
7. Inspect the JSON panel to see the current object shape. The panel only displays the object; it is not involved in updating it.
8. Select Reset profile to return to the initial profile and clear the saved snapshot.

## What to notice

- Every handler passes a new object to setPerson.
- Unchanged fields are preserved by spreading the previous object.
- Nested updates copy the nested object and each outer object along the changed path.
- The saved snapshot remains unchanged after later edits because the previous objects are not mutated.
- The state setter replaces the complete object; it does not merge partial objects automatically.
