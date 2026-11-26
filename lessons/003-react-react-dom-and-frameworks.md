# 003. React, React DOM, and the framework ecosystem

## Learning goals

By the end of this lesson, you should be able to:

- Distinguish the React library from React DOM and a React framework.
- Recognize where common React packages are imported from.
- Explain what a renderer does and why browser and native UI use different renderers.
- Describe what frameworks and ecosystem libraries can add around React.
- Choose a sensible starting architecture for a small feature, a learning project, or a larger application.

## React is the UI foundation

React provides a component model and APIs for describing user interfaces. Components receive inputs and return React nodes. React coordinates rendering work so the displayed interface can match those inputs.

React does not require one particular way to organize all application code. It does not include one universal router, backend, database client, styling system, project generator, or deployment service. Those pieces come from browser APIs, separate packages, build tools, or frameworks.

The word **React** can refer to the library and also to the wider ecosystem built around it. Keep the actual package responsibilities clear when reading code or choosing tools.

## What the main packages do

### The `react` package

The `react` package contains APIs used by components and application logic, including built-in Hooks and component utilities.

```jsx
import { useState } from 'react';

function Greeting() {
  const [name, setName] = useState('Priya');

  return <h1>Hello, {name}</h1>;
}
```

Most component files import React APIs they use from `react`. JSX syntax is transformed by the project tooling; modern JSX projects usually do not need to import a `React` variable just to write JSX.

### React DOM

The `react-dom` package connects React to the web platform. Its entry points separate browser client rendering from server rendering:

- `react-dom/client` contains browser APIs such as `createRoot` and `hydrateRoot`.
- `react-dom/server` contains APIs used to produce HTML on a server or during a build.
- The main `react-dom` entry contains other web-specific APIs, such as portals and resource-related APIs.

A browser entry module often looks like this:

```jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const container = document.getElementById('root');

if (!container) {
  throw new Error('The app container is missing.');
}

createRoot(container).render(<App />);
```

This is application bootstrap code. It usually runs once, outside the ordinary component tree. A framework may perform this setup itself.

### A renderer

A renderer connects React's component model to a host environment. React DOM renders into browser DOM. React Native uses React concepts to render native platform UI. They share some ideas, but their elements, styling, accessibility and platform APIs are not interchangeable.

When writing a normal web application, use React DOM or a framework that uses it. You generally do not create a renderer yourself.

## What a framework adds

A framework combines React with conventions and tools for building a complete application. Depending on the framework and project configuration, it may provide:

- URL routing and navigation.
- Route-level data loading and mutations.
- Client rendering, static generation, and server rendering.
- Server and client component boundaries.
- Caching and revalidation.
- Error, loading, and not-found UI conventions.
- Build tooling, code splitting, asset handling, and development workflows.
- Deployment adapters or integration with a hosting platform.

Frameworks differ in what they provide and how they work. The term does not mean that every feature is mandatory, that every page runs on a server, or that the project cannot be deployed as static files.

A framework may call React DOM APIs for you. Application components usually work with components, props, state, and framework-specific APIs rather than manually creating roots or routing every request themselves.

## Libraries around React

A library usually solves a focused problem and can be combined with other tools. Common categories include:

| Category | Typical responsibility | Questions to ask |
| --- | --- | --- |
| Router | Maps URLs and navigation to UI. | Does it support nested layouts, route errors, and the rendering mode the app needs? |
| Data and query library | Fetches, caches, and synchronizes remote data. | How does it handle loading, errors, stale data, mutations, and cancellation? |
| State store | Shares client state across distant components. | Is local state, props, or context insufficient for the actual use case? |
| Styling system | Organizes CSS and reusable design decisions. | Does it fit the existing CSS, accessibility needs, and team workflow? |
| Component library | Provides reusable controls and patterns. | Are its components semantic, keyboard-accessible, customizable, and maintained? |
| Testing tools | Checks logic, UI behavior, or browser workflows. | Does the test observe user-facing behavior at the right level? |

These are ecosystem choices, not requirements of React itself. Add a package when it solves a real need; avoid collecting overlapping tools before understanding the problem.

## Build tools are a separate layer

A build tool and a framework can overlap, but the concepts are different:

- A **build tool** processes source files and assets, serves a local development app, and creates deployable output.
- A **framework** defines a broader application structure and may use build tools internally.
- A **library** can be added to an existing setup without defining the whole application structure.

A custom React setup can use `react`, `react-dom`, and a JSX-capable build tool. This can be appropriate for learning, embedding React in an existing project, or a constrained application. It also means the project team must choose and connect other needs such as routing, data loading, and production rendering.

For a new full application, evaluate a React framework early. It may already solve routing, rendering, data, and deployment problems that would otherwise need separate decisions. A framework is not automatically the right answer for every embedded feature, learning exercise, or static site.

This course uses a build-tool workflow to teach the React foundations and later explains when a full-stack framework may be a better fit. The project setup itself is covered in a dedicated lesson.

## Three common ways to use React

### 1. Add a React island to an existing page

Keep the existing site and mount a React root into a dedicated DOM container. This is useful for a focused interactive feature or incremental migration.

The host site still owns the rest of the page. Keep DOM ownership boundaries clear and load React's client entry through the existing project's configured tooling.

### 2. Build a client-rendered React application

Use one client root for the app, a build tool to process and serve the code, and separate choices for routing, data, and other needs. This is useful for learning the fundamentals, for client-focused experiences, or when the project has a clear reason to assemble its own stack.

The project team must understand how its pieces fit together. A build tool by itself does not automatically provide a router, backend, or complete production architecture.

### 3. Build an application with a React framework

Start with a framework that integrates React with features such as routing, server rendering, data loading, or server-side code. This can reduce the number of independent systems the team must connect for a full application.

Understand the framework's conventions, deployment model, and server/client boundaries. React knowledge still matters, but some startup and rendering details are handled by the framework.

## Client rendering, server APIs, and Server Components

React DOM has client APIs for rendering in a browser and server APIs for producing HTML. A framework may coordinate them.

- **Client rendering** starts with browser JavaScript and creates the UI in a client root.
- **Server rendering** produces HTML before or while sending a response.
- **Hydration** attaches React behavior to existing React-generated HTML; the browser entry uses `hydrateRoot`.
- **React Server Components (RSC)** describe where components execute in a server/client architecture. They are not the same as ordinary server rendering and require compatible framework or bundler integration.

Do not assume that using a framework means the whole app runs on the server. Many frameworks support client-rendered pages and static output as well. Check which code runs in each environment before using browser-only APIs or server-only secrets.

## How to choose without overengineering

Start from the shape of the problem:

| Situation | A reasonable starting point |
| --- | --- |
| One interactive control on an existing site | Add React to a dedicated region using the existing build integration. |
| Learning components and React fundamentals | Use the course's small build-tool setup and focus on the React model. |
| A new multi-page production application | Compare full-stack React frameworks and their routing, data, rendering, and deployment support. |
| A mostly static document with little interaction | HTML and CSS may be simpler than adding React. |
| A native mobile application | Explore React Native and its native-specific APIs rather than using React DOM. |

Before adding a library, framework, or separate service, state which problem it solves. Prefer the smallest architecture that meets current requirements and can evolve with the product.

## Common misconceptions

- **React and React DOM are the same package.** React provides UI APIs; React DOM renders to the web platform.
- **Every React app needs a framework.** A project can use React with a build tool or incrementally inside an existing page. A framework can be useful for a full application.
- **A framework is just a bundler.** A framework may include a build system, but it also provides broader app conventions and capabilities.
- **A framework always means server rendering.** Frameworks can support client rendering and static output too; the project chooses or configures its behavior.
- **React includes routing and data fetching.** These are usually provided by a framework or separate library.
- **The same UI code works identically in React DOM and React Native.** The component model is shared, but platform elements and APIs differ.
- **More dependencies make an application more complete.** Every tool adds concepts, configuration, and maintenance. Add only what the project needs.
- **If a framework creates the root, React is no longer involved.** The framework still uses React; it is handling application setup and integration.

## Key takeaways

- `react` supplies component APIs; `react-dom` renders React into the web platform.
- `react-dom/client` is for browser roots; `react-dom/server` is for server HTML rendering.
- A renderer connects React to a target platform.
- A framework combines React with broader application systems such as routing, data, rendering, and deployment.
- Focused libraries can solve specific needs without controlling the whole app.
- A build tool processes source and assets; it is not automatically a full application framework.
- Choose architecture according to the application, not by adding every available tool.
- The course teaches the foundations with a build-tool workflow, then treats frameworks as an informed application choice.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="002-how-react-fits-into-the-web-platform.md">002 - How React fits into the web platform</a></td><td><a href="../qnas/003-react-react-dom-and-frameworks.md">Q&amp;A</a></td><td><a href="../examples/003-react-react-dom-and-frameworks.md">Example</a></td><td><a href="004-javascript-and-browser-prerequisites.md">004 - JavaScript and browser prerequisites</a></td></tr>
  </tbody>
</table>
