# 013. Function components and component naming - Q&A

## Component fundamentals

### 1. What is a function component?

A function component is a JavaScript function that React uses to describe part of the interface. It returns a React node, commonly JSX, and is rendered by using its component name in JSX.

~~~jsx
function Welcome() {
  return <h1>Welcome</h1>;
}
~~~

### 2. Is a component the same thing as a DOM element?

No. A component is a JavaScript function and a boundary in the React component tree. Its returned React node describes the UI. React handles how that UI is represented in the browser DOM.

### 3. What is the difference between a component and a React element?

A component is the function definition, such as `Welcome`. A JSX expression such as `<Welcome />` creates an element description that tells React to render that component at that point in the tree.

### 4. Does every function that returns JSX count as a component?

Not automatically. A function used as a component should follow the component conventions and be rendered by React through JSX. A helper function may return a value or even a JSX fragment for another function to use, but it is not a component boundary unless it is rendered as one.

### 5. What can a function component return?

It can return a React node. JSX is the common choice. A component can also return a string, a number, an array of nodes, or `null` when those values fit the UI. It should return a valid React node rather than falling through and returning `undefined`.

### 6. Can a component return multiple sibling elements?

A component must return one JavaScript expression. Wrap sibling elements in a parent element or a Fragment:

~~~jsx
function PageHeading() {
  return (
    <>
      <h1>Lessons</h1>
      <p>Choose a topic to begin.</p>
    </>
  );
}
~~~

The Fragment groups the JSX without adding a wrapper element to the DOM.

## Component names and JSX

### 7. Why should component names start with a capital letter?

JSX uses capitalization to distinguish a JavaScript component from an intrinsic tag name. A capitalized name such as `Welcome` refers to a component in scope. A lowercase name such as `button` is treated as a native element name.

### 8. Should component names use PascalCase?

Yes. Start each word with a capital letter, as in `CourseCard` or `LessonHeading`. React needs the initial capital to recognize a component in JSX; PascalCase is the common naming convention for the full name.

### 9. What happens if I write a custom component with a lowercase JSX name?

JSX treats the lowercase name as an intrinsic tag name, not as a reference to your JavaScript function. Use a capitalized function and JSX name:

~~~jsx
function CourseCard() {
  return <article>React foundations</article>;
}

function App() {
  return <CourseCard />;
}
~~~

### 10. Can I define a component with an arrow function?

Yes. Give the variable a capitalized name:

~~~jsx
const CourseCard = () => <article>React foundations</article>;
~~~

Function declarations and other function expressions can also define components.

### 11. Do component names affect readability?

Yes. A name such as `CourseCard` communicates more than a vague name such as `Thing`. Clear names make a component tree, JSX, and debugging output easier to understand.

### 12. Are lowercase HTML tags and capitalized component names interchangeable?

No. Lowercase JSX names describe intrinsic elements such as `main`, `button`, and `svg`. Capitalized JSX names refer to JavaScript values that must be in scope, usually React components.

## Rendering components

### 13. How do I render a component?

Use it as a JSX element:

~~~jsx
function LessonHeading() {
  return <h1>Components</h1>;
}

function App() {
  return <LessonHeading />;
}
~~~

React then manages the component as part of the tree.

### 14. Why should I not call a component like a regular function?

Calling `LessonHeading()` directly bypasses the component boundary that React manages. This can break React features such as Hooks and makes the component tree inaccurate. Render it with `<LessonHeading />` instead.

### 15. Can I call ordinary helper functions directly?

Yes. A helper function that calculates a value can be called like any other JavaScript function. Keep the distinction clear: call helpers directly, and render components through JSX.

~~~jsx
function formatLabel(topic) {
  return topic.toUpperCase();
}

function TopicTitle() {
  return <h1>{formatLabel('components')}</h1>;
}
~~~

### 16. Does writing `<CourseCard />` immediately create a browser DOM node?

No. JSX creates an element description. React uses the component tree and its returned nodes to update the browser DOM as needed.

### 17. Can I render the same component more than once?

Yes. Each JSX occurrence asks React to render that component at that position. Reuse the definition instead of copying its implementation:

~~~jsx
function Topic() {
  return <li>Components</li>;
}

function TopicList() {
  return (
    <ul>
      <Topic />
      <Topic />
    </ul>
  );
}
~~~

When repeated items come from data, lists and keys are the appropriate pattern and are covered later.

## Component definitions and organization

### 18. Why should component definitions usually be at module scope?

A component function defined inside another component is recreated whenever the parent function runs. React can see a new function identity and treat it as a different component, which can reset its state and cause unnecessary work. Define reusable component functions at the top level of the module.

### 19. Can components still be nested visually if their definitions are at the top level?

Yes. Component composition happens through JSX. A top-level `App` can render `<LessonHeading />` inside its returned markup even when `LessonHeading` is defined separately at module scope.

### 20. Should every component get its own file?

No. Small, closely related components can share a file. Give a component its own module when that improves organization, keeps a file manageable, or makes reuse and imports clearer.

### 21. Is a component file required to have the same name as the component?

No. File naming is a project convention. Matching a file such as `LessonHeading.jsx` to its main component, `LessonHeading`, is a common way to make projects easier to navigate.

### 22. When is a new component boundary useful?

Create one when it names a meaningful piece of UI, separates a responsibility, or supports reuse. A boundary for every tiny markup fragment can make simple code harder to follow.

### 23. Is a component the right place for every calculation?

No. Keep small calculations in helper functions when that makes the code clearer. Components describe UI; helpers can prepare values used by that UI.

## Chapter navigation

| Lesson | Example | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [013 - Function components and component naming](../lessons/013-function-components-and-naming.md) | [Open Example](../examples/013-function-components-and-naming.md) | [012 - Strict Mode and development checks](../lessons/012-strict-mode-and-development-checks.md) | [014 - Props and passing data](../lessons/014-props-and-passing-data.md) |
