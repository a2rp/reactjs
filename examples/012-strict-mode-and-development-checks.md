# 012. Strict Mode and development checks — Example

## Goal

Enable Strict Mode around a Vite app, then observe two useful development checks:

- Rendering derives a new list without mutating the input data.
- An Effect removes the browser event listener it installs.

The Effect here is a small preview of a topic covered in depth later. The same example runs in production, but the extra Strict Mode checks and their setup-cleanup-setup sequence are development-only.

## Files

Use these files in the Vite project:

- src/main.jsx
- src/App.jsx

No extra package or CSS file is required.

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const container = document.getElementById('root');

if (container === null) {
  throw new Error('The root container was not found in index.html.');
}

const root = createRoot(container);

root.render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

StrictMode wraps the component tree and enables its development checks. It adds no DOM element.

## src/App.jsx

~~~jsx
import { useEffect } from 'react';

const startingTopics = [
  { id: 'jsx', label: 'JSX' },
  { id: 'components', label: 'Components' },
];

function TopicList({ topics }) {
  const displayedTopics = [
    ...topics,
    { id: 'create', label: 'Create a new topic' },
  ];

  return (
    <ul>
      {displayedTopics.map((topic) => (
        <li key={topic.id}>{topic.label}</li>
      ))}
    </ul>
  );
}

function ResizeListenerCheck() {
  useEffect(() => {
    function handleResize() {
      console.log('Window resize event received.');
    }

    window.addEventListener('resize', handleResize);
    console.log('Resize listener connected.');

    return () => {
      window.removeEventListener('resize', handleResize);
      console.log('Resize listener disconnected.');
    };
  }, []);

  return <p>Resize the browser window and check the console.</p>;
}

export default function App() {
  return (
    <main>
      <h1>Strict Mode checks</h1>
      <p>The topic list is derived without changing its source array.</p>
      <TopicList topics={startingTopics} />
      <ResizeListenerCheck />
    </main>
  );
}
~~~

## Run and observe

From the project folder containing package.json, start the Vite development server:

~~~sh
npm run dev
~~~

Open the local address Vite prints, then open the browser developer console.

In development, the initial Effect logs a sequence like:

~~~text
Resize listener connected.
Resize listener disconnected.
Resize listener connected.
~~~

The extra cleanup between setups is Strict Mode checking that the Effect can undo its work. Resize the window; each resize should produce one event message because the earlier listener was removed before the active one was installed.

The page should show a heading, a short explanation, and three topic items. Strict Mode's extra render check should not add duplicate list entries because TopicList creates a new array and does not change startingTopics.

## Try these changes

1. Temporarily replace the displayedTopics array with the original topics array and append the new item to it with push. Observe that development checks can expose repeated mutation. Restore the copied-array version.
2. Temporarily remove removeEventListener from the Effect cleanup. Resize the window and observe that extra active listeners can produce repeated messages. Restore the cleanup.
3. Change a topic label and confirm the list updates without changing its id.
4. Remove the StrictMode wrapper and compare the initial Effect logs in development. Restore Strict Mode after comparing.
5. Run npm run build and then npm run preview from the project folder. Open the preview URL and compare its console behavior with the development server; the extra Strict Mode checks are development-only.

## What this example demonstrates

- StrictMode belongs around the app tree in the entry module.
- The list component calculates output from its input without mutating the input array.
- Effects that install browser listeners should remove those listeners during cleanup.
- The development setup-cleanup-setup sequence is a diagnostic, not an instruction to keep duplicate listeners.
- Removing Strict Mode hides the check; it does not repair mutation or missing cleanup.

## Chapter navigation

| Lesson | Q&A | Previous chapter | Next chapter |
| --- | --- | --- | --- |
| [012 - Strict Mode and development checks](../lessons/012-strict-mode-and-development-checks.md) | [Open Q&A](../qnas/012-strict-mode-and-development-checks.md) | [011 - Create a root and render a React application](../lessons/011-create-a-root-and-render-an-application.md) | [013 - Function components and component naming](../lessons/013-function-components-and-naming.md) |
