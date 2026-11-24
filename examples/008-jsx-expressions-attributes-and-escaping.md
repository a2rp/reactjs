# 008. Example: Render data and expressions in JSX

## Goal

Build a course summary component that uses JavaScript values in child text and attributes, selects a status label, shows a conditional section, applies a style object, and displays tag-looking text as text.

## Step 1: Define the data

In `src/App.jsx`, start with the data and helper below:

```jsx
const course = {
  title: 'JSX expressions',
  summary: 'Place JavaScript values into readable interface markup.',
  minutes: 25,
  published: true,
  level: 'Beginner',
  cardClass: 'course-card',
  detailsId: 'course-details',
  detailsHref: '#course-details',
  markupExample: '<strong>This remains text</strong>',
  theme: {
    backgroundColor: '#f4f1ff',
    borderColor: '#5b45a5',
  },
};

const topicCount = 3;

function formatDuration(minutes) {
  return minutes + ' minutes';
}
```

The `course` value is a JavaScript object. Do not try to render the whole object as a child; read the individual properties that the interface needs.

## Step 2: Use expressions in the component

Add the component below in the same file:

```jsx
export default function App() {
  const status = course.published ? 'Published' : 'Coming soon';

  return (
    <main>
      <article
        className={course.cardClass}
        data-level={course.level}
        style={course.theme}
      >
        <h1 id="course-title" title={course.title}>
          {course.title}
        </h1>

        <p>{course.summary}</p>
        <p>Estimated time: {formatDuration(course.minutes)}</p>
        <p>Status: {status}</p>

        <p>
          Markup-looking text: <code>{course.markupExample}</code>
        </p>

        <a href={course.detailsHref}>Jump to topics</a>

        <section id={course.detailsId} aria-label="Course topics">
          {topicCount > 0 && (
            <ul>
              <li>Text expressions</li>
              <li>Attribute values</li>
              <li>Safe text output</li>
            </ul>
          )}
        </section>
      </article>
    </main>
  );
}
```

The `className`, `data-level`, `style`, `title`, `href`, and `id` values come from JavaScript expressions. The fixed `id` and `aria-label` values are literal strings. The `status` ternary selects one of two strings. The conditional section uses a comparison so a zero topic count will not print as a stray `0`.

The helper function returns a string, which JSX displays as text. The style prop receives a JavaScript object; its CSS property names use camelCase.

## Step 3: Run and inspect the page

Start the Vite development server from the project directory:

```sh
npm run dev
```

Open the address printed in the terminal. The page should show the title, summary, estimated time, published status, sample text, and three topics.

The string `<strong>This remains text</strong>` should appear with its tag characters visible inside the `code` element. It should not create a bold element. Normal JSX child text is treated as text, not parsed as HTML.

Stop the development server with Ctrl+C when finished.

## Step 4: Change the data and observe the output

Try these changes one at a time:

1. Change `course.title`. The heading and its title attribute should use the new value.
2. Change `course.minutes`. The helper should display the updated duration.
3. Set `course.published` to `false`. The status should become “Coming soon.”
4. Set `topicCount` to `0`. The topic list should disappear without displaying a stray `0`.
5. Change `course.theme.backgroundColor`. The card background should change.
6. Change `course.markupExample` to another string containing angle brackets. It should still display as text.

## What each expression demonstrates

| Code | What happens |
| --- | --- |
| `{course.title}` | Reads a property and renders the string as text. |
| `{formatDuration(course.minutes)}` | Calls a function and renders its returned string. |
| `{course.published ? 'Published' : 'Coming soon'}` | Selects one value using a conditional expression. |
| `className={course.cardClass}` | Passes a dynamic string to an attribute. |
| `style={course.theme}` | Passes an object to the style prop. |
| `data-level={course.level}` | Passes a dynamic value to a custom data attribute. |
| `{topicCount > 0 && (...)}` | Renders the list only when the condition is true. |
| `{course.markupExample}` | Displays a tag-looking string as text. |

## Common mistakes

- **Writing `className="{course.cardClass}"`:** this passes literal text. Use `className={course.cardClass}`.
- **Writing `<p>{course}</p>`:** a plain object is not a text child. Choose `course.title` or another property.
- **Using `topicCount && <ul>...`:** when the count is zero, the expression evaluates to zero and React displays it. Compare with `topicCount > 0`.
- **Passing a CSS string to `style`:** pass an object or use a CSS class.
- **Assuming the tag-looking message becomes markup:** normal child strings stay text.
- **Using raw HTML for the message:** do not use `dangerouslySetInnerHTML` for ordinary content or untrusted strings.
- **Assuming text rendering validates a URL:** validate untrusted destinations separately before using them in an `href`.

## Completion checklist

- [ ] The component reads values from the `course` object.
- [ ] The duration helper's return value appears in the page.
- [ ] The ternary renders the correct status for either boolean value.
- [ ] The course style object applies to the card.
- [ ] The topic list appears only when `topicCount` is greater than zero.
- [ ] The tag-looking example appears as text instead of HTML.
- [ ] I can distinguish a quoted literal attribute from an expression value.

## Chapter navigation

| Previous chapter | Lesson | Q&A | Next chapter |
| --- | --- | --- | --- |
| [007 - JSX syntax and markup](../lessons/007-jsx-syntax-and-markup.md) | [Open lesson](../lessons/008-jsx-expressions-attributes-and-escaping.md) | [Open Q&A](../qnas/008-jsx-expressions-attributes-and-escaping.md) | [009 - JSX styles, event props, and DOM properties](../lessons/009-jsx-styles-events-and-dom-properties.md) |
