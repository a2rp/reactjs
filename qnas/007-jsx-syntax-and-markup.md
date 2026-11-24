# 007. Q&A: JSX syntax and markup

Use these questions to review how JSX is written inside JavaScript modules and how its markup rules differ from plain HTML.

## JSX basics

### 1. What is JSX?

JSX is a JavaScript syntax extension for writing HTML-like markup in JavaScript source. React components commonly return JSX to describe the UI they render.

### 2. Is JSX the same thing as HTML?

No. JSX resembles HTML, but it has stricter structure and some different property names. Project tooling transforms JSX into JavaScript before the browser runs the app.

### 3. Is JSX the same thing as React?

No. JSX is syntax; React is a UI library. They are commonly used together, but they are separate ideas. React can be used without JSX, and JSX-like syntax is also used with other UI libraries.

### 4. Does the browser understand raw JSX?

No. The configured build tool or framework transforms JSX into browser-runnable JavaScript. A browser loading source with untransformed JSX will report a syntax error.

### 5. Why do components often put markup in their JavaScript file?

A component's logic determines what it renders. Keeping that logic near its markup makes it easier to see and change them together. Larger interfaces can still be split into smaller components and files.

## Structure and tags

### 6. Why does JSX require one root around adjacent markup?

A single JSX expression cannot contain two adjacent root tags. Wrap static siblings in a meaningful element such as `<section>`, or use a Fragment if an extra DOM element is not wanted. React components can also return arrays, but wrappers or Fragments are usually clearer for static siblings.

### 7. What is a Fragment?

A Fragment groups child elements without adding a DOM element around them. The short syntax is `<></>`. A keyed Fragment in a dynamic list uses a different syntax and is covered with list rendering.

### 8. Do all JSX tags have to be closed?

Yes. Use a matching closing tag for elements with children, such as `<p>Text</p>`, and a self-closing tag for elements without children, such as `<img />`, `<br />`, or `<input />`.

### 9. What does correct tag nesting mean?

Close the most recently opened tag first. For example, `<p><strong>Text</strong></p>` is properly nested; closing the paragraph before the strong element is not.

### 10. Can JSX markup be returned on one line?

Yes. A simple expression can follow `return` directly, such as `return <h1>Welcome</h1>;`. For multiline markup, parentheses make the return value clear.

### 11. Why should the opening parenthesis be beside `return`?

If you put a newline immediately after `return`, JavaScript may insert a semicolon there. The function can then return `undefined` before reaching the JSX. Write `return (` on one line and put the markup on the following lines.

### 12. How do comments work inside JSX children?

Put a JavaScript comment inside curly braces: `{/* A useful comment */}`. A regular HTML comment such as `<!-- comment -->` is not valid JSX syntax.

## Names and components

### 13. Why do JSX elements use `className` instead of `class`?

JSX property names follow JavaScript and DOM property conventions. Use `className` to set the DOM class property. HTML-to-JSX conversions must update this attribute.

### 14. Are all JSX attribute names camelCase?

Many DOM and SVG property names use camelCase, such as `tabIndex`. However, `aria-*` and `data-*` attributes keep their hyphenated spelling, for example `aria-label` and `data-topic`.

### 15. How can I tell an HTML tag from a React component in JSX?

Built-in HTML tags are lowercase, such as `<main>` and `<button>`. A React component starts with an uppercase letter, such as `<CourseCard />`, and must be in scope where it is used.

### 16. What happens if I write a component name in lowercase?

JSX treats a lowercase tag name as an element type string, not as a component variable. Use an uppercase name for the component function and its JSX tag.

### 17. Do I have to import React just to use JSX in a modern Vite template?

Usually not in a current Vite React starter with the modern JSX transform configured. You still import React APIs that your code actually uses. Follow the project's template and build-tool configuration.

## Markup and errors

### 18. Why can't I paste any valid HTML directly into JSX?

Some HTML permits omitted closing tags and attribute spellings that JSX does not accept. JSX needs explicit closures, properly nested tags, one root per JSX expression, and JSX-compatible property names.

### 19. Does indentation affect JSX validity?

Indentation does not define the markup hierarchy, but consistent indentation makes nesting errors easier to see. Matching tags and valid syntax determine the structure.

### 20. Is a JSX expression a string of HTML?

No. `<h1>Welcome</h1>` in a JavaScript module is JSX syntax, not the string `'<h1>Welcome</h1>'`. React uses the transformed element description to render the UI.

### 21. Should I manually edit the DOM to change JSX-rendered UI?

Generally, no. Describe the UI through React components. Manual DOM changes can conflict with React's rendering. Refs provide controlled access for specific cases and are covered later.

### 22. What should I do when the JSX compiler reports several errors?

Fix the first structural error and recheck. One missing or incorrectly nested tag can cause many later messages. Look near the reported line and compare opening and closing tags.

### 23. Should I use a semantic HTML element or a Fragment?

Use a semantic element when it accurately describes the content or provides useful structure. Use a Fragment when you need to group siblings but an extra DOM node would have no meaning. Do not add a generic wrapper automatically.

### 24. Where do JavaScript expressions and JSX attribute values fit in?

They are written inside JSX using curly braces and have their own rules for strings, values, escaping, and property syntax. Those topics are the focus of the next lesson.

## Chapter navigation

| Previous chapter | Lesson | Example | Next chapter |
| --- | --- | --- | --- |
| [006 - Project files, scripts, and the development server](../lessons/006-project-files-scripts-and-dev-server.md) | [Open lesson](../lessons/007-jsx-syntax-and-markup.md) | [Open Example](../examples/007-jsx-syntax-and-markup.md) | [008 - JavaScript expressions, attributes, and escaping in JSX](../lessons/008-jsx-expressions-attributes-and-escaping.md) |
