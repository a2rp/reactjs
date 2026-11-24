# 014. Props and passing data

## Learning goals

By the end of this lesson, you should be able to:

- Explain how props carry information from a parent component to a child.
- Pass strings and JavaScript values through JSX attributes.
- Read values from a component's `props` object.
- Pass objects, arrays, functions, and nested JSX as props.
- Keep props read-only and send change requests through callbacks.
- Distinguish props from component state.

## Props connect components

A component can be reused with different content when its parent supplies values. React calls the child component with a `props` object containing those values.

~~~jsx
function CourseCard(props) {
  return <h2>{props.title}</h2>;
}

function App() {
  return <CourseCard title="React components" />;
}
~~~

Here, `App` is the parent and `CourseCard` is its child. The JSX attribute `title` becomes the `title` property on the child's `props` object. The child reads that value to describe its UI.

Props work like the arguments to a function. A component receives one props object; the individual props are properties on that object. A parent can render the same component several times with different values.

## Pass values through JSX attributes

A string value can be written in quotes:

~~~jsx
<CourseCard title="React components" />
~~~

Use curly braces to pass a JavaScript expression:

~~~jsx
<CourseCard
  title={currentTitle}
  duration={25}
  featured={true}
  topics={['JSX', 'Components']}
/>
~~~

The braces mean “evaluate this JavaScript expression.” They can contain a variable, number, boolean, array, object, function, or an expression that produces one of those values.

For a boolean prop whose value is true, JSX also allows the shorter form:

~~~jsx
<CourseCard featured />
~~~

This is equivalent to `featured={true}`. A prop omitted from the JSX is not supplied, so its value will be `undefined` unless the component provides a default. Default values and prop destructuring are the focus of the next chapter.

An object literal needs braces both to enter JavaScript and to write the object:

~~~jsx
<CourseCard course={{ title: 'React components', duration: 25 }} />
~~~

The outer braces start a JavaScript expression. The inner braces delimit the object literal.

## Read values from props

A component can read the properties it needs from its props object:

~~~jsx
function CourseCard(props) {
  return (
    <article>
      <h2>{props.course.title}</h2>
      <p>{props.course.duration} minutes</p>
    </article>
  );
}
~~~

When the expression is inside JSX text, braces let JavaScript evaluate it. Use a prop in the part of the UI it describes; React renders the resulting values as part of the component.

A component may receive many values, but it should use the ones that help it describe its UI or behavior. Keep each value's meaning clear in the parent JSX.

## Props can carry different kinds of values

Props are not limited to strings. They can carry values such as:

- Numbers and booleans.
- Objects and arrays.
- Functions.
- React elements and other JSX content.

~~~jsx
<CourseCard
  course={{ title: 'React components', duration: 25 }}
  topics={['JSX', 'Components']}
  featured
  onOpen={handleOpen}
/>
~~~

The component decides how to use these values. For example, it can read a field from the `course` object, map over the `topics` array, display a featured marker, or call `onOpen` in response to an event.

## Nested JSX is passed as children

When one component is written with content between its opening and closing tags, React passes that nested content to the child in a prop named `children`:

~~~jsx
function Panel(props) {
  return (
    <section>
      <h2>{props.title}</h2>
      <div>{props.children}</div>
    </section>
  );
}

function App() {
  return (
    <Panel title="Before you begin">
      <p>Read the lesson, then try its example.</p>
    </Panel>
  );
}
~~~

Here, the paragraph is the `children` value. A wrapper such as `Panel` can display its children without needing to know which specific elements the parent placed there. The children may be text, one element, several elements grouped in a Fragment, or another component.

## Pass functions to let children request an action

A parent can pass a function as a prop. The child can call it when an event occurs, allowing the parent to decide what should happen:

~~~jsx
function OpenButton(props) {
  return (
    <button type="button" onClick={props.onOpen}>
      Open lesson
    </button>
  );
}

function App() {
  function handleOpen() {
    console.log('The lesson was selected.');
  }

  return <OpenButton onOpen={handleOpen} />;
}
~~~

Pass the function itself with `onOpen={handleOpen}`. Do not call it during rendering with `onOpen={handleOpen()}`; that runs it immediately instead of waiting for the event. Event handling and state updates are covered in later chapters.

This is still one-way data flow: the parent owns the handler and passes it down. The child can call the handler, but the parent remains responsible for deciding how its data changes.

## Treat props as read-only

A component must not change a prop it receives. Props are inputs to a render, not storage for edits:

~~~jsx
function TopicList(props) {
  // Do not call props.topics.push('New topic').
  const visibleTopics = [...props.topics, 'New topic'];

  return (
    <ul>
      {visibleTopics.map((topic) => (
        <li key={topic}>{topic}</li>
      ))}
    </ul>
  );
}
~~~

The copied array lets the component calculate a changed display without modifying the parent's array. The same principle applies to objects: do not assign to a property on an object received through props.

If the displayed data should change, the parent can provide different props on a later render. For user-triggered changes, a child can call a callback prop and the parent can update its own state. State is introduced in a later module.

## Props can change between renders

A prop describes the value supplied for that particular render. If a parent later renders a child with another value, React calls the child with the current props so the UI can reflect the new input.

Props are not just setup values that the child reads once. Avoid copying a prop into another variable merely to keep a second, unsynchronized version of the same data. Use state only when the component truly owns a value that changes over time.

## Props and state have different owners

| Question | Props | State |
| --- | --- | --- |
| Who supplies the value? | A parent component supplies it to a child. | The component that owns the state manages it. |
| How does it change? | The parent renders the child with a different value. | The owning component updates it through React's state APIs. |
| Should the receiving component mutate it? | No. Treat it as read-only. | Do not mutate state directly either; use the relevant state API. |
| Common use | Configure a component or give it data. | Remember information that changes during interaction or over time. |

A component can receive props and own state at the same time. They solve related but different problems: props communicate between components, while state stores changing information owned by a component.

## Special case: the key prop

React uses `key` to identify elements when rendering lists. It is a special attribute and is not passed to the component as a regular prop. If the component also needs the same identifier, pass it separately:

~~~jsx
<CourseCard key={course.id} courseId={course.id} course={course} />
~~~

The list rendering chapter explains when and how to choose keys.

## Forward props carefully

JSX spread syntax can forward a group of props:

~~~jsx
function CardFrame(props) {
  return <CourseCard {...props} />;
}
~~~

Use this when a wrapper intentionally forwards the same interface. Usually, passing the specific values a child needs is easier to read and keeps component boundaries clear:

~~~jsx
<CourseCard title={title} duration={duration} />
~~~

Do not spread props everywhere by default. An explicit list shows what information crosses the boundary.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Writing `duration="25"` when a number is needed | Quoted JSX attributes are strings. | Write `duration={25}`. |
| Using a variable without curly braces | JSX treats the variable name as literal text or an attribute string. | Write `title={currentTitle}`. |
| Calling a callback while rendering | The function runs immediately instead of after the event. | Pass its reference, such as `onOpen={handleOpen}`. |
| Mutating an array or object from props | The parent-owned input changes unexpectedly and rendering becomes unreliable. | Treat props as read-only and calculate from a copy. |
| Expecting nested JSX to arrive in a named prop automatically | Nested JSX is supplied through `children`. | Read `props.children` in the child component. |
| Trying to read `key` as an ordinary prop | React reserves `key` for its own element identity handling. | Pass a second prop such as `courseId` when the child needs the value. |
| Using props as the child's private editable storage | The child does not own the parent's input. | Ask the parent to update its value through a callback; use state for changing local data. |

## Key takeaways

- Props are values a parent passes to a child through JSX.
- A component receives one props object and can read its properties.
- Curly braces pass JavaScript expressions; quoted JSX attributes pass strings.
- Props may contain objects, arrays, functions, and nested JSX.
- Nested JSX is available to the child as `children`.
- Treat props as read-only; pass callbacks when a child needs to request an action.
- React reserves `key`; pass a separate prop if the component needs the same identifier.
- Props communicate data between components; state stores changing data owned by a component.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [013 - Function components and component naming](013-function-components-and-naming.md) | [Open Q&A](../qnas/014-props-and-passing-data.md) | [Open Example](../examples/014-props-and-passing-data.md) | [015 - Destructuring props and default values](015-destructuring-props-and-default-values.md) |
