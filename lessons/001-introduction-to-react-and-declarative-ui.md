# 001. Introduction to React and declarative UI

## Learning goals

By the end of this lesson, you should be able to:

- Explain what React is used for and where it fits in a web application.
- Describe a component as a reusable description of part of a user interface.
- Explain the difference between describing UI and manually changing individual DOM nodes.
- Recognize that React updates the rendered interface when its inputs change.
- Identify what React does not provide by itself, such as a complete routing or backend system.

## What is React?

React is a JavaScript library for building user interfaces. It lets you divide an interface into components, describe what each component should show, and compose those components into screens and applications.

A component can be small, such as a button, or large, such as a complete page. A React application is usually made from many components that form a tree.

React is especially useful when an interface has changing data and many interactive states: search results, forms, dashboards, menus, dialogs, shopping carts, and similar features. It can also be added to selected parts of an existing page; the whole site does not have to be rewritten in React.

### React works with the web platform

React works alongside the technologies already used on the web:

- **HTML** provides the document structure and semantic elements.
- **CSS** controls presentation and layout.
- **JavaScript** provides the programming language and application logic.
- **React** provides a component model and a way to keep the rendered interface in sync with application data.
- **React DOM** connects React to browser DOM nodes.

React does not replace HTML, CSS, JavaScript, or the browser. JSX may look like HTML, but it is JavaScript syntax that a project build tool transforms before the browser runs the application.

React itself is a UI library, not a complete full-stack framework. A real application may also need a build tool, routing, data-loading choices, a server, and deployment setup. Later lessons explain how these pieces fit together.

## Describe the UI instead of issuing DOM commands

A browser page contains DOM nodes. With an imperative approach, application code can find and mutate individual nodes:

```js
const heading = document.querySelector('#welcome');
heading.textContent = 'Welcome back, Priya';
```

This works for a small isolated change. As an interface grows, however, many pieces of code can try to update the DOM. It becomes harder to know which code owns a value, whether the screen is up to date, and how several related parts should change together.

React encourages a declarative approach. Describe the UI for the current data, and let React update the DOM to match that description:

```jsx
function WelcomeMessage({ name }) {
  return <h1>Welcome back, {name}</h1>;
}

function App() {
  const name = 'Priya';

  return <WelcomeMessage name={name} />;
}
```

Here, `WelcomeMessage` describes the heading. If its parent later supplies a different `name`, React renders the heading using that value and applies the needed DOM update. The component describes the desired result; it does not search for and manually change the heading node.

This approach gives the UI a clear relationship:

```text
current inputs and state
          |
          v
     React render
          |
          v
 description of the UI
          |
          v
 React updates the DOM
```

You focus on what should be visible for a given situation. React coordinates how that result is reflected in the browser.

## Components are the building blocks

A component is a JavaScript function whose name starts with a capital letter and whose return value describes UI. Components are used in JSX with tag-like syntax:

```jsx
function PageTitle() {
  return <h1>Learning React</h1>;
}

function App() {
  return (
    <main>
      <PageTitle />
      <p>Build the interface from small, understandable pieces.</p>
    </main>
  );
}
```

In this example:

- `PageTitle` and `App` are React components.
- `<h1>`, `<main>`, and `<p>` are browser HTML elements.
- `<PageTitle />` tells React to include the output of the `PageTitle` component.
- `App` composes the heading and paragraph into a larger UI.

Component names start with uppercase letters so JSX can distinguish them from built-in lowercase HTML tags. A component should be used through JSX; do not call a component function directly as an ordinary helper.

A component can be rendered many times. For example, a page can reuse the same `CourseCard` component for several courses while supplying different data to each one. This keeps structure consistent and makes changes easier to maintain.

## JSX is how many React projects describe UI

JSX lets you write markup-like descriptions inside JavaScript. It supports JavaScript expressions inside curly braces:

```jsx
const learner = 'Priya';

function Greeting() {
  return <p>Hello, {learner}.</p>;
}
```

The browser does not understand JSX directly. A build tool transforms it into JavaScript that React can use. You will learn JSX syntax, attributes, expressions, and fragments in the next lessons.

JSX resembles HTML but follows JavaScript conventions in some places. For example, a React DOM element uses `className` for CSS classes. This lesson only introduces JSX; the details come later.

## The component tree

Components can contain other components. Their relationships form a tree:

```text
App
├── Header
│   └── SiteTitle
└── MainContent
    ├── CourseCard
    └── CourseCard
```

The tree helps you reason about an interface:

- A parent component can arrange and compose its children.
- A parent can provide data to a child.
- A child can describe one focused piece of the interface.
- Reusable children can appear in more than one place.

A component tree is a design tool, not necessarily a one-to-one copy of the visual layout. Choose boundaries that make the interface understandable and components useful.

## React UI follows its inputs

A component's output depends on inputs such as props and state. Props are values provided by a parent; state is component-owned information that can change over time. State is introduced in later lessons.

For now, think of the general relationship as:

```text
props + state -> component render -> UI description
```

When an input changes, React can call the component again and update what appears on screen. You describe the interface for the new inputs rather than manually issuing a separate DOM instruction for every possible change.

This does not mean React automatically knows how to fetch your data or choose the application's behavior. Your JavaScript and application architecture decide what the inputs should be. React helps render the UI from them.

## A small component example

This example creates two course cards by composing a reusable component:

```jsx
function CourseCard({ title, description }) {
  return (
    <article>
      <h2>{title}</h2>
      <p>{description}</p>
    </article>
  );
}

export default function App() {
  return (
    <main>
      <h1>My learning plan</h1>

      <CourseCard
        title="HTML"
        description="Structure pages with meaningful elements."
      />

      <CourseCard
        title="React"
        description="Build user interfaces from components."
      />
    </main>
  );
}
```

The browser displays a main heading followed by two articles. Both cards have the same structure, but each receives different text. This is a small example of describing UI with reusable components.

The code uses JSX, function components, nesting, and props. Each idea receives a deeper treatment later in the roadmap.

## When React is a good fit

React can be useful when:

- Many parts of the interface depend on changing data.
- The same UI patterns appear in several places.
- Interactions create several related UI states.
- You want a component structure that can be maintained as a project grows.
- A team needs reusable UI pieces with explicit inputs.

React is not required for every web page. A mostly static document may be simpler with plain HTML and CSS. Choose React when its component and update model helps solve the actual interface problem.

## What this first lesson does not cover yet

This lesson establishes the mental model. It does not yet teach how to:

- Create or configure a React project.
- Mount a React root into a page.
- Write all JSX syntax rules.
- Handle user events or update state.
- Render collections with stable keys.
- Fetch data, route between pages, or create a server application.

Those topics have dedicated lessons so you can learn each concept in order.

## Key takeaways

- React is a JavaScript library for building user interfaces from components.
- A component describes a piece of UI and can be composed with other components.
- JSX is a JavaScript syntax extension used to describe UI in many React projects.
- Declarative UI means describing what should appear for current inputs, rather than manually changing each DOM node.
- React DOM applies rendered UI to the browser DOM.
- React works with HTML, CSS, JavaScript, build tools, and, when needed, a separate framework.
- A component tree helps organize how interface pieces fit together.
- React is useful when its model makes an interface easier to build and maintain; it is not mandatory for every page.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| — | [Open Q&A](../qnas/001-introduction-to-react-and-declarative-ui.md) | [Open Example](../examples/001-introduction-to-react-and-declarative-ui.md) | [002 - How React fits into the web platform](002-how-react-fits-into-the-web-platform.md) |
