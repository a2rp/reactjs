# 007. JSX syntax and markup

## Learning goals

By the end of this lesson, you should be able to:

- Explain what JSX is and how it relates to JavaScript, HTML, and React.
- Write a component's returned markup with valid JSX structure.
- Close and nest JSX tags correctly.
- Use fragments when a component needs to return sibling elements without an extra DOM wrapper.
- Recognize common JSX naming differences such as `className`.
- Read a compiler error and locate the markup that needs correction.

## What is JSX?

JSX is a JavaScript syntax extension for writing HTML-like markup in JavaScript source files. React developers commonly use it to describe what a component should render:

```jsx
function WelcomeMessage() {
  return <h1>Welcome to the course</h1>;
}
```

The component is a JavaScript function. The element between `return` and the semicolon is JSX. Project tooling transforms JSX into JavaScript that the browser can run. Browsers do not understand raw JSX directly.

JSX and React are related in everyday projects, but they are separate ideas: JSX is syntax; React is a UI library. JSX can be transformed into element-creation calls, and React can also be used without JSX, though JSX is the usual way to write React interfaces.

## Why keep markup with a component?

A component's logic determines which content it renders. Keeping the component's markup near that logic makes the relationship visible and keeps the change in one place. The markup can still be split across components and files when that improves clarity.

For example, the following function describes one piece of the page:

```jsx
function CourseHeading() {
  return <h1>React foundations</h1>;
}
```

When a component returns more than one adjacent element, JSX requires a single root. The next section shows the valid ways to group those elements.

## Rule 1: Return one root element

A component cannot return two sibling JSX elements as separate values:

```jsx
function CourseHeader() {
  return (
    <h1>React foundations</h1>
    <p>Learn to describe user interfaces.</p>
  );
}
```

Wrap the siblings in one parent element:

```jsx
function CourseHeader() {
  return (
    <header>
      <h1>React foundations</h1>
      <p>Learn to describe user interfaces.</p>
    </header>
  );
}
```

Choose a meaningful semantic element such as `header`, `main`, or `section` when it represents the content. If an extra DOM element would have no meaning or layout purpose, use a Fragment. React components can also return arrays of elements, but an element or Fragment is usually clearer for static sibling markup:

```jsx
function CourseHeader() {
  return (
    <>
      <h1>React foundations</h1>
      <p>Learn to describe user interfaces.</p>
    </>
  );
}
```

The short Fragment syntax `<></>` groups its children without adding an element to the browser DOM. Keyed fragments for dynamic lists have a different syntax and are covered with list rendering.

## Rule 2: Close every tag

JSX requires explicit closing tags. HTML sometimes permits omitted closing tags, but JSX makes the structure explicit.

Close a normal element with a matching end tag:

```jsx
<p>Each paragraph needs an end tag.</p>
```

Close an element that has no children with a slash before the closing angle bracket:

```jsx
<img src="/course-cover.png" alt="A notebook labeled React" />
<br />
<input type="search" />
```

Tags must also be nested in the correct order. This is invalid because the `strong` element closes after its parent paragraph:

```jsx
<p>Read the <strong>first chapter.</p></strong>
```

Correct the nesting so each opening tag closes before its parent does:

```jsx
<p>Read the <strong>first chapter.</strong></p>
```

Clear nesting matters both to the JSX parser and to the resulting document structure.

## Rule 3: Use JSX attribute names and values correctly

JSX looks like HTML but is written inside JavaScript. Some attributes use JavaScript-friendly names. For example, use `className` instead of HTML's `class`:

```jsx
<article className="course-card">
  <h2>React foundations</h2>
</article>
```

Many DOM and SVG property names use camelCase, such as `tabIndex`. The accessibility attributes `aria-*` and custom `data-*` attributes keep their hyphens:

```jsx
<button aria-pressed="false" data-topic="jsx">
  Save topic
</button>
```

Basic attribute values can be written as quoted text. JavaScript values inside attributes use curly braces; those expressions are the focus of the next lesson. Avoid copying HTML to JSX without checking names, nesting, and tag closure.

## Rule 4: Distinguish HTML tags from React components

Built-in HTML tag names are lowercase:

```jsx
<main>
  <h1>Course overview</h1>
</main>
```

A custom React component starts with a capital letter:

```jsx
function CourseCard() {
  return <article>A course card</article>;
}

function CourseList() {
  return (
    <main>
      <CourseCard />
    </main>
  );
}
```

The capital letter lets JSX distinguish a component variable from a built-in HTML tag. The component has to be in scope where it is used. Component composition and imports are covered in later lessons.

## Put JSX in a return statement

A one-line JSX expression can follow `return` directly:

```jsx
function CourseTitle() {
  return <h1>React foundations</h1>;
}
```

For multiline JSX, wrap the returned markup in parentheses. This makes it clear that the markup belongs to the return value:

```jsx
function CourseTitle() {
  return (
    <section>
      <h1>React foundations</h1>
      <p>A practical introduction.</p>
    </section>
  );
}
```

The opening parenthesis should stay on the same line as `return`. If a line break appears immediately after `return`, JavaScript's automatic semicolon insertion can make the function return `undefined` before it reaches the JSX.

## JSX comments

JavaScript comments can be used around the returned expression. To place a comment among JSX children, put the JavaScript comment inside curly braces:

```jsx
function CourseDetails() {
  return (
    <section>
      {/* Keep this heading before the summary. */}
      <h2>Course details</h2>
      <p>Short description goes here.</p>
    </section>
  );
}
```

Curly braces also introduce JavaScript expressions inside JSX. Those expressions are covered next; here, recognize the comment form and keep comments useful rather than using them to explain obvious markup.

## JSX is not a string of HTML

This is JSX syntax in a JavaScript module:

```jsx
const heading = <h1>Course overview</h1>;
```

It is not a string such as `'<h1>Course overview</h1>'`, and it is not a command to manually edit the browser DOM. The configured project tooling transforms the syntax, and React uses the resulting element description to render the interface.

Keep UI updates in React components instead of using `document.write` or manually changing the same DOM nodes React controls. Direct DOM operations can conflict with React's rendering. Refs provide specific, controlled DOM access and are covered later.

## A small valid component

This component uses a single semantic root, correctly nested elements, a self-closing image, a JSX class name, and a list:

```jsx
export default function CourseOverview() {
  return (
    <main>
      <article className="course-card">
        <h1>React foundations</h1>
        <p>Learn how to describe a user interface with components.</p>
        <img
          src="/course-cover.png"
          alt="A notebook labeled React"
        />
        <ul>
          <li>Start with markup</li>
          <li>Compose interface pieces</li>
        </ul>
      </article>
    </main>
  );
}
```

The file path in `src` should match the project where this component is used. The image itself is illustrative: if the file is not present, the browser will report a missing image. The JSX structure remains valid either way.

## Common JSX errors

| Error | Why it fails | Correction |
| --- | --- | --- |
| Returning adjacent root elements | A component must return one JSX root value. | Wrap them in a meaningful element or Fragment. |
| Leaving `img` or `br` unclosed | JSX requires explicit tag closure. | Write `<img />` or `<br />`. |
| Closing tags in the wrong order | The markup tree is not properly nested. | Close the most recently opened tag first. |
| Writing `class` on a JSX element | JSX uses the DOM property name. | Write `className`. |
| Writing `<courseCard />` for a component | Lowercase names are treated as built-in/custom HTML tags. | Name the component with an uppercase first letter, such as `CourseCard`. |
| Putting a newline after `return` before JSX | JavaScript may insert a semicolon after `return`. | Write `return (` and put the markup on the next line. |
| Pasting HTML unchanged | JSX has stricter syntax and different property names. | Check roots, nesting, closure, and JSX attribute names. |

Read the first compiler error carefully. It often points near the opening or closing tag where the structure stopped making sense. Fix the earliest structural error first; later error messages may be consequences of it.

## Key takeaways

- JSX is an HTML-like syntax extension written in JavaScript source.
- Tooling transforms JSX; browsers do not run raw JSX.
- A single JSX expression cannot contain adjacent root tags; wrap static siblings in an element or Fragment.
- Close every element and nest tags in the proper order.
- Use `className` and other JSX-compatible property names.
- Lowercase tags represent HTML; capitalized names represent components.
- Use parentheses for multiline returned JSX, with the opening parenthesis beside `return`.
- Treat JSX as React's description of UI, not as a string or direct DOM-editing instruction.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="006-project-files-scripts-and-dev-server.md">006 - Project files, scripts, and the development server</a></td><td><a href="../qnas/007-jsx-syntax-and-markup.md">Q&amp;A</a></td><td><a href="../examples/007-jsx-syntax-and-markup.md">Example</a></td><td><a href="008-jsx-expressions-attributes-and-escaping.md">008 - JavaScript expressions, attributes, and escaping in JSX</a></td></tr>
  </tbody>
</table>
