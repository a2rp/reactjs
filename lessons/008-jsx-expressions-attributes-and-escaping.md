# 008. JavaScript expressions, attributes, and escaping in JSX

## Learning goals

By the end of this lesson, you should be able to:

- Put JavaScript expressions in JSX children and attributes.
- Tell a literal string from a variable or computed attribute value.
- Use simple expressions, function calls, conditions, and objects in JSX.
- Explain what JSX does with strings, numbers, booleans, null values, and plain objects.
- Keep untrusted text as text and recognize the risks of rendering raw HTML.

This lesson builds on JSX's markup rules from the previous chapter. It does not cover every DOM property or event prop; those have their own lessons.

## Curly braces open an expression

Inside JSX, a pair of curly braces switches from markup text to a JavaScript expression. The expression must produce a value:

```jsx
const learnerName = 'Maya';

function Greeting() {
  return <h1>Welcome, {learnerName}!</h1>;
}
```

The braces do not make JSX a general JavaScript block. You can put an expression that produces a value there, not a statement such as an `if` declaration or a `for` loop.

Common expressions include:

```jsx
<p>{course.title}</p>
<p>{price * quantity}</p>
<p>{formatDuration(course.minutes)}</p>
<p>{isPublished ? 'Available' : 'Coming soon'}</p>
```

For longer logic, calculate a value before the `return`, or move the work into a named function. Keeping the markup easy to read helps you understand what the component will render.

## Use JavaScript in JSX text

JSX text can combine literal words and expressions:

```jsx
const topicCount = 3;

function TopicSummary() {
  return <p>This lesson has {topicCount} topics.</p>;
}
```

You can also put the whole message in an expression:

```jsx
<p>{'This lesson has ' + topicCount + ' topics.'}</p>
```

The first form is often easier to read. A template string can be used inside an expression too:

```jsx
<p>{`This lesson has ${topicCount} topics.`}</p>
```

This chapter’s code fences show valid source code; when writing a template string inside Markdown, keep its backticks distinct from the surrounding code fence.

### Whitespace and line breaks

JSX collapses ordinary whitespace between text segments in ways similar to HTML. Indentation in your source does not guarantee a particular number of spaces in the rendered page. Use CSS for layout and spacing, and explicit text when the words themselves need a space or line break.

For example, adjacent text around an expression is readable:

```jsx
<p>
  Welcome, {learnerName}!
</p>
```

## Use JavaScript in attributes

Quoted attributes pass literal strings:

```jsx
<img className="avatar" alt="Profile portrait" />
```

Curly braces pass the value of a JavaScript expression:

```jsx
const avatarUrl = '/images/maya.png';
const avatarDescription = 'Maya, course author';

<img className="avatar" src={avatarUrl} alt={avatarDescription} />
```

Do not put quotes around an expression when you want its value. This passes the literal characters `{avatarUrl}` as the source, not the variable:

```jsx
<img src="{avatarUrl}" alt="Profile portrait" />
```

The correct version is:

```jsx
<img src={avatarUrl} alt="Profile portrait" />
```

A string can also be written as a JavaScript expression, but ordinary quoted JSX attributes are simpler for fixed text:

```jsx
<button title={'Open course details'}>Details</button>
```

## Boolean attributes and values

Use a JavaScript boolean for a value that should be on or off:

```jsx
const isLocked = false;

<button disabled={isLocked}>Open lesson</button>
```

A quoted string such as `"false"` is text, not the boolean value `false`. When a property expects a boolean, use `true`, `false`, or a boolean expression. For a true boolean attribute, the shorthand is also available:

```jsx
<input type="checkbox" disabled />
```

Whether an attribute accepts a boolean, string, number, or another type depends on the element and property. Use the right value type.

## JavaScript objects in JSX

To pass an object as an attribute value, the outer braces open a JavaScript expression and the inner braces create the object:

```jsx
<div style={{ backgroundColor: 'lavender', color: 'navy' }}>
  A styled note
</div>
```

The double braces are ordinary syntax: an object literal `{ backgroundColor: 'lavender' }` inside JSX expression braces.

React's `style` prop expects an object, not a CSS text string. The object uses camelCase property names:

```jsx
<p style={{ fontSize: 18, lineHeight: 1.5 }}>
  Readable course text
</p>
```

For most styles, a CSS class is easier to maintain. Inline styles are useful when the values genuinely come from JavaScript data. React adds `px` to numeric values for many length properties; some properties such as `lineHeight` are unitless. A string can express a specific unit, such as `'1.25rem'`.

Objects are valid attribute values in places where an object is expected, but a plain object is not renderable as a text child. For example, `<p>{course}</p>` fails if `course` is an ordinary object. Select a property instead: `<p>{course.title}</p>`.

## Which JavaScript values render as children?

As children in JSX:

- Strings render as text.
- Numbers render as text.
- React elements and arrays of renderable nodes can produce nested UI.
- `null`, `undefined`, and boolean values render no visible text.
- A plain JavaScript object cannot be rendered directly as a child.

For example:

```jsx
function Status({ completed, score }) {
  return (
    <p>
      {completed ? 'Complete' : 'In progress'} — score: {score}
    </p>
  );
}
```

The condition produces one of two strings; the number appears as text. By contrast, trying to render a plain object directly usually produces an error that an object is not a valid React child.

Booleans are often used for conditions, but they do not print the words “true” or “false” as text. Convert a boolean into a display string when you want to show it.

## Conditional expressions

A ternary expression chooses between two values:

```jsx
<p>{isPublished ? 'Published' : 'Draft'}</p>
```

Logical AND can include an element when a condition is true:

```jsx
{isPublished && <span>Available now</span>}
```

Be careful when the left side can be a number. If `items.length` is zero, then `items.length && <List />` evaluates to `0`, and React renders the number. Prefer an explicit comparison:

```jsx
{items.length > 0 && <List />}
```

For several branches or complex checks, calculate a result above the return or use a clear `if` statement to choose what to render. The full set of conditional rendering patterns appears later in the course.

## Escaping and displaying data

A string interpolated as a normal JSX child is rendered as text, not parsed as markup. For example, if `message` contains the characters `<strong>hello</strong>`, then `<p>{message}</p>` displays those characters as text. It does not create a `strong` element.

This distinction lets you display user-entered text without turning it into HTML markup. Do not concatenate an untrusted string into a raw HTML document and inject it into the page.

React's `dangerouslySetInnerHTML` prop deliberately bypasses the normal child rendering path and inserts raw HTML into a DOM element. It can create a cross-site scripting vulnerability if the markup is not trusted and sanitized. Avoid it for ordinary content. If a feature truly requires raw HTML, use a well-understood sanitizer and only pass trusted, sanitized markup. Do not treat the prop name as a security mechanism by itself.

Normal text handling does not make every dynamic value safe in every context. For example, an untrusted URL still needs validation appropriate to the application before it is used as a link or resource address. Escaping text and validating a URL solve different problems.

## Display literal special characters

If JSX source characters would otherwise begin markup or an expression, you can place the desired characters in a JavaScript string expression:

```jsx
<p>{'Write {"name"} to show the pattern.'}</p>
<p>{'The tag text <CourseCard /> is an example.'}</p>
```

The string is rendered as text. For common HTML character references in literal JSX text, entities such as `&amp;` and `&lt;` can also represent characters. Use these for actual text content, not as a substitute for correctly writing JSX structure.

## A complete data-driven example

```jsx
const course = {
  title: 'JSX expressions',
  durationMinutes: 25,
  published: true,
  summary: 'Use JavaScript values in markup.',
};

export default function CourseSummary() {
  const status = course.published ? 'Available' : 'Coming soon';

  return (
    <article className="course-card">
      <h1>{course.title}</h1>
      <p>{course.summary}</p>
      <p>Estimated time: {course.durationMinutes} minutes</p>
      <p>Status: {status}</p>
      <p>
        This value is displayed as text: {'<not-an-element>'}
      </p>
      <div style={{ borderColor: 'slateblue', padding: 12 }}>
        Read the lesson and try its example.
      </div>
    </article>
  );
}
```

The component reads object properties, renders strings and a number, selects a status string, renders tag-like characters as text, and passes a style object. The browser shows no element for the string `<not-an-element>`; it shows its characters.

## Common mistakes

| Mistake | Result | Better form |
| --- | --- | --- |
| `src="{imageUrl}"` | The attribute receives literal text. | Write `src={imageUrl}`. |
| `<p>{course}</p>` where `course` is an object | React cannot render a plain object as a child. | Render a property such as `course.title`. |
| Putting an `if` statement inside braces | JSX expression braces do not accept a statement block. | Calculate a value before `return` or use a ternary expression. |
| `items.length && <List />` when the list can be empty | A zero may appear in the UI. | Use `items.length > 0 && <List />`. |
| `style="color: navy"` | The `style` prop expects an object. | Use `style={{ color: 'navy' }}` or a CSS class. |
| Treating `"false"` as a boolean | A string is not the boolean value `false`. | Pass `false` or a boolean expression. |
| Injecting untrusted HTML with `dangerouslySetInnerHTML` | The browser may parse unsafe markup. | Render text normally; only use trusted, sanitized markup when required. |

## Key takeaways

- Curly braces place JavaScript expressions inside JSX text or immediately after an attribute's equals sign.
- Quotes pass literal strings; braces evaluate expressions.
- Objects in attributes need an expression wrapper; style objects use camelCase properties.
- Strings and numbers render as text, booleans and empty values render no text, and plain objects are not valid children.
- Normal JSX text interpolation displays a string as text, not parsed HTML.
- Raw HTML insertion is a security-sensitive exception; use only trusted, sanitized content.
- Validate dynamic URLs separately from rendering text.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [007 - JSX syntax and markup](007-jsx-syntax-and-markup.md) | [Open Q&A](../qnas/008-jsx-expressions-attributes-and-escaping.md) | [Open Example](../examples/008-jsx-expressions-attributes-and-escaping.md) | [009 - JSX styles, event props, and DOM properties](009-jsx-styles-events-and-dom-properties.md) |
