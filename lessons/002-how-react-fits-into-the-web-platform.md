# 002. How React fits into the web platform

## Learning goals

By the end of this lesson, you should be able to:

- Describe the main browser, React, React DOM, build-tool, and server responsibilities.
- Trace a simplified path from source code to pixels in a browser.
- Explain how a React root connects a component tree to a DOM container.
- Trace an interaction and a server-data response through a React app.
- Distinguish client rendering, server rendering, and hydration at a high level.
- Explain why React can be added to part of an existing page.

## The web platform is a set of cooperating parts

A React page still runs on the web platform. The browser, JavaScript, HTML, CSS, network, and server each have their own jobs. React fits into that system as the UI library.

| Part | Main responsibility |
| --- | --- |
| Browser | Loads documents and assets, runs JavaScript, implements browser APIs, manages the DOM, handles user input, and displays pixels. |
| HTML | Describes document structure and semantic content. |
| CSS | Describes presentation, layout, and visual rules. |
| JavaScript | Implements application logic and can respond to events, request data, and call APIs. |
| React | Helps describe a UI as components and keep the rendered result aligned with current inputs and state. |
| React DOM | Connects React components to browser DOM nodes and provides browser rendering APIs. |
| Build and development tools | Usually transform JSX and modules, serve code during development, and produce assets for deployment. Exact responsibilities vary by setup. |
| Web server | Delivers documents and assets, and may run application code or forward requests to other services. |
| Backend/API | Applies server-side rules, handles authorized operations, and may read or write a database before returning a response. |

React does not replace the browser or make network requests happen automatically. It is one part of a larger system.

## A simplified path from source code to the screen

A client-rendered React application commonly follows these steps:

1. A developer writes JavaScript modules and JSX components.
2. The development or build tool processes the project. It may transform JSX, resolve imports, and prepare files for a browser.
3. A browser requests the HTML document and the JavaScript and CSS assets it needs.
4. The browser parses HTML into a document tree, loads CSS, and runs JavaScript.
5. The app's entry module finds an HTML element reserved for React and creates a React root there.
6. React calls components to calculate the UI description.
7. React DOM creates or updates browser DOM nodes in the root container.
8. The browser applies CSS, calculates layout, paints the result, and composites it on the screen.

This is a simplified mental model. Tooling, caching, resource loading, concurrent rendering, server rendering, and browser scheduling can change the exact order and details.

## The React root and its container

An HTML page can reserve an element for a client-rendered React app:

```html
<div id="root"></div>
```

The JavaScript entry module can create a client root and render the app into that element:

```jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const container = document.getElementById('root');

if (!container) {
  throw new Error('The React root element was not found.');
}

createRoot(container).render(<App />);
```

The container is a real browser DOM node. The component tree is React's description of the UI that it manages inside that container. In a client-only root, React controls the contents of the root; unrelated page content outside it stays outside React.

A typical application has one root for its main app. A page that is gradually adopting React may have separate roots for independent interactive regions. Avoid two roots trying to manage the same DOM region.

If a framework starts the application, it will usually create the root for you. Most components should not call `createRoot`; it belongs at the application entry point.

### A root is not a component

These terms refer to different things:

- The **container** is the DOM element in the HTML page.
- The **root** is the connection between React and that container.
- The **root component**, often called `App`, describes the top of the component tree.
- A **component** is one reusable part of that tree.

For example, `createRoot(container)` connects React to a DOM element, while `root.render(<App />)` tells that root which UI to display.

## What the build tool does

Browsers execute JavaScript, but a React project commonly uses source syntax and module conventions that need processing for the target browser and deployment environment.

A project's build tooling may:

- Transform JSX into JavaScript calls.
- Resolve and package imports from multiple files.
- Process CSS, images, fonts, and other assets.
- Provide a local development server.
- Optimize and emit production assets.
- Support source maps or fast refresh during development.

These jobs belong to the project's tooling, not to the React component model itself. A framework may combine many of these jobs behind one interface. The exact configuration is project-specific and is covered in the setup chapters.

A browser can run JavaScript modules directly, but it does not parse JSX syntax as written. That is why a normal JSX project uses a tool or environment that understands JSX.

## What happens when a user interacts?

Consider a user clicking a button that changes a selected tab:

1. The browser detects the click and dispatches an event.
2. React's event handling calls the component's event handler.
3. The handler requests a state update.
4. React renders the affected component tree from the new state.
5. React DOM applies the necessary DOM updates.
6. The browser recalculates and displays the result.

The important shift is that application code changes the data that describes the UI. It does not need to manually find every text node, hide every old panel, and show every new panel.

State, event handlers, and update scheduling receive their own lessons later. For now, remember this flow:

```text
user action
    -> event handler
    -> state or input changes
    -> React render
    -> React DOM update
    -> browser displays the result
```

## Where server data fits

React components can display data, but React core does not define the application's backend or automatically retrieve data. The surrounding app architecture decides how to load and update data.

A simplified API request can work like this:

```text
React UI
   | user action or route load
   v
browser sends an HTTP request
   |
   v
server checks input and permissions
   |
   v
server may read or update a database
   |
   v
server returns a response, often JSON
   |
   v
application updates its data model
   |
   v
React renders the current UI
```

The browser and the server have different responsibilities:

- Browser code is visible to the person using the app. Do not put private credentials or trusted authorization decisions only in client code.
- Server code can protect secrets and perform privileged operations, but it must validate incoming data and permissions.
- A returned response can be JSON, HTML, a file, or another format; the app decides how to use it.
- A database is normally accessed by server-side code, not directly by a browser React component.

There are multiple ways to load data. A framework can provide route loaders or server rendering. A client-side library can manage caching and synchronization. A focused Effect can synchronize with an external system in some cases. The right choice depends on the app, and is covered later.

## Client rendering, server rendering, and hydration

The browser can receive an almost-empty HTML container and render the React UI after JavaScript loads. This is **client rendering**.

A server or build process can also produce HTML from React before the browser runs the client app. This is **server rendering** or **static rendering**, depending on when it is produced.

When React attaches client behavior to existing React-generated HTML, it is called **hydration**. For a server-rendered root, React DOM uses `hydrateRoot`, not `createRoot`. The first client output should match the server HTML so React can reuse the existing structure.

| Approach | Initial HTML | What client React does |
| --- | --- | --- |
| Client rendering | Often contains an empty app container. | Creates the UI in the browser after JavaScript loads. |
| Server or static rendering | Contains HTML generated before the app's client code runs. | Hydrates the matching HTML to attach React behavior and manage later updates. |

A framework may handle these rendering APIs and data flow for you. This lesson introduces the distinction; later chapters cover rendering strategies and their tradeoffs.

## React can be adopted gradually

React does not require a whole site to be rewritten. A page can keep its existing HTML, CSS, and server technology while rendering a React component into a dedicated element.

For example, a page could keep a server-rendered header and footer, while React manages an interactive search area inside one container. The surrounding page and the React-managed region must have clear ownership so that two systems do not try to modify the same nodes.

This incremental approach is useful when:

- An existing application needs one interactive feature.
- A team wants to migrate one region at a time.
- Some pages are static while other areas need client-side interaction.
- A React app is embedded into a larger server-rendered site.

## React library, framework, and renderer

These terms are related but not interchangeable:

- **React** defines components and UI APIs.
- **React DOM** renders those components into browser DOM.
- A **framework** may combine React with routing, server rendering, data loading, build configuration, and deployment conventions.
- A **renderer** connects React's component model to a target environment. React DOM targets the web; React Native targets native mobile UI.

Choose an architecture that fits the product. A small interactive island may not need a full application framework. A large application may benefit from a framework that coordinates routing, data, server rendering, and deployment.

## Common misconceptions

- **"React is the browser."** React runs as JavaScript in an environment such as a browser; the browser provides the DOM and rendering engine.
- **"React is a backend."** React is a UI library. Server APIs and databases have separate responsibilities.
- **"React compiles JSX by itself."** Project tooling or a compatible runtime processes JSX; browsers do not parse it directly.
- **"React replaces all existing HTML."** React can manage one dedicated page region.
- **"Every component creates a new root."** Roots are created at application entry points; components are nested inside a root.
- **"React always uses client-only rendering."** React supports client rendering and APIs used for server rendering and hydration; a framework often coordinates these modes.
- **"The browser knows my application state."** State belongs to application code or a chosen data system. The browser displays the result it is given.
- **"I can freely change DOM nodes that React owns."** Direct mutations can conflict with React's next render. Use React to describe the managed region and use refs only for bounded imperative integrations.

## Key takeaways

- A React web app runs within the browser platform; HTML, CSS, JavaScript, network, and server still matter.
- Build tools prepare JSX and modules for the browser.
- React DOM connects a component tree to a real DOM container through a root.
- A typical interaction travels from a browser event to application state, a React render, a DOM update, and a browser paint.
- React can render into one part of an existing page.
- React core does not prescribe a backend, router, or single data-fetching strategy.
- Client rendering, server rendering, and hydration are related but distinct.
- A framework may coordinate these pieces, while React remains the UI foundation.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="001-introduction-to-react-and-declarative-ui.md">001 - Introduction to React and declarative UI</a></td><td><a href="../qnas/002-how-react-fits-into-the-web-platform.md">Q&amp;A</a></td><td><a href="../examples/002-how-react-fits-into-the-web-platform.md">Example</a></td><td><a href="003-react-react-dom-and-frameworks.md">003 - React, React DOM, and the framework ecosystem</a></td></tr>
  </tbody>
</table>
