# Example: Preserve Panel State and Profile React 19.2 Work

This example shows a small workspace with two panels. Switching panels uses React 19.2's `<Activity>` so the inactive panel is hidden while its React state is retained. One panel also runs a timer in an Effect; hiding it cleans up the timer, and showing it sets the Effect up again.

The app is intentionally small. It demonstrates the behavior and how to inspect a repeatable interaction. It is not a benchmark and does not promise that Activity makes every screen faster.

## Requirements

- React and React DOM 19.2 or later.
- A React application using Vite or another toolchain configured for the modern JSX transform.
- A browser with a Performance panel that supports custom tracks; these steps use Chrome DevTools.

If you need a new project:

```sh
npm create vite@latest activity-performance-demo -- --template react
cd activity-performance-demo
npm install
npm run dev
```

## 1. Build the workspace

Replace `src/App.jsx` with:

```jsx
import { Activity, useEffect, useId, useState } from "react";
import "./App.css";

function NotesPanel({ id }) {
  const [draft, setDraft] = useState("");
  const draftId = useId();

  return (
    <section className="panel-content" id={id} aria-labelledby={draftId + "-title"}>
      <h2 id={draftId + "-title"}>Notes</h2>
      <label htmlFor={draftId}>Draft text</label>
      <textarea
        id={draftId}
        rows={7}
        value={draft}
        onChange={(event) => setDraft(event.target.value)}
        placeholder="Write a few words, then switch panels."
      />
      <p className="hint">
        This draft is kept in this component's React state while the panel is hidden.
      </p>
      <p className="character-count">{draft.length} characters</p>
    </section>
  );
}

function PreviewPanel({ id }) {
  const [ticks, setTicks] = useState(0);
  const refreshId = useId();

  useEffect(() => {
    const timerId = window.setInterval(() => {
      setTicks((currentTicks) => currentTicks + 1);
    }, 1000);

    return () => {
      window.clearInterval(timerId);
    };
  }, []);

  return (
    <section className="panel-content" id={id} aria-labelledby={refreshId + "-title"}>
      <h2 id={refreshId + "-title"}>Preview</h2>
      <p>
        This panel has been visible for {ticks} seconds in total. Its timer stops
        when the panel is hidden and continues after it becomes visible again.
      </p>
      <button type="button" onClick={() => setTicks((value) => value + 10)}>
        Add ten seconds to the demo counter
      </button>
      <p className="hint">
        The counter is React state. The interval is an external resource managed
        by an Effect.
      </p>
    </section>
  );
}

export default function App() {
  const [activeView, setActiveView] = useState("notes");
  const viewGroupName = useId();

  function chooseView(nextView) {
    if (typeof performance !== "undefined") {
      performance.mark("activity-demo:view-" + nextView);
    }

    setActiveView(nextView);
  }

  return (
    <main className="app">
      <header className="intro">
        <p className="eyebrow">React 19.2</p>
        <h1>Activity and performance trace</h1>
        <p>
          Switch panels, check which state returns, then record the interaction
          in the browser Performance panel.
        </p>
      </header>

      <fieldset className="view-picker">
        <legend>Choose a panel</legend>

        <label>
          <input
            type="radio"
            name={viewGroupName}
            value="notes"
            checked={activeView === "notes"}
            onChange={() => chooseView("notes")}
            aria-controls="notes-panel"
          />
          Notes
        </label>

        <label>
          <input
            type="radio"
            name={viewGroupName}
            value="preview"
            checked={activeView === "preview"}
            onChange={() => chooseView("preview")}
            aria-controls="preview-panel"
          />
          Preview
        </label>
      </fieldset>

      <div className="workspace">
        <Activity mode={activeView === "notes" ? "visible" : "hidden"}>
          <NotesPanel id="notes-panel" />
        </Activity>

        <Activity mode={activeView === "preview" ? "visible" : "hidden"}>
          <PreviewPanel id="preview-panel" />
        </Activity>
      </div>

      <aside className="instructions">
        <h2>What to notice</h2>
        <ul>
          <li>The Notes draft returns when you switch back.</li>
          <li>The Preview timer pauses while Preview is hidden.</li>
          <li>The browser trace places React work beside browser activity.</li>
        </ul>
      </aside>
    </main>
  );
}
```

## 2. Add basic styling

Replace `src/App.css` with:

```css
:root {
  color: #182238;
  background: #f2f5fa;
  font-family: system-ui, sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
  box-sizing: border-box;
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

button {
  min-height: 2.75rem;
  padding: 0.65rem 1rem;
  border: 0;
  border-radius: 0.5rem;
  color: white;
  background: #234db4;
  font-weight: 650;
  cursor: pointer;
}

button:hover {
  background: #173a91;
}

button:focus-visible,
input:focus-visible,
textarea:focus-visible {
  outline: 3px solid #d78300;
  outline-offset: 3px;
}

.app {
  width: min(100% - 2rem, 48rem);
  margin: 0 auto;
  padding: 3rem 0;
}

.intro {
  margin-bottom: 1.5rem;
}

.intro h1 {
  margin: 0.25rem 0 0.75rem;
  font-size: clamp(2rem, 6vw, 3.1rem);
  line-height: 1.1;
}

.intro > p:last-child,
.hint,
.instructions {
  color: #4c5870;
  line-height: 1.6;
}

.eyebrow {
  color: #234db4;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.view-picker,
.panel-content,
.instructions {
  margin: 1rem 0;
  padding: 1.25rem;
  border: 1px solid #d6deeb;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 0.25rem 1rem rgb(24 44 82 / 6%);
}

.view-picker {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem 1.25rem;
}

.view-picker legend {
  padding: 0 0.35rem;
  font-weight: 700;
}

.view-picker label {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  min-height: 2rem;
}

.view-picker input {
  width: 1.1rem;
  height: 1.1rem;
  accent-color: #234db4;
}

.panel-content h2,
.instructions h2 {
  margin-top: 0;
}

.panel-content label {
  display: block;
  margin: 0.75rem 0 0.4rem;
  font-weight: 650;
}

.panel-content textarea {
  width: 100%;
  min-height: 10rem;
  padding: 0.75rem;
  border: 1px solid #8793a8;
  border-radius: 0.5rem;
  resize: vertical;
  line-height: 1.5;
}

.hint,
.character-count {
  font-size: 0.92rem;
}

.instructions ul {
  padding-left: 1.4rem;
}

@media (max-width: 38rem) {
  .app {
    padding: 1.5rem 0;
  }
}
```

Replace `src/main.jsx` with:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

Keep the Vite `index.html` root element and its module script for `/src/main.jsx`.

## 3. Observe Activity behavior

1. Run the development server and open the app.
2. Type a sentence in Notes.
3. Switch to Preview. The draft is hidden but its component state is preserved.
4. Watch the timer value for a few seconds.
5. Switch back to Notes. The draft remains, and the Preview timer stopped while hidden.
6. Switch to Preview again. The timer Effect is set up again and its preserved counter continues.
7. Navigate the radio controls using the keyboard. The hidden panel should not be reachable by tabbing while it is hidden.

The component state and the timer have different lifecycles: the state is retained, but the Effect's interval is cleaned up and recreated.

## 4. Compare with conditional rendering

For comparison, replace the two Activity boundaries with conditional rendering:

```jsx
{activeView === "notes" && <NotesPanel id="notes-panel" />}
{activeView === "preview" && <PreviewPanel id="preview-panel" />}
```

Now each panel is removed when inactive. Its local state is discarded, and showing it later starts it from its initial state. Choose between the two approaches based on the desired user experience and measured resource cost.

## 5. Record a browser performance trace

While the development server is running:

1. Open Chrome DevTools and select **Performance**.
2. Start a recording.
3. Switch Notes and Preview several times, and type a short draft.
4. Stop the recording.
5. Find the `activity-demo:view-notes` and `activity-demo:view-preview` user timing marks.
6. Inspect nearby React Scheduler and Components tracks.
7. Look for Activity mount, disconnect, or reconnect work and compare it with JavaScript, network, and browser activity on the same timeline.

React Performance Tracks are available in development and profiling builds. They are disabled in production builds by default because instrumentation adds overhead. This small app is intended to make the tracks and lifecycle visible; its trace does not prove a speed improvement.

React Strict Mode may add development-only Effect setup and cleanup cycles. Use the browser trace and actual interaction to understand behavior; do not treat a count of console messages or Effect setups in development as a production benchmark.

## 6. Optional: inspect a transition in the Scheduler track

Use a transition only when the result view is measurably expensive and can update in the background. For example, if a query drives a large report list:

```jsx
import { useState, useTransition } from "react";

function ReportSearch() {
  const [inputValue, setInputValue] = useState("");
  const [query, setQuery] = useState("");
  const [isPending, startTransition] = useTransition();

  function handleChange(event) {
    const nextValue = event.target.value;

    setInputValue(nextValue); // Keep the controlled input immediate.
    startTransition(() => {
      setQuery(nextValue); // Defer the expensive result update.
    });
  }

  return (
    <>
      <label>
        Search reports
        <input value={inputValue} onChange={handleChange} />
      </label>
      {isPending && <p role="status">Updating results…</p>}
      <LargeReportList query={query} />
    </>
  );
}
```

This snippet assumes `LargeReportList` has real work worth deferring. Do not add a large artificial list only to produce a longer trace. Record the interaction and verify that the input remains responsive and that the transition track shows the expected background work.

## 7. Optional: use an Effect Event for a connection callback

When an external connection emits an event, its connection key may be a dependency while presentation details such as the current theme should be read at callback time:

```jsx
import { useEffect, useEffectEvent } from "react";

function RoomConnection({ roomId, theme }) {
  const handleConnected = useEffectEvent(() => {
    showNotification("Connected!", theme);
  });

  useEffect(() => {
    const connection = createConnection(roomId);

    connection.on("connected", () => {
      handleConnected();
    });

    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return null;
}
```

`createConnection` and `showNotification` are application-specific functions. Call this Effect Event only from the Effect in this component. Do not use it for a regular button click or to omit a real dependency such as `roomId`.

## 8. Production considerations

- Activity can retain hidden component trees and their state, which uses memory.
- Hidden Effects are cleaned up; make every subscription and timer safe to reconnect.
- Hidden content may still rerender when props change, at lower priority.
- Server-rendered hidden Activity content is not part of the initial server HTML; check the framework's hydration behavior.
- Profiling builds and development traces add instrumentation; measure production behavior separately.
- Use Activity only for screens users are likely to revisit or content with a clear measured benefit.
