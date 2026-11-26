# 004. Example: Use JavaScript data and functions in a React component

## Goal

Build a small course list using JavaScript objects, arrays, functions, modules, expressions in JSX, and one browser event handler.

The example is static from React's point of view: the button logs to the browser Console but does not update UI state. State is introduced later in the course.

## What you will practice

- Keeping data and a helper function in a JavaScript module.
- Importing named exports into a JSX module.
- Destructuring props and object properties.
- Filtering an array and mapping its items into a list.
- Passing an event handler function to a button.
- Using browser developer tools to observe the handler.

## Prerequisite

Use a React project with a working JSX build setup. The project entry point and its `App` import should already be configured. This example only adds the two source files below.

## Step 1: Create the course data module

Create `src/courseData.js`:

```js
export const courses = [
  {
    id: 'javascript',
    title: 'JavaScript',
    summary: 'Practice functions, arrays, objects, and modules.',
    durationMinutes: 30,
    published: true,
  },
  {
    id: 'jsx',
    title: 'JSX',
    summary: 'Describe user interfaces with JavaScript expressions.',
    durationMinutes: 25,
    published: true,
  },
  {
    id: 'server-rendering',
    title: 'Server rendering',
    summary: 'Explore how React can produce HTML before the browser runs.',
    durationMinutes: 40,
    published: false,
  },
];

export function formatDuration(minutes) {
  return `${minutes} minutes`;
}
```

The array contains course objects. Each object has the same shape: an ID, title, summary, duration, and publication flag. `formatDuration` is a named export that converts a number into display text.

## Step 2: Create the React page

Create or update `src/App.jsx`:

```jsx
import { courses, formatDuration } from './courseData.js';

function CourseCard({ course }) {
  const { title, summary, durationMinutes } = course;

  function handlePreview() {
    console.log(`Selected course: ${title}`);
  }

  return (
    <li className="course-item">
      <article>
        <h2>{title}</h2>
        <p>{summary}</p>
        <p>Estimated time: {formatDuration(durationMinutes)}</p>
        <button type="button" onClick={handlePreview}>
          Log title to Console
        </button>
      </article>
    </li>
  );
}

export default function App() {
  const availableCourses = courses.filter(course => course.published);

  return (
    <main>
      <h1>Available courses</h1>
      <p>{availableCourses.length} topics are currently available.</p>

      <ul>
        {availableCourses.map(course => (
          <CourseCard key={course.id} course={course} />
        ))}
      </ul>
    </main>
  );
}
```

## Expected result

The page displays two course cards: **JavaScript** and **JSX**. The unpublished **Server rendering** course is filtered out.

Each card displays its title, summary, estimated duration, and a button. When you click a button, the browser Console displays a message such as:

```text
Selected course: JavaScript
```

The current UI does not change after the click. The event is used only to demonstrate how a JavaScript function can be passed to a React event prop.

## Trace the JavaScript and React parts

### Data module

`courseData.js` exports a named array and a named function. `App.jsx` imports both names inside braces.

### Filtering

`courses.filter(course => course.published)` returns a new array containing only the published courses. It does not modify the original data array.

### Rendering with `map`

`availableCourses.map(...)` transforms each remaining course into a `CourseCard`. Each card receives its course object through a prop and has a stable `key` based on the course ID.

### Destructuring and formatting

`CourseCard` destructures the `course` prop, then reads its properties. The JSX braces evaluate variables and call `formatDuration` to create display text.

### Event callback

`onClick={handlePreview}` passes a function to the browser button through React. React calls it when the user clicks. It logs to the browser Console, not to a server or a React state store.

## Follow a click through the layers

```text
user clicks the HTML button
          |
          v
browser dispatches a click event
          |
          v
React calls handlePreview
          |
          v
JavaScript writes a message to Console
```

Because the handler does not update state, React has no new UI state to render. A later lesson will add state and show how the page changes in response to an interaction.

## Try it yourself

1. Add a fourth course object and set `published: true`.
2. Set one existing course to `published: false` and observe that it disappears.
3. Change `formatDuration` to display `min` instead of `minutes`.
4. Change the Console message to include the course ID.
5. Rename `formatDuration` in the export, then update its import and use.
6. Open the browser Console, click both buttons, and compare the messages.

### Suggested change for a shorter duration label

Update the helper in `courseData.js`:

```js
export function formatDuration(minutes) {
  return `${minutes} min`;
}
```

The component imports the helper, so it displays the new format without duplicating the formatting logic in every card.

## Common mistakes

- **Import and export names do not match:** named imports must use a name exported by the target module.
- **The file path is wrong:** `./courseData.js` is relative to `App.jsx`; check file name and letter casing.
- **The map callback returns nothing:** use parentheses for an implicit JSX return or an explicit `return` inside braces.
- **The list item has no key:** provide a stable ID so React can match list entries. The ID is not automatically passed as a prop.
- **The click handler runs during rendering:** pass `onClick={handlePreview}`, not `onClick={handlePreview()}`.
- **The click handler is expected to change the page:** this handler only logs. A visible update requires state, which comes later.
- **A browser error is hidden:** check the Console and Network panels in developer tools.

## Completion checklist

- [ ] `courseData.js` exports the array and formatting function.
- [ ] `App.jsx` imports both named exports successfully.
- [ ] The unavailable course is filtered out.
- [ ] The two available courses appear with formatted durations.
- [ ] Each rendered card uses the course ID as its key.
- [ ] Clicking a button logs the selected title in the browser Console.
- [ ] You can explain which code is JavaScript, which code describes React UI, and which behavior comes from the browser.

## Related chapters

- [Lesson 004: JavaScript and browser prerequisites](../lessons/004-javascript-and-browser-prerequisites.md)
- [Q&A 004: JavaScript and browser prerequisites](../qnas/004-javascript-and-browser-prerequisites.md)
- [Next: Choose a React project setup](../lessons/005-choose-a-react-project-setup.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="003-react-react-dom-and-frameworks.md">003 - Example: Map a React app to its ecosystem layers</a></td><td><a href="../lessons/004-javascript-and-browser-prerequisites.md">Lesson</a></td><td><a href="../qnas/004-javascript-and-browser-prerequisites.md">Q&amp;A</a></td><td><a href="005-choose-a-react-project-setup.md">005 - Example: Choose and start a React learning project</a></td></tr>
  </tbody>
</table>
