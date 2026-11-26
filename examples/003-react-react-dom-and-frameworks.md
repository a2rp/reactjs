# 003. Example: Map a React app to its ecosystem layers

## Goal

Create a small course page and identify which part of the stack is responsible for each job: React components, React DOM, the browser, CSS, and project tooling.

This example uses a JSX-enabled build-tool project. The project setup determines how the browser loads `src/main.jsx`; frameworks may use a different entry convention.

## The project structure

A simple setup can have a structure like this:

```text
project/
├── index.html
└── src/
    ├── main.jsx
    ├── App.jsx
    └── styles.css
```

Each file has a distinct role:

| File | Role |
| --- | --- |
| `index.html` | Supplies the browser document and the DOM container for the React root. |
| `src/main.jsx` | Application entry point; connects React DOM to the container. |
| `src/App.jsx` | Describes the page with React components. |
| `src/styles.css` | Supplies visual rules that the browser applies. |
| Build-tool configuration | Processes JSX, modules, and CSS and determines how the entry module is served or built. |

A framework can organize these jobs differently or perform some of the entry setup automatically.

## Step 1: Reserve a root container

The HTML page contains a container:

```html
<div id="root"></div>
```

A JSX-enabled build-tool project must load the `src/main.jsx` entry according to its configuration. Do not copy a script path from another tool without checking how your project is set up.

## Step 2: Create the React page

Add `src/App.jsx`:

```jsx
function CourseCard({ title, description }) {
  return (
    <article className="course-card">
      <h2>{title}</h2>
      <p>{description}</p>
    </article>
  );
}

export default function App() {
  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">Learning plan</p>
        <h1>Choose your next topic</h1>
        <p>Three parts of the web platform to study.</p>
      </header>

      <section className="course-list" aria-label="Available topics">
        <CourseCard
          title="HTML"
          description="Describe meaningful page structure."
        />
        <CourseCard
          title="CSS"
          description="Control layout, color, and presentation."
        />
        <CourseCard
          title="React"
          description="Compose user interfaces from components."
        />
      </section>
    </main>
  );
}
```

The component tree describes one page and three cards. It is intentionally static; the example is about how React fits with the other layers, not about state or routing.

## Step 3: Connect React DOM to the page

Add `src/main.jsx`:

```jsx
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './styles.css';

const container = document.getElementById('root');

if (!container) {
  throw new Error('The root container was not found.');
}

createRoot(container).render(<App />);
```

The module imports `createRoot` from React DOM's browser entry point and imports the page component. The build tool processes the JSX and CSS import according to the project's configuration.

This entry module runs once to bootstrap the React tree. Child components do not create their own root.

## Step 4: Add browser styling

Add `src/styles.css`:

```css
:root {
  font-family: system-ui, sans-serif;
  color: #182230;
  background: #f4f7fb;
}

body {
  margin: 0;
}

.page {
  width: min(100% - 2rem, 56rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.eyebrow {
  color: #2457a6;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.course-list {
  display: grid;
  gap: 1rem;
}

.course-card {
  padding: 1.25rem;
  border: 1px solid #c9d3e0;
  border-radius: 0.75rem;
  background: white;
}

.course-card h2 {
  margin-top: 0;
}
```

The browser applies these CSS rules to the elements React DOM places in the root container. CSS layout and painting are browser jobs, not React jobs.

## Expected result

The browser displays a centered learning-plan page with:

- A small "Learning plan" label.
- A main heading and introduction.
- Three cards for HTML, CSS, and React.
- A responsive single-column layout with simple spacing and borders.

The exact build command and development URL depend on the JSX-enabled tool configured for the project.

## Trace each layer

When the page loads:

1. The browser receives and parses `index.html`, creating the document and the empty `#root` element.
2. The configured tool makes the JavaScript entry and stylesheet available to the browser.
3. The browser runs the transformed JavaScript module.
4. `main.jsx` imports `createRoot` from `react-dom/client` and `App` from the app source.
5. React evaluates `App`, `CourseCard`, and their JSX descriptions.
6. React DOM updates the DOM inside `#root`.
7. The browser applies `styles.css`, computes layout, and displays the page.

## Assign responsibility correctly

| Need | Layer used in this example |
| --- | --- |
| Describe reusable cards and page structure | React components in `App.jsx` |
| Render the component tree into browser DOM | React DOM in `main.jsx` |
| Transform JSX and process module/CSS imports | Configured build tool |
| Parse HTML, apply CSS, handle input, and display pixels | Browser |
| Map URLs to screens | Not needed for this one-page example |
| Retrieve private data or enforce authorization | Not needed here; belongs on a server when required |

The missing rows are intentional. A small static page does not need a router or backend just because it uses React.

## Decide what to add next

Imagine this page changes in one of these ways:

| New requirement | What to evaluate |
| --- | --- |
| Add a second URL and nested pages | A router or a framework that supplies routing. |
| Load course information from an API | A data-loading approach, such as a framework loader or a query/cache library. |
| Render HTML before client JavaScript runs | A framework or server-rendering setup, plus hydration. |
| Keep the page as a single static screen | The current React plus build-tool setup may be enough. |
| Add only one interactive panel to an older site | Mount a React root in a dedicated region of that page. |

Do not install each possible library in advance. Identify the requirement first, then choose a tool that covers it and fits the project.

## Try it yourself

1. Remove the CSS import and observe that the structure still comes from React while the presentation changes.
2. Change the card text in `App.jsx`. Which layer determines the new UI?
3. Remove the `#root` element from the HTML. What happens when the entry module looks for it?
4. Add a fourth static card. Does this require a router or a server?
5. Imagine the page has ten nested routes and server-loaded data. What kind of tool should you evaluate?

### Suggested answers

1. React still describes and renders the elements; the browser applies no project stylesheet from that import.
2. React renders the changed component output, React DOM updates the root's DOM, and the browser displays it.
3. The entry module throws the intentional error because no container exists.
4. No. The current one-page static UI can render another component.
5. Evaluate a React framework or a combination of routing and data tools, based on deployment and rendering needs.

## Common mistakes

- **Importing `createRoot` from the wrong package:** browser roots come from `react-dom/client`.
- **Creating a root inside `App`:** the root belongs in the application entry module.
- **Assuming JSX runs untransformed in every browser:** use a project setup that supports the JSX source.
- **Assuming React styles the page automatically:** React outputs UI; CSS and the browser determine presentation.
- **Adding routing for a one-screen example:** use a router when the product needs URL-based navigation.
- **Assuming React provides the API server:** data services and authorization are separate application responsibilities.
- **Putting secrets in browser code:** treat all delivered client code as visible to the user.

## Completion checklist

- [ ] The page has one React root container.
- [ ] `main.jsx` imports React DOM's `createRoot` and renders `<App />`.
- [ ] `App.jsx` composes reusable course-card components.
- [ ] The CSS import is processed by the configured project tooling.
- [ ] You can identify which layer handles components, DOM rendering, source processing, and styling.
- [ ] You can explain why this example does not need a router or backend.

## Related chapters

- [Lesson 003: React, React DOM, and the framework ecosystem](../lessons/003-react-react-dom-and-frameworks.md)
- [Q&A 003: React, React DOM, and the framework ecosystem](../qnas/003-react-react-dom-and-frameworks.md)
- [Next: JavaScript and browser prerequisites](../lessons/004-javascript-and-browser-prerequisites.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="002-how-react-fits-into-the-web-platform.md">002 - Example: Mount a React feature inside an existing page</a></td><td><a href="../lessons/003-react-react-dom-and-frameworks.md">Lesson</a></td><td><a href="../qnas/003-react-react-dom-and-frameworks.md">Q&amp;A</a></td><td><a href="004-javascript-and-browser-prerequisites.md">004 - Example: Use JavaScript data and functions in a React component</a></td></tr>
  </tbody>
</table>
