# 014. Props and passing data - Example

## Goal

Build a small course catalog that passes different data into one reusable component. The example demonstrates:

- Passing strings, numbers, objects, arrays, and booleans through JSX.
- Reading values from the component's props object.
- Passing a callback function to a child.
- Rendering nested JSX through the `children` prop.
- Using `key` for a mapped list without treating it as a regular prop.

## Files

In an existing Vite React project, create or update these files:

- src/main.jsx
- src/App.jsx
- src/components/CourseCard.jsx

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

## src/components/CourseCard.jsx

~~~jsx
export default function CourseCard(props) {
  return (
    <article>
      <p>
        {props.course.level} | {props.course.duration} minutes
      </p>

      {props.featured && <p>Featured lesson</p>}

      <h3>{props.course.title}</h3>
      <p>{props.course.summary}</p>

      <ul>
        {props.topics.map((topic) => (
          <li key={topic}>{topic}</li>
        ))}
      </ul>

      <button type="button" onClick={props.onOpen}>
        Select lesson
      </button>
    </article>
  );
}
~~~

The component reads each value from `props`. The `course` prop is an object, `topics` is an array, `featured` is a boolean, and `onOpen` is a function. The button gives React the callback to run on click; the callback is not called during rendering.

## src/App.jsx

~~~jsx
import CourseCard from './components/CourseCard.jsx';

const courses = [
  {
    id: 'components',
    title: 'Function components',
    level: 'Beginner',
    duration: 20,
    summary: 'Name and compose the functions that describe your interface.',
    topics: ['Component names', 'JSX composition'],
    featured: true,
  },
  {
    id: 'props',
    title: 'Props and passing data',
    level: 'Beginner',
    duration: 25,
    summary: 'Configure a component with values supplied by its parent.',
    topics: ['Strings and expressions', 'Objects and callbacks'],
    featured: false,
  },
];

function NotePanel(props) {
  return (
    <aside>
      <h2>{props.title}</h2>
      {props.children}
    </aside>
  );
}

export default function App() {
  function handleCourseOpen(title) {
    console.log('Selected lesson:', title);
  }

  return (
    <main>
      <header>
        <h1>React course catalog</h1>
        <p>Choose a lesson to explore.</p>
      </header>

      <NotePanel title="How this page works">
        <p>
          The parent supplies course data, and each card renders the values it
          receives.
        </p>
      </NotePanel>

      <section aria-labelledby="courses-title">
        <h2 id="courses-title">Available lessons</h2>

        {courses.map((course) => (
          <CourseCard
            key={course.id}
            course={course}
            topics={course.topics}
            featured={course.featured}
            onOpen={() => handleCourseOpen(course.title)}
          />
        ))}
      </section>
    </main>
  );
}
~~~

`App` owns the course data and passes each value to `CourseCard`. The object and its topic array are supplied as props. The boolean controls whether the featured message appears. The callback receives the selected title and runs after a button click.

The list's `key` helps React identify each card while rendering the array. React does not pass `key` to `CourseCard` as a normal prop. The course object has its own `id` field if a component needs to read that data.

`NotePanel` receives the nested paragraph through `props.children` and decides where to render it.

## Run the example

From the project folder that contains `package.json`, start Vite:

~~~sh
npm run dev
~~~

Open the local address printed by Vite. The page should show an explanation panel and two course cards with different titles, durations, topics, and featured states. Select either button and check the browser developer console for the selected lesson title.

## Try these changes

1. Change a title, duration, summary, or topic in the `courses` array. The matching card should display the new value.
2. Change one course's `featured` value to `true` or `false` and observe the featured message.
3. Add another course object with a unique `id`. The mapping should render another card.
4. Change `handleCourseOpen` to log a different message. Both cards use the callback supplied by the parent.
5. Replace the paragraph inside `NotePanel` with another element. The panel should render the new nested content through `props.children`.
6. Remove `key={course.id}` temporarily and inspect the development console. Restore the key after observing React's list warning.

## What this example demonstrates

- The parent controls the data and passes it down through JSX.
- A child can read primitive values and fields from an object prop.
- Arrays can be used to render repeated content; each mapped element has a stable key.
- A boolean prop can control whether a section of UI appears.
- A callback prop lets a child request an action from the parent.
- Nested JSX is received as `props.children`.
- Props are inputs to a component; the child does not mutate the parent's data.

## Chapter navigation

| Lesson | Q&A | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [014 - Props and passing data](../lessons/014-props-and-passing-data.md) | [Open Q&A](../qnas/014-props-and-passing-data.md) | [013 - Function components and component naming](../lessons/013-function-components-and-naming.md) | [015 - Destructuring props and default values](../lessons/015-destructuring-props-and-default-values.md) |
