# 013. Function components and component naming

## Learning goals

By the end of this lesson, you should be able to:

- Define a React component as a JavaScript function that returns UI.
- Give a component a name that React recognizes in JSX.
- Tell a built-in HTML tag from a custom React component by its capitalization.
- Compose components by writing them in JSX.
- Define components at the top level instead of inside another component.
- Choose when to keep related components in one file or move one into its own file.

## A component is a function that describes UI

A function component is a JavaScript function whose return value describes what React should display. Most components return JSX:

~~~jsx
function Greeting() {
  return <h1>Welcome to React</h1>;
}
~~~

A component can also return a larger JSX tree:

~~~jsx
function LessonIntro() {
  return (
    <section>
      <h1>React components</h1>
      <p>Components are reusable parts of an interface.</p>
    </section>
  );
}
~~~

JSX makes the structure readable, while JavaScript lets the component use values and logic. The component itself does not directly create or update browser DOM nodes; React manages how its returned UI becomes part of the page.

A component can return a React node, which is often JSX. JSX rules from the earlier chapters still apply: a return statement must produce one expression, and sibling elements can be grouped with a Fragment.

## Name components with an initial capital letter

Use PascalCase for component names: each word starts with a capital letter, as in Greeting, LessonIntro, or CourseCard.

~~~jsx
function CourseCard() {
  return <article>A React course</article>;
}
~~~

JSX uses capitalization to distinguish custom components from built-in browser tags:

~~~jsx
function App() {
  return (
    <main>
      <CourseCard />
      <button type="button">Start lesson</button>
    </main>
  );
}
~~~

The lowercase main and button names refer to built-in HTML elements. The capitalized CourseCard name refers to the JavaScript component. If a custom component is written with a lowercase JSX name, JSX treats it like a built-in tag name instead.

Function declarations and named function expressions can define components. If you use an arrow function, give the variable a capitalized name:

~~~jsx
const CourseCard = () => {
  return <article>A React course</article>;
};
~~~

The function's name should communicate what the component renders or represents. Meaningful names make JSX easier to scan and help identify components in debugging tools.

## Use components through JSX

Write a component as a JSX element when you want React to render it:

~~~jsx
function LessonHeading() {
  return <h1>Components</h1>;
}

function App() {
  return (
    <main>
      <LessonHeading />
      <p>Now the app uses the heading component.</p>
    </main>
  );
}
~~~

Do not call a component like an ordinary helper function:

~~~jsx
function App() {
  return (
    <main>
      {LessonHeading()}
    </main>
  );
}
~~~

Instead, write the component in JSX:

~~~jsx
function App() {
  return (
    <main>
      <LessonHeading />
    </main>
  );
}
~~~

When you write a component element, React controls when the function runs and tracks it as part of the component tree. Directly calling the function hides that component boundary from React and can break features such as Hooks. Hooks are covered later; the component rule applies even before you use them.

## Define components at the top level

Define component functions at the top level of a module, not inside another component:

~~~jsx
function App() {
  function LessonHeading() {
    return <h1>Components</h1>;
  }

  return <LessonHeading />;
}
~~~

This nested definition creates a new function type every time App runs. React can treat the nested component as a different component, which can reset its state and do extra work. Move the definition outside:

~~~jsx
function LessonHeading() {
  return <h1>Components</h1>;
}

function App() {
  return <LessonHeading />;
}
~~~

Components can still be nested visually by using one in another's JSX. Defining at the top level keeps each component's identity stable while the parent renders.

## Compose a page from smaller components

A page component can combine smaller components that each describe a meaningful part of the interface:

~~~jsx
function CourseTitle() {
  return <h1>React Foundations</h1>;
}

function CourseSummary() {
  return <p>Learn how React builds interfaces from components.</p>;
}

function CoursePage() {
  return (
    <main>
      <CourseTitle />
      <CourseSummary />
    </main>
  );
}
~~~

CoursePage is the parent component in this composition. CourseTitle and CourseSummary are child components. This relationship describes the React component tree; the browser ultimately receives the elements returned by those components.

You can use the same component more than once. A reusable component definition is not copied into separate functions; React renders it at each place where its JSX element appears.

## Organize component files

Small, closely related components can live in the same file. This keeps a short example easy to follow. When a file becomes crowded, or a component needs to be reused from another part of the app, move it into its own module and import it where needed.

The file name is a project convention, but matching a component file to its main component makes projects easier to navigate. For example, LessonHeading.jsx can contain a component named LessonHeading.

Do not make every few lines a new component automatically. A component boundary is useful when it names a piece of UI, separates a responsibility, or supports reuse. File organization and exports are covered in the project and module chapters.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Naming a component with a lowercase first letter | JSX treats a lowercase name like a built-in browser tag. | Use a PascalCase name such as CourseCard. |
| Writing a custom component name in lowercase JSX | It is interpreted as a native tag name. | Use a capitalized JSX component name. |
| Calling a component function directly | React does not see that call as a component boundary in its tree. | Render the component in JSX. |
| Defining a component inside another component | Each parent render creates a new component function type. | Define components at the top level of the module. |
| Giving a component a vague name such as Thing | The JSX does not explain the component's role. | Choose a name based on the UI or responsibility. |
| Assuming every JavaScript function is a component | A helper may calculate data without returning UI. | Use component naming and JSX where the function describes UI. |
| Creating a component for every small markup fragment | Excessive boundaries can make a simple UI harder to follow. | Extract components when the name, responsibility, or reuse helps. |

## Key takeaways

- A function component is a JavaScript function that returns a React node, often JSX.
- Use PascalCase component names so JSX can distinguish them from lowercase HTML tags.
- Render components as JSX elements; do not call component functions directly.
- Define component functions at the top level of a module.
- Compose larger interfaces by nesting component elements in JSX.
- Keep small related components together and split files when their size or reuse calls for it.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="012-strict-mode-and-development-checks.md">012 - Strict Mode and development checks</a></td><td><a href="../qnas/013-function-components-and-naming.md">Q&amp;A</a></td><td><a href="../examples/013-function-components-and-naming.md">Example</a></td><td><a href="014-props-and-passing-data.md">014 - Props and passing data</a></td></tr>
  </tbody>
</table>
