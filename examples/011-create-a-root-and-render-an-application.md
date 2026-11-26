# 011. Create a root and render a React application - Example

## Goal

Connect the browser document to a React app using Vite's usual file layout:

1. index.html provides an empty container element.
2. src/main.jsx finds that element and creates one React DOM root.
3. The root renders the App component from src/App.jsx.

This example uses the project setup from the earlier chapters. It does not add packages or configure Strict Mode; that is the next lesson.

## Project files

~~~text
react-learning-lab/
├── index.html
└── src/
    ├── main.jsx
    └── App.jsx
~~~

## index.html

Keep this at the Vite project root, beside package.json:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React Root Example</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

The id root is the mount container. The JavaScript entry looks up this same id. The module script starts the app through Vite.

## src/main.jsx

~~~jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const container = document.getElementById('root');

if (container === null) {
  throw new Error('Could not find the #root element in index.html.');
}

const root = createRoot(container);
root.render(<App />);
~~~

This is startup code, so create the root once here. The null check ensures the HTML element exists before createRoot is called.

## src/App.jsx

~~~jsx
export default function App() {
  return (
    <main>
      <h1>My first React root</h1>
      <p>
        The browser document provides a container, and React renders this app
        inside it.
      </p>
    </main>
  );
}
~~~

App returns the UI that React will render. It does not create another root; it is the component tree rendered inside the root created by main.jsx.

## Run and inspect

From the project folder that contains package.json, start the Vite development server:

~~~sh
npm run dev
~~~

Open the local address Vite prints. The page should show the heading and paragraph.

In browser developer tools, inspect the page:

- The document has one div with id root.
- That div contains the rendered main, h1, and p.
- The React component tree starts at App inside the root.
- There is no second root created by App.

The existing Vite project may have additional files such as global CSS, a favicon, or a different title. Keep those project details as needed; the important connection is the matching container id and entry module.

## Try these changes

1. Change the container id in index.html to app-root but leave main.jsx unchanged. The clear missing-container error should appear. Restore the matching id afterward.
2. Change the heading in App.jsx and confirm the development server updates the page.
3. Put a paragraph inside the root div in index.html. The first client render replaces the contents of that container. Remove the placeholder afterward.
4. Add a static header before the root div in index.html. It remains outside React's managed container.
5. Add a second call to createRoot for the same container and observe the warning/error. Remove it; one app root is enough here.

## Expected result

The browser shows a heading and short paragraph rendered by React inside the root container. The app has one createRoot call, made in the entry module, and one root.render call to display App.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="010-fragments-and-multiple-elements.md">010 - Fragments and returning multiple elements - Example</a></td><td><a href="../lessons/011-create-a-root-and-render-an-application.md">Lesson</a></td><td><a href="../qnas/011-create-a-root-and-render-an-application.md">Q&amp;A</a></td><td><a href="012-strict-mode-and-development-checks.md">012 - Strict Mode and development checks - Example</a></td></tr>
  </tbody>
</table>
