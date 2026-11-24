# Controlled and Uncontrolled Component APIs — Example

This example demonstrates three ownership patterns:

- A controlled Disclosure receives its open state from its parent. The parent can open or close it externally.
- An uncontrolled Disclosure initializes its own state from defaultOpen. It reports changes to the parent, but the parent does not supply its current open value.
- A form combines a controlled name field with an uncontrolled nickname field. The name is needed for a live preview; the nickname is only read at submit time.

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
import { useId, useState } from "react";

function Disclosure({
  isOpen,
  defaultOpen = false,
  onOpenChange,
  title,
  children,
}) {
  const generatedId = useId();
  const isControlled = isOpen !== undefined;
  const [internalIsOpen, setInternalIsOpen] = useState(defaultOpen);
  const open = isControlled ? isOpen : internalIsOpen;

  function requestOpenChange(nextOpen) {
    if (!isControlled) {
      setInternalIsOpen(nextOpen);
    }

    onOpenChange?.(nextOpen);
  }

  return (
    <section className="disclosure">
      <h3>
        <button
          className="disclosure-toggle"
          type="button"
          aria-expanded={open}
          aria-controls={generatedId}
          onClick={() => requestOpenChange(!open)}
        >
          <span>{title}</span>
          <span aria-hidden="true">{open ? "−" : "+"}</span>
        </button>
      </h3>

      <div className="disclosure-content" id={generatedId} hidden={!open}>
        {children}
      </div>
    </section>
  );
}

function ControlledDisclosureDemo() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <section className="card" aria-labelledby="controlled-heading">
      <p className="eyebrow">Parent-owned value</p>
      <h2 id="controlled-heading">Controlled disclosure</h2>
      <p className="section-copy">
        The parent owns isOpen. The disclosure reports toggle requests, and the
        parent updates the value passed back to it.
      </p>

      <div className="button-row">
        <button
          className="secondary-button"
          type="button"
          onClick={() => setIsOpen(true)}
          disabled={isOpen}
        >
          Open from parent
        </button>
        <button
          className="secondary-button"
          type="button"
          onClick={() => setIsOpen(false)}
          disabled={!isOpen}
        >
          Close from parent
        </button>
      </div>

      <Disclosure
        isOpen={isOpen}
        onOpenChange={setIsOpen}
        title="Toggle this controlled content"
      >
        <p>
          This content is controlled by the isOpen prop. Clicking the disclosure
          calls onOpenChange; the parent then changes the value.
        </p>
      </Disclosure>

      <p className="status-line">
        Parent state: <strong>{isOpen ? "open" : "closed"}</strong>
      </p>
    </section>
  );
}

function UncontrolledDisclosureDemo() {
  const [lastNotification, setLastNotification] = useState("No interaction yet");

  return (
    <section className="card" aria-labelledby="uncontrolled-heading">
      <p className="eyebrow">Component-owned value</p>
      <h2 id="uncontrolled-heading">Uncontrolled disclosure</h2>
      <p className="section-copy">
        The disclosure starts closed and manages its own open state. The parent
        only receives a notification when the user toggles it.
      </p>

      <Disclosure
        defaultOpen={false}
        onOpenChange={(nextOpen) =>
          setLastNotification(nextOpen ? "Opened" : "Closed")
        }
        title="Toggle this uncontrolled content"
      >
        <p>
          The component owns this open value. The callback reports each request
          without making the parent the source of truth.
        </p>
      </Disclosure>

      <p className="status-line">
        Last notification: <strong>{lastNotification}</strong>
      </p>
    </section>
  );
}

function MixedOwnershipForm() {
  const [displayName, setDisplayName] = useState("");
  const [submittedNickname, setSubmittedNickname] = useState(null);

  function handleSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);
    const nickname = formData.get("nickname");
    setSubmittedNickname(typeof nickname === "string" ? nickname : "");
  }

  function handleReset() {
    setDisplayName("");
    setSubmittedNickname(null);
  }

  return (
    <section className="card" aria-labelledby="form-heading">
      <p className="eyebrow">Mixed ownership</p>
      <h2 id="form-heading">Controlled and uncontrolled fields</h2>
      <p className="section-copy">
        The controlled name drives a live preview. The nickname is uncontrolled
        and is read with FormData when the form is submitted.
      </p>

      <form
        className="profile-form"
        onSubmit={handleSubmit}
        onReset={handleReset}
      >
        <label className="field">
          <span>Display name (controlled)</span>
          <input
            name="displayName"
            value={displayName}
            onChange={(event) => setDisplayName(event.target.value)}
            placeholder="Type a display name"
          />
        </label>

        <label className="field">
          <span>Nickname (uncontrolled)</span>
          <input
            name="nickname"
            defaultValue="Rae"
            placeholder="Enter a nickname"
          />
        </label>

        <div className="button-row">
          <button className="primary-button" type="submit">
            Read nickname
          </button>
          <button className="secondary-button" type="reset">
            Reset form
          </button>
        </div>
      </form>

      <div className="form-output" aria-live="polite">
        <p>
          Live display name:{" "}
          <strong>{displayName || "No name entered"}</strong>
        </p>
        {submittedNickname !== null && (
          <p>
            Nickname read at submit:{" "}
            <strong>{submittedNickname || "Empty"}</strong>
          </p>
        )}
      </div>
    </section>
  );
}

export default function App() {
  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React component APIs</p>
        <h1>Who owns the current value?</h1>
        <p className="intro">
          Compare parent-controlled values with component-owned values. A
          callback can report an interaction in either design.
        </p>
      </header>

      <div className="demo-grid">
        <ControlledDisclosureDemo />
        <UncontrolledDisclosureDemo />
      </div>

      <MixedOwnershipForm />
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
input {
  font: inherit;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #82aaff;
  outline-offset: 3px;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.page {
  width: min(100% - 2rem, 1080px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  max-width: 700px;
  margin-bottom: 1.5rem;
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
  font-size: clamp(2rem, 6vw, 3.2rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0;
  font-size: 1.2rem;
}

.intro,
.section-copy {
  color: #526071;
  line-height: 1.65;
}

.intro {
  max-width: 64ch;
  margin-bottom: 0;
}

.demo-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  align-items: start;
  gap: 1rem;
  margin-bottom: 1rem;
}

.card {
  min-width: 0;
  padding: clamp(1rem, 3vw, 1.4rem);
  border: 1px solid #d8e0eb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 35px rgb(32 53 84 / 7%);
}

.card h2 {
  margin-bottom: 0.65rem;
}

.section-copy {
  margin-bottom: 1rem;
  font-size: 0.94rem;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-block: 0.9rem;
}

.primary-button,
.secondary-button {
  padding: 0.65rem 0.85rem;
  border-radius: 0.55rem;
  cursor: pointer;
}

.primary-button {
  border: 1px solid #315db5;
  color: #fff;
  background: #315db5;
  font-weight: 700;
}

.primary-button:hover:not(:disabled) {
  background: #244b9a;
}

.secondary-button {
  border: 1px solid #bac8dc;
  color: #244578;
  background: #f7f9fc;
}

.secondary-button:hover:not(:disabled) {
  background: #edf3fb;
}

.disclosure {
  overflow: hidden;
  border: 1px solid #dce4ef;
  border-radius: 0.7rem;
}

.disclosure h3 {
  margin: 0;
  font-size: 1rem;
}

.disclosure-toggle {
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.9rem;
  border: 0;
  color: #18212f;
  background: #fff;
  font-weight: 700;
  text-align: left;
  cursor: pointer;
}

.disclosure-toggle:hover {
  background: #f5f8fc;
}

.disclosure-toggle span:last-child {
  color: #315db5;
  font-size: 1.2rem;
}

.disclosure-content {
  padding: 0 0.9rem 0.9rem;
  color: #526071;
  line-height: 1.6;
}

.disclosure-content p {
  margin: 0;
}

.status-line {
  margin: 0.8rem 0 0;
  color: #637185;
  font-size: 0.9rem;
}

.status-line strong {
  color: #344257;
}

.profile-form {
  display: grid;
  gap: 0.9rem;
}

.field {
  display: grid;
  gap: 0.45rem;
  font-size: 0.92rem;
  font-weight: 650;
}

.field input {
  width: 100%;
  padding: 0.7rem 0.8rem;
  border: 1px solid #bac8dc;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
}

.profile-form .button-row {
  margin: 0;
}

.form-output {
  margin-top: 1rem;
  padding: 0.9rem;
  border-radius: 0.65rem;
  color: #344257;
  background: #f4f7fc;
  line-height: 1.6;
  overflow-wrap: anywhere;
}

.form-output p {
  margin: 0;
}

.form-output p + p {
  margin-top: 0.4rem;
}

@media (max-width: 720px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .demo-grid {
    grid-template-columns: 1fr;
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
      content="A React example comparing controlled and uncontrolled component APIs."
    />
    <title>Controlled and Uncontrolled — React Example</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How the example works

### Controlled disclosure

ControlledDisclosureDemo owns isOpen and passes it to Disclosure. The parent buttons can open or close the content without using the disclosure toggle.

When the disclosure toggle is activated, Disclosure calls onOpenChange with the requested next boolean. The parent updates isOpen, then passes the new value back. Disclosure does not keep a competing internal open state for this instance.

### Uncontrolled disclosure with a notification

UncontrolledDisclosureDemo passes defaultOpen but does not pass isOpen. Disclosure initializes internal state from defaultOpen and owns later changes.

The parent receives onOpenChange notifications and displays the last one. That notification is a different fact from the current open value, so it does not make the disclosure controlled.

### Mixed form

The display name is controlled by React state because it drives a live preview. The nickname uses defaultValue; the browser owns its current value until submission. The form reads the nickname using FormData when submitted.

The reset button resets the uncontrolled native field to its default and the onReset handler clears the controlled display name and submission result.

## Run it

From the Vite project root, install the project's dependencies if needed and start its development server:

~~~sh
npm install
npm run dev
~~~

Open the local address printed by Vite.

- Use the parent buttons to open and close the controlled disclosure.
- Toggle either disclosure and compare how the values are owned.
- Type in both form fields. The display name preview changes immediately; the nickname is reported when you submit.
- Reset the form and observe both fields return to their intended initial values.

## What to notice

- The controlled disclosure always renders the parent's isOpen prop.
- The uncontrolled disclosure owns internal state initialized from defaultOpen.
- Both components can call onOpenChange, but the callback does not decide ownership.
- The form demonstrates that one interface can mix controlled and uncontrolled fields.
- The current value has one owner in each case, avoiding a prop and internal state competing for the same fact.
