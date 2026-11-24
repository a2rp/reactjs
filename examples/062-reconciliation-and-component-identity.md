# Example: Reconciliation, Component Identity, and Keys

This runnable React app demonstrates two key behaviors:

- Two reorderable lists show why stable data keys keep row state with its item, while index keys can leave that state attached to the wrong row.
- A profile editor uses a key based on the selected profile ID to intentionally reset its local draft when switching profiles.

Each row stores a small draft in its own state. The app makes no external requests.

## Create the project

~~~sh
npm create vite@latest reconciliation-keys-demo -- --template react
cd reconciliation-keys-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following and start the app:

~~~sh
npm run dev
~~~

## Project files

~~~text
reconciliation-keys-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f6fb" />
    <title>React Reconciliation and Keys Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## src/main.jsx

~~~jsx
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
~~~

## src/App.jsx

~~~jsx
import { useId, useState } from "react";

const STARTING_PEOPLE = [
  { id: "person-ada", name: "Ada" },
  { id: "person-grace", name: "Grace" },
  { id: "person-katherine", name: "Katherine" },
];

const PROFILES = [
  { id: "profile-ada", name: "Ada Lovelace" },
  { id: "profile-grace", name: "Grace Hopper" },
];

function PersonRow({ person }) {
  const [draft, setDraft] = useState(`A note about ${person.name}`);
  const inputId = useId();

  return (
    <li className="person-row">
      <div className="person-row__heading">
        <div>
          <span className="row-id">Data ID: {person.id}</span>
          <h3>{person.name}</h3>
        </div>
        <span className="state-tag">Local state</span>
      </div>
      <label htmlFor={inputId}>Personal draft for this row</label>
      <input
        id={inputId}
        value={draft}
        onChange={(event) => setDraft(event.target.value)}
      />
      <p className="row-explanation">
        If the row identity is preserved, this draft stays with its component.
      </p>
    </li>
  );
}

function ReorderableList({ title, description, people, setPeople, useIndexKeys }) {
  function reversePeople() {
    setPeople((currentPeople) => [...currentPeople].reverse());
  }

  return (
    <section className="list-demo" aria-label={title}>
      <div className="list-demo__heading">
        <div>
          <p className="eyebrow">{useIndexKeys ? "Position identity" : "Data identity"}</p>
          <h3>{title}</h3>
        </div>
        <button className="button" type="button" onClick={reversePeople}>
          Reverse order
        </button>
      </div>
      <p className="list-demo__description">{description}</p>
      <ul className="person-list">
        {people.map((person, index) => (
          <PersonRow
            key={useIndexKeys ? index : person.id}
            person={person}
          />
        ))}
      </ul>
    </section>
  );
}

function KeyComparison() {
  const [stablePeople, setStablePeople] = useState(STARTING_PEOPLE);
  const [indexPeople, setIndexPeople] = useState(STARTING_PEOPLE);

  return (
    <section className="panel" aria-labelledby="comparison-title">
      <p className="eyebrow">Reorder a collection</p>
      <h2 id="comparison-title">Which row owns the draft?</h2>
      <p className="panel-copy">
        Edit each draft, then reverse both lists. The left list uses each
        person’s stable ID. The right list uses the current array index.
      </p>

      <div className="comparison-grid">
        <ReorderableList
          title="Stable data keys"
          description="key={person.id}"
          people={stablePeople}
          setPeople={setStablePeople}
          useIndexKeys={false}
        />
        <ReorderableList
          title="Index keys"
          description="key={index}"
          people={indexPeople}
          setPeople={setIndexPeople}
          useIndexKeys={true}
        />
      </div>

      <p className="result-note">
        After reversing, the stable-key drafts follow their people. In the
        index-key list, React reuses rows by position, so a draft can appear
        beside another person’s name.
      </p>
    </section>
  );
}

function ProfileEditor({ profile }) {
  const [draftName, setDraftName] = useState(profile.name);
  const inputId = useId();

  return (
    <div className="profile-editor">
      <span className="row-id">Editor key: {profile.id}</span>
      <label htmlFor={inputId}>Local profile draft</label>
      <input
        id={inputId}
        value={draftName}
        onChange={(event) => setDraftName(event.target.value)}
      />
      <p>
        This input starts from the selected profile name. Switching profiles
        changes the component key, so the editor remounts with a fresh draft.
      </p>
    </div>
  );
}

function KeyedResetDemo() {
  const [profileIndex, setProfileIndex] = useState(0);
  const profile = PROFILES[profileIndex];
  const nextProfileIndex = (profileIndex + 1) % PROFILES.length;

  return (
    <section className="panel" aria-labelledby="reset-title">
      <p className="eyebrow">Intentional reset</p>
      <h2 id="reset-title">Use a key when a new identity needs fresh state</h2>
      <p className="panel-copy">
        Edit the draft, then switch profiles. The editor gets a different key
        and starts with that profile’s name.
      </p>
      <div className="profile-switcher">
        <p className="active-profile">
          Current profile: <strong>{profile.name}</strong>
        </p>
        <button
          className="button button--primary"
          type="button"
          onClick={() => setProfileIndex(nextProfileIndex)}
        >
          Switch profile
        </button>
      </div>
      <ProfileEditor key={profile.id} profile={profile} />
      <p className="result-note">
        A key change destroys the old editor’s local state; it does not save the
        old draft for when you switch back. Lift drafts into parent state if
        they must be restored.
      </p>
    </section>
  );
}

export default function App() {
  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React reconciliation</p>
        <h1>Keys describe identity</h1>
        <p className="page-intro">
          React uses tree structure, component type, and keys to decide which
          component state belongs to which item between renders.
        </p>
      </header>

      <div className="identity-summary" aria-label="Component identity factors">
        <span>Parent position</span>
        <span aria-hidden="true">+</span>
        <span>Component type</span>
        <span aria-hidden="true">+</span>
        <span>Sibling key</span>
      </div>

      <KeyComparison />
      <KeyedResetDemo />

      <section className="panel panel--notes" aria-labelledby="rules-title">
        <p className="eyebrow">Key rules</p>
        <h2 id="rules-title">Keep keys stable and data-based</h2>
        <ul className="rule-list">
          <li>Use a stable ID stored on the item.</li>
          <li>Keys must be unique among siblings, not across the entire app.</li>
          <li>React consumes key; pass a separate prop if the child needs the ID.</li>
          <li>Changing a key resets that component subtree’s local state.</li>
          <li>Do not generate keys with indexes for changing lists or with random values during render.</li>
        </ul>
      </section>

      <footer className="page-footer">
        Keys do not move state across different parents. They identify siblings
        within one parent’s rendered children.
      </footer>
    </main>
  );
}
~~~

## Run the experiment

1. In both lists, change each **Personal draft for this row** field so the drafts are easy to recognize.
2. Click **Reverse order** in both lists.
3. In **Stable data keys**, each draft remains with its matching person.
4. In **Index keys**, the people change position but React reuses row instances by index, so the draft text can appear beside a different person.
5. Edit the profile draft and click **Switch profile**. The profile editor’s key changes, so React discards the old editor state and mounts a fresh editor.
6. Switch back. The previous local draft is gone, because a key is not a cache for unmounted components.

## Why the comparison works

**PersonRow** owns a controlled text field in local state. With a stable **person.id** key, React can match that row component to the same person after the array is reversed. Its local draft follows the person.

With **index** as the key, React matches the first row with the first position, the second row with the second position, and so on. After reversal, a row instance receives a different **person** prop while retaining its local **draft** state. That is why the state appears associated with the wrong item.

The profile editor demonstrates a different use of keys. The selected profile ID is intentionally part of the editor identity. Changing it unmounts the old editor and initializes the new editor from its profile.

## If the draft should survive switching profiles

A keyed reset is not appropriate when each profile’s edits must be restored later. Store each draft in parent state keyed by profile ID, then pass the active draft and change handler to the editor:

~~~jsx
const [drafts, setDrafts] = useState({});
const draft = drafts[profile.id] ?? profile.name;

<ProfileEditor
  profile={profile}
  draftName={draft}
  onDraftChange={(value) => {
    setDrafts((current) => ({ ...current, [profile.id]: value }));
  }}
/>
~~~

The parent state outlives the editor instance, so switching profiles no longer loses their drafts.

## More key examples

### A mapped component uses an ID from its data

~~~jsx
const rows = people.map((person) => (
  <PersonRow key={person.id} person={person} personId={person.id} />
));
~~~

The component receives **personId** as a regular prop. It does not receive the special **key** prop.

### A keyed Fragment groups sibling elements

~~~jsx
import { Fragment } from "react";

const entries = posts.map((post) => (
  <Fragment key={post.id}>
    <h2>{post.title}</h2>
    <p>{post.summary}</p>
  </Fragment>
));
~~~

The long Fragment syntax supports a key without adding a wrapper element to the DOM. The short **<>...</>** syntax cannot accept one.

## Try these changes

1. Change the stable list to use **key={index}** and observe how the draft can become associated with the wrong person.
2. Give two list items the same key and inspect the development console warning; then restore unique IDs.
3. Change the profile editor key to a constant. Observe that the component identity no longer changes when switching profiles, so its local state is preserved.
4. Remove **key={profile.id}** from the editor and observe the same-position component behavior.
5. Add a new person with an ID stored in the data before rendering it. Give it a draft and reorder the list.
6. Move the profile drafts into parent state so switching away and back restores each one.
7. Define a small stateful component inside **App** and trigger an App update. Observe why nested component definitions can remount their children; then move the definition to module scope.
