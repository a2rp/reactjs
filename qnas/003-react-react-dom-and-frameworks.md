# 003. Q&A: React, React DOM, and the framework ecosystem

Use these questions to check that the package and architecture boundaries are clear. The goal is not to memorize a specific tool list; it is to understand what each layer is responsible for.

## React packages and renderers

### 1. What does the `react` package provide?

It provides React APIs used to define and compose UI, including Hooks and component utilities. It is the foundation for React renderers.

### 2. What does React DOM provide?

React DOM connects React to the browser web platform. It provides client APIs for browser roots, server APIs for generating HTML, and other web-specific integration APIs.

### 3. Where does `useState` come from?

It is imported from `react`:

```jsx
import { useState } from 'react';
```

### 4. Where does `createRoot` come from?

It is imported from the client entry of React DOM:

```jsx
import { createRoot } from 'react-dom/client';
```

### 5. Where do browser and server rendering APIs live?

Browser root APIs such as `createRoot` and `hydrateRoot` are in `react-dom/client`. Server HTML-rendering APIs are in `react-dom/server`. The app or framework chooses when to use them.

### 6. What is a renderer?

A renderer connects React's component model to a target platform. React DOM targets browser DOM. React Native targets native platform UI. Other renderers may target other environments.

### 7. Can React DOM render native Android views?

No. React DOM targets the web. React Native provides a separate renderer and native components for mobile platforms. Some React concepts carry over, but DOM elements and browser APIs do not become native views.

### 8. Should components create roots?

Usually not. An application entry point or framework creates the root. Components are nested inside it and should describe part of the UI.

### 9. Can a framework create the root for me?

Yes. Frameworks often handle application bootstrap and call the relevant React DOM APIs internally. Most application components do not need to know how the root is initialized.

## Build tools, libraries, and frameworks

### 10. What does a build tool do?

A build tool processes source files and assets, commonly transforms JSX, resolves imports, serves files during development, and creates deployable output. The exact features depend on the tool and configuration.

### 11. Is a build tool the same as a framework?

No. A build tool focuses on preparing and serving project files. A framework defines broader application conventions and may integrate routing, data loading, rendering, and deployment as well as build tooling.

### 12. Is React a framework?

React core is a UI library. Frameworks built around React can provide a broader application structure. In conversation, people sometimes use "React" to refer broadly to this ecosystem, so check which package or tool is actually meant.

### 13. Does every React project need a framework?

No. A small feature inside an existing page or a focused learning project can use React with a build tool. A full application may benefit from a framework that coordinates routing, rendering, data, and deployment.

### 14. Does every framework render on the server?

No. Frameworks can support client rendering, static output, and server rendering in different configurations. The presence of a framework does not tell you exactly where every component executes.

### 15. What is a library in the React ecosystem?

A library typically solves a focused problem, such as routing, remote-data caching, shared client state, styling, testing, or reusable controls. It can be used with React without defining the entire application structure.

### 16. Should I install a state library, router, and query library at the start?

Not automatically. Begin with the app's actual requirements. Local state, props, and context may be enough for some needs. Add specialized tools when they solve a problem that the current approach cannot handle clearly.

### 17. What does the framework provide that React core does not?

Depending on the framework, it may provide route definitions, navigation, server and client data loading, server rendering, static generation, caching, error conventions, asset handling, and deployment integrations.

### 18. Does using a framework mean my app no longer uses React?

No. A React framework uses React as its UI foundation and may manage startup, routing, rendering, and other integration details around it.

### 19. Can I use React without a bundler?

Some modern browser environments can load JavaScript modules directly, and simple experiments can use browser tooling. Typical JSX projects need a compatible transform or runtime and package/module setup. The right setup depends on the project; this course uses a build-tool workflow for its fundamentals.

## Rendering and architecture choices

### 20. What is the difference between client rendering and server rendering?

Client rendering creates the UI in the browser after its JavaScript runs. Server rendering generates HTML before it reaches the browser. Frameworks can coordinate either approach and can use them together.

### 21. When do I use `hydrateRoot`?

Use it when the container already contains React-generated HTML that should be reused and connected to client behavior. Use `createRoot` for a client-only container that does not contain server-rendered React markup.

### 22. Are Server Components just another name for server-rendered HTML?

No. Server rendering describes how HTML is generated. React Server Components describe a server/client component execution architecture and need compatible framework or bundler support. These concepts can work together but are distinct.

### 23. What is the simplest approach for one interactive widget in an existing application?

A dedicated React root in a clearly owned DOM container may be appropriate. The rest of the page can stay with its existing system.

### 24. What might make a framework a good choice for a new multi-page application?

A framework may already coordinate routing, data loading, rendering, server/client boundaries, assets, and deployment. Evaluate its conventions and hosting model against the product's needs.

### 25. What might make a small build-tool setup appropriate?

It can be useful for learning React, embedding it into an existing project, or building a focused client-rendered app with constraints that justify choosing the surrounding pieces separately.

### 26. Does a framework automatically add a backend or database?

No. Frameworks can provide server-side execution and integrations, but a database and business-specific backend behavior still require implementation and configuration. Some apps are static and need no application server at runtime.

### 27. Which layer is responsible for CSS layout?

The browser's CSS engine computes layout and paints styled elements. React can render class names, style props, and other markup, but React does not replace CSS or the browser's rendering engine.

### 28. What is the main rule for choosing ecosystem tools?

Start with the requirements and identify which layer needs a capability. Choose the smallest set of tools that meets those requirements and that the team can maintain.

## Quick scenarios

### Scenario A: A component imports `useState`

Which package should that import come from?

**Answer:** `react`.

### Scenario B: A browser entry module needs to mount `<App />`

Which package entry point provides `createRoot`?

**Answer:** `react-dom/client`.

### Scenario C: A project needs routing and route-level data loading

Does React core supply both of those as built-in universal systems?

**Answer:** No. A framework or separate ecosystem libraries provide these capabilities.

### Scenario D: An existing page has a React-generated HTML shell

Should the page use a client-only `createRoot` and replace it?

**Answer:** No. It should hydrate matching server-generated React HTML with `hydrateRoot`.

### Scenario E: A marketing page has no interactions and mostly static content

Must it use React?

**Answer:** No. Plain HTML and CSS may be the simpler choice.

## Quick self-check

1. Which package contains component Hooks?
2. Which package entry point contains browser root APIs?
3. Does React core define one required router?
4. Is a build tool automatically a full-stack framework?
5. What renderer targets browser DOM?
6. Why should a new application compare frameworks before assembling many independent tools?

### Suggested answers

1. `react`.
2. `react-dom/client`.
3. No.
4. No. Their responsibilities can overlap, but a framework handles broader application structure.
5. React DOM.
6. A framework may already coordinate routing, data, rendering, and deployment, reducing separate integration work.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="002-how-react-fits-into-the-web-platform.md">002 - Q&amp;A: How React fits into the web platform</a></td><td><a href="../lessons/003-react-react-dom-and-frameworks.md">Lesson</a></td><td><a href="../examples/003-react-react-dom-and-frameworks.md">Example</a></td><td><a href="004-javascript-and-browser-prerequisites.md">004 - Q&amp;A: JavaScript and browser prerequisites</a></td></tr>
  </tbody>
</table>
