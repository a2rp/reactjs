# 008. Q&A: JavaScript expressions, attributes, and escaping in JSX

Use these questions to review how JavaScript values enter JSX and how ordinary text differs from raw HTML.

## Expressions in JSX

### 1. What do curly braces do inside JSX?

They open a JavaScript expression inside markup. For example, `<p>{course.title}</p>` evaluates `course.title` and renders its value as a child.

### 2. What is an expression?

An expression produces a value. A variable reference, property access, function call, arithmetic operation, ternary expression, or object literal can be an expression.

### 3. Can I put an `if` statement inside JSX braces?

No. JSX braces in a child or attribute position expect an expression, not a statement block. Compute a value before the return, use a ternary, or choose a clear conditional rendering pattern.

### 4. Where can JSX use curly braces?

You can use them in a JSX child's content or as the value after an attribute's equals sign. They do not let you dynamically create a tag name such as `<{tag}>`.

### 5. Can I call a function inside JSX braces?

Yes, if the function call produces a value appropriate for that position. For example, `<p>{formatDuration(minutes)}</p>` renders the returned value. Keep render-time calculations predictable and avoid side effects in code that runs while rendering.

### 6. How do I combine literal text and a variable?

Put the expression between text segments:

```jsx
<p>Welcome, {learnerName}!</p>
```

A JavaScript string expression or template string can also produce the whole message.

## Attribute values

### 7. What is the difference between quotes and braces in an attribute?

Quotes pass literal text; braces evaluate JavaScript:

```jsx
<img alt="Course cover" src={imageUrl} />
```

Here, the alt text is fixed and `src` receives the value of `imageUrl`.

### 8. Why is `src="{imageUrl}"` usually wrong for a dynamic image?

It passes the literal string `{imageUrl}`, including the braces. Write `src={imageUrl}` to pass the variable's value.

### 9. Is `title={'Open details'}` valid?

Yes. It is a string expression. For a fixed literal, `title="Open details"` is simpler.

### 10. How do I pass a boolean property?

Pass a boolean value or expression, for example `disabled={isLocked}`. A boolean prop can also use shorthand when it should be true, such as `<input disabled />`.

### 11. Does `disabled="false"` mean the button is enabled?

No. That is a string, not the boolean value `false`. Use `disabled={false}` or pass a boolean variable. The expected value type depends on the specific property.

### 12. What are “double curly braces” in `style={{ color: 'navy' }}`?

The outer braces enter a JavaScript expression, and the inner braces create an object literal. It is ordinary JSX containing a JavaScript object; it is not a special double-brace syntax.

### 13. Why are inline style property names camelCase?

The `style` prop receives a JavaScript object, so its keys use JavaScript-compatible names such as `backgroundColor` instead of the CSS text spelling `background-color`. Most styling can still be easier to maintain in a CSS class.

## Rendering values

### 14. Can a plain object be rendered as a child?

No. An ordinary object such as `{ title: 'React' }` is not a renderable text child. Select a property, such as `course.title`, or use the object to create elements.

### 15. Which common values render visibly as JSX children?

Strings and numbers render as text. React elements and arrays of renderable nodes can create UI. Booleans, `null`, and `undefined` produce no visible text. A plain object cannot be rendered directly.

### 16. How do I show the word “true” or “false” to the user?

Convert the boolean into a display string, for example `{isReady ? 'Ready' : 'Not ready'}`. A boolean used directly as a child does not print the word.

### 17. How does a ternary expression help in JSX?

It chooses one of two values:

```jsx
<p>{isPublished ? 'Published' : 'Draft'}</p>
```

Use it when both outcomes are clear. For more complex logic, calculate the result before the return or split the UI into clearer conditions.

### 18. What is the `&&` conditional rendering pitfall with a number?

If the left side is `0`, the expression `0 && <List />` evaluates to `0`, and React renders the number. Compare explicitly, such as `items.length > 0 && <List />`.

## Escaping and safety

### 19. Does `<p>{message}</p>` parse a string as HTML?

No. A string rendered as an ordinary JSX child is displayed as text. If it contains tag-looking characters such as `<strong>`, React does not turn that string into a strong element.

### 20. What is `dangerouslySetInnerHTML`?

It is a prop that deliberately inserts raw HTML into a DOM element. It bypasses normal child rendering and should be used only when the markup is trusted and properly sanitized. Untrusted markup can create a cross-site scripting vulnerability.

### 21. Is normal text rendering the same as URL validation?

No. Rendering a string as text and validating an untrusted `href` or resource URL are different tasks. Apply URL rules appropriate to the application before using untrusted values as destinations.

### 22. How can I display a literal string that looks like JSX?

Put it in a JavaScript string expression, for example `{'<CourseCard />'}`. React renders the characters as text.

### 23. Can I use HTML character references in JSX text?

Common character references such as `&amp;` and `&lt;` can represent characters in literal JSX text. For dynamic content, pass the string value instead of trying to construct markup.

### 24. Does indentation control spaces in the rendered page?

No. JSX source indentation helps people read the structure, but ordinary whitespace is normalized similarly to HTML. Use CSS for visual spacing and explicit text for meaningful spaces or line breaks.

## Chapter navigation

| Previous chapter | Lesson | Example | Next chapter |
| --- | --- | --- | --- |
| [007 - JSX syntax and markup](../lessons/007-jsx-syntax-and-markup.md) | [Open lesson](../lessons/008-jsx-expressions-attributes-and-escaping.md) | [Open Example](../examples/008-jsx-expressions-attributes-and-escaping.md) | [009 - JSX styles, event props, and DOM properties](../lessons/009-jsx-styles-events-and-dom-properties.md) |
