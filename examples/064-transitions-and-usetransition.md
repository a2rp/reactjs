# Example: Async Actions with useTransition

This runnable React app demonstrates a local profile-draft save flow:

- Text fields update urgently and remain editable while saving.
- An async Action keeps **isPending** true while the simulated service is working.
- The saved result is committed in a nested Transition after **await**.
- A one-shot failure control demonstrates inline error handling.
- The currently saved draft remains visible until a new save succeeds.

The app makes no network requests. Its local timer stands in for service latency so the pending state is observable; a real application would await its service function instead.

## Create the project

~~~sh
npm create vite@latest transition-actions-demo -- --template react
cd transition-actions-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following and start the app:

~~~sh
npm run dev
~~~

## Project files

~~~text
transition-actions-demo/
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
    <meta name="theme-color" content="#f4f7fb" />
    <title>React Async Transition Actions</title>
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
import { useState, useTransition } from "react";

function persistDraft(draft, shouldFail) {
  return new Promise((resolve, reject) => {
    window.setTimeout(() => {
      if (shouldFail) {
        reject(new Error("The simulated save failed."));
        return;
      }

      resolve({
        id: Date.now(),
        title: draft.title,
        body: draft.body,
        savedAt: new Date().toISOString(),
      });
    }, 700);
  });
}

function formatSavedTime(isoTime) {
  return new Intl.DateTimeFormat(undefined, {
    dateStyle: "medium",
    timeStyle: "short",
  }).format(new Date(isoTime));
}

function DraftEditor() {
  const [title, setTitle] = useState("Project notes");
  const [body, setBody] = useState("Keep this draft while I edit.");
  const [savedDraft, setSavedDraft] = useState(null);
  const [errorMessage, setErrorMessage] = useState("");
  const [failNextSave, setFailNextSave] = useState(false);
  const [isPending, startTransition] = useTransition();

  function handleSubmit(event) {
    event.preventDefault();

    if (isPending) {
      return;
    }

    const submittedDraft = {
      title: title.trim(),
      body: body.trim(),
    };

    if (!submittedDraft.title) {
      setErrorMessage("Add a title before saving.");
      return;
    }

    const shouldFail = failNextSave;
    setFailNextSave(false);
    setErrorMessage("");

    startTransition(async () => {
      try {
        const saved = await persistDraft(submittedDraft, shouldFail);

        // Updates after await need another Transition call when they
        // should remain non-urgent.
        startTransition(() => {
          setSavedDraft(saved);
        });
      } catch {
        // Keep failure feedback urgent so it appears promptly.
        setErrorMessage("The draft was not saved. Review it and try again.");
      }
    });
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React Actions</p>
        <h1>Save a draft without blocking the interface</h1>
        <p className="page-intro">
          Start a Transition for the async save, keep the fields responsive,
          and show pending or error feedback near the work.
        </p>
      </header>

      <div className="demo-grid">
        <section className="panel" aria-labelledby="editor-title">
          <p className="eyebrow">Urgent fields + async Action</p>
          <h2 id="editor-title">Draft editor</h2>
          <p className="panel-copy">
            Save the current snapshot. You can keep editing the fields while
            that snapshot is being saved.
          </p>

          <form className="draft-form" onSubmit={handleSubmit}>
            <label htmlFor="draft-title">Title</label>
            <input
              id="draft-title"
              name="title"
              value={title}
              onChange={(event) => setTitle(event.target.value)}
              maxLength={80}
              required
            />

            <label htmlFor="draft-body">Draft</label>
            <textarea
              id="draft-body"
              name="body"
              rows={6}
              value={body}
              onChange={(event) => setBody(event.target.value)}
              required
            />

            <label className="failure-option">
              <input
                type="checkbox"
                checked={failNextSave}
                onChange={(event) => setFailNextSave(event.target.checked)}
              />
              Make the next simulated save fail
            </label>

            <button
              className="button button--primary"
              type="submit"
              disabled={isPending}
            >
              {isPending ? "Saving…" : "Save draft"}
            </button>
          </form>

          <p className="status-line" role="status" aria-live="polite">
            {isPending ? "Saving your draft…" : "Ready to save."}
          </p>
          {errorMessage && (
            <p className="error-message" role="alert">
              {errorMessage}
            </p>
          )}
          <p className="note">
            Only the save button is disabled while saving. The title and draft
            stay editable because their state updates remain urgent.
          </p>
        </section>

        <section
          className="panel saved-panel"
          aria-labelledby="saved-title"
          aria-busy={isPending}
        >
          <p className="eyebrow">Committed result</p>
          <h2 id="saved-title">Last saved draft</h2>
          {savedDraft ? (
            <article className="saved-card" aria-live="polite">
              <h3>{savedDraft.title}</h3>
              <p className="saved-card__body">{savedDraft.body}</p>
              <p className="saved-card__time">
                Saved {formatSavedTime(savedDraft.savedAt)}
              </p>
            </article>
          ) : (
            <p className="empty-state">
              Nothing has been saved yet. Your current edits stay in the form.
            </p>
          )}
          <p className="note">
            The previous successful result stays visible until the next save
            succeeds. A failed save does not replace it.
          </p>
        </section>
      </div>

      <footer className="page-footer">
        The mock service delay represents waiting for an async operation. Do
        not add a timer around a Transition just to make pending UI appear.
      </footer>
    </main>
  );
}

export default function App() {
  return <DraftEditor />;
}
~~~

## src/styles.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  color: #172033;
  background: #f4f7fb;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input,
textarea {
  font: inherit;
}

button:focus-visible,
input:focus-visible,
textarea:focus-visible {
  outline: 3px solid #6c63ff;
  outline-offset: 3px;
}

.page-shell {
  width: min(100% - 2rem, 960px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 720px;
  margin-bottom: 2rem;
}

.eyebrow {
  color: #6256ce;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2,
h3 {
  line-height: 1.2;
}

h1 {
  margin-top: 0.6rem;
  font-size: clamp(2rem, 5vw, 3.3rem);
  letter-spacing: -0.045em;
}

.page-intro,
.panel-copy {
  margin-top: 0.8rem;
  color: #536078;
  line-height: 1.65;
}

.demo-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1.25rem;
  align-items: start;
}

.panel {
  min-width: 0;
  padding: 1.35rem;
  border: 1px solid #d9dfeb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 36px rgb(29 42 70 / 7%);
}

.panel h2 {
  margin-top: 0.45rem;
  font-size: 1.45rem;
}

.draft-form {
  display: grid;
  gap: 0.55rem;
  margin-top: 1.25rem;
}

.draft-form > label:not(.failure-option) {
  margin-top: 0.55rem;
  font-size: 0.9rem;
  font-weight: 700;
}

.draft-form > input,
.draft-form > textarea {
  width: 100%;
  padding: 0.7rem 0.8rem;
  border: 1px solid #bfc8d8;
  border-radius: 0.55rem;
  color: #172033;
  background: #fff;
  line-height: 1.5;
}

.draft-form > textarea {
  min-height: 9rem;
  resize: vertical;
}

.failure-option {
  display: flex;
  gap: 0.55rem;
  align-items: center;
  margin-block: 0.55rem;
  color: #536078;
  font-size: 0.86rem;
  line-height: 1.5;
}

.failure-option input {
  width: 1rem;
  height: 1rem;
  accent-color: #6256ce;
}

.button {
  min-height: 2.7rem;
  padding: 0.6rem 0.9rem;
  border: 1px solid transparent;
  border-radius: 0.55rem;
  cursor: pointer;
  font-weight: 700;
}

.button--primary {
  color: #fff;
  background: #5145b8;
}

.button--primary:hover:not(:disabled) {
  background: #4035a0;
}

.button:disabled {
  cursor: wait;
  opacity: 0.7;
}

.status-line {
  min-height: 1.5rem;
  margin-top: 0.9rem;
  color: #5145b8;
  font-size: 0.88rem;
  font-weight: 700;
}

.error-message {
  margin-top: 0.55rem;
  padding: 0.7rem 0.8rem;
  border: 1px solid #efb4b4;
  border-radius: 0.55rem;
  color: #8f2525;
  background: #fff1f1;
  line-height: 1.5;
}

.saved-panel {
  min-height: 25rem;
}

.saved-panel[aria-busy="true"] {
  border-color: #9a91eb;
}

.saved-card,
.empty-state {
  margin-top: 1.25rem;
  padding: 1rem;
  border: 1px solid #e1e6ef;
  border-radius: 0.7rem;
  background: #fbfcfe;
}

.saved-card h3 {
  font-size: 1.1rem;
}

.saved-card__body {
  margin-top: 0.7rem;
  color: #536078;
  line-height: 1.6;
  white-space: pre-wrap;
}

.saved-card__time {
  margin-top: 1rem;
  color: #68758b;
  font-size: 0.8rem;
}

.empty-state {
  color: #536078;
  line-height: 1.6;
}

.note {
  margin-top: 1rem;
  color: #68758b;
  font-size: 0.82rem;
  line-height: 1.55;
}

.page-footer {
  margin-top: 1.5rem;
  padding: 1rem 1.1rem;
  border-left: 4px solid #6256ce;
  color: #536078;
  background: #ebe9ff;
  line-height: 1.6;
}

@media (max-width: 720px) {
  .page-shell {
    width: min(100% - 1rem, 580px);
    padding-block: 1.5rem;
  }

  .demo-grid {
    grid-template-columns: 1fr;
  }

  .saved-panel {
    min-height: auto;
  }
}
~~~

## Run the experiment

1. Change the title and draft, then select **Save draft**.
2. The save button reports its pending state. The saved panel keeps its previous successful content while the simulated service runs.
3. Keep typing into the fields while saving. The inputs respond immediately because their state is not wrapped in the Transition.
4. When saving finishes, the submitted snapshot appears as the last saved draft. Edits made during the save remain in the form for the next save.
5. Select **Make the next simulated save fail**, then save again. The inline error appears and the last successful draft remains visible.
6. Retry without the failure option to save successfully.

## Why the state updates are placed this way

The title and body setters are called directly from input events. They must be urgent so controlled fields reflect every keystroke.

The save handler takes a snapshot before starting the Action. This makes it clear which values the user submitted, even if they continue editing while the service is pending. The button is disabled only to prevent a duplicate save in this example.

The async Action awaits **persistDraft**. After the await, it wraps **setSavedDraft** in a nested **startTransition** because current React does not automatically mark that update as a Transition. The error state is set in the catch block without that wrapper so failure feedback can appear urgently.

The timer is inside **persistDraft**, which stands in for an asynchronous service call. It is not a timer around **startTransition**. Replace **persistDraft** with the real service function in an application, and handle request ordering if multiple operations may overlap.

## Reusable Action callback pattern

A reusable button can accept a synchronous or asynchronous callback and await it within its Action:

~~~jsx
function ActionButton({ action, children }) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      type="button"
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await action();
        });
      }}
    >
      {isPending ? "Working…" : children}
    </button>
  );
}
~~~

Awaiting the callback keeps the Transition pending for its asynchronous work. Disable the button only when repeating that particular operation would be confusing or unsafe.

## What this example does not do

- It does not make a network request; the service function is local.
- It does not make synchronous JavaScript calculations run in the background.
- It does not cancel requests or resolve overlapping request races.
- It does not use a Transition to control the fields’ values.
