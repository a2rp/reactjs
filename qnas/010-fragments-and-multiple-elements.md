# 010. Fragments and returning multiple elements — Q&A

## Returning and grouping JSX

### 1. Why can a component not return two adjacent JSX elements directly?

A component's return expression must produce one value. Two sibling JSX expressions need to be grouped into one expression, such as a Fragment or a real parent element.

### 2. What does a Fragment do?

A Fragment groups multiple React children so they can be returned together. It does not add a wrapper element to the browser DOM.

### 3. What is the Fragment shorthand?

The shorthand is an empty opening JSX tag and a matching closing tag surrounding the children:

~~~jsx
<>
  <h1>Overview</h1>
  <p>Read the course outline.</p>
</>
~~~

### 4. Do I need to import anything to use the shorthand?

No. The empty-tag shorthand is part of JSX syntax and does not need an import.

### 5. Does a component have to return one DOM element?

No. It must return one JSX expression. That expression may be a Fragment whose children become separate sibling DOM elements.

### 6. Can I use a Fragment in a conditional expression?

Yes. A branch can return a Fragment when that branch needs to show several siblings. The conditional still evaluates to one value.

### 7. Can a Fragment be used somewhere other than a component's return?

Yes. You can use it anywhere JSX expects one element expression, such as a conditional branch or as a child grouping. Use it when grouping is needed and no DOM wrapper is desired.

## DOM, semantics, and styling

### 8. Does a Fragment appear as a node in the browser DOM?

No. Its children appear in the DOM as siblings. The Fragment itself does not become a browser element.

### 9. Does a Fragment change layout or add a CSS box?

No. It adds no DOM wrapper box. Its children participate in the surrounding DOM and CSS as siblings.

### 10. Can I put className, id, or an event handler on the shorthand?

No. The shorthand cannot receive props. It does not create a DOM node to style or attach an event handler to.

### 11. Should I use a Fragment or a div?

Use a Fragment when you only need to group siblings. Use a real element such as section, ul, or div when the group needs meaning, styling, layout, or DOM attributes. Choose a semantic element when one fits the content.

### 12. Does a Fragment add semantics or accessibility behavior?

No. A Fragment is only a grouping mechanism in React; it does not add a role, landmark, label, or other browser semantics. Use appropriate HTML elements for those needs.

### 13. Can a Fragment fix invalid HTML nesting?

No. It adds no wrapper, so its children still become children of the same real parent. The resulting HTML structure must obey that parent's content rules.

## Explicit and keyed Fragments

### 14. When should I use the explicit Fragment component?

Use the explicit Fragment form when the Fragment needs a supported prop such as key. Import Fragment from React and render it as a JSX component.

### 15. Why can the shorthand not be used for a keyed list group?

The shorthand cannot take props, including key. Use the explicit form:

~~~jsx
import { Fragment } from 'react';

items.map((item) => (
  <Fragment key={item.id}>
    <dt>{item.title}</dt>
    <dd>{item.description}</dd>
  </Fragment>
))
~~~

### 16. Where does the key go when a list item returns multiple siblings?

Put the key on the outer Fragment returned for that list item. The key identifies that group among its siblings. You do not need to repeat that key on each child inside the Fragment.

### 17. What should I use as a Fragment key?

Use a stable, unique identifier from the item data, such as a database ID. Avoid generating a random value during rendering. Chapter 019 explains list keys in more detail.

### 18. Can I style an explicit Fragment with className?

No. The explicit Fragment does not create a DOM element either. It supports grouping props such as key, but not DOM styling attributes like className. Use a real element if the group needs a class.

### 19. Can Fragments be nested?

Yes. A Fragment can contain elements and other Fragments. Keep the structure as simple as needed; adding unnecessary nested groups can make JSX harder to read.

### 20. Is a Fragment the only way to return a group?

No. A real HTML element can group the children when it has a useful purpose. Arrays of React nodes can also represent multiple children, but each item in a rendered list needs an appropriate key. A Fragment is a clear choice when you need grouping without a wrapper element.

### 21. Are React Fragments HTML tags?

No. There is no HTML element named fragment for this purpose. Use the JSX shorthand or import React's Fragment component.

### 22. What should I inspect to confirm a Fragment added no wrapper?

Inspect the rendered elements in browser developer tools. You should see the Fragment's child elements as siblings, without an extra wrapper between them.

## Chapter navigation

| Lesson | Example | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [010 - Fragments and returning multiple elements](../lessons/010-fragments-and-multiple-elements.md) | [Open Example](../examples/010-fragments-and-multiple-elements.md) | [009 - JSX styles, event props, and DOM properties](../lessons/009-jsx-styles-events-and-dom-properties.md) | [011 - Create a root and render a React application](../lessons/011-create-a-root-and-render-an-application.md) |
