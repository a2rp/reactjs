# 009. JSX styles, event props, and DOM properties - Q&A

## CSS classes and the style prop

### 1. How do you assign a CSS class in JSX?

Use the className prop:

~~~jsx
<div className="course-card">...</div>
~~~

The className value should match a selector in a stylesheet that is loaded by the project.

### 2. Why does JSX use className instead of class?

Built-in JSX elements use React DOM property names. For a CSS class, React uses className. The spelling differs from the HTML attribute name.

### 3. When should I use a CSS class instead of the style prop?

Use classes for ordinary, reusable, responsive, or state-independent presentation. Classes work with media queries, pseudo-classes, and the full CSS cascade.

Use the style prop when a declaration depends on a JavaScript value at render time, such as an item's data-driven color.

### 4. What shape does the style prop expect?

It expects a JavaScript object whose keys are CSS property names in camelCase:

~~~jsx
<p style={{ backgroundColor: 'lavender', marginTop: 12 }}>
  Notice
</p>
~~~

The outer JSX braces evaluate an expression; the inner braces create the object.

### 5. How do I write a CSS property such as font-size in a style object?

Use camelCase: fontSize. Do not use the CSS spelling font-size as an ordinary object key.

### 6. Does a numeric style value automatically get a unit?

For most CSS length properties, React appends px to a numeric value. For example, fontSize: 18 is 18 pixels. Unitless properties such as opacity do not receive px. Use a string for a specific unit, for example width: '50%'.

### 7. Can the style prop replace a stylesheet?

No. It is best for a small number of dynamic declarations. Use stylesheets and classes for static rules, responsive layout, pseudo-classes, and shared presentation.

## Event props and handler functions

### 8. How do you attach a click handler to a JSX button?

Define a function and pass it to onClick:

~~~jsx
function handleSave() {
  console.log('Save requested');
}

<button type="button" onClick={handleSave}>Save</button>
~~~

React calls the function when the user activates the button.

### 9. Why is onClick={handleSave} different from onClick={handleSave()}?

The first form passes the function for React to call later. The second calls it immediately while the component renders, then passes its return value. For a normal event handler, pass the function reference without parentheses.

### 10. How can I pass an argument to a handler?

Pass an arrow function that calls the handler with the argument:

~~~jsx
<button onClick={() => handleSelect(course.id)}>
  Select
</button>
~~~

This defers the call until the event. Writing onClick={handleSelect(course.id)} evaluates the call during rendering.

### 11. Are event prop names case-sensitive?

Yes. Use React's camelCase event prop names, such as onClick, onChange, and onMouseEnter.

### 12. Can an event prop receive a string containing a function name?

No. Event props expect a function value, not a string. Define or reference a function in JavaScript and pass that value.

### 13. Should a component perform an action while rendering JSX?

Do not trigger interaction side effects while rendering. Rendering should describe the interface. Put work such as saving or responding to a click in an event handler.

### 14. Do I need to understand the event object to write a basic handler?

No. A basic handler can ignore the event object. The later event chapters explain the event parameter, propagation, and interaction-specific behavior.

## DOM properties and values

### 15. What is the JSX spelling for an HTML label's for attribute?

Use htmlFor:

~~~jsx
<label htmlFor="email">Email</label>
<input id="email" name="email" />
~~~

The htmlFor value should match the control's id.

### 16. How do I write aria-* and data-* props?

Keep the hyphens:

~~~jsx
<button aria-pressed={isPinned} data-course-id="react-basics">
  Pin
</button>
~~~

ARIA attributes describe accessible information; data attributes attach string data to an element.

### 17. How should I pass a boolean prop such as disabled?

Pass a JavaScript boolean or an expression that evaluates to one:

~~~jsx
<button disabled={isSaving}>Save</button>
~~~

Do not pass the string "false". A string is not the boolean false.

### 18. What does readOnly do, and how is it spelled in JSX?

It marks a form control as read-only. JSX uses camelCase:

~~~jsx
<input value="Reference text" readOnly />
~~~

This differs from disabling a control: read-only content remains available for selection and focus behavior according to the browser, while a disabled control is disabled.

### 19. Why should a clickable action use a button rather than a div?

A native button provides built-in keyboard activation, focus handling, and button semantics. A div with onClick does not automatically provide those behaviors. Use a button for an action and an anchor for navigation.

### 20. Is aria-pressed useful on a button?

Use aria-pressed for a button that represents a toggle state, and pass the current boolean state. It helps assistive technology expose whether the toggle is pressed. The stateful toggle behavior itself is covered later.

### 21. Is it always safe to spread all component props onto a native DOM element?

No. Component props may include application-only values that the browser element should not receive. Remove those values or explicitly select the DOM props to forward.

### 22. How can I recognize a style or DOM prop mistake?

Check the browser console and the JSX spelling. Common issues include using class instead of className, for instead of htmlFor, kebab-case in a style object, a string instead of a boolean, or calling a handler during render.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="008-jsx-expressions-attributes-and-escaping.md">008 - Q&amp;A: JavaScript expressions, attributes, and escaping in JSX</a></td><td><a href="../lessons/009-jsx-styles-events-and-dom-properties.md">Lesson</a></td><td><a href="../examples/009-jsx-styles-events-and-dom-properties.md">Example</a></td><td><a href="010-fragments-and-multiple-elements.md">010 - Fragments and returning multiple elements - Q&amp;A</a></td></tr>
  </tbody>
</table>
