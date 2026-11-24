# Example: useDeferredValue with a Search Results List

This runnable React app lets you compare two modes:

- **Deferred on:** the input updates immediately while a memoized topic list catches up using **useDeferredValue**.
- **Deferred off:** the topic list receives the current query and updates along with the input.

The local list contains a small pure calculation in each row to make rendering work easier to observe. It makes no network requests and is only a teaching aid; a real application should profile and optimize slow work before adding scheduling complexity.

## Create the project

~~~sh
npm create vite@latest deferred-search-demo -- --template react
cd deferred-search-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following and start the app:

~~~sh
npm run dev
~~~

## Project files

~~~text
deferred-search-demo/
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
    <title>React Deferred Search Demo</title>
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
import { memo, useDeferredValue, useState } from "react";

const TOPIC_NAMES = [
  "Components",
  "JSX",
  "Props",
  "State",
  "Events",
  "Effects",
  "Context",
  "Reducers",
  "Transitions",
  "Suspense",
  "Accessibility",
  "Testing",
];

const CATEGORIES = [
  "Foundations",
  "Rendering",
  "Data flow",
  "Interaction",
  "Performance",
  "Accessibility",
];

const TOPICS = Array.from({ length: 180 }, (_, index) => {
  const topicName = TOPIC_NAMES[index % TOPIC_NAMES.length];
  const category = CATEGORIES[index % CATEGORIES.length];

  return {
    id: "topic-" + (index + 1),
    title: topicName + " pattern " + (Math.floor(index / TOPIC_NAMES.length) + 1),
    category,
    description:
      "Explore a practical " + category.toLowerCase() +
      " example related to " + topicName.toLowerCase() + ".",
  };
});

function TopicCard({ topic }) {
  // This pure loop simulates small per-row rendering work for the comparison.
  // React can yield between rows, but cannot interrupt this loop halfway through.
  let renderMarker = 0;
  for (let index = 0; index < 6500; index += 1) {
    renderMarker = (renderMarker + index + topic.id.length) % 4093;
  }

  return (
    <li className="topic-card" data-render-marker={renderMarker}>
      <div className="topic-card__heading">
        <h3>{topic.title}</h3>
        <span>{topic.category}</span>
      </div>
      <p>{topic.description}</p>
    </li>
  );
}

const TopicResults = memo(function TopicResults({ query }) {
  const normalizedQuery = query.trim().toLowerCase();

  const visibleTopics = TOPICS.filter((topic) => {
    const searchableText =
      topic.title + " " + topic.category + " " + topic.description;
    return searchableText.toLowerCase().includes(normalizedQuery);
  });

  return (
    <>
      <p className="result-count">
        {visibleTopics.length} {visibleTopics.length === 1 ? "topic" : "topics"}
        {" "}for {query.trim() ? "“" + query.trim() + "”" : "all searches"}
      </p>
      <ul className="topic-list" aria-label="Search results">
        {visibleTopics.map((topic) => (
          <TopicCard key={topic.id} topic={topic} />
        ))}
        {visibleTopics.length === 0 && (
          <li className="empty-state">No topics match this search.</li>
        )}
      </ul>
    </>
  );
});

function DeferredSearchDemo() {
  const [query, setQuery] = useState("");
  const [deferredMode, setDeferredMode] = useState(true);
  const deferredQuery = useDeferredValue(query);
  const resultsQuery = deferredMode ? deferredQuery : query;
  const isStale = deferredMode && query !== deferredQuery;

  return (
    <section className="panel" aria-labelledby="search-heading">
      <p className="eyebrow">Urgent input + slower consumer</p>
      <h2 id="search-heading">Search React topics</h2>
      <p className="panel-copy">
        Type while the result list renders. Toggle deferred mode to compare
        which value the slow region receives.
      </p>

      <label className="field-label" htmlFor="topic-search">
        Search terms
      </label>
      <input
        className="search-input"
        id="topic-search"
        type="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
        placeholder="Try “state”, “rendering”, or “effects”"
      />

      <label className="mode-toggle">
        <input
          type="checkbox"
          checked={deferredMode}
          onChange={(event) => setDeferredMode(event.target.checked)}
        />
        Use a deferred query for the results
      </label>

      <p className="status-line" role="status" aria-live="polite">
        {isStale
          ? "Showing previous results while the list catches up."
          : deferredMode
            ? "Results reflect the current search."
            : "Deferred mode is off; results update with the input."}
      </p>

      <div
        className={isStale ? "results-region results-region--stale" : "results-region"}
        aria-busy={isStale}
      >
        <TopicResults query={resultsQuery} />
      </div>
    </section>
  );
}

export default function App() {
  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React rendering priority</p>
        <h1>Let a slower view catch up</h1>
        <p className="page-intro">
          The search field always uses its immediate state. The list can either
          receive that query directly or use the deferred version.
        </p>
      </header>

      <DeferredSearchDemo />

      <aside className="explanation" aria-labelledby="rules-heading">
        <h2 id="rules-heading">What to notice</h2>
        <ul>
          <li>The input stays controlled by the latest query in both modes.</li>
          <li>
            With deferred mode on, the memoized list can skip the urgent parent
            render while its deferred query is unchanged.
          </li>
          <li>
            The old list remains visible while it catches up; the status and
            busy state describe that stale content.
          </li>
          <li>
            Deferred rendering has no fixed delay and does not reduce network
            requests.
          </li>
        </ul>
      </aside>

      <footer className="page-footer">
        The per-row calculation is artificial demonstration work. React can
        yield between component work, not halfway through one synchronous loop.
      </footer>
    </main>
  );
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
input {
  font: inherit;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #6c63ff;
  outline-offset: 3px;
}

.page-shell {
  width: min(100% - 2rem, 940px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 740px;
  margin-bottom: 1.5rem;
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
  font-size: clamp(2rem, 5vw, 3.25rem);
  letter-spacing: -0.045em;
}

.page-intro,
.panel-copy {
  margin-top: 0.8rem;
  color: #536078;
  line-height: 1.65;
}

.panel,
.explanation {
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

.field-label {
  display: block;
  margin-top: 1.2rem;
  font-size: 0.9rem;
  font-weight: 700;
}

.search-input {
  width: 100%;
  min-height: 2.8rem;
  margin-top: 0.45rem;
  padding: 0.65rem 0.8rem;
  border: 1px solid #bfc8d8;
  border-radius: 0.55rem;
  color: #172033;
  background: #fff;
}

.search-input::placeholder {
  color: #718096;
}

.mode-toggle {
  display: flex;
  gap: 0.55rem;
  align-items: center;
  margin-top: 0.9rem;
  color: #354158;
  font-size: 0.88rem;
}

.mode-toggle input {
  width: 1rem;
  height: 1rem;
  accent-color: #5145b8;
}

.status-line {
  min-height: 1.5rem;
  margin-top: 0.85rem;
  color: #5145b8;
  font-size: 0.88rem;
  font-weight: 700;
}

.results-region {
  max-height: 34rem;
  margin-top: 0.65rem;
  overflow: auto;
  border: 1px solid #e1e6ef;
  border-radius: 0.7rem;
  background: #fbfcfe;
}

.results-region--stale {
  opacity: 0.62;
}

.result-count {
  position: sticky;
  top: 0;
  z-index: 1;
  padding: 0.75rem 0.85rem;
  border-bottom: 1px solid #e1e6ef;
  color: #536078;
  background: #f7f8fc;
  font-size: 0.82rem;
  font-weight: 700;
}

.topic-list {
  display: grid;
  gap: 0.55rem;
  padding: 0.75rem;
  list-style: none;
}

.topic-card,
.empty-state {
  padding: 0.8rem;
  border: 1px solid #e1e6ef;
  border-radius: 0.6rem;
  background: #fff;
}

.topic-card__heading {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 0.5rem;
  align-items: center;
}

.topic-card h3 {
  font-size: 0.98rem;
}

.topic-card__heading span {
  padding: 0.22rem 0.5rem;
  border-radius: 999px;
  color: #5145b8;
  background: #efedff;
  font-size: 0.72rem;
  font-weight: 700;
}

.topic-card p {
  margin-top: 0.35rem;
  color: #536078;
  font-size: 0.86rem;
  line-height: 1.5;
}

.empty-state {
  color: #536078;
}

.explanation {
  margin-top: 1rem;
  box-shadow: none;
}

.explanation h2 {
  font-size: 1.1rem;
}

.explanation ul {
  display: grid;
  gap: 0.55rem;
  margin-top: 0.7rem;
  padding-left: 1.25rem;
  color: #536078;
  line-height: 1.55;
}

.page-footer {
  margin-top: 1rem;
  padding: 1rem 1.1rem;
  border-left: 4px solid #6256ce;
  color: #536078;
  background: #ebe9ff;
  line-height: 1.6;
}

@media (max-width: 600px) {
  .page-shell {
    width: min(100% - 1rem, 560px);
    padding-block: 1.5rem;
  }

  .panel,
  .explanation {
    padding: 1rem;
  }
}
~~~

## Run the experiment

1. Leave **Use a deferred query for the results** selected.
2. Type a search term such as “state” or “rendering”. The field updates immediately; the result region may briefly keep the previous query and displays a stale status if it is still catching up.
3. Turn deferred mode off and type again. The memoized results component now receives the input query directly, so its row work happens during the urgent update.
4. Search for text with no match, then clear the field to restore all topics.
5. Try the same interaction on another device. A fast device may complete the deferred render so quickly that the stale state is hard to notice.

## Why the list can skip an urgent render

**DeferredSearchDemo** owns the current **query**, which controls the input. **useDeferredValue** returns **deferredQuery** for the slower result region. While React is handling a keystroke, the deferred value can still equal the value from the previous render.

**TopicResults** is wrapped in **memo** and receives the same query until the deferred value catches up. It can therefore skip the urgent parent render. React later renders it using the latest deferred query. If the child were not memoized, it could repeat its filtering and row rendering during the urgent parent update anyway.

The source and deferred query are strings, so comparing them with **!==** is suitable for the stale indicator. The actual result region gets **aria-busy** only while it is showing results for an older query.

## About the simulated row work

Each **TopicCard** runs a deterministic, side-effect-free loop while rendering. It is there to make rendering cost noticeable; it is not an optimization technique and does not represent typical production row code.

React may yield between component work, but it cannot interrupt the loop inside one **TopicCard** halfway through. In a real app, first measure the slow part, then optimize the component or data work. For very large collections, consider list virtualization as a separate technique.

## Key points

- Keep the controlled input tied to the current query.
- Pass the deferred query to a memoized slower consumer.
- Use the source/deferred difference for stale feedback, not as a network status.
- There is no fixed delay, and a quick render may show no visible lag.
- **useDeferredValue** does not make the calculation faster or limit requests.
