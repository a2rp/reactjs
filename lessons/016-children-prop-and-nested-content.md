# 016. The children prop and nested content

## Learning goals

By the end of this lesson, you should be able to:

- Explain how nested JSX becomes a component's `children` prop.
- Render children in the place chosen by the receiving component.
- Pass one child, several children, a Fragment, or no child content.
- Choose between a general `children` area and named content props.
- Avoid assuming that `children` is always an array or that it renders automatically.

## Nested JSX becomes the children prop

A JSX component can have content between its opening and closing tags:

~~~jsx
<Panel title="Study notes">
  <p>Read the lesson, then try the example.</p>
</Panel>
~~~

React passes that nested content to `Panel` through a prop named `children`. The component can read and render it like any other prop:

~~~jsx
function Panel(props) {
  return (
    <section>
      <h2>{props.title}</h2>
      <div>{props.children}</div>
    </section>
  );
}
~~~

The parent chooses what content to provide. The panel chooses where that content appears. This lets a reusable wrapper provide a heading and layout without needing to know the exact markup inside it.

## Children do not render automatically

Receiving children does not make them appear by itself. The component must include the value in its returned JSX:

~~~jsx
function Panel(props) {
  return (
    <section>
      <h2>{props.title}</h2>
      {props.children}
    </section>
  );
}
~~~

If `props.children` is omitted from the return value, the nested content is not displayed. A component may also intentionally put the children in a different location or render them more than once.

The expression braces in `{props.children}` tell JSX to insert the JavaScript value into the returned UI.

## Receive children with destructuring

Because `children` is a regular prop, it can be destructured:

~~~jsx
function Panel({ title, children }) {
  return (
    <section>
      <h2>{title}</h2>
      {children}
    </section>
  );
}
~~~

The parent still supplies the content with nested JSX:

~~~jsx
<Panel title="Remember">
  <p>Props flow from a parent to its child.</p>
</Panel>
~~~

The component's function parameter receives both `title` and `children`. Destructuring changes how the values are read, not how JSX passes them.

## Children can be different kinds of React nodes

A child value can be text, a number, an element, another component, an array of nodes, or an empty value such as `null` or a boolean. For example:

~~~jsx
<Panel title="A text child">Read the next section.</Panel>
<Panel title="An element child"><p>Practice the example.</p></Panel>
<Panel title="No visible child">{null}</Panel>
~~~

Boolean children do not display visible text. A component should render children as React nodes rather than converting them to strings. For example, `String(children)` would turn elements into unhelpful text instead of rendering their UI.

If the parent supplies no nested content, `children` is undefined. A self-closing component is a clear way to show that it has no child content:

~~~jsx
<Panel title="Introduction" />
~~~

## One child and several children

A component may receive one child or several nested children:

~~~jsx
<Panel title="One child">
  <p>One paragraph.</p>
</Panel>

<Panel title="Several children">
  <p>A paragraph.</p>
  <ul>
    <li>One topic</li>
    <li>Another topic</li>
  </ul>
</Panel>
~~~

Do not assume `children` is always an array. Its value depends on what the parent supplied; a single paragraph is not the same shape as several sibling nodes. If a component needs to render a collection from data, pass an array prop and map over that data instead of assuming a particular `children` shape.

## Use a Fragment to group siblings

If a component needs several sibling nodes but should not add a wrapper element, use a Fragment:

~~~jsx
<Panel title="Next steps">
  <>
    <p>Review the key terms.</p>
    <p>Then build the example from memory.</p>
  </>
</Panel>
~~~

The Fragment lets the parent provide one JSX expression containing multiple siblings without adding an extra `div` to the rendered DOM. The receiving component can render the Fragment through `children` like other JSX.

Use a semantic element when the content has a meaning or structural role, such as `ul` or `section`. Use `div` for a generic container when one is needed. Use a Fragment when the only purpose of a wrapper would be to group JSX.

## Choose a children area or named content props

Use `children` when a component has one general content area, such as a panel, card frame, dialog body, or page layout:

~~~jsx
<Card title="Course overview">
  <p>Learn how components receive data.</p>
  <TopicList />
</Card>
~~~

Use named props when the component has distinct places for different pieces of content:

~~~jsx
function PageFrame({ header, children, footer }) {
  return (
    <div>
      <header>{header}</header>
      <main>{children}</main>
      <footer>{footer}</footer>
    </div>
  );
}
~~~

The parent can provide those areas as JSX values:

~~~jsx
<PageFrame
  header={<h1>React notes</h1>}
  footer={<p>End of page</p>}
>
  <LessonList />
</PageFrame>
~~~

A named prop such as `header` is not special React syntax; it is an ordinary prop whose value happens to be JSX. Choose names that make the component's content areas clear.

## Composition keeps content with the caller

A wrapper can render child components supplied by its parent without importing or defining those components itself:

~~~jsx
function Notice({ children }) {
  return <aside className="notice">{children}</aside>;
}

function TipContent() {
  return (
    <>
      <strong>Tip:</strong>
      <p>Try the code before reading the answer.</p>
    </>
  );
}
function App() {
  return (
    <Notice>
      <TipContent />
    </Notice>
  );
}
~~~

The code that uses `Notice` decides what goes inside. `Notice` decides how to wrap it. This separation makes the wrapper reusable for different content.

Nested components still render as components. Passing a JSX element through `children` does not call the component like a regular function; React renders it when the returned tree includes it.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Forgetting to render `children` | JSX between the tags is received but never included in the output. | Place `{children}` where the content should appear. |
| Assuming children are always an array | One child, several children, and empty content can have different shapes. | Render children directly or pass an explicit data array when mapping is needed. |
| Converting children to a string | Elements become unhelpful text instead of rendered UI. | Insert the React node directly with `{children}`. |
| Wrapping siblings in an unnecessary `div` | The extra DOM element may affect semantics, styling, or layout. | Use a Fragment when no real wrapper element is needed. |
| Using `children` for many unrelated named areas | The component API becomes unclear about where each piece belongs. | Use named JSX props such as `header` and `footer`. |
| Defining every nested child inside the wrapper | The wrapper becomes coupled to content that should belong to its caller. | Let the caller compose content and pass it through `children`. |

## Key takeaways

- JSX nested between a component's tags is passed as the `children` prop.
- The receiving component must include children in its returned JSX to display them.
- Children can be text, elements, components, arrays, or empty values.
- Do not assume children are always an array or turn them into strings.
- Use Fragments to group siblings without adding a DOM wrapper.
- Use named JSX props when a component has distinct content regions.
- The caller supplies content; the wrapper decides where and how it is rendered.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [015 - Destructuring props and default values](015-destructuring-props-and-default-values.md) | [Open Q&A](../qnas/016-children-prop-and-nested-content.md) | [Open Example](../examples/016-children-prop-and-nested-content.md) | [017 - Composition, containment, and reusable layouts](017-composition-containment-and-layouts.md) |
