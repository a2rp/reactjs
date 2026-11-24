# Lifting State Up and Sharing Data — Example

This example shows two related patterns:

1. An editor and a preview are siblings. Their shared message is owned by App.
2. An accordion contains several panels. Its active panel ID is owned by Accordion, the closest common parent of those panels.

In both cases, the owner passes values down as props. A child reports a user action through a callback, and the owner updates state.

## Project files

In a Vite React project, use these files:

~~~text
index.html
src/
  App.jsx
  main.jsx
  styles.css
~~~

## src/App.jsx

~~~jsx
import { useState } from "react";

const accordionItems = [
  {
    id: "ownership",
    title: "Who owns shared state?",
    content:
      "The closest common parent that needs to coordinate the consumers owns the shared value.",
  },
  {
    id: "flow",
    title: "How does an interaction update it?",
    content:
      "A child calls a callback prop. The parent handles the request, updates state, and passes the new value down.",
  },
  {
    id: "local",
    title: "Should every state value be lifted?",
    content:
      "No. Keep independent state local. Lift only values that multiple components need to share or coordinate.",
  },
];

function MessageEditor({ message, onMessageChange }) {
  return (
    <label className="field">
      <span className="field-label">Edit the shared message</span>
      <textarea
        value={message}
        onChange={(event) => onMessageChange(event.target.value)}
        rows={5}
      />
      <span className="field-help">
        The textarea value comes from parent state.
      </span>
    </label>
  );
}

function MessagePreview({ message }) {
  const displayedMessage = message.trim()
    ? message
    : "Your preview will appear here.";

  return (
    <section className="preview" aria-labelledby="preview-heading">
      <div className="preview-heading-row">
        <h3 id="preview-heading">Live preview</h3>
        <span className="character-count">
          {message.length} {message.length === 1 ? "character" : "characters"}
        </span>
      </div>
      <p className="preview-message" aria-live="polite">
        {displayedMessage}
      </p>
    </section>
  );
}

function AccordionPanel({ id, title, isOpen, onToggle, children }) {
  const headingId = "accordion-" + id + "-heading";
  const panelId = "accordion-" + id + "-panel";

  return (
    <section className="accordion-panel" aria-labelledby={headingId}>
      <h3 className="accordion-heading" id={headingId}>
        <button
          className="accordion-toggle"
          type="button"
          aria-expanded={isOpen}
          aria-controls={panelId}
          onClick={onToggle}
        >
          <span>{title}</span>
          <span aria-hidden="true">{isOpen ? "−" : "+"}</span>
        </button>
      </h3>

      <div className="accordion-content" id={panelId} hidden={!isOpen}>
        <p>{children}</p>
      </div>
    </section>
  );
}

function Accordion() {
  const [activePanelId, setActivePanelId] = useState("ownership");

  function handleToggle(panelId) {
    setActivePanelId((currentId) =>
      currentId === panelId ? null : panelId
    );
  }

  return (
    <div className="accordion">
      {accordionItems.map((item) => (
        <AccordionPanel
          key={item.id}
          id={item.id}
          title={item.title}
          isOpen={activePanelId === item.id}
          onToggle={() => handleToggle(item.id)}
        >
          {item.content}
        </AccordionPanel>
      ))}
    </div>
  );
}

export default function App() {
  const [message, setMessage] = useState(
    "One parent value keeps the editor and preview in sync."
  );

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React state sharing</p>
        <h1>Lifting state up</h1>
        <p className="intro">
          Shared values live in a common parent. Children display props and
          report interactions through callbacks.
        </p>
      </header>

      <section className="card" aria-labelledby="shared-message-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Example 1</p>
            <h2 id="shared-message-heading">Editor and preview</h2>
          </div>
          <button
            className="secondary-button"
            type="button"
            onClick={() => setMessage("")}
          >
            Clear message
          </button>
        </div>

        <div className="editor-preview-grid">
          <MessageEditor
            message={message}
            onMessageChange={setMessage}
          />
          <MessagePreview message={message} />
        </div>

        <p className="explanation">
          App owns message. MessageEditor receives message and reports edits
          with onMessageChange. MessagePreview receives the same message, so
          both siblings stay in sync.
        </p>
      </section>

      <section className="card" aria-labelledby="accordion-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Example 2</p>
            <h2 id="accordion-heading">One open panel at a time</h2>
          </div>
        </div>

        <Accordion />

        <p className="explanation">
          Accordion owns one activePanelId value and derives each panel's
          isOpen prop from it. Selecting a different panel closes the previous
          one. Selecting the open panel closes it.
        </p>
      </section>
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
  color: #18212f;
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
textarea {
  font: inherit;
}

.page {
  width: min(100% - 2rem, 960px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  max-width: 680px;
  margin-bottom: 2rem;
}

.eyebrow {
  margin: 0 0 0.45rem;
  color: #315db5;
  font-size: 0.76rem;
  font-weight: 750;
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
  font-size: clamp(2rem, 6vw, 3.3rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0;
  font-size: 1.35rem;
}

h3 {
  margin-bottom: 0;
}

.intro {
  max-width: 62ch;
  margin-bottom: 0;
  color: #526071;
  line-height: 1.65;
}

.card {
  margin-top: 1.25rem;
  padding: clamp(1rem, 3vw, 1.5rem);
  border: 1px solid #d8e0eb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 35px rgb(32 53 84 / 7%);
}

.section-heading,
.preview-heading-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.section-heading {
  margin-bottom: 1.25rem;
}

.secondary-button {
  flex: 0 0 auto;
  padding: 0.6rem 0.8rem;
  border: 1px solid #bac8dc;
  border-radius: 0.55rem;
  color: #244578;
  background: #f7f9fc;
  cursor: pointer;
}

.secondary-button:hover {
  background: #edf3fb;
}

.secondary-button:focus-visible,
.accordion-toggle:focus-visible,
textarea:focus-visible {
  outline: 3px solid #82aaff;
  outline-offset: 3px;
}

.editor-preview-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
}

.field,
.preview {
  min-width: 0;
  padding: 1rem;
  border: 1px solid #dce4ef;
  border-radius: 0.75rem;
  background: #fbfcfe;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 0.65rem;
}

.field-label {
  font-weight: 700;
}

textarea {
  width: 100%;
  min-height: 8rem;
  resize: vertical;
  padding: 0.75rem;
  border: 1px solid #b9c6d8;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
  line-height: 1.5;
}

.field-help,
.character-count {
  color: #637185;
  font-size: 0.85rem;
}

.preview-heading-row {
  align-items: baseline;
  margin-bottom: 0.75rem;
}

.preview-message {
  min-height: 5rem;
  margin-bottom: 0;
  overflow-wrap: anywhere;
  color: #344257;
  line-height: 1.65;
  white-space: pre-wrap;
}

.explanation {
  margin: 1rem 0 0;
  color: #526071;
  font-size: 0.94rem;
  line-height: 1.65;
}

.accordion {
  overflow: hidden;
  border: 1px solid #dce4ef;
  border-radius: 0.75rem;
}

.accordion-panel + .accordion-panel {
  border-top: 1px solid #dce4ef;
}

.accordion-heading {
  font-size: 1rem;
}

.accordion-toggle {
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem;
  border: 0;
  color: #18212f;
  background: #fff;
  font-weight: 700;
  text-align: left;
  cursor: pointer;
}

.accordion-toggle:hover {
  background: #f5f8fc;
}

.accordion-toggle span:last-child {
  color: #315db5;
  font-size: 1.25rem;
}

.accordion-content {
  padding: 0 1rem 1rem;
  color: #526071;
  line-height: 1.65;
}

.accordion-content p {
  margin-bottom: 0;
}

@media (max-width: 640px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .editor-preview-grid {
    grid-template-columns: 1fr;
  }

  .section-heading {
    align-items: flex-start;
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
      content="A React example of lifting state up and sharing data between components."
    />
    <title>Lifting State Up — React Example</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How the example works

### The shared message

App is the closest common parent of MessageEditor and MessagePreview, so it owns message.

- App passes message to both siblings.
- MessageEditor renders the prop as the textarea value.
- When the user types, MessageEditor calls onMessageChange with the new text.
- App updates its state.
- React renders both siblings with the updated message.
- The preview's character count is calculated from the prop; it is not stored as a second state value.
- The Clear message button updates the same parent state, so the editor and preview both change.

### The accordion

Accordion is the closest common parent of its panels, so it owns activePanelId.

- The parent compares activePanelId with each item ID and passes the result as isOpen.
- AccordionPanel renders from isOpen and calls onToggle when its button is activated.
- Accordion handles the request and updates its single active ID.
- The previous panel closes when another is selected.
- Activating the open panel sets the ID to null, so all panels close.
- The button's aria-expanded value reflects the same state as the visible panel.

AccordionPanel does not keep its own isOpen state. It is controlled by the Accordion parent. The message editor and preview also do not store duplicate message state.

## Run it

From the Vite project root, install the project's dependencies if needed and start its development server:

~~~sh
npm install
npm run dev
~~~

Open the local address printed by Vite. Edit the message and confirm the preview and character count update. Clear it and confirm both siblings reflect the empty value. Open a second accordion panel and confirm the previous panel closes; activate the open panel again and confirm it closes.

## What to notice

The example uses two different state owners because it has two independent coordination boundaries:

- App owns the message shared by the editor and preview.
- Accordion owns the active panel ID shared by its panels.

This keeps each piece of state at the nearest level that needs it. The values flow down through props, and interactions flow back through callbacks.
