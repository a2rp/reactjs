# 004. Q&A: JavaScript and browser prerequisites

Use this review to check the JavaScript and browser basics that appear in React projects. This is a practical foundation, not a complete JavaScript language reference.

## JavaScript fundamentals

### 1. Why does learning JavaScript matter for React?

React is a JavaScript library, and function components are JavaScript functions. Props, state, conditions, array transformations, event handlers, and modules all use JavaScript language features.

### 2. When should I use `const` and `let`?

Use `const` when the variable binding will not be reassigned. Use `let` when you need to reassign the binding. In modern React code, `var` is generally not needed.

### 3. Does `const` make an object immutable?

No. `const` prevents reassigning the variable binding. The object's properties could still be mutated. React code should treat props and state objects as immutable and create replacement values when changing them.

### 4. What is the difference between an object and an array?

An object groups named properties, such as `{ title: 'React' }`. An array stores an ordered collection, such as `['HTML', 'CSS']`. React applications commonly use objects and arrays to represent data.

### 5. What does a function do?

A function is reusable JavaScript behavior. It can receive values through parameters and return a result. A function component uses this language feature to describe UI.

### 6. What is an arrow function?

It is another way to define a function, for example `const double = number => number * 2`. Arrow functions are commonly used for callbacks and short array transformations.

### 7. What is a callback?

A callback is a function passed to other code so it can be called later. React event handlers are callbacks: the function is passed to an event prop and React calls it when the event occurs.

### 8. What is destructuring?

Destructuring extracts properties from an object or values from an array. A component can write `function Card({ title })` to read its `title` prop directly.

### 9. What is a template string?

A string written with backticks that can embed expressions using `${...}`, for example `` `Hello, ${name}` ``.

### 10. Why prefer `===` for equality comparisons?

Strict equality compares without first coercing values to another type. It usually makes the intended comparison more predictable than loose equality.

## JavaScript inside JSX

### 11. What do curly braces mean inside JSX?

They let JSX evaluate a JavaScript expression. For example, `<h1>{course.title}</h1>` displays the value of the expression `course.title`.

### 12. How do quotes and braces differ in a JSX attribute?

Quotes pass literal text, while braces evaluate JavaScript:

```jsx
<img alt="Course cover" src={course.imageUrl} />
```

The `alt` is literal text; `src` receives the value of `course.imageUrl`.

### 13. Is JSX itself JavaScript or HTML?

JSX is a JavaScript syntax extension that looks like markup. A configured build tool or runtime transforms it; browsers do not parse JSX syntax directly.

### 14. How do I render several array values?

Use JavaScript methods such as `map` to transform data into elements:

```jsx
<ul>
  {courses.map(course => (
    <li key={course.id}>{course.title}</li>
  ))}
</ul>
```

This example uses stable keys; list-key rules are taught in a later chapter.

### 15. What do `map` and `filter` do?

`map` creates a new array by transforming each item. `filter` creates a new array containing items that pass a condition. They are often used to prepare data for a rendered list.

### 16. Why can `0 && <p>Message</p>` display a zero?

In JavaScript, `&&` returns one of its operands. When the left operand is `0`, the expression evaluates to `0`, and React renders that number. Use a boolean test such as `items.length > 0 && ...` if that is the intended condition.

### 17. What is the difference between a string and an expression in JSX?

`<p>"{name}"</p>` displays quoted literal text. `<p>{name}</p>` evaluates the variable and displays its value.

## Modules and project files

### 18. What do `import` and `export` do?

They let JavaScript modules share values. A module exports functions, data, or components; another file imports them using a path and the appropriate default or named syntax.

### 19. What is the difference between a default and a named export?

A file can have one default export, imported without braces using the importing name of your choice. It can have multiple named exports, imported by their exported names inside braces.

### 20. Why can an import fail even when the function exists?

The file path, file name, named export, or letter casing may not match. Check that the path points to the right file and that import syntax matches the export type.

### 21. What does the build tool do with JSX and modules?

It processes project source so the browser can load it. Depending on setup, it may transform JSX, resolve imports, process CSS, and prepare production assets.

## Browser, events, and asynchronous work

### 22. What is the difference between passing and calling an event handler?

Passing `onClick={handleClick}` gives React a function to call later. Writing `onClick={handleClick()}` executes the function immediately during rendering and passes its result instead.

### 23. What is a Promise?

A Promise represents a value or failure that will be available later. Network requests are asynchronous and commonly use Promises.

### 24. What does `async` and `await` do?

An `async` function returns a Promise. Inside it, `await` pauses that function's flow until a Promise settles, while other JavaScript work can continue. Errors still need to be handled.

### 25. Does React automatically call `fetch` for a component?

No. Application code, a framework, or a data library decides when to fetch data and how to handle loading, success, and error states. React renders the UI from the data it receives.

### 26. Can browser JavaScript use every server API?

No. The environments provide different APIs. Browser code can use `document` and browser UI APIs; server code can use server capabilities and protect private credentials. Server-only values must not be exposed in browser bundles.

### 27. Should a component manually find and rewrite its own DOM nodes?

Usually no. React should manage the DOM inside its root from the component description. Refs provide limited DOM access for tasks such as focus, but arbitrary direct mutations can conflict with React updates.

### 28. Which developer tools help when a React page is blank?

Start with the browser Console for errors and the Network panel for missing scripts or failed requests. Use Elements to inspect the rendered DOM and CSS; use Sources/Debugger for breakpoints and source inspection.

### 29. Does editing the Elements panel update my source code?

No. It changes the current in-memory browser page. Update the project source and reload or let the development server refresh the browser to see a lasting change.

### 30. Do I need to know all of JavaScript before starting React?

No. A working understanding of functions, values, objects, arrays, modules, conditions, and callbacks is enough to begin. Learn additional language features as they appear.

## Quick scenarios

### Scenario A: A component expects `name` as a prop

Should it display `"{name}"` or `{name}` to show the variable value?

**Answer:** Use `{name}` as a JavaScript expression.

### Scenario B: A click action runs as soon as the page loads

The JSX contains `onClick={handleSave()}`. What is the likely mistake?

**Answer:** The code calls the function during render. Pass the function instead: `onClick={handleSave}`.

### Scenario C: A list of courses should include only published courses

Which array method can create the filtered data?

**Answer:** `filter`.

### Scenario D: An import reports that a named export does not exist

What should you check?

**Answer:** Verify the exported name, the file path and casing, and whether the code uses named or default import syntax.

### Scenario E: A network request fails

Where can you see its request and response details?

**Answer:** In the browser developer tools' Network panel; JavaScript errors and rejected work may also appear in the Console.

## Quick self-check

1. What does a function parameter receive?
2. What does `{course.title}` do in JSX?
3. Which method transforms every array item?
4. Which method keeps only matching array items?
5. Does a browser bundle keep secrets private?
6. What does `onClick={save}` pass to React?
7. Which browser panel is useful for request failures?
8. Do React and the browser provide identical APIs?

### Suggested answers

1. An input value supplied when the function is called.
2. It evaluates the JavaScript property and renders its value.
3. `map`.
4. `filter`.
5. No. Client-delivered code and values are inspectable.
6. The `save` function as a callback.
7. Network, and Console for related errors.
8. No. Browser and server environments expose different APIs.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="003-react-react-dom-and-frameworks.md">003 - Q&amp;A: React, React DOM, and the framework ecosystem</a></td><td><a href="../lessons/004-javascript-and-browser-prerequisites.md">Lesson</a></td><td><a href="../examples/004-javascript-and-browser-prerequisites.md">Example</a></td><td><a href="005-choose-a-react-project-setup.md">005 - Q&amp;A: Choose a React project setup</a></td></tr>
  </tbody>
</table>
