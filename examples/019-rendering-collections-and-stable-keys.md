# 019. Rendering collections and choosing stable keys - Example

## Goal

Build a small React learning page from arrays of data. It demonstrates:

- Rendering repeated rows with `map()`.
- Filtering the source array before rendering.
- Using a stable record ID as each mapped sibling's key.
- Passing the same ID under a normal prop when a child component needs it.
- Showing an empty-state message when a filtered array has no items.
- Using a keyed Fragment when one glossary record produces two sibling elements.

## Files

In an existing Vite React project, create or update these files:

- `src/main.jsx`
- `src/App.jsx`

No extra package or stylesheet is required.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
import { Fragment } from 'react';

const lessons = [
  {
    id: 'jsx-basics',
    title: 'Components and JSX',
    description: 'Describe an interface with components and JSX.',
    level: 'beginner',
  },
  {
    id: 'props',
    title: 'Passing data with props',
    description: 'Pass values from a parent component to a child.',
    level: 'beginner',
  },
  {
    id: 'composition',
    title: 'Composition and layouts',
    description: 'Arrange UI with children and named content.',
    level: 'beginner',
  },
  {
    id: 'conditional-ui',
    title: 'Conditional rendering',
    description: 'Choose which interface to display for a condition.',
    level: 'intermediate',
  },
  {
    id: 'collections',
    title: 'Collections and stable keys',
    description: 'Render data collections and give each item a stable key.',
    level: 'intermediate',
  },
];

const glossary = [
  {
    id: 'record',
    term: 'Record',
    definition: 'One object that contains the values for a data item.',
  },
  {
    id: 'sibling',
    term: 'Sibling',
    definition: 'An element rendered beside another element under the same parent.',
  },
];

function LessonRow({ lessonId, title, description }) {
  return (
    <li id={`lesson-${lessonId}`}>
      <article>
        <h3>{title}</h3>
        <p>{description}</p>
      </article>
    </li>
  );
}

function LessonList({ heading, items, emptyMessage }) {
  return (
    <section>
      <h2>{heading}</h2>

      {items.length === 0 ? (
        <p>{emptyMessage}</p>
      ) : (
        <ul>
          {items.map((lesson) => (
            <LessonRow
              key={lesson.id}
              lessonId={lesson.id}
              title={lesson.title}
              description={lesson.description}
            />
          ))}
        </ul>
      )}
    </section>
  );
}

function Glossary({ entries }) {
  return (
    <section>
      <h2>Glossary</h2>
      <dl>
        {entries.map((entry) => (
          <Fragment key={entry.id}>
            <dt>{entry.term}</dt>
            <dd>{entry.definition}</dd>
          </Fragment>
        ))}
      </dl>
    </section>
  );
}

export default function App() {
  const beginnerLessons = lessons.filter(
    (lesson) => lesson.level === 'beginner',
  );
  const advancedLessons = lessons.filter(
    (lesson) => lesson.level === 'advanced',
  );

  return (
    <main>
      <h1>React learning path</h1>

      <LessonList
        heading="Beginner lessons"
        items={beginnerLessons}
        emptyMessage="No beginner lessons are available."
      />

      <LessonList
        heading="Advanced lessons"
        items={advancedLessons}
        emptyMessage="No advanced lessons are available yet."
      />

      <Glossary entries={glossary} />
    </main>
  );
}
~~~

## What to notice

- `lessons` and `glossary` are arrays of records. Each record has an `id` that stays with that item.
- `filter()` derives the beginner and advanced lesson arrays. The source array is left unchanged.
- `LessonList` maps over its `items` and places `key={lesson.id}` on the `LessonRow` returned for each record.
- `lessonId` is passed separately because `key` is reserved for React and is not available as a component prop. `LessonRow` uses that regular prop to create a document `id`.
- The advanced filter currently has no matching records, so the empty-state message appears. Change one lesson's `level` to `advanced` to see the filtered row render.
- Each glossary record creates both a `<dt>` and a `<dd>`. The keyed `Fragment` groups them for React without adding a wrapper element.

## Practice

1. Add a lesson with a new stable ID and confirm it appears in the correct filtered group.
2. Change the beginner filter to select one lesson by ID instead of by level.
3. Add another glossary record with its own ID.
4. Temporarily replace `key={lesson.id}` with `key={Math.random()}` and observe that the keys change whenever the list renders again, so React treats the rows as new items. Restore the stable ID afterward.
5. Try `key={index}` and explain why that choice becomes unsafe if the lessons are reordered or removed.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [018 - Conditional rendering patterns](../lessons/018-conditional-rendering-patterns.md) | [019 - Rendering collections and choosing stable keys](../lessons/019-rendering-collections-and-stable-keys.md) | [Open Q&A](../qnas/019-rendering-collections-and-stable-keys.md) | [020 - Component boundaries and responsibility](../lessons/020-component-boundaries-and-responsibility.md) |