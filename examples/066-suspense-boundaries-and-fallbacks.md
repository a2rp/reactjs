# Example: Suspense Boundaries and Progressive Fallbacks

This runnable React app compares two boundary layouts over the same cached Promise resources:

- **Reveal together:** one Suspense boundary keeps the summary and lesson list behind one fallback until both are ready.
- **Reveal progressively:** an outer boundary reveals the summary first; a nested boundary shows its own fallback while the lesson list is still pending.

The timers simulate two independent async resources. The app uses no network requests. Its stable Promise objects are created by a click handler and stored in state, not recreated during rendering.

## Create the project

~~~sh
npm create vite@latest suspense-boundaries-demo -- --template react
cd suspense-boundaries-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following and start the app:

~~~sh
npm run dev
~~~

## Project files

~~~text
suspense-boundaries-demo/
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
    <title>React Suspense Boundaries Demo</title>
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
import { Suspense, use, useState } from "react";

const LESSONS = [
  "Choose a meaningful reveal group",
  "Keep loading feedback near pending content",
  "Preserve useful page context",
];

function waitForValue(value, delay) {
  return new Promise((resolve) => {
    window.setTimeout(() => resolve(value), delay);
  });
}

function createCourseResources() {
  return {
    summary: waitForValue(
      {
        title: "Build an accessible course dashboard",
        description:
          "Plan the page shell, course summary, lesson list, and loading states.",
      },
      700,
    ),
    lessons: waitForValue(LESSONS, 1800),
  };
}

function LoadingCard({ title, description, compact = false }) {
  return (
    <div
      className={compact ? "loading-card loading-card--compact" : "loading-card"}
      role="status"
      aria-live="polite"
    >
      <span className="loading-mark" aria-hidden="true" />
      <div>
        <h3>{title}</h3>
        <p>{description}</p>
      </div>
    </div>
  );
}

function SummaryContent({ summary }) {
  return (
    <section className="content-card" aria-labelledby="course-title">
      <p className="eyebrow">Course summary</p>
      <h3 id="course-title">{summary.title}</h3>
      <p>{summary.description}</p>
    </section>
  );
}

function LessonsContent({ lessons }) {
  return (
    <section className="content-card" aria-labelledby="lessons-title">
      <p className="eyebrow">Course plan</p>
      <h3 id="lessons-title">Included lessons</h3>
      <ol className="lesson-list">
        {lessons.map((lesson, index) => (
          <li key={lesson}>
            <span className="lesson-number">
              {String(index + 1).padStart(2, "0")}
            </span>
            {lesson}
          </li>
        ))}
      </ol>
    </section>
  );
}

function LessonResource({ promise }) {
  const lessons = use(promise);
  return <LessonsContent lessons={lessons} />;
}

function RevealTogether({ resources }) {
  const summary = use(resources.summary);
  const lessons = use(resources.lessons);

  return (
    <div className="revealed-content">
      <SummaryContent summary={summary} />
      <LessonsContent lessons={lessons} />
    </div>
  );
}

function RevealProgressively({ resources }) {
  const summary = use(resources.summary);

  return (
    <div className="revealed-content">
      <SummaryContent summary={summary} />
      <Suspense
        fallback={
          <LoadingCard
            compact
            title="Loading the lesson list"
            description="The course summary is already available."
          />
        }
      >
        <LessonResource promise={resources.lessons} />
      </Suspense>
    </div>
  );
}

function SuspenseDemo() {
  const [mode, setMode] = useState(null);
  const [resources, setResources] = useState(null);

  function startDemo(nextMode) {
    setMode(nextMode);
    setResources(createCourseResources());
  }

  function resetDemo() {
    setResources(null);
    setMode(null);
  }

  return (
    <section className="panel" aria-labelledby="demo-title">
      <p className="eyebrow">One resource pair, two boundary layouts</p>
      <h2 id="demo-title">Choose how the course content appears</h2>
      <p className="panel-copy">
        Both resources start together. The summary resolves first, followed
        later by the lesson list.
      </p>

      <div className="controls">
        <button
          className="button button--primary"
          type="button"
          disabled={resources !== null}
          onClick={() => startDemo("together")}
        >
          Reveal together
        </button>
        <button
          className="button"
          type="button"
          disabled={resources !== null}
          onClick={() => startDemo("progressive")}
        >
          Reveal progressively
        </button>
        {resources && (
          <button className="button button--quiet" type="button" onClick={resetDemo}>
            Reset
          </button>
        )}
      </div>

      <p className="mode-description" role="status" aria-live="polite">
        {mode === "together"
          ? "One boundary keeps both sections behind its fallback."
          : mode === "progressive"
            ? "The summary can appear before the nested lesson-list boundary resolves."
            : "Choose a reveal pattern to start the simulated resources."}
      </p>

      {resources && (
        <div className="course-region" role="region" aria-label="Course content">
          {mode === "together" ? (
            <Suspense
              fallback={
                <LoadingCard
                  title="Loading the course"
                  description="The summary and lesson list will appear together."
                />
              }
            >
              <RevealTogether resources={resources} />
            </Suspense>
          ) : (
            <Suspense
              fallback={
                <LoadingCard
                  title="Loading the course summary"
                  description="The course page is waiting for its first section."
                />
              }
            >
              <RevealProgressively resources={resources} />
            </Suspense>
          )}
        </div>
      )}
    </section>
  );
}

export default function App() {
  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React Suspense</p>
        <h1>Put fallbacks around meaningful content</h1>
        <p className="page-intro">
          Suspense boundaries control which parts of the interface are
          replaced while a supported resource is pending.
        </p>
      </header>

      <SuspenseDemo />

      <aside className="explanation" aria-labelledby="notes-title">
        <h2 id="notes-title">What this demo teaches</h2>
        <ul>
          <li>
            The fallback belongs to the nearest boundary above the content
            that suspends.
          </li>
          <li>
            A single boundary reveals its grouped content together.
          </li>
          <li>
            A nested boundary can reveal ready outer content and keep a local
            fallback for its slower section.
          </li>
          <li>
            The Promise resources are created outside rendering and reused
            across retries.
          </li>
        </ul>
      </aside>

      <footer className="page-footer">
        The delay is only a local stand-in for async data. An ordinary request
        in an Effect does not activate Suspense by itself.
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

button {
  font: inherit;
}

button:focus-visible {
  outline: 3px solid #6c63ff;
  outline-offset: 3px;
}

.page-shell {
  width: min(100% - 2rem, 980px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 760px;
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

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-top: 1.2rem;
}

.button {
  min-height: 2.65rem;
  padding: 0.6rem 0.85rem;
  border: 1px solid #c7cfe0;
  border-radius: 0.55rem;
  color: #28334a;
  background: #fff;
  cursor: pointer;
  font-weight: 700;
}

.button:hover:not(:disabled) {
  border-color: #6256ce;
  color: #5145b8;
}

.button--primary {
  border-color: #5145b8;
  color: #fff;
  background: #5145b8;
}

.button--primary:hover:not(:disabled) {
  color: #fff;
  background: #4035a0;
}

.button--quiet {
  border-color: transparent;
  background: #f1f2f7;
}

.button:disabled {
  cursor: not-allowed;
  opacity: 0.58;
}

.mode-description {
  min-height: 1.5rem;
  margin-top: 0.9rem;
  color: #5145b8;
  font-size: 0.88rem;
  font-weight: 700;
  line-height: 1.5;
}

.course-region {
  margin-top: 0.6rem;
}

.revealed-content {
  display: grid;
  gap: 0.75rem;
}

.loading-card,
.content-card {
  display: flex;
  gap: 0.85rem;
  align-items: flex-start;
  min-height: 8rem;
  padding: 1rem;
  border: 1px solid #e1e6ef;
  border-radius: 0.75rem;
  background: #fbfcfe;
}

.loading-card--compact {
  min-height: 5.5rem;
}

.loading-mark {
  flex: 0 0 0.7rem;
  width: 0.7rem;
  height: 0.7rem;
  margin-top: 0.25rem;
  border-radius: 50%;
  background: #6256ce;
}

.loading-card h3,
.content-card h3 {
  font-size: 1rem;
}

.loading-card p,
.content-card > p:not(.eyebrow) {
  margin-top: 0.45rem;
  color: #536078;
  line-height: 1.55;
}

.content-card {
  display: block;
}

.content-card h3 {
  margin-top: 0.4rem;
}

.content-card .eyebrow {
  margin-top: 0;
}

.lesson-list {
  display: grid;
  gap: 0.55rem;
  margin-top: 0.9rem;
  list-style: none;
}

.lesson-list li {
  display: flex;
  gap: 0.65rem;
  align-items: center;
  color: #536078;
  line-height: 1.45;
}

.lesson-number {
  display: inline-grid;
  flex: 0 0 1.8rem;
  width: 1.8rem;
  height: 1.8rem;
  place-items: center;
  border-radius: 50%;
  color: #5145b8;
  background: #efedff;
  font-size: 0.74rem;
  font-weight: 800;
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

  .controls {
    display: grid;
  }
}
~~~

## Run the experiment

1. Select **Reveal together**. The outer fallback stays visible until both the summary and lesson list are ready.
2. Select **Reset**, then choose **Reveal progressively**. The outer fallback shows while the summary is pending.
3. When the summary resolves, it appears. The nested boundary keeps its own fallback visible until the slower lesson list resolves.
4. Reset and repeat either mode. Each run creates a new pair of Promise resources outside rendering and stores them for reuse.
5. Compare which fallback matches the amount of content that is temporarily unavailable.

## Why the two layouts differ

Both buttons call **createCourseResources** in an event handler. It creates two Promise instances with different simulated completion times. The app stores those Promise references in state, so retries read the same cached instances instead of creating new Promises during render.

**RevealTogether** reads both resources under one Suspense boundary. If either one is still pending, that boundary shows its fallback. The summary is not revealed early.

**RevealProgressively** reads the summary under the outer boundary, then renders the lesson resource inside a nested boundary. Once the summary is ready, the outer content can appear while the nested fallback describes the pending lesson list.

The controls remain outside both boundaries, so they remain available while content is pending. The reset button removes the active resource tree and lets you compare the other layout.

## Key points

- A boundary owns the fallback for its suspended descendants.
- One boundary groups content; nested boundaries allow progressive reveal.
- The demo reads Promise resources with React **use**, so they can activate Suspense.
- Reuse the same Promise instance across retries; do not create the resource during render.
- Suspense fallbacks represent pending content. Use an Error Boundary for failures.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="065-usedeferredvalue.md">065 - Example: useDeferredValue with a Search Results List</a></td><td><a href="../lessons/066-suspense-boundaries-and-fallbacks.md">Lesson</a></td><td><a href="../qnas/066-suspense-boundaries-and-fallbacks.md">Q&amp;A</a></td><td><a href="067-lazy-loading-components.md">067 - Example: On-Demand Dashboard Panels with lazy</a></td></tr>
  </tbody>
</table>
