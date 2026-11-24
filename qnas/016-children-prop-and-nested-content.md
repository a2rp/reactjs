# 016. The children prop and nested content - Q&A

## The children prop

### 1. What is the `children` prop?

It is the prop React uses for content nested between a component's opening and closing JSX tags:

~~~jsx
<Panel>
  <p>Read the lesson first.</p>
</Panel>
~~~

The paragraph is supplied to `Panel` as `children`.

### 2. Is `children` a special JavaScript keyword?

No. It is a conventional prop name that JSX fills with nested content. A component can read it like any other prop, including by destructuring it.

### 3. How does a component read and display children?

Read the prop and include it in the component's returned JSX:

~~~jsx
function Panel({ children }) {
  return <section>{children}</section>;
}
~~~

The braces insert the JavaScript value into the returned JSX.

### 4. Does nested JSX render automatically even if the component ignores `children`?

No. If the component does not put its children into the returned tree, that content is not displayed.

### 5. Can I pass `children` as a normal explicit prop?

Yes, though nested JSX is usually clearer for content:

~~~jsx
<Panel children={<p>Read the lesson first.</p>} />
~~~

The equivalent nested form is:

~~~jsx
<Panel>
  <p>Read the lesson first.</p>
</Panel>
~~~

### 6. What is the value of `children` if nothing is nested?

For a self-closing component such as `<Panel />`, the component receives no `children` value; reading it gives `undefined`. Explicitly nesting `{null}` gives a null value, which also renders no visible content.

## Child values and shape

### 7. What kinds of values can children contain?

Children can be text, numbers, React elements, other components, arrays of nodes, or empty values such as `null` and booleans. React renders React nodes as UI; a component should not convert them to strings.

### 8. Does a boolean child display the words `true` or `false`?

No. Boolean values are empty React nodes, so they produce no visible text. If the user should see a word, render a string such as `"Enabled"` explicitly.

### 9. Does a numeric child display?

Yes. Numbers are rendered as text. For example, `<Panel>{0}</Panel>` displays `0`. This differs from boolean children, which do not display.

### 10. Is `children` always an array?

No. It may be a single element, a string, a number, an array, or an empty value. Do not assume that `children.map(...)` is always valid.

### 11. What should I do if I need to render a list of data?

Pass the list as an explicit array prop and map over the data. The `children` prop is for already composed React nodes; its shape is not a reliable substitute for a data collection.

### 12. Can children include another component?

Yes. The parent can nest a component element inside another component:

~~~jsx
<Notice>
  <LessonLink />
</Notice>
~~~

When `Notice` renders `children`, React renders `LessonLink` as part of the returned component tree.

### 13. Should the receiving component call a child component function directly?

No. Include its JSX element in the children or returned tree. React should render components through JSX so it can manage their component boundaries.

## Fragments and layout

### 14. When should I use a Fragment inside children?

Use a Fragment to group sibling elements when the parent needs one JSX expression but an extra DOM wrapper would not help:

~~~jsx
<Panel>
  <>
    <p>Review the terms.</p>
    <p>Then try the example.</p>
  </>
</Panel>
~~~

The Fragment groups the siblings without adding a DOM element.

### 15. Is a Fragment the same as an empty child?

No. An empty child such as `null` renders nothing. A Fragment can contain one or more children and renders those children without its own DOM wrapper.

### 16. Should I always wrap children in a `div`?

No. Use a real element when it provides meaning or a useful structural container. Use a Fragment when only grouping is needed and no extra DOM node should be added.

### 17. Can a wrapper render the same children more than once?

Yes. A component can insert `children` in multiple places in its JSX, and React renders that content at each location. Do this only when duplicating the content is the intended design.

### 18. When should I use named JSX props instead of `children`?

Use `children` for one general content area. Use named props such as `header`, `footer`, or `actions` when the component has separate, meaningful content regions:

~~~jsx
<PageFrame
  header={<h1>Course</h1>}
  footer={<p>End of course</p>}
>
  <LessonList />
</PageFrame>
~~~

Those named props are ordinary props that happen to contain JSX.

## Composition and ownership

### 19. Which component decides what content goes inside a wrapper?

The caller supplies the content; the wrapper decides where and how to render it. For example, `App` can supply a `LessonLink` to `Notice`, and `Notice` can choose to place it inside an `aside`.

### 20. Does passing a child through `children` make the wrapper responsible for that child's implementation?

No. The wrapper receives a React node and can render it without knowing its implementation. This keeps the wrapper reusable for different content.

### 21. Can the parent decide which children to pass?

Yes. The parent can use its data and conditional rendering to include different JSX for different situations. The receiving component renders whichever children it gets.

### 22. Is the `children` prop the same as the browser DOM's `childNodes`?

No. `children` is a React prop containing React nodes supplied through JSX. React uses the returned component tree to manage the browser DOM; the prop itself is not a DOM node list.

## Chapter navigation

| Lesson | Example | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [016 - The children prop and nested content](../lessons/016-children-prop-and-nested-content.md) | [Open Example](../examples/016-children-prop-and-nested-content.md) | [015 - Destructuring props and default values](../lessons/015-destructuring-props-and-default-values.md) | [017 - Composition, containment, and reusable layouts](../lessons/017-composition-containment-and-layouts.md) |
