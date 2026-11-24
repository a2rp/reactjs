# 039. Queued updates, batching, and functional updates - Example

## What this example demonstrates

This runnable React app contains three small demonstrations:

1. Compare one direct increment, three direct updates, and three functional updates.
2. Change the order of replacement values and updater functions to see that queue order matters.
3. Update several related state values from one event handler and render their consistent result.

The update buttons use React state only; no external package is needed beyond an existing React app.

## Project files

Add these files to an existing React app created with Vite:

- src/App.jsx
- src/App.css
- src/main.jsx
- src/index.css

Run the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useState } from 'react';
import './App.css';

export default function App() {
  const [count, setCount] = useState(0);
  const [counterNote, setCounterNote] = useState(
    'Try each counter button and compare its result.'
  );

  const [queueValue, setQueueValue] = useState(0);
  const [queueNote, setQueueNote] = useState(
    'Reset the queue value before comparing the ordered examples.'
  );

  const [saveStage, setSaveStage] = useState('Idle');
  const [savedItems, setSavedItems] = useState(0);
  const [saveError, setSaveError] = useState('');

  function handleAddOneDirectly() {
    setCount(count + 1);
    setCounterNote(
      'This handler used its count snapshot (' +
        count +
        ') and requested ' +
        (count + 1) +
        '.'
    );
  }

  function handleAddThreeDirectly() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);

    setCounterNote(
      'All three direct calls used the same snapshot (' +
        count +
        '). They each requested ' +
        (count + 1) +
        ', so the result increased by one.'
    );
  }

  function handleAddThreeWithUpdaters() {
    setCount((currentCount) => currentCount + 1);
    setCount((currentCount) => currentCount + 1);
    setCount((currentCount) => currentCount + 1);

    setCounterNote(
      'Three updater functions ran in queue order, each adding one to the pending count.'
    );
  }

  function handleResetCounter() {
    setCount(0);
    setCounterNote('The handler queued a replacement value of 0.');
  }

  function handleReplaceThenIncrement() {
    setQueueValue(5);
    setQueueValue((currentValue) => currentValue + 1);
    setQueueNote(
      'Replace with 5, then add one to that pending value. The result is 6.'
    );
  }

  function handleIncrementThenReplace() {
    setQueueValue((currentValue) => currentValue + 1);
    setQueueValue(5);
    setQueueNote(
      'Add one to the pending value, then replace it with 5. The result is 5.'
    );
  }

  function handleReplaceAtTheEnd() {
    setQueueValue(5);
    setQueueValue((currentValue) => currentValue + 1);
    setQueueValue(42);
    setQueueNote(
      'The updater calculates 6, then the final replacement makes the result 42.'
    );
  }

  function handleResetQueue() {
    setQueueValue(0);
    setQueueNote('Queue value reset to 0. Try the order examples again.');
  }

  function handleSaveSuccessfully() {
    setSaveStage('Saved');
    setSavedItems((currentSavedItems) => currentSavedItems + 1);
    setSaveError('');
  }

  function handleSaveWithError() {
    setSaveStage('Needs attention');
    setSaveError('The simulated save failed. Your previous saved count remains.');
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React state</p>
        <h1>Queued, batched, and functional updates</h1>
        <p>
          Setters queue updates. A direct value replaces pending state; an
          updater calculates from the pending value.
        </p>
      </header>

      <section className="demo-card" aria-labelledby="counter-heading">
        <p className="eyebrow">Demo 1</p>
        <h2 id="counter-heading">Direct values and updater functions</h2>
        <p className="large-value" aria-live="polite">
          Count: {count}
        </p>

        <div className="button-row">
          <button type="button" onClick={handleAddOneDirectly}>
            Add one directly
          </button>
          <button type="button" onClick={handleAddThreeDirectly}>
            Add three directly
          </button>
          <button type="button" onClick={handleAddThreeWithUpdaters}>
            Add three with updaters
          </button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleResetCounter}
          >
            Reset
          </button>
        </div>

        <p className="status-box" role="status">
          {counterNote}
        </p>

        <p className="explanation">
          Add one directly uses the current render's count to calculate one
          known next value. The two “Add three” buttons queue three updates in
          one handler, so compare direct replacements with updater functions.
        </p>
      </section>

      <section className="demo-card" aria-labelledby="queue-heading">
        <p className="eyebrow">Demo 2</p>
        <h2 id="queue-heading">Update order changes the result</h2>
        <p className="large-value" aria-live="polite">
          Queue result: {queueValue}
        </p>

        <div className="button-row">
          <button type="button" onClick={handleReplaceThenIncrement}>
            Replace with 5, then add one
          </button>
          <button type="button" onClick={handleIncrementThenReplace}>
            Add one, then replace with 5
          </button>
          <button type="button" onClick={handleReplaceAtTheEnd}>
            Finish with replacement 42
          </button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleResetQueue}
          >
            Reset queue value
          </button>
        </div>

        <p className="status-box" role="status">
          {queueNote}
        </p>

        <p className="explanation">
          Each button adds a short sequence to the queue. A replacement later
          in the same sequence can replace the result calculated by earlier
          entries.
        </p>
      </section>

      <section className="demo-card" aria-labelledby="batching-heading">
        <p className="eyebrow">Demo 3</p>
        <h2 id="batching-heading">Related state updates</h2>
        <p>
          Save status: <strong>{saveStage}</strong>
        </p>
        <p>
          Saved items: <strong>{savedItems}</strong>
        </p>
        {saveError !== '' && (
          <p className="error-box" role="status">
            {saveError}
          </p>
        )}

        <div className="button-row">
          <button type="button" onClick={handleSaveSuccessfully}>
            Simulate successful save
          </button>
          <button
            className="secondary-button"
            type="button"
            onClick={handleSaveWithError}
          >
            Simulate save error
          </button>
        </div>

        <p className="explanation">
          A successful save handler updates the status, increments the saved
          count from its pending value, and clears the error. React processes
          these related updates so the rendered interface reflects their
          combined result.
        </p>
      </section>

      <footer className="page-footer">
        <p>
          Use a direct value for a known next value. Use an updater when the
          next value depends on pending state.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/App.css

~~~css
.page-shell {
  width: min(100% - 2rem, 920px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.page-header h1 {
  margin-block: 0.35rem 0.75rem;
  font-size: clamp(2rem, 5vw, 3.25rem);
  line-height: 1.1;
}

.page-header > p:last-child,
.explanation,
.page-footer {
  max-width: 70ch;
  line-height: 1.65;
  color: #4b5563;
}

.eyebrow {
  color: #4d3ab8;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.demo-card {
  margin-block: 1.25rem;
  padding: clamp(1.1rem, 3vw, 1.75rem);
  border: 1px solid #d5dae4;
  border-radius: 1rem;
  background: #ffffff;
  box-shadow: 0 0.5rem 1.5rem rgb(24 33 54 / 7%);
}

.demo-card h2 {
  margin-block: 0.35rem 1rem;
}

.large-value {
  margin-block: 1rem;
  font-size: 2rem;
  font-weight: 750;
  font-variant-numeric: tabular-nums;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.65rem;
  margin-block: 1rem;
}

button {
  min-height: 2.75rem;
  padding: 0.6rem 0.9rem;
  border: 1px solid #4d3ab8;
  border-radius: 0.55rem;
  background: #5b46d6;
  color: #ffffff;
  cursor: pointer;
  font: inherit;
}

button:hover {
  background: #4835b5;
}

button:focus-visible {
  outline: 3px solid #a99cff;
  outline-offset: 2px;
}

.secondary-button {
  border-color: #abb3c0;
  background: #ffffff;
  color: #202735;
}

.secondary-button:hover {
  background: #f0f2f6;
}

.status-box,
.error-box {
  min-height: 3rem;
  margin-block: 1rem;
  padding: 0.85rem 1rem;
  border-left: 4px solid #5b46d6;
  border-radius: 0.35rem;
  background: #f3f1ff;
  line-height: 1.55;
}

.error-box {
  border-left-color: #b42318;
  background: #fff1f0;
  color: #7a2018;
}

.explanation {
  margin-top: 1rem;
  font-size: 0.95rem;
}

.page-footer {
  margin-top: 2rem;
  padding-inline: 0.25rem;
}

@media (max-width: 560px) {
  .page-shell {
    width: min(100% - 1.25rem, 920px);
    padding-block: 1.5rem;
  }
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.jsx';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
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
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #202735;
  background: #f6f7fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button {
  font: inherit;
}
~~~

## Try it

1. Select Add one directly. The count increases by one.
2. Select Add three directly. The three calls each use the same render snapshot, so the count increases by one.
3. Select Add three with updaters. The three updater functions build on the pending value, so the count increases by three.
4. In the queue demo, reset the value. Select Replace with 5, then add one. The sequence produces 6.
5. Reset and select Add one, then replace with 5. The final replacement makes the result 5.
6. Reset and select Finish with replacement 42. The final replacement wins after the earlier queued work.
7. In Related state updates, try both simulated outcomes. The UI displays the resulting status, count, and error together.

## What to notice

- A state setter does not change the state variable in the running handler.
- A direct value queues a replacement; it does not accumulate with other replacements automatically.
- An updater receives the pending value and returns the next pending value.
- Queue order matters, especially when a replacement follows an updater.
- A handler can update multiple state variables; batching helps React render their combined result.
- An updater should calculate state only. Keep side effects in the appropriate handler or Effect.
