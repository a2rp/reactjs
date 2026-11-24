# 016. The children prop and nested content - Example

## Goal

Build a small lesson page that uses nested JSX in reusable wrappers. The example demonstrates:

- Receiving nested content with the `children` prop.
- Rendering one child, several children, and no children.
- Nesting a component inside another component's children.
- Grouping siblings with a Fragment.
- Using named props for separate header and footer regions.

## Files

In an existing Vite React project, create or update these files:

- src/main.jsx
- src/App.jsx

No additional package or CSS file is required.

## src/main.jsx

~~~jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('The root element was not found in index.html.');
}

createRoot(rootElement).render(<App />);
~~~

The Vite starter `index.html` should contain an element with the `root` id.

## src/App.jsx

~~~jsx
function Panel({ title, children }) {
  return (
    <section>
      <h2>{title}</h2>
      <div>{children}</div>
    </section>
  );
}

function TopicList() {
  return (
    <ul>
      <li>Choose a useful wrapper element.</li>
      <li>Render children where the content belongs.</li>
    </ul>
  );
}

function PageFrame({ header, children, footer }) {
  return (
    <div>
      <header>{header}</header>
      <section>
        <h2>Lesson content</h2>
        {children}
      </section>
      <footer>{footer}</footer>
    </div>
  );
}

export default function App() {
  return (
    <main>
      <h1>Nested content with children</h1>

      <Panel title="One child">
        <p>This paragraph is passed to Panel as its children.</p>
      </Panel>

      <Panel title="Several children">
        <>
          <p>A Fragment groups these sibling elements.</p>
          <TopicList />
        </>
      </Panel>

      <Panel title="No child content" />

      <PageFrame
        header={<h2>Named content regions</h2>}
        footer={<p>End of the lesson page.</p>}
      >
        <p>
          The parent supplies the header, main content, and footer separately.
        </p>
      </PageFrame>
    </main>
  );
}
~~~

`Panel` displays its nested content wherever it renders `children`. Its second use receives a Fragment containing a paragraph and a `TopicList` component. The self-closing `Panel` receives no visible child content.

`PageFrame` has three content areas. The parent passes `header` and `footer` as named JSX props, and the main area as nested `children`. Those named props are ordinary values; `PageFrame` decides where to place each one.

## Run the example

From the project folder that contains `package.json`, start Vite:

~~~sh
npm run dev
~~~

Open the local address printed by Vite. You should see a panel with one paragraph, a panel with a paragraph and topic list, an empty panel body, and a frame with separate header, main, and footer content.

## Try these changes

1. Remove `{children}` from the `Panel` return value. The nested paragraphs and list should disappear. Restore it afterward.
2. Replace the Fragment inside the second `Panel` with a `div`. Inspect the DOM and compare the extra wrapper element.
3. Add another `<p>` inside the Fragment. It should appear in the same panel body.
4. Replace `<TopicList />` with text. The wrapper should not need to change.
5. Change the content passed to `header` or `footer` in `PageFrame`. The frame should render the new JSX in the corresponding region.
6. Add a self-closing `<Panel title="Another empty panel" />` and observe that its body has no visible content.

## What this example demonstrates

- Nested JSX becomes the receiving component's `children` prop.
- A wrapper must render `children` to display the nested content.
- The children value can be one element, a Fragment with siblings, or empty.
- A child component can be supplied by the caller and rendered by the wrapper.
- Fragments group sibling JSX without adding a DOM wrapper.
- Named JSX props are useful when a component has distinct content regions.

## Chapter navigation

| Lesson | Q&A | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [016 - The children prop and nested content](../lessons/016-children-prop-and-nested-content.md) | [Open Q&A](../qnas/016-children-prop-and-nested-content.md) | [015 - Destructuring props and default values](../lessons/015-destructuring-props-and-default-values.md) | [017 - Composition, containment, and reusable layouts](../lessons/017-composition-containment-and-layouts.md) |
