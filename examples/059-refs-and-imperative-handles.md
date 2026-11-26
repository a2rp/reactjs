# Example: React 19 Refs and Imperative Handles

This runnable React 19 app demonstrates how a parent can use refs across function-component boundaries without owning the child’s markup:

- **NoteComposer** keeps its input DOM node private and exposes only **focus()** and **select()**.
- **NoteList** keeps its list DOM node private and exposes only **scrollToLatest()**.
- The parent owns the notes, draft, search query, and help-panel visibility with state. Those visible values are not hidden in refs.
- A controlled help panel shows why ordinary UI state belongs in props and state, not an imperative **open()** method.

The example uses no external assets or network requests. Its component ref-prop syntax requires React 19.

## Create the project

~~~sh
npm create vite@latest react19-ref-api -- --template react
cd react19-ref-api
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following. Start the development server with:

~~~sh
npm run dev
~~~

## Project files

~~~text
react19-ref-api/
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
    <meta name="theme-color" content="#f5f7fb" />
    <title>React 19 Ref API Demo</title>
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
import { useImperativeHandle, useRef, useState } from "react";

function NoteComposer({ ref, value, onChange, onSubmit }) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => ({
      focus() {
        inputRef.current?.focus();
      },
      select() {
        inputRef.current?.select();
      },
    }),
    [],
  );

  return (
    <form className="composer" onSubmit={onSubmit}>
      <label htmlFor="note-draft">New note</label>
      <div className="composer__row">
        <input
          id="note-draft"
          ref={inputRef}
          value={value}
          onChange={onChange}
          placeholder="Write a short note"
          autoComplete="off"
        />
        <button className="button button--primary" type="submit">
          Add note
        </button>
      </div>
    </form>
  );
}

function NoteList({ ref, notes }) {
  const listRef = useRef(null);

  useImperativeHandle(
    ref,
    () => ({
      scrollToLatest() {
        listRef.current?.lastElementChild?.scrollIntoView({
          behavior: "smooth",
          block: "nearest",
        });
      },
    }),
    [],
  );

  if (notes.length === 0) {
    return <p className="empty-state">No matching notes. Add one or change the search.</p>;
  }

  return (
    <ul className="note-list" ref={listRef} aria-label="Notes">
      {notes.map((note) => (
        <li className="note-card" key={note.id}>
          <span className="note-card__number">Note {note.id}</span>
          <p>{note.text}</p>
        </li>
      ))}
    </ul>
  );
}

export default function App() {
  const [draft, setDraft] = useState("");
  const [query, setQuery] = useState("");
  const [showHelp, setShowHelp] = useState(false);
  const [notes, setNotes] = useState([
    { id: 1, text: "A child can expose a small imperative API." },
    { id: 2, text: "Use state for values that change visible UI." },
    { id: 3, text: "React 19 function components can receive ref as a prop." },
  ]);
  const nextIdRef = useRef(4);
  const composerRef = useRef(null);
  const noteListRef = useRef(null);

  const visibleNotes = notes.filter((note) =>
    note.text.toLowerCase().includes(query.trim().toLowerCase()),
  );

  function addNote(event) {
    event.preventDefault();
    const text = draft.trim();

    if (!text) {
      composerRef.current?.focus();
      return;
    }

    const note = { id: nextIdRef.current, text };
    nextIdRef.current += 1;
    setNotes((currentNotes) => [...currentNotes, note]);
    setDraft("");
    composerRef.current?.focus();
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React 19 component API</p>
        <h1>Notes with a focused ref handle</h1>
        <p className="page-intro">
          The parent manages what appears. Each child exposes only the imperative
          action the parent needs.
        </p>
      </header>

      <section className="panel" aria-labelledby="composer-title">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Child component: NoteComposer</p>
            <h2 id="composer-title">Create a note</h2>
          </div>
          <span className="api-badge">focus · select</span>
        </div>

        <NoteComposer
          ref={composerRef}
          value={draft}
          onChange={(event) => setDraft(event.target.value)}
          onSubmit={addNote}
        />

        <div className="action-row" aria-label="Composer ref actions">
          <button
            className="button button--secondary"
            type="button"
            onClick={() => composerRef.current?.focus()}
          >
            Focus input
          </button>
          <button
            className="button button--secondary"
            type="button"
            onClick={() => composerRef.current?.select()}
          >
            Select text
          </button>
          <p className="action-explanation">
            The parent receives these two methods, not the input DOM node.
          </p>
        </div>
      </section>

      <section className="panel" aria-labelledby="notes-title">
        <div className="section-heading section-heading--wrap">
          <div>
            <p className="eyebrow">Child component: NoteList</p>
            <h2 id="notes-title">Your notes</h2>
          </div>
          <button
            className="button button--secondary"
            type="button"
            onClick={() => noteListRef.current?.scrollToLatest()}
          >
            Scroll to latest
          </button>
        </div>

        <label className="search-label" htmlFor="note-search">Filter notes</label>
        <input
          className="search-input"
          id="note-search"
          type="search"
          value={query}
          onChange={(event) => setQuery(event.target.value)}
          placeholder="Search note text"
        />
        <p className="result-count" role="status" aria-live="polite">
          Showing {visibleNotes.length} of {notes.length} notes
        </p>

        <NoteList ref={noteListRef} notes={visibleNotes} />
      </section>

      <section className="panel panel--hint" aria-labelledby="state-title">
        <div className="section-heading section-heading--wrap">
          <div>
            <p className="eyebrow">Declarative UI</p>
            <h2 id="state-title">Help panel</h2>
          </div>
          <button
            className="button button--secondary"
            type="button"
            aria-expanded={showHelp}
            aria-controls="help-panel"
            onClick={() => setShowHelp((isVisible) => !isVisible)}
          >
            {showHelp ? "Hide explanation" : "Show explanation"}
          </button>
        </div>

        <div className="help-panel" id="help-panel" hidden={!showHelp}>
          <p>
            This panel’s visibility is ordinary UI state, so the parent renders
            it from <strong>showHelp</strong>. A ref method such as
            <strong> open()</strong> would hide visible state outside React’s
            normal data flow.
          </p>
        </div>
      </section>

      <footer className="page-footer">
        <p>
          Ref handles are useful for focused imperative actions. Props and state
          remain the source for visible UI.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #172033;
  background: #f2f5fa;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-weight: 400;
}

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input {
  font: inherit;
}

button {
  cursor: pointer;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #93c5fd;
  outline-offset: 2px;
}

.page-shell {
  width: min(100% - 32px, 860px);
  margin: 0 auto;
  padding: 56px 0 40px;
}

.page-header {
  margin-bottom: 28px;
}

.eyebrow {
  margin-bottom: 8px;
  color: #53627a;
  font-size: 0.76rem;
  font-weight: 750;
  letter-spacing: 0.11em;
  text-transform: uppercase;
}

h1,
h2,
p {
  overflow-wrap: anywhere;
}

h1 {
  max-width: 700px;
  font-size: clamp(2rem, 6vw, 3.15rem);
  line-height: 1.08;
  letter-spacing: -0.045em;
}

h2 {
  font-size: 1.25rem;
  line-height: 1.25;
}

.page-intro {
  max-width: 650px;
  margin-top: 14px;
  color: #56647a;
  font-size: 1.05rem;
  line-height: 1.65;
}

.panel {
  margin-top: 18px;
  padding: 24px;
  border: 1px solid #d9e0eb;
  border-radius: 18px;
  background: #fff;
  box-shadow: 0 12px 32px rgb(35 52 79 / 6%);
}

.panel--hint {
  background: #f9fbff;
}

.section-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 22px;
}

.section-heading .eyebrow {
  margin-bottom: 6px;
}

.section-heading--wrap {
  align-items: center;
  flex-wrap: wrap;
}

.api-badge {
  padding: 7px 10px;
  border: 1px solid #c7d7f2;
  border-radius: 999px;
  color: #244b87;
  background: #edf4ff;
  font-size: 0.82rem;
  font-weight: 700;
  white-space: nowrap;
}

.composer label,
.search-label {
  display: block;
  margin-bottom: 8px;
  color: #344158;
  font-size: 0.92rem;
  font-weight: 700;
}

.composer__row {
  display: flex;
  gap: 10px;
}

.composer input,
.search-input {
  width: 100%;
  min-width: 0;
  padding: 12px 14px;
  border: 1px solid #cbd4e1;
  border-radius: 10px;
  color: #172033;
  background: #fff;
}

.composer input::placeholder,
.search-input::placeholder {
  color: #8792a3;
}

.button {
  min-height: 42px;
  padding: 9px 14px;
  border: 1px solid transparent;
  border-radius: 10px;
  font-size: 0.91rem;
  font-weight: 700;
  transition: background-color 120ms ease, border-color 120ms ease,
    transform 120ms ease;
}

.button:active {
  transform: translateY(1px);
}

.button--primary {
  flex: 0 0 auto;
  color: #fff;
  background: #245fc5;
}

.button--primary:hover {
  background: #194ea9;
}

.button--secondary {
  border-color: #cbd4e1;
  color: #26344b;
  background: #fff;
}

.button--secondary:hover {
  border-color: #9aa9bd;
  background: #f5f8fc;
}

.action-row {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 9px;
  margin-top: 14px;
}

.action-explanation {
  flex-basis: 100%;
  margin-top: 2px;
  color: #66748a;
  font-size: 0.87rem;
  line-height: 1.5;
}

.search-label {
  margin-bottom: 7px;
}

.result-count {
  margin-top: 9px;
  color: #66748a;
  font-size: 0.88rem;
}

.note-list {
  display: grid;
  max-height: 300px;
  gap: 10px;
  overflow: auto;
  margin-top: 14px;
  padding: 2px 4px 2px 2px;
  list-style: none;
  scroll-behavior: smooth;
}

.note-card {
  padding: 15px 16px;
  border: 1px solid #dce3ed;
  border-radius: 12px;
  background: #fbfcfe;
}

.note-card__number {
  color: #62718a;
  font-size: 0.76rem;
  font-weight: 750;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.note-card p {
  margin-top: 5px;
  line-height: 1.5;
}

.empty-state {
  margin-top: 15px;
  padding: 16px;
  border: 1px dashed #bdc9d9;
  border-radius: 12px;
  color: #59677d;
  line-height: 1.5;
}

.help-panel {
  padding: 15px 16px;
  border-left: 4px solid #5a84d6;
  border-radius: 8px;
  color: #394860;
  background: #edf4ff;
  line-height: 1.65;
}

.help-panel[hidden] {
  display: none;
}

.page-footer {
  padding: 22px 4px 0;
  color: #66748a;
  font-size: 0.9rem;
  line-height: 1.55;
}

@media (max-width: 560px) {
  .page-shell {
    width: min(100% - 22px, 860px);
    padding-top: 32px;
  }

  .panel {
    padding: 18px;
    border-radius: 14px;
  }

  .composer__row {
    align-items: stretch;
    flex-direction: column;
  }

  .button--primary {
    width: 100%;
  }

  .api-badge {
    font-size: 0.75rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    transition-duration: 0.01ms !important;
  }
}
~~~

## Why the API is designed this way

### The child owns the DOM details

**NoteComposer** stores its input element in **inputRef** and returns only two methods. The parent can focus or select the input, but cannot access its classes, replace its value, or depend on the child’s exact markup.

**NoteList** owns the list element and returns one scrolling method. It performs a non-destructive browser operation on the React-rendered list. The parent does not query through the child’s DOM.

### Visible values stay declarative

The parent owns **draft**, **query**, **notes**, and **showHelp** with state. These values affect visible output, so changing them schedules React renders. The child components receive values and callbacks through props.

The parent uses **nextIdRef** only for non-visual bookkeeping. Incrementing it does not need to render anything. This value is read and changed in the submit event handler, not during rendering.

### The parent handles a missing child safely

Before a child is committed, and after it unmounts, its ref can be null. Optional chaining in the button handlers makes the action safe if the child is not currently attached.

## React 18 compatibility pattern

The runnable project above intentionally uses React 19’s **ref** prop. For a project that supports React 18, wrap the child with **forwardRef** and accept the ref as the second argument:

~~~jsx
import { forwardRef, useImperativeHandle, useRef } from "react";

const NoteComposer = forwardRef(function NoteComposer(
  { value, onChange, onSubmit },
  ref,
) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => ({
      focus() {
        inputRef.current?.focus();
      },
      select() {
        inputRef.current?.select();
      },
    }),
    [],
  );

  return (
    <form onSubmit={onSubmit}>
      <input ref={inputRef} value={value} onChange={onChange} />
      <button type="submit">Add note</button>
    </form>
  );
});
~~~

The parent’s **<NoteComposer ref={composerRef} />** usage is unchanged. Keep this compatibility wrapper when React 18 is in the supported range; do not use the React 19-only ref-prop implementation in a React 18 app.

## Try these changes

1. Add a **clearSearch()** method to a search field handle. Clear the query by calling a parent callback or by keeping the query in the child’s state; do not directly mutate a controlled input’s DOM value.
2. Add **focusComposer()** to **NoteList** only if there is a real reason for the list component to coordinate that action. Consider whether the parent should call the composer handle instead.
3. Add an **isHelpVisible** prop to a separate help component and keep toggling it with state. Compare that design with a ref method named **toggleHelp()**.
4. Add a **scrollToNote(id)** method that finds the matching rendered note. Keep the item data in props and use refs only for the DOM operation.
5. Remove the **select()** method and confirm that the parent can no longer request text selection. This illustrates how the child’s handle defines the public API.
6. Temporarily change a note or a draft to a ref and observe why ref mutation alone does not update the visible interface. Restore state for those values.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="058-useref.md">058 - Example: useRef for DOM Nodes and Mutable Values</a></td><td><a href="../lessons/059-refs-and-imperative-handles.md">Lesson</a></td><td><a href="../qnas/059-refs-and-imperative-handles.md">Q&amp;A</a></td><td><a href="060-memoization-useid-usememo-usecallback.md">060 - Example: Memoization and useId</a></td></tr>
  </tbody>
</table>
