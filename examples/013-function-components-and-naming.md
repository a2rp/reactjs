# 013. Function components and component naming - Example

## Goal

Build a small lesson page from function components. The example demonstrates:

- Defining components with PascalCase names.
- Using lowercase names for native HTML elements.
- Composing components through JSX.
- Defining component functions at the top level.
- Reusing the same component more than once.
- Keeping one reusable component in its own file.

The two lesson cards intentionally show the same content. Passing different data to each instance is the next chapter's topic.

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

The entry file finds the HTML root and asks React to render the `App` component there. The Vite starter `index.html` already includes an element with the `root` id.

## src/components/CourseCard.jsx

~~~jsx
export default function CourseCard() {
  return (
    <article>
      <p>Lesson 013</p>
      <h3>Function components</h3>
      <p>
        Learn how named functions describe parts of a React interface.
      </p>
      <a href="#lesson-outline">Review the lesson outline</a>
    </article>
  );
}
~~~

`CourseCard` is defined at the top level of this module. Its name starts with a capital letter, and the component returns one article element containing the card content.

## src/App.jsx

~~~jsx
import CourseCard from './components/CourseCard.jsx';

function PageIntro() {
  return (
    <header>
      <p>ReactJS Notes</p>
      <h1>Learn React with small building blocks</h1>
      <p>
        A lesson page composed from reusable function components.
      </p>
    </header>
  );
}

function LessonOutline() {
  return (
    <section id="lesson-outline" aria-labelledby="outline-title">
      <h2 id="outline-title">What you will practice</h2>
      <ul>
        <li>Use a function component in JSX.</li>
        <li>Recognize custom components by their capitalized names.</li>
        <li>Compose a page from smaller pieces.</li>
      </ul>
    </section>
  );
}

export default function App() {
  return (
    <main>
      <PageIntro />

      <section aria-labelledby="featured-title">
        <h2 id="featured-title">Featured lesson</h2>
        <CourseCard />
        <CourseCard />
      </section>

      <LessonOutline />
    </main>
  );
}
~~~

`PageIntro`, `LessonOutline`, and `App` are defined at module scope. `App` composes them by using capitalized JSX names. Names such as `main`, `header`, `section`, `h2`, and `ul` are lowercase native HTML elements.

The two `<CourseCard />` elements render the same component definition at two places. Updating the component's markup updates both instances. Each instance is still represented separately in the rendered component tree.

## Run the example

From the project folder that contains `package.json`, start Vite:

~~~sh
npm run dev
~~~

Open the local address printed by Vite. The page should show a heading, two identical lesson cards, and a short outline. Selecting either card's link moves to the outline section on the same page.

## Try these changes

1. Change the heading or description inside `CourseCard`. Both displayed cards should reflect the one component change.
2. Remove one `<CourseCard />` line from `App`. The page should show one card while the component definition remains available.
3. Add a third `<CourseCard />` element. It should render another copy of the same card.
4. Rename `PageIntro` to `CourseIntro` in both its function declaration and JSX usage. Keep the first letter capitalized.
5. Move `CourseCard` into `App.jsx` temporarily and update the import. Then decide which file arrangement is easier to follow for this small project.

## What this example demonstrates

- A component is a JavaScript function rendered through JSX.
- PascalCase names identify custom components in JSX; lowercase names identify native elements.
- Components can be composed visually while their function definitions stay at module scope.
- A component definition can be reused at several locations.
- Small components can be grouped or moved into separate modules based on clarity and reuse.
- Props, which let component instances display different data, are introduced in the next chapter.

## Chapter navigation

| Lesson | Q&A | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [013 - Function components and component naming](../lessons/013-function-components-and-naming.md) | [Open Q&A](../qnas/013-function-components-and-naming.md) | [012 - Strict Mode and development checks](../lessons/012-strict-mode-and-development-checks.md) | [014 - Props and passing data](../lessons/014-props-and-passing-data.md) |
