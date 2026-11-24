# 037. State with useState - Example

This example renders two independent study panels. Each panel remembers how many practice cards were completed and whether its hint is visible. The count, progress element, button states, and hint all render from useState values.

## Project files

Create these files in an existing React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

Start the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useState } from 'react';

const totalCards = 5;

function StudyPanel({ subject, idPrefix }) {
  const [completedCount, setCompletedCount] = useState(0);
  const [showHint, setShowHint] = useState(false);

  const progressId = idPrefix + '-progress';
  const headingId = idPrefix + '-heading';
  const hintId = idPrefix + '-hint';

  function handleCompleteOne() {
    setCompletedCount(completedCount + 1);
  }

  function handleRemoveOne() {
    setCompletedCount(completedCount - 1);
  }

  function handleReset() {
    setCompletedCount(0);
  }

  function handleToggleHint() {
    setShowHint(!showHint);
  }

  return (
    <section className="study-panel" aria-labelledby={headingId}>
      <h2 id={headingId}>{subject} practice</h2>
      <p className="panel-intro">
        Complete up to {totalCards} practice cards in this panel.
      </p>

      <label className="progress-label" htmlFor={progressId}>
        {subject} cards completed
      </label>
      <progress id={progressId} max={totalCards} value={completedCount}>
        {completedCount} of {totalCards}
      </progress>
      <p className="progress-status" role="status">
        {completedCount} of {totalCards} {subject} cards completed.
      </p>

      <div className="button-row">
        <button
          type="button"
          onClick={handleRemoveOne}
          disabled={completedCount === 0}
          aria-label={'Remove one ' + subject + ' practice card'}
        >
          Remove one
        </button>
        <button
          type="button"
          onClick={handleCompleteOne}
          disabled={completedCount === totalCards}
          aria-label={'Complete one ' + subject + ' practice card'}
        >
          Complete one
        </button>
        <button
          type="button"
          className="secondary-button"
          onClick={handleReset}
          disabled={completedCount === 0}
        >
          Reset count
        </button>
      </div>

      <button
        type="button"
        className="hint-button"
        aria-expanded={showHint}
        aria-controls={hintId}
        onClick={handleToggleHint}
      >
        {showHint ? 'Hide hint' : 'Show hint'}
      </button>

      <p className="hint" id={hintId} hidden={!showHint}>
        {subject === 'React'
          ? 'Ask which values need to be remembered between renders.'
          : 'Try one small practice task, then describe what changed.'}
      </p>
    </section>
  );
}

function App() {
  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React state practice</p>
        <h1>Track your study cards</h1>
        <p>
          Use the controls to update each panel. The displayed interface is
          rendered from the state stored by that panel.
        </p>
      </header>

      <div className="study-grid">
        <StudyPanel subject="React" idPrefix="react" />
        <StudyPanel subject="CSS" idPrefix="css" />
      </div>

      <section className="explanation" aria-labelledby="state-heading">
        <h2 id="state-heading">What to notice</h2>
        <ul>
          <li>
            Each panel calls useState at the top level of its component.
          </li>
          <li>
            Clicking a button calls a setter, and React renders the panel
            again with the new value.
          </li>
          <li>
            The progress bar and disabled buttons are calculated from the
            completed-card count.
          </li>
          <li>
            The React and CSS panels have independent count and hint state.
          </li>
          <li>
            The hint paragraph stays in the document and is shown or hidden
            from the boolean state.
          </li>
        </ul>
      </section>
    </main>
  );
}

export default App;
~~~

The decrement and increment handlers each set one known next value from the current render. A later chapter explains why functional updater functions are useful when several updates depend on pending state.

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
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
  color: #1b2435;
  background: #f3f5f9;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 20rem;
  min-height: 100vh;
}

button {
  font: inherit;
}

.page {
  width: min(100% - 2rem, 64rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 42rem;
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin-bottom: 0.5rem;
  color: #354b9c;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  margin-bottom: 0.75rem;
  font-size: clamp(2rem, 6vw, 3.25rem);
}

h2 {
  margin-bottom: 0.75rem;
  font-size: 1.35rem;
}

.page-header > p:last-child,
.panel-intro,
.explanation li {
  color: #465166;
  line-height: 1.6;
}

.study-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 19rem), 1fr));
  gap: 1rem;
}

.study-panel,
.explanation {
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d1d8e3;
  border-radius: 0.9rem;
  background: #fff;
  box-shadow: 0 0.25rem 1rem rgb(27 36 53 / 6%);
}

.panel-intro {
  margin-bottom: 1.25rem;
}

.progress-label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 700;
}

progress {
  display: block;
  width: 100%;
  height: 1rem;
  accent-color: #2e4b9b;
}

.progress-status {
  margin-top: 0.6rem;
  color: #34415a;
  font-weight: 600;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-top: 1.25rem;
}

.button-row button,
.hint-button {
  min-height: 2.75rem;
  padding: 0.65rem 0.9rem;
  border: 1px solid #2e468d;
  border-radius: 0.5rem;
  color: #fff;
  background: #304a99;
  font-weight: 700;
  cursor: pointer;
}

.button-row .secondary-button {
  color: #25355f;
  background: #fff;
}

.button-row button:hover:not(:disabled),
.hint-button:hover {
  background: #243b80;
  color: #fff;
}

.button-row button:disabled {
  border-color: #8b94a4;
  color: #4b5567;
  background: #e5e8ee;
  cursor: not-allowed;
}

.hint-button {
  margin-top: 1rem;
}

.hint {
  margin-top: 0.75rem;
  padding: 0.85rem;
  border-left: 4px solid #536bb8;
  border-radius: 0.25rem;
  color: #273654;
  background: #eef1fb;
  line-height: 1.55;
}

.explanation {
  margin-top: 1rem;
}

.explanation ul {
  display: grid;
  gap: 0.55rem;
  padding-left: 1.3rem;
}

:where(button):focus-visible {
  outline: 3px solid #354b9c;
  outline-offset: 3px;
}
~~~

## What to try

1. Complete cards in the React panel and observe its progress bar and status text.
2. Complete cards in the CSS panel. Its count should change independently.
3. Remove a card until the count reaches zero. The remove and reset buttons should then be disabled.
4. Complete cards until the progress reaches five. The complete button should then be disabled.
5. Show the React hint, then show the CSS hint. Each panel controls its own hint.
6. Use the keyboard to reach every button and check that the focus outline is visible.
7. Render another StudyPanel and observe that the new instance starts with its own state.

## Expected behavior

Each StudyPanel begins with a count of zero and its hint hidden. The count cannot go below zero or above five because the related buttons become disabled at those limits. Activating the hint button changes its label, aria-expanded value, and hint visibility. State in one panel does not change the other panel.