# Example: Preserving and Resetting State with Keys

This runnable Vite example demonstrates three related behaviors:

1. A profile editor receives a key based on the selected profile, so switching profiles resets its local draft.
2. Editable cards use stable person IDs as keys, so reordering the list keeps each draft with its person.
3. A conditionally rendered search panel loses its local query when it is unmounted and later mounted again.

All changes stay in local React state. The example does not make network requests or save data outside the page.

## Project files

~~~text
state-identity-demo/
├── index.html
└── src/
    ├── App.jsx
    ├── main.jsx
    └── styles.css
~~~

Use these files in an existing Vite React project. From the project root, install dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

## src/App.jsx

~~~jsx
import { useState } from "react";

const initialProfiles = [
  { id: "profile-ada", name: "Ada Lovelace", role: "Mathematician" },
  { id: "profile-grace", name: "Grace Hopper", role: "Computer scientist" },
  { id: "profile-katherine", name: "Katherine Johnson", role: "Mathematician" },
];

const initialPeople = [
  { id: "person-ada", name: "Ada" },
  { id: "person-grace", name: "Grace" },
  { id: "person-katherine", name: "Katherine" },
];

function ProfileEditor({ profile, onSave }) {
  const [draftName, setDraftName] = useState(profile.name);

  function handleSubmit(event) {
    event.preventDefault();
    onSave(profile.id, draftName.trim());
  }

  return (
    <form className="editor-card" onSubmit={handleSubmit}>
      <div className="card-heading">
        <div>
          <p className="eyebrow">Keyed editor</p>
          <h3>Edit {profile.name}</h3>
        </div>
        <span className="identity-badge">ID: {profile.id}</span>
      </div>

      <p className="muted">
        Switch profiles after typing. A different profile ID changes this
        component key and starts a fresh draft.
      </p>

      <label className="field">
        Display name
        <input
          value={draftName}
          onChange={(event) => setDraftName(event.target.value)}
          required
        />
      </label>

      <div className="button-row">
        <button className="primary-button" type="submit">
          Save name
        </button>
        <button
          className="secondary-button"
          type="button"
          onClick={() => setDraftName(profile.name)}
        >
          Restore saved name
        </button>
      </div>
    </form>
  );
}

function PersonCard({ person }) {
  const [note, setNote] = useState("");
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <li className="person-card">
      <div className="person-heading">
        <div>
          <h3>{person.name}</h3>
          <p className="identity-badge">Stable ID: {person.id}</p>
        </div>
        <button
          className="secondary-button"
          type="button"
          aria-expanded={isExpanded}
          onClick={() => setIsExpanded((expanded) => !expanded)}
        >
          {isExpanded ? "Hide note" : "Edit note"}
        </button>
      </div>

      {isExpanded && (
        <label className="field">
          Private note for {person.name}
          <input
            value={note}
            onChange={(event) => setNote(event.target.value)}
            placeholder={"A note that belongs to " + person.name}
          />
        </label>
      )}
    </li>
  );
}

function SearchPanel() {
  const [query, setQuery] = useState("");

  return (
    <div className="search-panel">
      <label className="field">
        Temporary search text
        <input
          value={query}
          onChange={(event) => setQuery(event.target.value)}
          placeholder="Type something, then close the panel"
        />
      </label>
      <p className="muted">
        Current local query: <strong>{query || "(empty)"}</strong>
      </p>
    </div>
  );
}

export default function App() {
  const [profiles, setProfiles] = useState(initialProfiles);
  const [selectedProfileId, setSelectedProfileId] = useState(
    initialProfiles[0].id
  );
  const [people, setPeople] = useState(initialPeople);
  const [showSearch, setShowSearch] = useState(true);

  const selectedProfile = profiles.find(
    (profile) => profile.id === selectedProfileId
  );

  function saveProfileName(profileId, nextName) {
    if (!nextName) {
      return;
    }

    setProfiles((currentProfiles) =>
      currentProfiles.map((profile) =>
        profile.id === profileId
          ? { ...profile, name: nextName }
          : profile
      )
    );
  }

  function reversePeople() {
    setPeople((currentPeople) => [...currentPeople].reverse());
  }

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React state identity</p>
        <h1>Preserving and resetting state</h1>
        <p className="intro">
          Try each section. Observe what stays with a component, what follows
          an item, and what starts fresh after React removes a component.
        </p>
      </header>

      <section className="demo-section" aria-labelledby="profile-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Identity boundary</p>
            <h2 id="profile-heading">Reset a draft when the profile changes</h2>
          </div>
        </div>

        <label className="field profile-picker">
          Select a profile
          <select
            value={selectedProfileId}
            onChange={(event) => setSelectedProfileId(event.target.value)}
          >
            {profiles.map((profile) => (
              <option key={profile.id} value={profile.id}>
                {profile.name}
              </option>
            ))}
          </select>
        </label>

        {selectedProfile && (
          <ProfileEditor
            key={selectedProfile.id}
            profile={selectedProfile}
            onSave={saveProfileName}
          />
        )}

        <p className="explanation">
          The selected ID is the editor key. Switching to another profile
          creates a fresh editor and initializes its draft from that profile.
          Saving changes the profile data but keeps the same editor identity.
        </p>
      </section>

      <section className="demo-section" aria-labelledby="list-heading">
        <div className="section-heading section-heading-row">
          <div>
            <p className="eyebrow">Stable sibling keys</p>
            <h2 id="list-heading">Keep each draft with its person</h2>
          </div>
          <button className="secondary-button" type="button" onClick={reversePeople}>
            Reverse list
          </button>
        </div>

        <p className="muted">
          Open a few notes, type different text, then reverse the order. Each
          PersonCard uses its person's stable ID as its key.
        </p>

        <ul className="people-list">
          {people.map((person) => (
            <PersonCard key={person.id} person={person} />
          ))}
        </ul>

        <p className="explanation">
          React can match a card to its person after reordering because the
          identity stays in the data. An array index would describe a position,
          not the person, and could attach local state to the wrong row.
        </p>
      </section>

      <section className="demo-section" aria-labelledby="mount-heading">
        <div className="section-heading section-heading-row">
          <div>
            <p className="eyebrow">Conditional rendering</p>
            <h2 id="mount-heading">Unmounting discards local state</h2>
          </div>
          <button
            className="secondary-button"
            type="button"
            aria-pressed={showSearch}
            onClick={() => setShowSearch((visible) => !visible)}
          >
            {showSearch ? "Close search panel" : "Open search panel"}
          </button>
        </div>

        {showSearch ? (
          <SearchPanel />
        ) : (
          <p className="empty-state">
            The search panel is unmounted. Open it again and its query will be
            empty.
          </p>
        )}

        <p className="explanation">
          This condition removes SearchPanel from the tree. When it returns,
          React creates a new instance, so its local query starts over.
        </p>
      </section>

      <footer className="page-footer">
        <p>
          The examples use local state only. Refreshing the page clears all
          unsaved drafts.
        </p>
      </footer>
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
  color: #202a3a;
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

button {
  cursor: pointer;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #86aaff;
  outline-offset: 3px;
}

.page {
  width: min(100% - 2rem, 920px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  max-width: 68ch;
  margin-bottom: 2rem;
}

.eyebrow {
  margin: 0 0 0.4rem;
  color: #315db5;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

h1,
h2,
h3,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 6vw, 3.5rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0.45rem;
  font-size: clamp(1.25rem, 3vw, 1.7rem);
  letter-spacing: -0.025em;
}

h3 {
  margin-bottom: 0;
  font-size: 1rem;
}

.intro,
.muted,
.explanation,
.page-footer {
  color: #566477;
  line-height: 1.65;
}

.intro {
  margin-bottom: 0;
  font-size: 1.05rem;
}

.demo-section {
  margin-top: 1.25rem;
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d9e1ec;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 10px 28px rgb(32 53 84 / 5%);
}

.section-heading {
  margin-bottom: 1rem;
}

.section-heading-row,
.card-heading,
.person-heading,
.button-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
}

.section-heading-row {
  flex-wrap: wrap;
}

.field {
  display: grid;
  gap: 0.45rem;
  color: #29374a;
  font-size: 0.93rem;
  font-weight: 700;
}

.field input,
.field select {
  width: 100%;
  min-height: 2.75rem;
  padding: 0.65rem 0.75rem;
  border: 1px solid #bac8da;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
  font-weight: 450;
}

.profile-picker {
  max-width: 28rem;
  margin-bottom: 1rem;
}

.editor-card,
.search-panel {
  padding: 1rem;
  border: 1px solid #d9e1ec;
  border-radius: 0.75rem;
  background: #f9fbfe;
}

.card-heading {
  align-items: flex-start;
}

.identity-badge {
  display: inline-block;
  margin: 0.35rem 0 0;
  padding: 0.25rem 0.45rem;
  border-radius: 999px;
  color: #40516a;
  background: #eaf0f8;
  font-size: 0.75rem;
  font-weight: 650;
  overflow-wrap: anywhere;
}

.card-heading .identity-badge {
  margin-top: 0;
}

.editor-card .muted {
  margin: 0.8rem 0 1rem;
}

.button-row {
  justify-content: flex-start;
  flex-wrap: wrap;
  margin-top: 1rem;
}

.primary-button,
.secondary-button {
  min-height: 2.6rem;
  padding: 0.6rem 0.85rem;
  border-radius: 0.55rem;
}

.primary-button {
  border: 1px solid #315db5;
  color: #fff;
  background: #315db5;
  font-weight: 750;
}

.primary-button:hover {
  background: #244b9a;
}

.secondary-button {
  border: 1px solid #bac8da;
  color: #294775;
  background: #f6f8fc;
}

.secondary-button:hover {
  background: #eaf0f8;
}

.people-list {
  display: grid;
  gap: 0.7rem;
  margin: 1rem 0;
  padding: 0;
  list-style: none;
}

.person-card {
  padding: 0.9rem;
  border: 1px solid #d9e1ec;
  border-radius: 0.7rem;
  background: #fff;
}

.person-heading {
  align-items: flex-start;
}

.person-card .field {
  margin-top: 0.9rem;
}

.explanation {
  margin: 1rem 0 0;
  padding-top: 0.85rem;
  border-top: 1px solid #e4eaf2;
  font-size: 0.9rem;
}

.search-panel .field {
  margin-bottom: 0.75rem;
}

.search-panel .muted {
  margin-bottom: 0;
}

.empty-state {
  margin: 0;
  padding: 1rem;
  border: 1px dashed #bac8da;
  border-radius: 0.65rem;
  color: #566477;
  background: #f9fbfe;
}

.page-footer {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 0.9rem;
}

.page-footer p {
  margin-bottom: 0;
}

@media (max-width: 560px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .card-heading,
  .person-heading {
    align-items: flex-start;
    flex-direction: column;
  }

  .section-heading-row > button {
    width: 100%;
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
      content="An interactive React example of preserving and resetting component state."
    />
    <title>React State Identity Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How to explore the example

### Change the profile identity

1. Edit Ada's display name, but do not save it.
2. Select Grace from the profile dropdown.
3. Notice that the editor draft starts with Grace's saved name.
4. Switch back to Ada. Because switching removed Ada's keyed editor instance, her unsaved local draft is gone.
5. Save a name, then switch profiles and return. Saved profile data is owned by App, so it remains available even though the editor instance was recreated.

The editor key is the selected profile ID. Saving a new name changes profile data but does not change that ID, so it does not reset the editor.

### Reorder stateful list items

1. Open the note editor on two or more people.
2. Type different notes.
3. Select Reverse list.
4. The order changes, while each note remains attached to its person.

Each PersonCard uses person.id. As an experiment, replace that key with the array index and reorder the list. With stateful children, position-based identity can make notes or expanded state follow a position instead of the person. Restore the stable ID key afterward.

### Unmount and mount a component

1. Type a query in the temporary search panel.
2. Close the panel.
3. Open it again.
4. The query is empty because the first SearchPanel was removed and the new one initialized its state again.

## What to notice

- State preservation is about the component identity React matches in the returned tree, not merely about a JSX expression remaining in the source.
- A prop update does not necessarily create a new component instance.
- A changed key intentionally marks a different identity and resets that subtree.
- Stable keys let local state follow list items through reordering.
- Conditional removal discards the removed component's local state.
- Saved profile data lives in App, so it survives editor resets.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="047-reducer-design-and-state-machines.md">047 - Reducer Design, Transitions, and State Machines - Example</a></td><td><a href="../lessons/048-preserving-and-resetting-state.md">Lesson</a></td><td><a href="../qnas/048-preserving-and-resetting-state.md">Q&amp;A</a></td><td><a href="049-rules-of-hooks.md">049 - Example: Following the Rules of Hooks</a></td></tr>
  </tbody>
</table>
