# Example: Effect Dependencies, Closures, and Synchronization

This runnable Vite example connects to a small local mock connection. It demonstrates that:

- changing the room or server key re-synchronizes the connection;
- an unrelated component render does not restart it;
- the connection callback appends messages with a functional state updater;
- messages are filtered during render, without duplicate filtered state;
- Effect callbacks keep the values from the render that created their setup.

The mock uses a browser timer only. It does not contact a server, make network requests, or persist data.

## Project files

~~~text
effect-dependencies-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

Use these files in a React project created with Vite. From the project root, install dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f3f6fb" />
    <title>Effect Dependencies Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

const rootElement = document.getElementById("root");

if (!rootElement) {
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

Strict Mode is enabled. In development, the first connection may show an additional connect/disconnect cycle while React checks that cleanup mirrors setup.

## src/App.jsx

~~~jsx
import { useEffect, useState } from "react";

let nextConnectionId = 0;

function createMockConnection(serverKey, roomId) {
  const connectionId = nextConnectionId + 1;
  nextConnectionId = connectionId;
  let timerId = null;
  let messageNumber = 0;

  return {
    connect(onMessage) {
      console.info("[demo] connect", { connectionId, serverKey, roomId });

      timerId = window.setInterval(() => {
        messageNumber += 1;

        onMessage({
          id: connectionId + "-" + messageNumber,
          roomId,
          serverKey,
          text:
            "Mock message " +
            messageNumber +
            " from room " +
            roomId +
            " on server " +
            serverKey,
        });
      }, 1800);
    },

    disconnect() {
      window.clearInterval(timerId);
      console.info("[demo] disconnect", { connectionId, serverKey, roomId });
    },
  };
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  const [serverKey, setServerKey] = useState("blue");
  const [renderPulse, setRenderPulse] = useState(0);
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createMockConnection(serverKey, roomId);

    connection.connect((message) => {
      setMessages((currentMessages) => [message, ...currentMessages].slice(0, 12));
    });

    return () => connection.disconnect();
  }, [serverKey, roomId]);

  const currentMessages = messages.filter(
    (message) =>
      message.roomId === roomId && message.serverKey === serverKey
  );

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React Hooks · chapter 053</p>
        <h1>Dependencies keep synchronization in step with renders</h1>
        <p className="intro">
          Change a connection input to restart the mock connection. Trigger an
          unrelated render to see that unchanged dependencies leave it alone.
        </p>
      </header>

      <section className="card controls" aria-labelledby="connection-heading">
        <div>
          <p className="eyebrow">One Effect · two reactive inputs</p>
          <h2 id="connection-heading">Connection settings</h2>
        </div>

        <label className="field">
          <span>Room</span>
          <select value={roomId} onChange={(event) => setRoomId(event.target.value)}>
            <option value="general">General</option>
            <option value="music">Music</option>
            <option value="travel">Travel</option>
          </select>
        </label>

        <label className="field">
          <span>Server key</span>
          <input
            value={serverKey}
            onChange={(event) => setServerKey(event.target.value)}
          />
        </label>

        <div className="unrelated-control">
          <p>
            Unrelated render value: <strong>{renderPulse}</strong>
          </p>
          <button
            className="button-secondary"
            type="button"
            onClick={() => setRenderPulse((value) => value + 1)}
          >
            Trigger unrelated render
          </button>
        </div>

        <p className="hint">
          Open the browser console. Changing Room or Server key logs a
          disconnect followed by a new connection. Triggering an unrelated
          render should not log a reconnect.
        </p>
      </section>

      <section className="card feed" aria-labelledby="feed-heading">
        <div className="feed-heading">
          <div>
            <p className="eyebrow">Callback closure · state updater</p>
            <h2 id="feed-heading">Messages for {roomId}</h2>
          </div>
          <span className="connection-badge">Mock connection</span>
        </div>

        {currentMessages.length > 0 ? (
          <ul className="message-list">
            {currentMessages.map((message) => (
              <li key={message.id}>
                <span>{message.text}</span>
                <span className="message-meta">Server: {message.serverKey}</span>
              </li>
            ))}
          </ul>
        ) : (
          <p className="empty-state">
            Waiting for the first mock message from this connection…
          </p>
        )}
      </section>

      <section className="card explanation" aria-labelledby="reasoning-heading">
        <p className="eyebrow">What the code demonstrates</p>
        <h2 id="reasoning-heading">Why these dependencies?</h2>
        <ul>
          <li>
            The Effect reads roomId and serverKey, so both values are listed.
          </li>
          <li>
            The connection factory is outside the component, so it is not
            recreated by rendering.
          </li>
          <li>
            The callback uses a functional state updater, so it does not read
            the messages snapshot from setup.
          </li>
          <li>
            renderPulse is not read by the Effect, so changing it does not
            restart the connection.
          </li>
          <li>
            The visible message list is derived from messages during render;
            there is no Effect that copies or filters it into another state.
          </li>
        </ul>
      </section>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #182236;
  background: #f3f6fb;
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
input,
select {
  font: inherit;
}

button {
  min-height: 44px;
  border: 0;
  border-radius: 0.65rem;
  padding: 0.65rem 1rem;
  color: #ffffff;
  background: #285bd2;
  cursor: pointer;
}

button:hover {
  background: #1f49ad;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #e59c00;
  outline-offset: 3px;
}

.page {
  display: grid;
  gap: 1rem;
  width: min(900px, calc(100% - 2rem));
  margin: 0 auto;
  padding: 3rem 0;
}

.page-header {
  max-width: 760px;
  margin-bottom: 0.75rem;
}

.eyebrow {
  margin-bottom: 0.55rem;
  color: #285bd2;
  font-size: 0.78rem;
  font-weight: 750;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  max-width: 720px;
  font-size: clamp(2rem, 5vw, 3.35rem);
  letter-spacing: -0.04em;
}

h2 {
  font-size: 1.3rem;
}

.intro {
  max-width: 700px;
  margin-top: 1rem;
  color: #526078;
  font-size: 1.05rem;
  line-height: 1.65;
}

.card {
  border: 1px solid #dce3ee;
  border-radius: 1rem;
  padding: 1.35rem;
  background: #ffffff;
  box-shadow: 0 10px 30px rgb(30 48 80 / 6%);
}

.controls {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
}

.controls > div:first-child,
.unrelated-control,
.hint {
  grid-column: 1 / -1;
}

.field {
  display: grid;
  gap: 0.45rem;
  color: #35425a;
  font-weight: 650;
}

.field input,
.field select {
  width: 100%;
  min-height: 44px;
  border: 1px solid #aebbd0;
  border-radius: 0.65rem;
  padding: 0.65rem 0.8rem;
  color: #182236;
  background: #ffffff;
}

.unrelated-control {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
  border-top: 1px solid #e2e8f1;
  padding-top: 1rem;
}

.unrelated-control p {
  color: #35425a;
}

.unrelated-control strong {
  font-variant-numeric: tabular-nums;
}

.button-secondary {
  color: #25344e;
  background: #e8edf6;
}

.button-secondary:hover {
  background: #d9e2f0;
}

.hint {
  color: #58657a;
  line-height: 1.6;
}

.feed-heading {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
}

.connection-badge {
  border: 1px solid #b9ddca;
  border-radius: 999px;
  padding: 0.4rem 0.7rem;
  color: #17643b;
  background: #eef9f2;
  font-size: 0.85rem;
  font-weight: 650;
}

.message-list {
  display: grid;
  gap: 0.65rem;
  list-style: none;
}

.message-list li {
  display: grid;
  gap: 0.25rem;
  border-radius: 0.65rem;
  padding: 0.8rem;
  color: #26344d;
  background: #f3f6fb;
  line-height: 1.5;
}

.message-meta {
  color: #64718a;
  font-size: 0.85rem;
}

.empty-state {
  border-radius: 0.65rem;
  padding: 1rem;
  color: #58657a;
  background: #f3f6fb;
}

.explanation ul {
  display: grid;
  gap: 0.65rem;
  margin: 1rem 0 0 1.2rem;
  color: #46536b;
  line-height: 1.6;
}

@media (max-width: 620px) {
  .page {
    padding: 2rem 0;
  }

  .controls {
    grid-template-columns: 1fr;
  }

  .controls > div:first-child,
  .unrelated-control,
  .hint {
    grid-column: 1;
  }

  .unrelated-control {
    align-items: flex-start;
    flex-direction: column;
  }
}
~~~

## How to explore the example

1. Wait for mock messages to appear. Each message includes the room and server key captured by the active connection.
2. Change the room. The old connection is cleaned up and a new one starts with the new room value.
3. Change the server key. The connection is replaced so its callback reports the updated key.
4. Trigger an unrelated render. The component renders again, but neither dependency changed, so the connection remains active.
5. Watch the console in development. Strict Mode may show an initial extra connection and disconnection. After that check, room and server-key changes should pair each disconnect with a new connect.

## A stale-closure mistake

If the message callback read the messages array from the setup render, it could overwrite newer messages:

~~~jsx
useEffect(() => {
  const connection = createMockConnection(serverKey, roomId);

  connection.connect((message) => {
    setMessages([...messages, message]);
  });

  return () => connection.disconnect();
}, [serverKey, roomId]);
~~~

The callback closes over the messages array from the render that started the connection. The linter correctly reports that messages is missing. Adding messages to the dependency list would make each received message reconnect the mock connection.

The runnable example instead uses a functional updater:

~~~jsx
setMessages((currentMessages) => [message, ...currentMessages].slice(0, 12));
~~~

The updater receives the latest state when React processes it. The Effect no longer reads the messages snapshot, so messages is not a dependency.

## An unstable-object mistake

Creating an options object during rendering and depending on it would reconnect after every render:

~~~jsx
const options = { serverKey, roomId };

useEffect(() => {
  const connection = createMockConnection(options.serverKey, options.roomId);
  connection.connect((message) => {
    setMessages((currentMessages) => [...currentMessages, message]);
  });
  return () => connection.disconnect();
}, [options]);
~~~

Every render creates a new options object. The runnable version builds any setup-specific values inside the Effect and lists the actual inputs:

~~~jsx
useEffect(() => {
  const connection = createMockConnection(serverKey, roomId);
  connection.connect((message) => {
    setMessages((currentMessages) => [...currentMessages, message]);
  });
  return () => connection.disconnect();
}, [serverKey, roomId]);
~~~

The dependency list now follows the values that determine the connection rather than the identity of a temporary wrapper.
