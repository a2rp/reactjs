# 001. Example: Build a small course catalog with components

## Goal

Build a static learning-plan screen from reusable React components. The same `CourseCard` component will describe two cards with different content.

This example demonstrates the first lesson's central idea: describe the UI from component inputs and compose small pieces into a page.

## What you will practice

- Writing function components.
- Returning JSX from a component.
- Composing a page from child components.
- Passing different props to repeated component uses.
- Using semantic HTML for a simple page.

This example is intentionally static. It does not introduce state, event handling, routing, or data fetching.

## Prerequisite

Use any working React starter project. Place the code below in its main app component file, commonly `src/App.jsx`. Keep the starter project's existing entry file and build configuration.

## Build it step by step

### 1. Define a reusable card

The card receives three pieces of text from its parent:

```jsx
function CourseCard({ title, description, level }) {
  return (
    <article>
      <h3>{title}</h3>
      <p>{description}</p>
      <p>
        <strong>Level:</strong> {level}
      </p>
    </article>
  );
}
```

`title`, `description`, and `level` are props. The component uses them to describe its heading and paragraphs. It does not need to know which page contains it.

### 2. Compose the page

The page provides different prop values to two uses of the same component:

```jsx
export default function App() {
  return (
    <main>
      <header>
        <h1>My learning plan</h1>
        <p>A few topics I am studying next.</p>
      </header>

      <section aria-labelledby="courses-heading">
        <h2 id="courses-heading">Courses</h2>

        <CourseCard
          title="HTML"
          description="Structure pages with meaningful elements."
          level="Beginner"
        />

        <CourseCard
          title="React"
          description="Build user interfaces from reusable components."
          level="Getting started"
        />
      </section>
    </main>
  );
}
```

### 3. Put the components together

Save both components in `src/App.jsx`. The complete file is:

```jsx
function CourseCard({ title, description, level }) {
  return (
    <article>
      <h3>{title}</h3>
      <p>{description}</p>
      <p>
        <strong>Level:</strong> {level}
      </p>
    </article>
  );
}

export default function App() {
  return (
    <main>
      <header>
        <h1>My learning plan</h1>
        <p>A few topics I am studying next.</p>
      </header>

      <section aria-labelledby="courses-heading">
        <h2 id="courses-heading">Courses</h2>

        <CourseCard
          title="HTML"
          description="Structure pages with meaningful elements."
          level="Beginner"
        />

        <CourseCard
          title="React"
          description="Build user interfaces from reusable components."
          level="Getting started"
        />
      </section>
    </main>
  );
}
```

## Expected result

The page displays:

- A main heading: **My learning plan**
- A short introductory sentence.
- A **Courses** section.
- An HTML card with its description and level.
- A React card with its description and level.

The exact font, spacing, and colors depend on the CSS already present in your starter project. The example focuses on the rendered structure and data.

## Understand the component tree

The component relationship is:

```text
App
├── main
│   ├── header
│   └── section
│       ├── CourseCard (HTML props)
│       └── CourseCard (React props)
```

`App` describes the whole page and places two `CourseCard` components inside it. Each card uses the same JSX structure, while the props determine its text.

This is composition: a larger UI is assembled from smaller pieces. If the card structure needs to change, update `CourseCard` once and both cards use the updated structure.

## Why this is declarative

The component describes the page that should appear. It does not query the document, create heading nodes by hand, or set text on individual DOM nodes.

For example, to change the React description, edit the `description` prop:

```jsx
<CourseCard
  title="React"
  description="Compose interfaces from components and data."
  level="Getting started"
/>
```

React renders the updated description from the new input. The component remains responsible for describing a card, and the parent remains responsible for choosing the card's content.

## Try it yourself

1. Add a third card for CSS.
2. Change the level of the HTML card.
3. Change the section heading from `Courses` to `Topics`.
4. Add a `duration` prop and show it in each card.
5. Move `CourseCard` into its own file and import it into `App.jsx`.

### Suggested solution for the duration extension

Add `duration` to the component's props and render it:

```jsx
function CourseCard({ title, description, level, duration }) {
  return (
    <article>
      <h3>{title}</h3>
      <p>{description}</p>
      <p>
        <strong>Level:</strong> {level}
      </p>
      <p>
        <strong>Duration:</strong> {duration}
      </p>
    </article>
  );
}
```

Then pass a duration where the card is used:

```jsx
<CourseCard
  title="React"
  description="Build user interfaces from reusable components."
  level="Getting started"
  duration="4 weeks"
/>
```

A component can receive additional props when its callers need to supply additional content. Later lessons cover prop design in more detail.

## Common mistakes

- **Using a lowercase component name:** `<courseCard />` is treated like an unknown HTML tag. Use `CourseCard`.
- **Calling the component as a normal function:** write `<CourseCard />`, not `CourseCard()`. React should invoke components through JSX.
- **Forgetting curly braces for a JavaScript value:** use `{title}` inside JSX to display the variable.
- **Using invalid HTML nesting:** keep headings and sections in a meaningful order.
- **Expecting styling from React:** React describes the UI; CSS controls its presentation.
- **Expecting dynamic interaction:** this example is static. State and event handling come in later lessons.
- **Directly changing the DOM:** update the component inputs and let React render the result.

## Completion checklist

- [ ] The app renders without a JSX syntax error.
- [ ] The page has one main heading and a labelled courses section.
- [ ] Both course cards appear.
- [ ] Each card displays its own title, description, and level.
- [ ] Both cards use the same `CourseCard` component.
- [ ] You can explain which part describes the page and which part supplies card content.

## Related chapters

- [Lesson 001: Introduction to React and declarative UI](../lessons/001-introduction-to-react-and-declarative-ui.md)
- [Q&A 001: Introduction to React and declarative UI](../qnas/001-introduction-to-react-and-declarative-ui.md)
- [Next: How React fits into the web platform](../lessons/002-how-react-fits-into-the-web-platform.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td>Start of course</td><td><a href="../lessons/001-introduction-to-react-and-declarative-ui.md">Lesson</a></td><td><a href="../qnas/001-introduction-to-react-and-declarative-ui.md">Q&amp;A</a></td><td><a href="002-how-react-fits-into-the-web-platform.md">002 - Example: Mount a React feature inside an existing page</a></td></tr>
  </tbody>
</table>
