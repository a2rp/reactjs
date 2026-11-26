# 015. Destructuring props and default values - Example

## Goal

Build lesson cards that use destructured props, aliases, default values, and rest syntax. The page also makes the difference between an omitted prop, `undefined`, `null`, and `0` visible.

## Files

In an existing Vite React project, create or update these files:

- src/main.jsx
- src/App.jsx
- src/components/LessonCard.jsx

No additional package or CSS file is required.

## src/main.jsx

~~~jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('The root element was not found in index.html.');
}

createRoot(rootElement).render(<App />);
~~~

The Vite starter `index.html` should contain an element with the `root` id.

## src/components/LessonCard.jsx

~~~jsx
const EMPTY_TOPICS = [];

export default function LessonCard({
  title: heading = 'Untitled lesson',
  level = 'Beginner',
  duration = 20,
  topics = EMPTY_TOPICS,
  featured = false,
  ...articleProps
}) {
  return (
    <article {...articleProps}>
      {featured && <p>Featured lesson</p>}

      <h2>{heading}</h2>
      <p>Level: {level === null ? 'No level supplied' : level}</p>
      <p>Duration: {duration} minutes</p>

      <ul>
        {topics.map((topic) => (
          <li key={topic}>{topic}</li>
        ))}
      </ul>
    </article>
  );
}
~~~

The `title` prop is locally named `heading`. The component supplies defaults for missing or undefined values. If `level` is explicitly null, the default does not apply and the example displays a separate message. The value `0` remains a valid duration.

The component consumes `featured` itself. Other props, such as `id`, `className`, and `data-section`, are collected into `articleProps` and forwarded to the article element.

## src/App.jsx

~~~jsx
import LessonCard from './components/LessonCard.jsx';

const starterTopics = [
  'Destructure a props object',
  'Choose default values',
  'Forward remaining attributes',
];

function InstructorDetails({ instructor }) {
  if (instructor == null) {
    return <p>Instructor details are not available.</p>;
  }

  const {
    name = 'Guest instructor',
    role = 'Instructor',
  } = instructor;

  return (
    <p>
      Instructor: {name} ({role})
    </p>
  );
}

export default function App() {
  return (
    <main>
      <h1>Destructuring props in practice</h1>

      <LessonCard
        title="Destructuring props"
        level="Beginner"
        duration={25}
        topics={starterTopics}
        featured
        id="destructuring"
        className="lesson-card"
        data-section="destructuring"
      />

      <LessonCard
        title="Using defaults"
        duration={0}
        id="defaults"
        data-section="defaults"
      />

      <LessonCard
        title={undefined}
        level={null}
        duration={0}
        featured={false}
        topics={[]}
        id="explicit-values"
        data-section="explicit-values"
      />

      <InstructorDetails instructor={null} />
    </main>
  );
}
~~~

The first card receives all its main values explicitly. The second omits `level`, `topics`, and `featured`, so the component uses their defaults; its zero duration is preserved. The third passes `undefined` for `title`, which also triggers the default, while `null` for `level` is handled explicitly.

`InstructorDetails` checks the parent object before reading its fields. The field defaults only run after the component knows that the `instructor` object exists.

## Run the example

From the project folder that contains `package.json`, start Vite:

~~~sh
npm run dev
~~~

Open the local address printed by Vite. You should see three lesson cards. The second card shows the Beginner level, a 0 minute duration, no topic items, and no featured label. The third card shows the title default and the explicit null-level message.

## Try these changes

1. Remove `title="Destructuring props"` from the first card. It should use `Untitled lesson`.
2. Pass `title={undefined}` to the second card. The same title default should apply.
3. Pass `title={null}` and observe that the parameter default does not replace null. Restore a text title afterward.
4. Change `duration={0}` to `duration={undefined}`. The default duration should apply; restore zero to see that it is preserved.
5. Remove `topics={starterTopics}` from the first card. The module-level empty array supplies the default.
6. Add a valid DOM attribute, such as `aria-label="Lesson card"`, to a `LessonCard` use. It should be forwarded through `articleProps`.
7. Add a component-only prop such as `featured` and inspect the article in browser developer tools. It is consumed by the component and is not forwarded as an article attribute.
8. Change `instructor={null}` to `instructor={{ name: 'Ada', role: 'Teacher' }}`. The component should render the provided fields.

## What this example demonstrates

- Destructuring gives a component local names for selected props.
- Aliases change local variable names without changing the parent's JSX attributes.
- Defaults apply to omitted or undefined values, not null, false, zero, or an empty string.
- A module-level array can provide a stable fallback for an omitted array prop.
- A nested object is checked before its fields are read.
- Rest props can forward appropriate DOM attributes after component-specific props are consumed.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="014-props-and-passing-data.md">014 - Props and passing data - Example</a></td><td><a href="../lessons/015-destructuring-props-and-default-values.md">Lesson</a></td><td><a href="../qnas/015-destructuring-props-and-default-values.md">Q&amp;A</a></td><td><a href="016-children-prop-and-nested-content.md">016 - The children prop and nested content - Example</a></td></tr>
  </tbody>
</table>
