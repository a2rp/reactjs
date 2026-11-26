# 011. Create a root and render a React application

## Learning goals

By the end of this lesson, you should be able to:

- Identify the HTML element where a client-rendered React app starts.
- Connect that element to React DOM with createRoot.
- Render the root component with root.render.
- Explain why a typical React app creates its root once at startup.
- Distinguish client rendering with createRoot from hydration with hydrateRoot.
- Recognize when an app may need to unmount a root.

This lesson uses the Vite project structure from chapter 006. Frameworks may create and manage the root for you; the entry-point code here is for a browser app started directly with React DOM.

## The root is the connection point

A browser document is an HTML tree. React needs a real DOM element in that document where it can manage a component tree. That element is often called the root container or mount element.

A simple HTML document can provide an empty container:

~~~html
<div id="root"></div>
~~~

The id is a project convention, not a special React keyword. Your JavaScript can find a different id as long as the HTML and entry module agree. In a Vite project, the document is the project-root index.html and the JavaScript entry is commonly src/main.jsx.

The startup path is:

~~~text
index.html
   |
   | provides the root DOM element
   v
src/main.jsx
   |
   | creates a React DOM root
   v
root.render(<App />)
   |
   v
React component tree appears inside the root element
~~~

## Create the root and render the app

Import createRoot from react-dom/client. Find the container element, create one root for it, and render your root component:

~~~jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const container = document.getElementById('root');

if (!container) {
  throw new Error('The root container was not found in index.html.');
}

const root = createRoot(container);
root.render(<App />);
~~~

The steps have distinct jobs:

1. document.getElementById finds the browser DOM element that matches the HTML id.
2. createRoot tells React DOM to manage the content inside that element.
3. `root.render(<App />)` asks React to display the App component tree there.

Pass a JSX element such as `<App />` to render. Do not call the component function yourself as `App()`; React needs to manage the component as part of the tree.

The check for a missing container turns a blank screen or a confusing null-container error into a clear startup message. Make sure the id used in JavaScript matches the id in the HTML.

## Use one root for a full React application

A browser app built fully with React normally has one root for the whole app. Create it once in the entry module, then compose the rest of the interface under App:

~~~jsx
function App() {
  return (
    <main>
      <SiteHeader />
      <Dashboard />
      <SiteFooter />
    </main>
  );
}
~~~

Components are added by returning them from other components. Do not call createRoot inside every component, on every click, or each time a component renders. A root is the boundary where React takes ownership of a DOM container; it is not a replacement for component composition.

A page that gradually adopts React may have several independent roots for separate regions. Each root must be created for a different DOM element. This is an integration pattern; a normal single-page app usually needs only one.

## What root.render does

Call root.render to display a React node inside the container owned by that root. The first render of a client root clears existing content inside its container before rendering the React tree. Keep content that should remain outside React outside the root element.

For example, if the HTML is:

~~~html
<header>Static site header</header>
<div id="root"><p>This placeholder will be replaced.</p></div>
<footer>Static site footer</footer>
~~~

and JavaScript renders into the element with id root, the React app manages the div's contents. The static header and footer are outside the root and remain outside React's managed region.

You can call root.render again on the same root to provide a new React tree, but a regular app does not need to call it for every UI change. Components use props and state to describe updates; the entry module normally sets up the root and renders the app once.

## Choose createRoot or hydrateRoot

Use createRoot for a client-rendered app whose root container is empty or whose existing contents are intended to be replaced by the first client render.

If a server has already generated the initial HTML for the React app, use hydrateRoot from react-dom/client instead. Hydration attaches React to that existing React-generated HTML. It is not a substitute for createRoot on a normal empty client-rendered page, and the initial client output should match the server HTML.

Frameworks often set up hydration or client rendering internally. Follow the framework's entry conventions instead of adding a second root yourself.

## Unmount only when another system removes the container

A full React app usually stays mounted for the lifetime of the page. You rarely need to call root.unmount in its entry module.

Unmounting is useful when an external system may remove a DOM container that React was managing. Call root.unmount before handing that container back or removing it, so React can clean up the component tree. After unmounting, do not call render on that same root again; create a new root if you later need to reuse the container.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| The HTML id and JavaScript lookup do not match | The entry module cannot find its container. | Use the same id in both files and check that the container exists. |
| Calling createRoot with null | There is no DOM element for React to manage. | Check the lookup result before creating the root. |
| Calling `App()` directly | React does not receive the component as a managed element in its tree. | Render `<App />` through `root.render`. |
| Creating a root inside a component | Each render could create a new independent root and break normal composition. | Create the app root once in the entry module. |
| Calling createRoot more than once for the same container | The container should have one root owner at a time. | Reuse the root, or unmount it before creating another root for that DOM element. |
| Expecting existing container HTML to remain after root.render | The first client render clears the container's existing contents. | Put persistent static page content outside the React root. |
| Using createRoot for server-rendered React HTML | It replaces the server HTML instead of attaching to it. | Use hydrateRoot when React already rendered that HTML on the server. |
| Copying old ReactDOM.render setup into a current React app | Modern client entry points use the React DOM client root API. | Use createRoot from react-dom/client. |

## Key takeaways

- The HTML document supplies a real DOM element as the React root container.
- `createRoot` from `react-dom/client` creates the root; `root.render(<App />)` displays the app there.
- A fully React-built browser app usually creates one root once in its entry module.
- The first client render replaces existing content inside the root container.
- Use hydrateRoot for HTML already rendered by React on the server.
- Frameworks may create the root for you, so follow their setup.
- Unmount a root when an external system removes its container; it is rarely needed in a standalone app.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="010-fragments-and-multiple-elements.md">010 - Fragments and returning multiple elements</a></td><td><a href="../qnas/011-create-a-root-and-render-an-application.md">Q&amp;A</a></td><td><a href="../examples/011-create-a-root-and-render-an-application.md">Example</a></td><td><a href="012-strict-mode-and-development-checks.md">012 - Strict Mode and development checks</a></td></tr>
  </tbody>
</table>
