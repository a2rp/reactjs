# 015. Destructuring props and default values - Q&A

## Destructuring basics

### 1. What does destructuring props mean?

It is JavaScript object destructuring used to read selected properties from the props object:

~~~jsx
function LessonCard({ title, level }) {
  return <h2>{title}: {level}</h2>;
}
~~~

React still passes one props object. Destructuring creates local variables for the properties named in the pattern.

### 2. Is `{ title }` in a function parameter JSX syntax?

No. In `function LessonCard({ title })`, the braces are JavaScript destructuring inside the function parameter list. JSX syntax is used later in the function's returned UI.

### 3. Is this different from reading `props.title`?

It reads the same value. These two patterns are equivalent in purpose:

~~~jsx
function LessonCard(props) {
  return <h2>{props.title}</h2>;
}

function CompactLessonCard({ title }) {
  return <h2>{title}</h2>;
}
~~~

Choose the form that makes the component easiest to read.

### 4. Can I destructure props inside the function body instead?

Yes. For example, `const { title, level } = props;` can appear near the start of the function. This is useful when the parameter list would become long or the component needs to do some setup first.

### 5. Does destructuring create a copy of an object prop?

It creates a local binding to the property value. If that value is itself an object or array, destructuring does not clone it. Treat nested prop values as read-only too.

## Aliases

### 6. How can I give a prop a different local variable name?

Use `propName: localName` in the destructuring pattern:

~~~jsx
function LessonHeading({ title: heading }) {
  return <h1>{heading}</h1>;
}
~~~

The component still receives the prop named `title`; `heading` is only the local variable name.

### 7. Can I combine an alias and a default value?

Yes. Put the default after the local name:

~~~jsx
function LessonHeading({ title: heading = 'Untitled lesson' }) {
  return <h1>{heading}</h1>;
}
~~~

### 8. Does an alias rename the attribute the parent must use?

No. The parent still writes `<LessonHeading title="..." />`. The alias changes only the name used inside the component.

## Defaults

### 9. How do I set a default value for a prop?

Add `=` and the fallback value to the destructured property:

~~~jsx
function LessonCard({ level = 'Beginner' }) {
  return <p>{level}</p>;
}
~~~

The component uses the default when the prop is omitted or has the value `undefined`.

### 10. Does a default replace `null`?

No. Destructuring defaults apply to missing or `undefined` properties, not `null`. If the parent passes `level={null}`, the local value is `null`.

### 11. Do defaults replace `0`, `false`, or an empty string?

No. Those are supplied values, so they remain `0`, `false`, or `''`. This is useful when zero or false has a real meaning.

### 12. What is the difference between a parameter default and `||`?

A parameter default only applies to missing or undefined props. The `||` operator uses its fallback for any falsy value, including `0`, `false`, and `''`. Do not use `||` if those values should be preserved.

### 13. When should I use `??` instead of a parameter default?

Use nullish coalescing when both `null` and `undefined` should use a fallback:

~~~js
const displayedLevel = level ?? 'Beginner';
~~~

Unlike `||`, `??` preserves values such as `0`, `false`, and an empty string.

### 14. Can a default value be an expression?

Yes. A default can be any JavaScript expression that produces the fallback. It is evaluated when the property is missing or undefined, so keep it predictable and free of side effects.

### 15. What should I consider when the default is an array or object?

A literal default such as `topics = []` creates a new array each time that default is used. If stable identity matters, define the fallback once at module scope and reuse it. Do not mutate a shared default value.

### 16. What happens if I destructure a property from an object prop that is missing?

Nested destructuring assumes the object exists. For example, `function Card({ course: { title } })` fails if `course` is missing, undefined, or null. Default or check the `course` object before reading its fields.

### 17. Does a field default protect against a missing parent object?

No. A default for `title` cannot be applied if JavaScript cannot first read `title` from the parent `course` object. Handle the parent object before destructuring its fields.

## Rest props and forwarding

### 18. What does rest syntax do in a destructuring pattern?

It collects properties that were not named earlier:

~~~jsx
function SearchField({ label, ...inputProps }) {
  return (
    <label>
      {label}
      <input {...inputProps} />
    </label>
  );
}
~~~

Here, `label` is consumed by the component, and the remaining props are collected in `inputProps`.

### 19. Is it always safe to spread rest props onto a DOM element?

No. The remaining values may include custom component props that do not belong on that DOM element. Consume component-only props first and forward only values appropriate for the target element.

### 20. Is JSX spread the same thing as destructuring rest?

They work together but do different things. Destructuring rest collects remaining properties into an object; JSX spread passes an object's properties to a JSX element.

## Common usage questions

### 21. Should I destructure every prop?

No. Destructure the values that make the component clearer. Keeping the full `props` object can be useful when most values are forwarded or accessed dynamically.

### 22. Should I use a default for every optional prop?

No. Add a default when the component has a sensible fallback. If absence has a meaningful different behavior, handle that behavior explicitly instead of hiding it behind a default.

### 23. Does destructuring make props mutable?

No. It changes how JavaScript reads the values, not who owns them. Do not assign to a prop or mutate objects and arrays received through props.

### 24. Can I destructure `children` the same way?

Yes. `children` is a regular prop, so it can be read as `props.children` or destructured. The next chapter focuses on how nested JSX becomes the `children` prop and common ways to compose it.

## Chapter navigation

| Lesson | Example | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [015 - Destructuring props and default values](../lessons/015-destructuring-props-and-default-values.md) | [Open Example](../examples/015-destructuring-props-and-default-values.md) | [014 - Props and passing data](../lessons/014-props-and-passing-data.md) | [016 - The children prop and nested content](../lessons/016-children-prop-and-nested-content.md) |
