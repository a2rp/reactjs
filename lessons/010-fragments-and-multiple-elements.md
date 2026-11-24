# 010. Fragments and returning multiple elements

## Learning goals

By the end of this lesson, you should be able to:

- Explain why a component return needs one JSX expression.
- Group sibling JSX elements with the Fragment shorthand.
- Choose a real HTML element when the group needs semantics, attributes, or styling.
- Use an explicit keyed Fragment when one list item must render multiple sibling elements.
- Recognize that a Fragment groups React children without adding a wrapper node to the browser DOM.

## A component returns one JSX expression

A component can render many elements, but its return statement must evaluate to one value. This is not valid JSX because the heading and paragraph are separate expressions:

~~~jsx
function CourseIntro() {
  return (
    <h1>React foundations</h1>
    <p>Start with components and JSX.</p>
  );
}
~~~

You could put both elements in a real parent element, such as a section. Sometimes a parent has a useful meaning or role. Other times you only need to group siblings so the component can return them together. A Fragment does that without creating another browser element.

~~~jsx
function CourseIntro() {
  return (
    <>
      <h1>React foundations</h1>
      <p>Start with components and JSX.</p>
    </>
  );
}
~~~

The Fragment is one JSX expression, so the component has one return value. Its heading and paragraph are still separate children.

## Use the Fragment shorthand

Write an opening empty tag and a closing empty tag around sibling elements:

~~~jsx
<>
  <h1>Course outline</h1>
  <p>Four topics are included.</p>
</>
~~~

The shorthand is equivalent to using React's Fragment component in the common case. It is built into JSX syntax and does not need an import.

Fragments can be used anywhere a single JSX element is needed. For example, a component's return value can be a Fragment, or a conditional branch can group multiple siblings:

~~~jsx
function StatusMessage({ isReady }) {
  return isReady
    ? (
        <>
          <h2>Ready to begin</h2>
          <p>Choose a lesson below.</p>
        </>
      )
    : <p>Preparing the course...</p>;
}
~~~

The conditional expression still produces one value: either a paragraph or a Fragment.

## A Fragment does not add a DOM wrapper

The browser receives the Fragment's children as siblings. It does not receive an extra element named fragment, div, or anything else.

Given this JSX:

~~~jsx
<>
  <h1>Course outline</h1>
  <p>Four topics are included.</p>
</>
~~~

The relevant DOM shape is:

~~~text
h1
p
~~~

There is no wrapper between the component and those two DOM elements. This matters when a wrapper would change the HTML structure, CSS layout, or selectors.

For example, a description list groups terms and descriptions. When each data item produces a pair of siblings, a Fragment lets the dt and dd elements remain children of the dl without an extra div between them:

~~~jsx
<dl>
  <Fragment key="duration">
    <dt>Duration</dt>
    <dd>About two hours</dd>
  </Fragment>
  <Fragment key="level">
    <dt>Level</dt>
    <dd>Beginner</dd>
  </Fragment>
</dl>
~~~

The explicit Fragment form is used here because each group needs a key. A mapped list of keyed fragments appears in the chapter example. List keys are explained in more detail in chapter 019.

## Shorthand limits and explicit Fragments

The shorthand cannot receive props:

~~~jsx
<>
  <h2>Course facts</h2>
  <p>Updated this week.</p>
</>
~~~

You cannot add a key, className, id, event handler, or other prop to the shorthand. If a Fragment needs a key, import Fragment from React and use the explicit form:

~~~jsx
import { Fragment } from 'react';

<Fragment key={course.id}>
  <dt>{course.title}</dt>
  <dd>{course.description}</dd>
</Fragment>
~~~

For ordinary grouping, prefer the shorter shorthand. Use explicit Fragment when you need the Fragment's supported props, especially key in a list.

A Fragment is not a styling hook. It creates no DOM element, so there is no box to style and no element on which to place a className or id. If the group needs a class, layout behavior, an event prop, or another DOM attribute, use a suitable real element.

## Choose a meaningful element when you need one

Use a real wrapper when it describes or controls the content:

~~~jsx
<section className="course-summary">
  <h2>Course summary</h2>
  <p>Learn the React component model.</p>
</section>
~~~

Here, section creates a DOM element and can be styled. It also communicates that the content is a section of the page. A div is appropriate when a generic container is needed and no more specific HTML element fits.

Use a Fragment when grouping alone is enough:

~~~jsx
<>
  <CourseHeader />
  <CourseDetails />
</>
~~~

Do not add a wrapper just because it is the first fix that makes JSX compile. First decide whether the UI needs that DOM element. A Fragment does not repair invalid HTML nesting: the elements inside it must still be valid children of their actual parent.

## Fragments in lists

If one item in a mapped list must produce more than one sibling, group that item's output in an explicit Fragment and give the Fragment a stable key:

~~~jsx
import { Fragment } from 'react';

const facts = [
  { id: 'duration', term: 'Duration', detail: 'About two hours' },
  { id: 'level', term: 'Level', detail: 'Beginner' },
];

function CourseFacts() {
  return (
    <dl>
      {facts.map((fact) => (
        <Fragment key={fact.id}>
          <dt>{fact.term}</dt>
          <dd>{fact.detail}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
~~~

The key belongs on the outer Fragment returned for each item, not on each child. The Fragment shorthand cannot take a key. Prefer a stable identifier from the data; do not generate a fresh random key during rendering. Chapter 019 covers list rendering and keys in full.

## Common mistakes

| Mistake | Why it is a problem | Better approach |
| --- | --- | --- |
| Returning two adjacent JSX elements without grouping | The return expression has more than one root expression. | Group siblings in a Fragment or a meaningful HTML element. |
| Adding a div only to make JSX compile | It creates a DOM node and may affect layout or valid parent-child markup. | Use a Fragment when no wrapper element is needed. |
| Expecting a Fragment to accept className or id | A Fragment shorthand has no DOM node to receive those props. | Use a real element if the group needs attributes or styling. |
| Writing a lowercase fragment tag | There is no HTML element named fragment for this purpose. | Use the JSX shorthand or React's Fragment component. |
| Using `<>...</>` for a list group that needs a key | The shorthand cannot receive props. | Import Fragment and use `<Fragment key={item.id}>`. |
| Assuming a Fragment fixes invalid HTML nesting | A Fragment adds no wrapper, so the same children reach the same parent. | Follow the valid content model of the real parent element. |
| Giving a mapped Fragment an array index or random key by habit | Unstable keys can make React associate items incorrectly after list changes. | Prefer a stable identifier from the data. |

## Key takeaways

- A component returns one JSX expression; that expression can contain many child elements.
- The `<>...</>` shorthand groups children without adding a DOM element.
- Use a meaningful HTML container if the group needs semantics, styling, layout, or DOM props.
- Shorthand Fragments cannot receive props. Use imported Fragment when a group needs a key.
- In a mapped list, put the stable key on the Fragment returned for each item.
- Fragments do not make invalid HTML nesting valid.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [009 - JSX styles, event props, and DOM properties](009-jsx-styles-events-and-dom-properties.md) | [Open Q&A](../qnas/010-fragments-and-multiple-elements.md) | [Open Example](../examples/010-fragments-and-multiple-elements.md) | [011 - Create a root and render a React application](011-create-a-root-and-render-an-application.md) |
