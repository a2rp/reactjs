# React Example: Measure and Reduce Unnecessary Search Work

This example renders the same searchable records in two panels. The baseline filters records whenever its component renders. The optimized panel uses `memo` to skip a parent-driven render when its props have not changed, and `useMemo` to reuse the filtered records when its own unrelated display state changes.

The example is a way to observe the behavior, not proof that every real list needs memoization. The filtering operation is intentionally simple; use representative application data and measure a real symptom before applying the pattern.

## What this example demonstrates

- Recording the same parent update and local panel update in React Profiler.
- Using `<Profiler>` callbacks to identify commits and render duration.
- Seeing when `memo` can skip a child update because its props remain equal.
- Seeing when `useMemo` can reuse a calculation during an unrelated local update.
- Keeping controlled input state and search work easy to inspect.
- Understanding that memoization does not improve the calculation when its dependencies change.

## Prerequisites

Start from a React project with a development command such as `npm run dev`. Keep the React Developer Tools browser extension available. This example uses standard React APIs.

## File structure

```text
src/
  App.jsx
  App.css
  main.jsx
```

## 1. Create the profiling comparison

Create `src/App.jsx`:

```jsx
import { memo, Profiler, useMemo, useState } from 'react';

const records = Array.from({ length: 500 }, (_, index) => {
  const number = String(index + 1).padStart(3, '0');
  const group = index % 3 === 0 ? 'Planning' : index % 3 === 1 ? 'Design' : 'Build';

  return {
    id: `record-${number}`,
    name: `Record ${number}`,
    group,
  };
});

function filterRecords(items, query) {
  const normalizedQuery = query.trim().toLowerCase();

  if (!normalizedQuery) {
    return items;
  }

  return items.filter((record) =>
    `${record.name} ${record.group}`.toLowerCase().includes(normalizedQuery),
  );
}

function reportRender(id, phase, actualDuration, baseDuration) {
  console.log('[Profile]', {
    id,
    phase,
    actualDuration: `${actualDuration.toFixed(2)} ms`,
    baseDuration: `${baseDuration.toFixed(2)} ms`,
  });
}

function RecordRows({ items, showGroups }) {
  return (
    <ul className="record-list">
      {items.map((record) => (
        <li key={record.id}>
          <span>{record.name}</span>
          {showGroups && <span className="record-group">{record.group}</span>}
        </li>
      ))}
    </ul>
  );
}

function BaselineResults({ items, query }) {
  const [showGroups, setShowGroups] = useState(true);
  const visibleRecords = filterRecords(items, query);
  console.count('BaselineResults render; filter recalculated');

  return (
    <section className="panel" aria-labelledby="baseline-title">
      <h2 id="baseline-title">Baseline results</h2>
      <p>Filtering runs every time this component renders.</p>
      <button type="button" onClick={() => setShowGroups((value) => !value)}>
        Toggle group labels
      </button>
      <p role="status">
        Showing {visibleRecords.length} of {items.length} records
      </p>
      <RecordRows items={visibleRecords} showGroups={showGroups} />
    </section>
  );
}

const OptimizedResults = memo(function OptimizedResults({ items, query }) {
  const [showGroups, setShowGroups] = useState(true);
  const visibleRecords = useMemo(
    () => filterRecords(items, query),
    [items, query],
  );
  console.count('OptimizedResults render; filter reused when dependencies match');

  return (
    <section className="panel" aria-labelledby="optimized-title">
      <h2 id="optimized-title">Memoized results</h2>
      <p>
        Equal parent props can skip this component; unchanged filter inputs can
        reuse the filtered records.
      </p>
      <button type="button" onClick={() => setShowGroups((value) => !value)}>
        Toggle group labels
      </button>
      <p role="status">
        Showing {visibleRecords.length} of {items.length} records
      </p>
      <RecordRows items={visibleRecords} showGroups={showGroups} />
    </section>
  );
});

export default function App() {
  const [query, setQuery] = useState('');
  const [unrelatedCount, setUnrelatedCount] = useState(0);

  return (
    <main className="page">
      <h1>Measure search rendering</h1>
      <p>
        Search changes both panels. The counter changes neither panel's data.
      </p>

      <label htmlFor="record-search">Search records</label>
      <input
        id="record-search"
        type="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />

      <button
        type="button"
        onClick={() => setUnrelatedCount((count) => count + 1)}
      >
        Unrelated counter: {unrelatedCount}
      </button>

      <div className="comparison">
        <Profiler id="BaselineResults" onRender={reportRender}>
          <BaselineResults items={records} query={query} />
        </Profiler>
        <Profiler id="OptimizedResults" onRender={reportRender}>
          <OptimizedResults items={records} query={query} />
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
  color: #1b2933;
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

button,
input {
  min-height: 2.6rem;
  padding: 0.5rem 0.75rem;
  border: 1px solid #74818a;
  border-radius: 0.4rem;
  font: inherit;
}

button {
  color: white;
  border-color: #145c47;
  background: #145c47;
  cursor: pointer;
}

:focus-visible {
  outline: 3px solid #d47b00;
  outline-offset: 3px;
}

.page {
  display: grid;
  gap: 0.8rem;
  width: min(100% - 2rem, 76rem);
  margin: 2rem auto;
}

.page h1,
.page p {
  margin-block: 0;
}

#record-search {
  width: min(100%, 30rem);
}

.comparison {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
  align-items: start;
  margin-top: 0.5rem;
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

.panel > p:first-of-type {
  min-height: 3rem;
  color: #4c5b65;
}

.record-list {
  display: grid;
  gap: 0.25rem;
  max-height: 25rem;
  margin: 0;
  padding: 0;
  overflow: auto;
  list-style: none;
}

.record-list li {
  display: flex;
  justify-content: space-between;
  gap: 0.75rem;
  padding: 0.4rem;
  border-bottom: 1px solid #e4e9ec;
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

## 2. Profile an unrelated parent update

Open React Developer Tools **Profiler** and record clicking **Unrelated counter** once.

- `App` renders because its counter state changed.
- `BaselineResults` renders and recalculates the filter even though `items` and `query` are unchanged.
- `OptimizedResults` receives the same `items` array and `query` string, so `memo` can skip the parent-driven render.

The counter does not affect either panel's output. This is the specific reason the optimized panel can skip the work.

## 3. Profile a local display update

Record clicking **Toggle group labels** inside each panel:

- The baseline recalculates the filter after its local state changes.
- The optimized component must render because its own state changed, but `useMemo` can reuse the filtered array because `items` and `query` did not change.
- Both panels still update the group-label display.

This separates the two tools: `memo` can skip a component update from its parent; `useMemo` can skip a calculation during a render that still needs to happen.

## 4. Profile a changed search query

Record typing a new query:

- Both panels receive a changed `query`, so both need to produce new results.
- `useMemo` cannot skip filtering when a dependency changed.
- The input stays controlled and updates as the user types.

Use the browser console counts to see when each component body and filter runs. The counts are development clues, not timing measurements; Strict Mode can call render logic extra times. Use the Profiler duration and component tree for React work, and browser Performance tools for layout, paint, or other JavaScript.

## 5. Interpret the comparison carefully

`actualDuration` reports React render time for the profiled subtree in the current update; `baseDuration` estimates the cost of rendering the subtree without skipped work. Neither is the full time from a click to a painted frame. The `<Profiler>` callback and console logging add some overhead, so remove them after the experiment.

The sample uses 500 small records so updates are easy to exercise. It does not claim that this list is slow. In a real app, use representative records, reproduce an actual delay, and compare the same interaction before and after the change.

If React Compiler is enabled in the project, it may optimize the baseline too, so the difference can be smaller. Chapter 108 covers compiler behavior. Focus on the measured user experience rather than expecting one specific render count.

## Expected outcomes

- Both panels show the same records for a given query.
- An unrelated parent update can be skipped by the memoized panel when its props are unchanged.
- The optimized panel can reuse its filtered result during its local group-label update.
- Both panels calculate new results when the query changes.
- Profiler recordings show why each optimization helps only for a particular update path.
