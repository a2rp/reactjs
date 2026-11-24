# 009. JSX styles, event props, and DOM properties

## Learning goals

By the end of this lesson, you should be able to:

- Apply ordinary CSS classes to JSX with className.
- Use the style prop for values that depend on JavaScript data.
- Pass an event-handler function to a JSX event prop without calling it during render.
- Use React's DOM property names, including className, htmlFor, and readOnly.
- Pass boolean, aria-*, and data-* values with the right types and spelling.
- Choose semantic, keyboard-friendly HTML controls for interactive actions.

This chapter introduces the syntax and everyday rules. Chapter 025 studies event handling and user interactions in more depth; later chapters cover state, forms, and accessibility patterns.

## Add CSS classes with className

JSX uses className to set an element's CSS class. The class name is a string, and the corresponding CSS rule can live in a stylesheet:

~~~jsx
<article className="course-card">
  <h1 className="course-card__title">React basics</h1>
</article>
~~~

~~~css
.course-card {
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  padding: 1.5rem;
}

.course-card__title {
  margin: 0;
  color: #172554;
}
~~~

How a stylesheet is connected depends on the project setup. A project may import the CSS file from JavaScript or link it from the HTML document. React does not require one particular CSS-loading method.

Use classes for most layout, responsive rules, reusable visual patterns, pseudo-classes, and styles that are known in advance. This keeps presentation separate from component markup and lets normal CSS tools do their job.

In JSX, write className rather than the HTML spelling class:

~~~jsx
<div className="notice">Saved</div>
~~~

The class string can also come from a JavaScript value:

~~~jsx
<div className={isHighlighted ? 'notice notice--highlighted' : 'notice'}>
  Important update
</div>
~~~

Conditional class patterns and styling systems are covered in later chapters.

## Use the style prop for dynamic styles

The style prop accepts a JavaScript object. The object uses camelCase property names rather than CSS kebab-case names:

~~~jsx
<p style={{ color: 'navy', fontSize: 18 }}>
  A dynamically styled note
</p>
~~~

There are two brace pairs in this example:

- The outer braces tell JSX to evaluate a JavaScript expression for the prop.
- The inner braces create the JavaScript object passed as the style value.

React adds px to most numeric CSS length values. For example, fontSize: 18 means 18 pixels. Some CSS properties are unitless, such as opacity and lineHeight, so a number for those properties does not get px. Use a string when a value needs a particular unit, such as width: '50%' or marginTop: '1rem'.

Use style for values that genuinely depend on data at render time, such as a color selected from an item. Use a class for ordinary, reusable visual rules. Inline styles do not replace stylesheets: they are not a good place for media queries, pseudo-classes, or a large set of static declarations.

## Pass event handlers as props

An event handler is a function that React calls after an interaction. JSX event prop names use camelCase, such as onClick, onChange, and onMouseEnter. Pass a function value:

~~~jsx
function SaveButton() {
  function handleSave() {
    console.log('Save requested');
  }

  return (
    <button type="button" onClick={handleSave}>
      Save
    </button>
  );
}
~~~

The expression onClick={handleSave} passes a function reference. React can call that function when the click happens. Writing onClick={handleSave()} calls it immediately while the component renders, which is usually a mistake.

When a handler needs an argument, pass a new function that calls it later:

~~~jsx
<button type="button" onClick={() => handleSelect(course.id)}>
  Select course
</button>
~~~

The arrow function is created during rendering; handleSelect runs when the button is activated. Event objects, propagation, and interaction-specific details belong to the events chapters.

Keep render work pure: do not perform an action such as saving, navigating, or showing a dialog as a side effect of evaluating JSX. Put interaction work in an event handler. Do not pass a string as an event handler; event props expect functions.

## Use React DOM property names

JSX for built-in browser elements uses React DOM props. Many names follow the DOM property spelling and use camelCase. Several common differences from HTML are:

| HTML spelling | JSX spelling | Purpose |
| --- | --- | --- |
| class | className | Set CSS classes. |
| for | htmlFor | Associate a label with a form control. |
| tabindex | tabIndex | Set an element's tab order. |
| readonly | readOnly | Make a form control read-only. |
| maxlength | maxLength | Set a text control's length limit. |

For example, connect a label to an input with matching id and htmlFor values:

~~~jsx
<label htmlFor="email">Email address</label>
<input id="email" name="email" type="email" />
~~~

React keeps the hyphens in aria-* and data-* names:

~~~jsx
<button type="button" aria-pressed={isPinned} data-course-id={course.id}>
  Pin course
</button>
~~~

Use aria-* only to provide needed accessibility information. Prefer native HTML elements and attributes when they already express the behavior. A native button has keyboard and focus behavior built in; a clickable div does not become a button merely because it has an onClick prop.

## Pass values with the correct type

Use JavaScript booleans for boolean DOM props:

~~~jsx
<button type="button" disabled={isSaving}>
  Save
</button>
~~~

Do not use the string "false" for a boolean value. It is a string, not the boolean false, and may not produce the behavior you intend. For a controlled boolean attribute, calculate a real boolean in JavaScript.

Other common details:

- HTML text and URL-like attributes usually take strings.
- Numeric attributes should receive numbers when the DOM property expects a number.
- Boolean props such as disabled, checked, and readOnly should receive true or false.
- aria-* values use their documented accessible value; data-* values are strings in the DOM.
- A label's htmlFor value should match the id of its input.

## Keep application-only values off native elements

Props on a custom React component are inputs to that component. A built-in element such as button or div represents a real browser DOM element. Only pass it attributes and event props that are intended for that element.

Be careful when spreading an object onto a DOM element:

~~~jsx
// Avoid sending application-only fields to the browser element.
const { course, isPinned, ...buttonProps } = props;

return <button {...buttonProps} aria-pressed={isPinned}>{course.title}</button>;
~~~

This example first removes values used by the component itself, then applies the remaining intended button props. Do not forward arbitrary application data to the DOM just because it is available in an object.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| class="card" in JSX | JSX expects the React DOM prop name. | Use className="card". |
| for="email" on a label | The React DOM spelling is different. | Use htmlFor="email". |
| style="color: navy" | The style prop expects an object. | Use style={{ color: 'navy' }} or a class. |
| style={{ 'font-size': 18 }} | CSS property names in the object use camelCase. | Use style={{ fontSize: 18 }}. |
| onClick={save()} | The function runs while JSX is evaluated. | Use onClick={save}. |
| onClick="save" | An event handler must be a function. | Pass a function reference. |
| disabled="false" | A string is not the boolean false. | Use disabled={false} or a boolean expression. |
| Making a div clickable | It lacks a button's native interaction semantics. | Use a button for an action or a link for navigation. |
| Spreading all app props onto a DOM element | App-only values may leak to the browser element. | Forward only the DOM props the element should receive. |

## Key takeaways

- Use className to connect JSX elements to ordinary CSS classes.
- Use the style object for genuinely dynamic declarations; its property names are camelCase.
- Pass event-handler functions to camelCase event props such as onClick.
- Use className, htmlFor, and camelCase DOM property names in JSX; keep aria-* and data-* hyphenated.
- Pass booleans as booleans and keep component-only values off native DOM elements.
- Choose native semantic elements, especially buttons and links, for interactive controls.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [008 - JavaScript expressions, attributes, and escaping in JSX](008-jsx-expressions-attributes-and-escaping.md) | [Open Q&A](../qnas/009-jsx-styles-events-and-dom-properties.md) | [Open Example](../examples/009-jsx-styles-events-and-dom-properties.md) | [010 - Fragments and returning multiple elements](010-fragments-and-multiple-elements.md) |
