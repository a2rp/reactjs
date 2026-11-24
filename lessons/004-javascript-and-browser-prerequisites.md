# 004. JavaScript and browser prerequisites

## Learning goals

By the end of this lesson, you should be able to:

- Read the basic JavaScript expressions commonly used in React components.
- Recognize variables, values, objects, arrays, functions, and modules.
- Follow JavaScript expressions inside JSX.
- Understand why React code passes event handler functions instead of calling them during render.
- Distinguish browser APIs from React APIs and server-side JavaScript.
- Use the browser developer tools to inspect errors and loaded resources.

You do not need to master all of JavaScript before learning React. You do need enough JavaScript to read component inputs, transform data, and understand what a function will do.

## React is written in JavaScript

React is a JavaScript library. A React component is JavaScript code that returns a description of UI, commonly written with JSX. React does not replace the JavaScript language.

For example, this is an ordinary JavaScript value:

```js
const learnerName = 'Priya';
```

A JSX component can display that value:

```jsx
function Welcome() {
  return <h1>Welcome, {learnerName}</h1>;
}
```

The braces inside JSX let the code evaluate a JavaScript expression. The browser does not execute JSX as written; the configured project tooling transforms it.

## Variables and values

Use `const` when a variable binding will not be reassigned. Use `let` when the binding needs a new value later. Avoid `var` in modern React code unless you are reading older JavaScript.

```js
const courseName = 'React';
let completedLessons = 0;

completedLessons = completedLessons + 1;
```

A `const` binding cannot be reassigned, but an object or array stored in a `const` can still be mutated. In React, treat data used as props or state as immutable and create updated copies when changing it.

Common JavaScript values include:

- Strings: `'React'`
- Numbers: `12`
- Booleans: `true` and `false`
- `null`: an intentional absence of a value
- `undefined`: a value that has not been assigned or provided
- Objects: `{ title: 'React', published: true }`
- Arrays: `['HTML', 'CSS', 'React']`
- Functions: reusable blocks of JavaScript behavior

Use strict equality such as `===` for comparisons unless there is a specific reason to use another comparison operator. JavaScript is dynamically typed, so the same variable can receive values of different types, but predictable data shapes make React code easier to understand.

## Objects and arrays hold application data

An object groups related named values. An array stores an ordered list of values.

```js
const course = {
  id: 'react',
  title: 'React',
  beginnerFriendly: true,
};

const topics = ['components', 'JSX', 'props'];
```

Read an object property with dot notation:

```js
course.title;
```

Use a stable identifier such as `id` when objects will be rendered in a list. React later uses that identifier as a key to match list items across updates.

## Functions describe reusable work

A function can receive values through parameters and return a result:

```js
function formatDuration(minutes) {
  return minutes + ' minutes';
}

formatDuration(25); // '25 minutes'
```

Arrow functions are another common syntax:

```js
const formatDuration = (minutes) => {
  return minutes + ' minutes';
};
```

For a single expression, an arrow function can return implicitly:

```js
const double = number => number * 2;
```

Function values can be passed to other functions. This is important in React because event handlers are functions that React calls later when an interaction happens.

```jsx
<button onClick={handleClick}>Save</button>
```

This passes the function. Writing `onClick={handleClick()}` calls it immediately while the component is rendering. Event behavior is covered in its own lesson.

## Destructuring and template strings

Destructuring extracts values from objects or arrays:

```js
const course = { title: 'React', level: 'Beginner' };
const { title, level } = course;
```

Components often destructure their props in the parameter list:

```jsx
function CourseHeading({ title }) {
  return <h2>{title}</h2>;
}
```

Template strings use backticks and `${...}` to combine values:

```js
const message = `Now learning: ${title}`;
```

These are JavaScript language features, not React features. They are common because React components frequently read data and format it for display.

## Expressions in JSX

JSX allows JavaScript expressions inside curly braces. An expression produces a value:

```jsx
function CourseSummary({ course }) {
  return (
    <article>
      <h2>{course.title}</h2>
      <p>{course.lessonCount} lessons</p>
      <p>{course.published ? 'Available' : 'Coming soon'}</p>
    </article>
  );
}
```

Common expressions in JSX include:

- A variable: `{course.title}`
- A property: `{user.name}`
- A function call: `{formatDuration(course.minutes)}`
- A conditional expression: `{isReady ? 'Ready' : 'Loading'}`
- An array transformation: `{courses.map(course => ...)}`

Use ordinary quotes for a literal string attribute and braces for a JavaScript value:

```jsx
<img alt="Course cover" src={course.imageUrl} />
```

Here `alt` is a string, while `src` reads the value of a JavaScript expression. Writing `src="{course.imageUrl}"` would pass the literal text instead of the variable.

JavaScript object values in JSX need the expression braces around the object literal. For example, an inline style is written as `style={{ color: 'navy' }}`. For most styling, CSS classes are clearer.

## Transforming arrays for UI

JavaScript array methods commonly prepare data for rendering:

```js
const availableCourses = courses.filter(course => course.published);
const courseTitles = availableCourses.map(course => course.title);
```

- `filter` returns a new array containing values that pass a condition.
- `map` returns a new array created by transforming each item.
- `find` returns the first item that matches a condition.
- `length` gives the number of items in an array.

In React, `map` is often used to produce a list of elements:

```jsx
<ul>
  {availableCourses.map(course => (
    <li key={course.id}>{course.title}</li>
  ))}
</ul>
```

A stable key helps React match each item. The details of list rendering and keys are covered in a later lesson.

## Conditions and truthy values

JavaScript conditions can choose a value:

```jsx
<p>{course.published ? 'Available now' : 'Coming soon'}</p>
```

A component can also return different JSX using an `if` statement. The `&&` operator can conditionally include content, but remember that a false expression is not the only possible result. For example, `0 && <p>...</p>` evaluates to `0`, which React displays as text. Use a boolean condition such as `items.length > 0 && ...` when appropriate.

Optional chaining can safely read a nested value that might be missing:

```js
const instructorName = course.instructor?.name;
```

Do not use conditional tricks when a clear `if` statement or ternary is easier to read.

## JavaScript modules

Modules let a project split code across files. `export` makes a value available to other files; `import` reads it.

Default export:

```js
// course.js
export default function formatCourse(course) {
  return course.title.toUpperCase();
}
```

```js
// App.jsx
import formatCourse from './course.js';
```

Named exports:

```js
// courseData.js
export const courseCount = 3;
export function getCourseTitle(course) {
  return course.title;
}
```

```js
// App.jsx
import { courseCount, getCourseTitle } from './courseData.js';
```

A module's path must point to the correct file, and named imports must match exported names. A framework or build tool may impose additional conventions.

## Asynchronous JavaScript at a glance

A network request does not finish immediately. JavaScript represents work that completes later with Promises. An `async` function can use `await` to write that flow in a readable sequence:

```js
async function loadCourses() {
  const response = await fetch('/api/courses');

  if (!response.ok) {
    throw new Error('Could not load courses.');
  }

  return response.json();
}
```

This example requests data from the same site's `/api/courses` endpoint. It only defines a function; it does not run until called. A complete feature must decide when to call it and how to show loading, success, and error states. Those are covered in later lessons.

React does not automatically call `fetch` for a component. Frameworks, data libraries, or application code decide how data loading fits into the app.

## The browser environment

A browser provides APIs that JavaScript can use:

- `document` represents the current HTML document and its DOM.
- `window` represents the browser window and exposes browser-specific APIs.
- `fetch` can make HTTP requests.
- Browser events report actions such as clicks, typing, and form submission.
- The developer tools help inspect elements, network requests, JavaScript errors, and application behavior.

React DOM uses browser DOM APIs as its rendering target. Most React components should describe the UI through JSX rather than manually editing nodes through `document.querySelector`. Refs can provide controlled DOM access for tasks such as focusing an input; the root and component UI should remain under React's ownership.

JavaScript running in a browser and JavaScript running on a server share the language but have different available APIs. For example, browser code has `document`; server code generally does not. Never put private server credentials in browser JavaScript.

## Browser developer tools

The browser's developer tools are useful during React development:

- **Console:** read errors and inspect logged values.
- **Elements/Inspector:** inspect the rendered DOM and applied CSS.
- **Network:** see which documents, scripts, styles, and API requests loaded and what responses they received.
- **Sources/Debugger:** inspect source files, set breakpoints, and step through code.
- **Storage/Application:** inspect cookies, local storage, and other browser-managed data when relevant.

If the page is blank, start with the Console and Network panels. Check for a syntax or import error, a missing asset, a missing root container, or a script that never loaded. Do not treat a manually edited Elements panel as a permanent application change; it changes the current browser page, not your source code.

## Readiness checklist

You are ready to continue if you can:

- Read a simple function and identify its inputs and return value.
- Recognize an object, array, string, number, and boolean.
- Explain the difference between a string literal and a JavaScript expression inside JSX.
- Follow a basic `import` and `export`.
- Explain why `onClick={handleClick}` passes a function and `onClick={handleClick()}` calls it immediately.
- Use browser developer tools to find a console error and check whether a file or request loaded.
- Understand that React, browser APIs, and server APIs have different responsibilities.

You can learn advanced JavaScript alongside React. A basic working familiarity is enough to begin; revisit language features whenever a lesson uses one you have not seen before.

## Key takeaways

- React components are JavaScript functions, and JSX is embedded in JavaScript source.
- Objects and arrays commonly represent application data.
- Functions, destructuring, modules, conditions, and array methods appear frequently in React code.
- JSX braces evaluate JavaScript expressions; quoted attributes are literal strings.
- Pass callback functions for later execution instead of calling them during render.
- Browser APIs and server APIs differ even though both can run JavaScript.
- Use developer tools to inspect the actual browser page and diagnose errors.
- React renders UI; it does not replace JavaScript fundamentals or automatically load backend data.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [003 - React, React DOM, and the framework ecosystem](003-react-react-dom-and-frameworks.md) | [Open Q&A](../qnas/004-javascript-and-browser-prerequisites.md) | [Open Example](../examples/004-javascript-and-browser-prerequisites.md) | [005 - Choose a React project setup](005-choose-a-react-project-setup.md) |
