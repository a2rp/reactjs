# 015. Destructuring props and default values

## Learning goals

By the end of this lesson, you should be able to:

- Destructure the props a component uses directly in its function parameters.
- Explain how parameter destructuring relates to the full props object.
- Rename a prop locally with an alias.
- Set defaults for props that are missing or `undefined`.
- Distinguish defaults from fallback expressions that also replace `null` or other falsy values.
- Use rest syntax to forward selected remaining props without leaking component-only props to the DOM.

## Destructure props in the function parameter

A function component receives one props object. You can read its properties using `props.title`:

~~~jsx
function LessonCard(props) {
  return <h2>{props.title}</h2>;
}
~~~

JavaScript object destructuring lets you pull the properties you need into local variables when the function starts:

~~~jsx
function LessonCard({ title, level }) {
  return (
    <article>
      <h2>{title}</h2>
      <p>{level}</p>
    </article>
  );
}
~~~

The braces inside the parentheses belong to JavaScript parameter destructuring. They are not a JSX expression. The component still receives a single props object; destructuring is just a concise way to read some of its properties.

These two components read the same inputs:

~~~jsx
function LessonCard(props) {
  const title = props.title;
  const level = props.level;

  return <h2>{title}: {level}</h2>;
}

function CompactLessonCard({ title, level }) {
  return <h2>{title}: {level}</h2>;
}
~~~

Use the full `props` object when it improves clarity, such as when you need to inspect a property dynamically or are forwarding the entire object. Destructure the specific values a component uses when that makes the component easier to scan.

## Destructure inside the function body

Destructuring does not have to happen in the parameter list. You can destructure after entering the function:

~~~jsx
function LessonCard(props) {
  const { title, level } = props;

  return (
    <article>
      <h2>{title}</h2>
      <p>{level}</p>
    </article>
  );
}
~~~

This can be useful when the component needs early logic or when the parameter list would become difficult to read. It does not change where the props come from or who owns them.

## Rename a prop with an alias

Sometimes a prop name is useful at the component boundary but a different local name is clearer inside the component. Use the `propName: localName` syntax:

~~~jsx
function LessonHeading({ title: heading }) {
  return <h1>{heading}</h1>;
}
~~~

The parent still passes `title`:

~~~jsx
<LessonHeading title="Destructuring props" />
~~~

Inside `LessonHeading`, the local variable is named `heading`. The alias does not rename the prop in JSX and does not modify the props object.

## Set a default value for a prop

A destructured parameter can provide a default with `=`:

~~~jsx
function LessonCard({ title = 'Untitled lesson', level = 'Beginner' }) {
  return (
    <article>
      <h2>{title}</h2>
      <p>{level}</p>
    </article>
  );
}
~~~

If the parent leaves out `title`, the component uses `Untitled lesson`. If the parent leaves out `level`, it uses `Beginner`. The parent can override either default by passing a value.

A default is also used when the parent explicitly passes `undefined`:

~~~jsx
<LessonCard title={undefined} />
~~~

The default does not replace `null`, `false`, `0`, or an empty string:

| Value received | Does the parameter default apply? | Value inside the component |
| --- | --- | --- |
| Prop is omitted | Yes | The default value |
| `undefined` | Yes | The default value |
| `null` | No | `null` |
| `false` | No | `false` |
| `0` | No | `0` |
| `''` | No | The empty string |

This lets a parent intentionally pass values such as `0` or `false` without them being replaced.

## Choose the right fallback behavior

A parameter default is useful when only a missing or undefined value should use the fallback. If `null` should also mean “use the fallback,” use JavaScript's nullish coalescing operator:

~~~js
const displayedTitle = title ?? 'Untitled lesson';
~~~

The `??` operator uses the fallback for `null` and `undefined`, while preserving values such as an empty string, `0`, and `false`.

The logical OR operator has different behavior:

~~~js
const displayedDuration = duration || 15;
~~~

`||` uses the fallback for every falsy value, including `0`, `false`, and `''`. That can be wrong when those are valid values. Choose the behavior that matches the meaning of the prop.

## Combine aliases and defaults

An alias and a default can be used together. Put the default after the local name:

~~~jsx
function LessonHeading({ title: heading = 'Untitled lesson' }) {
  return <h1>{heading}</h1>;
}
~~~

The parent still passes the `title` prop. Inside the component, `heading` receives either the supplied title or the default.

## Defaults for object and array props

Props can contain objects and arrays. You can set a default for the entire prop:

~~~jsx
const EMPTY_TOPICS = [];

function TopicList({ topics = EMPTY_TOPICS }) {
  return (
    <ul>
      {topics.map((topic) => (
        <li key={topic}>{topic}</li>
      ))}
    </ul>
  );
}
~~~

This default covers an omitted or undefined `topics` prop. It does not cover `topics={null}`; null is an explicit value, so the component must decide whether it is valid or handle it separately.

A new object or array written directly as a parameter default is created whenever that default is needed. If the fallback is passed to another component, used as a dependency, or otherwise needs a stable identity, define it once at module scope as in `EMPTY_TOPICS`.

## Destructure nested objects carefully

You can destructure a property of an object prop:

~~~jsx
function InstructorLabel({ instructor }) {
  const { name, role } = instructor;

  return <p>{name} — {role}</p>;
}
~~~

This assumes the parent always provides a non-null `instructor` object. If it can be missing, give the whole object a suitable default or handle the missing case before reading its properties. A default for a nested property cannot help if the object needed to reach that property is itself `null`.

Keep nested destructuring readable. A named local variable often makes the data shape easier to understand:

~~~jsx
function InstructorLabel({ instructor }) {
  if (instructor == null) {
    return <p>Instructor details are unavailable.</p>;
  }

  const { name = 'Guest instructor', role = 'Instructor' } = instructor;

  return <p>{name} — {role}</p>;
}
~~~

Here, the object itself is checked first, then the individual fields receive defaults.

## Use rest syntax to forward remaining props

Destructuring can collect the remaining properties with rest syntax:

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

The component consumes `label` and forwards the remaining values, such as `type`, `name`, and `placeholder`, to the input element.

~~~jsx
<SearchField
  label="Search lessons"
  type="search"
  name="query"
  placeholder="Enter a topic"
/>
~~~

Only forward props that are appropriate for the destination element. If a component-only value such as `isFeatured` is included in `inputProps` and spread onto a DOM element, it may cause an invalid or unintended DOM attribute. Destructure and consume component-only values before forwarding the rest.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Writing `function Card(title)` | React passes one props object, not each prop as a separate positional argument. | Write `function Card({ title })` or read `props.title`. |
| Omitting the destructuring braces in the parameter | `function Card(title)` binds the whole props object to a variable named `title`. | Destructure the property with `function Card({ title })`. |
| Assuming a default replaces `null` | JavaScript parameter defaults apply to missing or undefined values only. | Handle null explicitly if it means “use the fallback.” |
| Using `value || fallback` when zero is valid | Logical OR replaces every falsy value. | Use a parameter default or `value ?? fallback`, as appropriate. |
| Renaming a prop in the parent when only a local alias is needed | JSX still uses the public prop name expected by the component. | Use `title: heading` inside the destructuring pattern. |
| Spreading every remaining prop onto a DOM element | Component-only values may become invalid DOM attributes. | Consume custom props first and forward only appropriate DOM props. |
| Mutating an object or array received through props | Destructuring creates local bindings, not independent copies of nested values. | Keep inputs read-only and make a copy before transforming data. |

## Key takeaways

- Destructuring is JavaScript syntax for reading properties from the props object.
- You can destructure in the function parameter or inside the component body.
- Use `propName: localName` to give a prop a clearer local alias.
- A destructured default applies when a prop is missing or `undefined`, not when it is `null`.
- Use `??` when null and undefined should both use a fallback; use `||` only when all falsy values should.
- Nested destructuring assumes the parent object exists; guard or default the object before reading nested fields.
- Rest syntax can forward remaining props, but filter out component-only values before spreading onto a DOM element.
- Destructuring does not clone objects or make props mutable.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [014 - Props and passing data](014-props-and-passing-data.md) | [Open Q&A](../qnas/015-destructuring-props-and-default-values.md) | [Open Example](../examples/015-destructuring-props-and-default-values.md) | [016 - The children prop and nested content](016-children-prop-and-nested-content.md) |
