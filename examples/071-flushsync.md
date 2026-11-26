# Synchronous DOM Updates with flushSync - Example

This small Vite app compares a normal React state update with an update forced through `flushSync`. Both buttons change the same React-managed message. Each handler immediately reads the message element's DOM text before the handler finishes:

- The normal update demonstrates why an immediate DOM read may still see the previous commit.
- The `flushSync` update commits before the read, so the DOM contains the new text.

This intentionally uses a DOM read to demonstrate an integration boundary. In ordinary application logic, prefer using the state value instead of reading it back from the DOM.

## Create a Vite project

Create a React app and install its dependencies:

~~~sh
npm create vite@latest flushsync-demo -- --template react
cd flushsync-demo
npm install
~~~

Replace or add these files and keep the generated Vite configuration.

## Project files

~~~text
flushsync-demo/
├── index.html
└── src/
    ├── App.jsx
    ├── main.jsx
    └── styles.css
~~~

## `index.html`

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>flushSync demonstration</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## `src/main.jsx`

~~~jsx
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The #root container was not found.");
}

createRoot(container).render(<App />);
~~~

The root is created once in the entry module. The example uses `createRoot` because its page starts with an empty container.

## `src/App.jsx`

~~~jsx
import { useRef, useState } from "react";
import { flushSync } from "react-dom";

export default function App() {
  const messageRef = useRef(null);
  const [message, setMessage] = useState("Initial message");
  const [observation, setObservation] = useState(
    "Choose an update to compare its DOM timing.",
  );

  function handleNormalUpdate() {
    setMessage("Updated with normal scheduling");

    const currentDomText = messageRef.current?.textContent ?? "(no DOM text)";
    setObservation("Immediate DOM read before this handler ends: " + currentDomText);
  }

  function handleSynchronousUpdate() {
    flushSync(() => {
      setMessage("Updated inside flushSync");
    });

    const currentDomText = messageRef.current?.textContent ?? "(no DOM text)";
    setObservation("DOM read after flushSync returns: " + currentDomText);
  }

  function handleReset() {
    setMessage("Initial message");
    setObservation("Choose an update to compare its DOM timing.");
  }

  return (
    <main className="demo">
      <p className="eyebrow">React DOM scheduling</p>
      <h1>Compare update timing</h1>
      <p>
        The next line reads the displayed DOM text immediately inside each
        button handler.
      </p>

      <section className="output" aria-labelledby="message-heading">
        <h2 id="message-heading">React-managed message</h2>
        <p ref={messageRef} className="message" aria-live="polite">
          {message}
        </p>
      </section>

      <p className="observation" role="status">
        {observation}
      </p>

      <div className="actions">
        <button type="button" onClick={handleNormalUpdate}>
          Update normally
        </button>
        <button type="button" onClick={handleSynchronousUpdate}>
          Update with flushSync
        </button>
        <button type="button" className="secondary" onClick={handleReset}>
          Reset
        </button>
      </div>

      <p className="note">
        This example reads the DOM only to show commit timing. Prefer React
        state for normal application logic.
      </p>
    </main>
  );
}
~~~

After selecting **Update normally**, the observation records the old DOM text because React has not committed the queued update at the point of that read. When the event handler finishes, React updates the message. After selecting **Update with flushSync**, React commits the message before the immediate DOM read, so the observation contains the new text.

The observation itself is updated outside the `flushSync` callback. It does not need to be read synchronously by the integration; React can render that status normally.

## `src/styles.css`

~~~css
:root {
  font-family: system-ui, sans-serif;
  color: #172033;
  background: #f2f5fa;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
  display: grid;
  place-items: center;
  padding: 1.5rem;
}

button {
  padding: 0.7rem 1rem;
  border: 0;
  border-radius: 0.55rem;
  color: white;
  background: #234ec5;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button:hover {
  background: #193c9e;
}

button:focus-visible {
  outline: 3px solid #e7a900;
  outline-offset: 3px;
}

.demo {
  width: min(100%, 44rem);
  padding: clamp(1.25rem, 5vw, 2.5rem);
  border: 1px solid #d6deea;
  border-radius: 1rem;
  background: white;
  box-shadow: 0 1rem 2.5rem rgb(23 32 51 / 10%);
}

.eyebrow {
  margin-block: 0 0.5rem;
  color: #52637e;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1 {
  margin-block: 0 1rem;
  font-size: clamp(1.6rem, 5vw, 2.4rem);
}

.output {
  margin-block: 1.5rem;
  padding: 1rem;
  border: 1px solid #d6deea;
  border-radius: 0.7rem;
  background: #f8faff;
}

.output h2 {
  margin-block: 0 0.5rem;
  font-size: 1rem;
}

.message {
  margin-block: 0;
  font-weight: 700;
}

.observation {
  min-height: 3rem;
  padding: 0.8rem 1rem;
  border-left: 4px solid #234ec5;
  background: #eef3ff;
  overflow-wrap: anywhere;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.65rem;
  margin-top: 1.25rem;
}

button.secondary {
  color: #172033;
  background: #e6ebf3;
}

button.secondary:hover {
  background: #d6deea;
}

.note {
  margin-block: 1.5rem 0;
  color: #52637e;
  font-size: 0.9rem;
  line-height: 1.5;
}
~~~

## Run the example

Start the development server:

~~~sh
npm run dev
~~~

Open the local Vite page and try the three buttons:

1. Select **Update normally**. The observation reads the prior message before React commits the queued update; the displayed message changes after the handler ends.
2. Select **Update with flushSync**. React commits the message inside the callback, so the observation reads the new text immediately afterward.
3. Select **Reset** to return to the initial state.

This is a demonstration of timing, not a recommendation to read React-owned text from the DOM in normal application code.

## Browser callback example: prepare print content

A browser callback may need a React update committed before it returns. This optional component demonstrates the `beforeprint` integration pattern. Its listener is installed and removed by an Effect; `flushSync` is called only when the browser invokes the callback.

~~~jsx
import { useEffect, useState } from "react";
import { flushSync } from "react-dom";

function PrintStatus() {
  const [isPrinting, setIsPrinting] = useState(false);

  useEffect(() => {
    function handleBeforePrint() {
      flushSync(() => {
        setIsPrinting(true);
      });
    }

    function handleAfterPrint() {
      setIsPrinting(false);
    }

    window.addEventListener("beforeprint", handleBeforePrint);
    window.addEventListener("afterprint", handleAfterPrint);

    return () => {
      window.removeEventListener("beforeprint", handleBeforePrint);
      window.removeEventListener("afterprint", handleAfterPrint);
    };
  }, []);

  return <p>Print mode: {isPrinting ? "on" : "off"}</p>;
}
~~~

For simple print-only styling, a `@media print` stylesheet is often enough and avoids a state update entirely.

## Common mistakes

- Importing `flushSync` from `react-dom/client` instead of `react-dom`.
- Wrapping every state update in `flushSync` even though no synchronous DOM read follows it.
- Expecting the state variable in the current render to change immediately.
- Calling `flushSync` from an Effect, render function, or class lifecycle method.
- Putting the DOM read inside the callback instead of after it returns.
- Forgetting that pending work or a Suspense fallback may also be flushed.
- Treating the normal update's previous-text observation as a bug; it is the expected result of reading before React commits.

## Key takeaway

Use `flushSync` only when a synchronous integration needs React's updated DOM before the current callback returns. Keep the callback small, read the DOM afterward, and let React schedule ordinary UI updates normally.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="070-react-dom-roots-and-client-apis.md">070 - React DOM Roots, Hydration, and Client APIs - Example</a></td><td><a href="../lessons/071-flushsync.md">Lesson</a></td><td><a href="../qnas/071-flushsync.md">Q&amp;A</a></td><td><a href="072-react-dom-resource-hints-and-browser-apis.md">072 - React DOM Resource Hints and Browser APIs - Example</a></td></tr>
  </tbody>
</table>
