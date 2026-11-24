# 002. Example: Mount a React feature inside an existing page

## Goal

Add a small React-powered help panel to a page that already has ordinary HTML content. The host page keeps its header, article, and footer. React manages only the dedicated panel container.

This example shows how React fits into the browser and web application stack. It does not build a backend, fetch data, or add interactions.

## What you will practice

- Identifying which parts belong to the host page and which part React manages.
- Connecting a React root to a browser DOM container.
- Rendering a component through React DOM.
- Keeping the component tree's DOM ownership clear.

## Prerequisite

Use a React project whose build tool is configured to process JSX and JavaScript modules. The exact script tag or asset helper used to load the client entry depends on the project's build tool or host framework. Do not serve raw JSX directly to the browser as if it were plain JavaScript.

## Step 1: Add a dedicated container to the host page

The host page can be ordinary HTML or a server-generated template. It reserves a specific element for React:

```html
<header>
  <h1>Northstar Learning</h1>
  <nav aria-label="Main navigation">
    <a href="/">Home</a>
    <a href="/courses">Courses</a>
  </nav>
</header>

<main>
  <article>
    <h2>Welcome to the learning portal</h2>
    <p>This article is provided by the existing host page.</p>
  </article>

  <section aria-labelledby="support-title">
    <h2 id="support-title">Support</h2>
    <div id="react-help-panel"></div>
  </section>
</main>

<footer>
  <p>Northstar Learning</p>
</footer>
```

The element with `id="react-help-panel"` is the mount container. The header, article, and footer are outside it, so React will not take them over.

## Step 2: Create the React component

Create `src/HelpPanel.jsx`:

```jsx
export default function HelpPanel() {
  return (
    <section aria-labelledby="help-heading">
      <h3 id="help-heading">Need a hand?</h3>
      <p>Our learning team can help you choose a course.</p>
      <a href="/contact">Contact the learning team</a>
    </section>
  );
}
```

This component describes the panel's content. It returns semantic HTML elements, and it can be changed without rewriting the host page's header or footer.

## Step 3: Connect React to the container

Create or update the project's client entry module, for example `src/main.jsx`:

```jsx
import { createRoot } from 'react-dom/client';
import HelpPanel from './HelpPanel.jsx';

const container = document.getElementById('react-help-panel');

if (!container) {
  throw new Error('The React help panel container was not found.');
}

const root = createRoot(container);
root.render(<HelpPanel />);
```

The configured build tool or framework must load this entry module in the browser. The exact way to include its generated asset depends on the project's setup.

The steps here are:

1. `document.getElementById` finds the existing host-page element.
2. `createRoot` connects React DOM to that DOM container.
3. `root.render` asks React to display the component there.
4. React manages the container's child UI; the rest of the page stays with the host.

## Expected result

The page continues to show the original header, navigation, article, and footer. Inside the Support section, React displays:

- A heading: **Need a hand?**
- A sentence about choosing a course.
- A link to the host site's contact page.

The support panel is React-rendered, but the surrounding page is not. Styling depends on the CSS already loaded by the host page and React project.

## Follow the browser-to-React flow

```text
host page HTML
   |
   +-- header, article, footer remain host-page content
   |
   +-- #react-help-panel is the reserved DOM container
              |
              v
       client entry module
              |
              v
       createRoot(container)
              |
              v
       root.render(<HelpPanel />)
              |
              v
       React DOM updates the container
              |
              v
       browser applies CSS and displays the page
```

The browser parses the host page and creates its DOM. The client entry runs, finds the container, and starts React in that region. React describes the panel, React DOM updates its DOM nodes, and the browser displays the combined page.

## Who owns which part?

| Page region or task | Owner in this example |
| --- | --- |
| Header, navigation, article, footer | Existing host page or its server template |
| Help panel's child UI | React, through the root mounted in `#react-help-panel` |
| CSS and layout | Browser CSS engine using stylesheets supplied by the project |
| JSX transformation and module loading | The configured build tool or framework |
| Contact route | The host site's routing/server architecture |

Keeping ownership clear prevents the host page and React from trying to rewrite the same DOM nodes.

## Important root behavior

For a client-only root, React manages the contents of its container. The first render clears existing child content within that container before placing the React UI. In this example the container is intentionally empty, so there is nothing to lose.

If a server or build process pre-rendered React HTML inside the container, do not mount it with `createRoot`. Use `hydrateRoot` for React-generated HTML so React can attach to the existing markup. Hydration is introduced later in the course.

Do not create a separate React root for every child component. Create roots only at integration boundaries. Components inside a root are composed with JSX.

## If the whole page becomes React

If the application later moves the header, navigation, article, and footer into React, the page could have one root for the whole app:

```jsx
function App() {
  return (
    <>
      <SiteHeader />
      <main>
        <WelcomeArticle />
        <HelpPanel />
      </main>
      <SiteFooter />
    </>
  );
}
```

Then the HTML page can provide one app container, and the `App` component describes all of its content. A real application may use an established React framework to coordinate routing, data loading, server rendering, and deployment.

## Try it yourself

1. Change the panel heading and paragraph.
2. Add a second link inside `HelpPanel`.
3. Move the panel to a different empty container in the host page and update the entry module's ID.
4. Add a CSS class to the panel and create a stylesheet rule in the project.
5. Identify which code would need to change if the contact link's route changes.

### Suggested answers

1. Edit the JSX returned by `HelpPanel`.
2. Add another semantic `<a>` with a meaningful destination and link text.
3. Change both the host page's container ID and the ID looked up by the entry module. The two IDs must match.
4. Add a `className` prop to the relevant JSX element and style that class in CSS.
5. Update the link destination and, if needed, the host site's route. React does not define the backend route.

## Common mistakes

- **Looking up the wrong ID:** the HTML container ID and `getElementById` value must match exactly.
- **Passing the component function to render:** use `root.render(<HelpPanel />)`, not `root.render(HelpPanel)`.
- **Calling `createRoot` on a missing element:** check that the container exists before mounting.
- **Rendering over server-generated React markup:** use hydration for existing React-generated HTML.
- **Manually changing React-owned children:** let React describe and update its root contents.
- **Assuming React transforms JSX in the browser:** make sure the configured build tool or framework processes the source.
- **Creating roots inside components:** roots belong at integration points such as application entry modules.

## Completion checklist

- [ ] The host page still displays its existing header, article, and footer.
- [ ] The React help panel appears only inside `#react-help-panel`.
- [ ] The host-page and React-managed regions have separate DOM ownership.
- [ ] The build setup loads the JSX entry module correctly.
- [ ] You can explain the roles of the browser, React, React DOM, and build tooling in this example.

## Related chapters

- [Lesson 002: How React fits into the web platform](../lessons/002-how-react-fits-into-the-web-platform.md)
- [Q&A 002: How React fits into the web platform](../qnas/002-how-react-fits-into-the-web-platform.md)
- [Next: React, React DOM, and the framework ecosystem](../lessons/003-react-react-dom-and-frameworks.md)
