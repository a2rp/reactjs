# 014. Props and passing data - Q&A

## Props basics

### 1. What are props?

Props are values a parent component passes to a child through JSX. They let the parent configure what the child displays or how it behaves.

~~~jsx
<CourseCard title="React components" />
~~~

The child receives a `title` prop with the string value `React components`.

### 2. Who supplies a component's props?

The component that renders a child supplies that child's props. In `<App><CourseCard title="React" /></App>`, `App` is the parent and supplies the `title` prop to `CourseCard`.

### 3. How does a function component receive props?

React passes one props object as the component function's argument. The component can read a value through a property:

~~~jsx
function CourseCard(props) {
  return <h2>{props.title}</h2>;
}
~~~

Destructuring that object and setting default values are explained in the next chapter.

### 4. Can a component receive several props?

Yes. A JSX element can include multiple attributes. They become properties on the one props object:

~~~jsx
<CourseCard title="React" duration={25} featured />
~~~

### 5. Does a component receive every JSX attribute as a regular prop?

Most values written on a custom component become props, but React reserves special attributes for its own use. For example, `key` helps React track elements and is not available as `props.key`.

## Passing values

### 6. When should I use quotes and when should I use curly braces?

Quotes provide a string:

~~~jsx
<CourseCard title="React components" />
~~~

Curly braces evaluate a JavaScript expression:

~~~jsx
<CourseCard title={currentTitle} duration={25} />
~~~

### 7. Can I pass an array, object, or boolean as a prop?

Yes. Curly braces let you pass JavaScript values:

~~~jsx
<CourseCard
  course={{ title: 'React components', duration: 25 }}
  topics={['JSX', 'Components']}
  featured={true}
/>
~~~

A boolean prop with the value `true` can use the shorthand `featured`.

### 8. Why are there two pairs of curly braces in `course={{ title: 'React' }}`?

The outer pair means “evaluate JavaScript here” in JSX. The inner pair starts an object literal in JavaScript. The value passed to `course` is one object.

### 9. What value does an omitted prop have?

If a prop is not provided, reading it returns `undefined` unless the component supplies a default. The next chapter explains default values.

### 10. Are props only used to pass text to the screen?

No. A component can receive data for its calculations and display, callback functions for actions, and React elements or other JSX for nested content.

### 11. Does passing a prop to a custom component automatically put it on a DOM element?

No. The custom component receives the prop and decides what to do with it. If a component accepts `className`, for example, it must use or pass that value to an appropriate DOM element itself.

~~~jsx
function Card(props) {
  return <article className={props.className}>{props.children}</article>;
}
~~~

### 12. How do I pass nested JSX to a component?

Put the content between the component's opening and closing tags:

~~~jsx
<Panel title="Study tip">
  <p>Practice by changing one prop at a time.</p>
</Panel>
~~~

The nested content is passed to `Panel` as the `children` prop.

### 13. What kinds of values can `children` contain?

It can contain text, an element, another component, or several pieces of JSX. The receiving component chooses where to render `props.children`. A Fragment can group siblings when needed.

## Callback props

### 14. Can I pass a function as a prop?

Yes. Functions are JavaScript values, so a parent can pass one to a child:

~~~jsx
<OpenButton onOpen={handleOpen} />
~~~

The child can use that callback when an event occurs.

### 15. Why should I pass `handleOpen` instead of `handleOpen()`?

`handleOpen` is the function itself. `handleOpen()` calls the function immediately while rendering and passes its return value instead. To run the action after a click, pass the function reference to the event handler.

### 16. How can a child request that a parent change something?

The parent can pass a callback prop. When an event happens, the child calls that callback; the parent decides how to update its own data or state. A child does not edit the parent's props directly.

## Updating and protecting props

### 17. Can a component change a prop it received?

No. Treat props as read-only inputs. Do not assign to a prop or mutate an object or array received through props.

### 18. What if I need to show a changed version of a prop value?

Calculate a derived value without changing the original. For arrays or objects, make a copy before applying a transformation that would otherwise mutate the input.

### 19. Can props have different values on later renders?

Yes. A parent can render a child again with current values. The child should use the props from its current render rather than assuming a prop is fixed forever.

### 20. Should I copy every prop into local state?

No. That creates a second copy that can become out of sync with the parent. Use a prop directly when it is parent-owned input. Use state only when the component truly owns a value that changes over time.

### 21. How are props different from state?

Props come from a parent and configure a child. State is changing information owned by a component. A component can use both, but it must not mutate either one directly.

### 22. If I need the value of a list item's `key`, how can I access it?

Pass the value in a second, ordinary prop. React reserves `key` for identifying elements, so the child cannot read it from `props.key`:

~~~jsx
<CourseCard key={course.id} courseId={course.id} course={course} />
~~~

### 23. What does JSX spread syntax do?

Spread syntax forwards properties from an object to a JSX element:

~~~jsx
<CourseCard {...courseProps} />
~~~

It can be useful when a component intentionally forwards a whole set of props. Prefer explicit attributes when they make the child interface easier to understand.

### 24. Are props and HTML attributes the same?

They look similar in JSX, but their meaning depends on the element. For a built-in element such as `<button>`, React interprets supported DOM props. For a custom component such as `<CourseCard>`, the component receives the values and decides how to use them.

## Chapter navigation

| Lesson | Example | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [014 - Props and passing data](../lessons/014-props-and-passing-data.md) | [Open Example](../examples/014-props-and-passing-data.md) | [013 - Function components and component naming](../lessons/013-function-components-and-naming.md) | [015 - Destructuring props and default values](../lessons/015-destructuring-props-and-default-values.md) |
