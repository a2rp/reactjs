# React Example: Profile an Effect-Driven Update

This example compares two searchable lists with the same data and UI. One stores filtered results in state and recalculates them in an Effect; the other derives the filtered list during render. Profiling the same search interaction makes the extra update from the Effect-based version visible.

## What this example demonstrates

- Comparing two implementations under the same development build.
- Observing render commits for a single search interaction.
- Using the React DevTools Profiler and React `<Profiler>` callback.
- Identifying a cascading update caused by redundant derived state.
- Replacing Effect-based derived state with a render calculation.
- Accounting for Strict Mode and profiling overhead when reading results.

## Prerequisites

Start from a React project with a development command such as `npm run dev`. Keep the React Developer Tools browser extension available. This example uses standard React Hooks and JSX.

## File structure

```text
src/
  App.jsx
  App.css
  main.jsx
```

## 1. Create the comparison app

Create `src/App.jsx`:

```jsx
import { Profiler, useEffect, useState } from 'react';

const records = Array.from({ length: 180 }, (_, index) => {
  const number = String(index + 1).padStart(3, '0');
  const group = index % 3 === 0 ? 'Planning' : index % 3 === 1 ? 'Design' : 'Build';

  return {
    id: `record-${number}`,
    label: `Record ${number}`,
    group,
  };
});

function matchesQuery(record, query) {
  const normalizedQuery = query.trim().toLowerCase();
  return (
    record.label.toLowerCase().includes(normalizedQuery) ||
    record.group.toLowerCase().includes(normalizedQuery)
  );
}

function reportRender(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  console.log('[React Profiler]', {
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime,
  });
}

function RecordList({ records: visibleRecords, total }) {
  return (
    <>
      <p role="status">
        Showing {visibleRecords.length} of {total} records
      </p>
      {visibleRecords.length > 0 ? (
        <ul className="record-list">
          {visibleRecords.map((record) => (
            <li key={record.id}>
              <span>{record.label}</span>
              <span className="record-group">{record.group}</span>
            </li>
          ))}
        </ul>
      ) : (
        <p>No matching records.</p>
      )}
    </>
  );
}

function EffectDerivedSearch() {
  const [query, setQuery] = useState('');
  const [visibleRecords, setVisibleRecords] = useState(records);

  useEffect(() => {
    setVisibleRecords(records.filter((record) => matchesQuery(record, query)));
  }, [query]);

  return (
    <section className="panel" aria-labelledby="effect-title">
      <h2 id="effect-title">Effect-derived results</h2>
      <p className="explanation">
        The filtered list is stored in state and synchronized from the query by
        an Effect.
      </p>
      <label htmlFor="effect-query">Search records</label>
      <input
        id="effect-query"
        type="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />
      <RecordList records={visibleRecords} total={records.length} />
    </section>
  );
}

function RenderDerivedSearch() {
  const [query, setQuery] = useState('');
  const visibleRecords = records.filter((record) => matchesQuery(record, query));

  return (
    <section className="panel" aria-labelledby="render-title">
      <h2 id="render-title">Render-derived results</h2>
      <p className="explanation">
        The filtered list is calculated directly from the query during render.
      </p>
      <label htmlFor="render-query">Search records</label>
      <input
        id="render-query"
        type="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />
      <RecordList records={visibleRecords} total={records.length} />
    </section>
  );
}

export default function App() {
  return (
    <main className="page">
      <h1>Profile component behavior</h1>
      <p>
        Enter the same search in both panels and compare the React updates.
      </p>
      <div className="comparison">
        <Profiler id="EffectDerivedSearch" onRender={reportRender}>
          <EffectDerivedSearch />
        </Profiler>
        <Profiler id="RenderDerivedSearch" onRender={reportRender}>
          <RenderDerivedSearch />
        </Profiler>
      </div>
    </main>
  );
}
```

Create `src/main.jsx`:

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './App.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

Create `src/App.css`:

```css
:root {
  font-family: system-ui, sans-serif;
  color: #1c2933;
  background: #f3f6f8;
  font-synthesis: none;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  margin: 0;
}

input {
  min-height: 2.6rem;
  padding: 0.5rem 0.7rem;
  border: 1px solid #74818a;
  border-radius: 0.35rem;
  font: inherit;
}

:focus-visible {
  outline: 3px solid #d47b00;
  outline-offset: 3px;
}

.page {
  width: min(100% - 2rem, 76rem);
  margin: 2rem auto;
}

.comparison {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
  align-items: start;
}

.panel {
  display: grid;
  gap: 0.75rem;
  min-width: 0;
  padding: 1rem;
  border: 1px solid #d2dce2;
  border-radius: 0.75rem;
  background: white;
}

.panel h2,
.panel p {
  margin: 0;
}

.explanation {
  min-height: 3rem;
  color: #465762;
}

.record-list {
  display: grid;
  gap: 0.35rem;
  max-height: 24rem;
  margin: 0;
  padding: 0;
  overflow: auto;
  list-style: none;
}

.record-list li {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.5rem;
  border-bottom: 1px solid #e3e8eb;
}

.record-group {
  color: #52616b;
}

@media (max-width: 48rem) {
  .comparison {
    grid-template-columns: 1fr;
  }
}
```

## 2. Profile the same interaction

Run the development server and open **Profiler** in React Developer Tools.

1. Start a recording.
2. Enter `record 12` into the left search field.
3. Stop the recording after the result settles.
4. Inspect the commits associated with `EffectDerivedSearch`.
5. Repeat the same steps in a new recording for the right search field.
6. Compare the update sequence and component work; then inspect the browser console for the `<Profiler>` callback output.

The left panel first renders with the new `query`. Its Effect then calculates a new `visibleRecords` array and sets state, causing another update. The right panel calculates `visibleRecords` from the current query during the same render, so it does not need a second state update for the filtered list.

Strict Mode is enabled. It can add development-only render and Effect checks, especially during initial mount. Let the page settle before recording the typing interaction, and compare equivalent user actions instead of comparing startup logs.

## 3. Fix the redundant state

The right panel shows the corrected pattern:

```jsx
function RenderDerivedSearch() {
  const [query, setQuery] = useState('');
  const visibleRecords = records.filter((record) => matchesQuery(record, query));

  return <RecordList records={visibleRecords} total={records.length} />;
}
```

There is no Effect or second state value to synchronize. If filtering were measurably expensive for the real data size, profile first and then consider `useMemo` for that calculation. Memoization is not needed merely because a value is derived.

## 4. Understand the measurements

Each `<Profiler>` calls `reportRender` with the subtree ID, phase, render durations, and timestamps. `actualDuration` is the time spent rendering that subtree for the current update. `baseDuration` estimates the cost of rendering the whole subtree without skipped work. Neither value includes all time from keystroke through browser paint.

The console callback is diagnostic code and can add overhead. Use it to see which commits happen and their phases, and use the DevTools Profiler to inspect the component tree. For the full page cost, record the same interaction in the browser Performance panel and correlate React work with JavaScript, layout, paint, and network activity.

## 5. Run a focused comparison

Try other queries such as `design` or `record 099`. Keep the data and environment unchanged when comparing. If you change the implementation, record the same query again and verify that the visible result is unchanged while the unnecessary update is gone.

This comparison is intentionally small. It demonstrates how an Effect can create an extra update; it is not evidence that a real application's largest bottleneck is filtering. Use realistic data and a user-visible symptom to guide an actual optimization.

## Expected outcomes

- Both panels return the same records for the same query.
- The Effect-derived panel shows a follow-up state update after the query changes.
- The render-derived panel calculates the filtered result without syncing duplicate state.
- React Profiler and the browser Performance panel provide complementary evidence.
- Profiling overhead and Strict Mode development behavior are accounted for in the comparison.
