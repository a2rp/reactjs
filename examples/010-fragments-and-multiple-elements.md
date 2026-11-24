# 010. Fragments and returning multiple elements — Example

## Goal

Render a course overview with:

- A component that returns multiple siblings using the shorthand Fragment.
- A page component that groups main content and a footer without adding a wrapper.
- Definition-list term and description pairs grouped by keyed Fragments.
- Valid semantic HTML with no unnecessary div elements.

## File

Use the source file from your React project:

- src/App.jsx

The example assumes the root component is already mounted by the project setup from the earlier chapters. It needs no CSS or extra package.

## src/App.jsx

~~~jsx
import { Fragment } from 'react';

const courseFacts = [
  {
    id: 'format',
    label: 'Format',
    description: 'Short lessons with a matching practice example.',
  },
  {
    id: 'level',
    label: 'Level',
    description: 'Beginner-friendly, with JavaScript fundamentals assumed.',
  },
  {
    id: 'goal',
    label: 'Goal',
    description: 'Build a clear mental model of React components.',
  },
];

function CourseIntro() {
  return (
    <>
      <h1>React Foundations</h1>
      <p>
        A practical path through React's core ideas, one lesson at a time.
      </p>
    </>
  );
}

function CourseFacts() {
  return (
    <dl>
      {courseFacts.map((fact) => (
        <Fragment key={fact.id}>
          <dt>{fact.label}</dt>
          <dd>{fact.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}

export default function App() {
  return (
    <>
      <main>
        <CourseIntro />
        <CourseFacts />
      </main>

      <footer>
        <p>Course notes for steady, hands-on practice.</p>
      </footer>
    </>
  );
}
~~~

## What renders

The Fragments do not create wrapper elements. The relevant browser DOM structure is:

~~~text
main
  h1
  p
  dl
    dt
    dd
    dt
    dd
    dt
    dd
footer
  p
~~~

The main element and footer are siblings. Inside the description list, each term and its description are also siblings directly under dl. React does not insert a div around the pair.

## What to notice

- CourseIntro returns one Fragment expression containing an h1 and a p.
- App returns one Fragment expression containing main and footer.
- CourseFacts uses an explicit Fragment because each mapped group needs a key.
- The key uses the stable id from each fact object.
- Each Fragment contains the two related elements, dt and dd.
- The real HTML elements provide the page and description-list semantics. The Fragments only group their children.
- If you wanted a styled box around all the content, replace the outer Fragment with a suitable real element and give that element a class.

## Try these changes

1. Inspect the rendered elements in browser developer tools. Confirm there is no wrapper between main and footer.
2. Expand dl and confirm each dt is immediately followed by its dd, with no div between them.
3. Temporarily replace the outer Fragment in App with a div. Inspect the DOM and see the new wrapper, then restore the Fragment.
4. Add a fourth object to courseFacts with a new unique id. Confirm its term and description appear in the list.
5. Temporarily remove key from the explicit Fragment and check the development console for React's missing-key warning. Restore the key afterward.

## Expected result

The page displays a heading and introduction, followed by three course facts and a short footer. The Fragments let each component return or group multiple elements while keeping the browser DOM limited to the meaningful main, footer, dl, dt, and dd elements.

## Chapter navigation

| Lesson | Q&A | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [010 - Fragments and returning multiple elements](../lessons/010-fragments-and-multiple-elements.md) | [Open Q&A](../qnas/010-fragments-and-multiple-elements.md) | [009 - JSX styles, event props, and DOM properties](../lessons/009-jsx-styles-events-and-dom-properties.md) | [011 - Create a root and render a React application](../lessons/011-create-a-root-and-render-an-application.md) |
