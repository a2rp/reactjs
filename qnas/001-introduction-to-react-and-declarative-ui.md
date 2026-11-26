# 001. Q&A: Introduction to React and declarative UI

Use these questions to review the first lesson. Try answering each one before reading its answer.

## React and the web

### 1. What is React?

React is a JavaScript library for building user interfaces from components. You describe the UI from its inputs and state; React coordinates rendering that description into a target such as the browser DOM.

### 2. Does React replace HTML, CSS, or JavaScript?

No. React is written in JavaScript and commonly uses JSX to describe UI. Browser elements still have HTML semantics, and CSS still controls presentation. React works with these technologies rather than replacing them.

### 3. Is React a complete full-stack framework?

React's core is a UI library. It does not prescribe one router, backend, data-fetching system, or deployment platform. A project can add separate tools or use a framework that integrates React with those concerns.

### 4. Do I need React for every website?

No. A simple, mostly static page may be easier to build with HTML and CSS. React is useful when its components and update model help with the interface's complexity, reuse, or interactions.

### 5. Can React be added to only part of an existing page?

Yes. React can render an interactive region inside a larger page. An application does not have to move every page or element into React at once.

### 6. What is React DOM?

React DOM is React's browser renderer. It connects React's UI descriptions to browser DOM nodes. React itself is not the browser DOM.

## Declarative UI

### 7. What does declarative mean?

Declarative UI describes what should appear for the current inputs and state. React figures out the DOM changes needed to make the visible UI match that description.

### 8. How is declarative UI different from imperative DOM code?

Imperative code tells the browser to perform individual steps, such as finding a heading and changing its text. Declarative React code describes the heading from current data, then React keeps the rendered result in sync as that data changes.

### 9. Does React always update the entire page when something changes?

No. A component may render again, but React calculates the next UI and applies the necessary changes to the rendering target. A render does not mean that the entire DOM is replaced.

### 10. Does React decide what my application should display?

No. Your application logic supplies the data and state. React renders the UI described by those inputs; it does not decide the product rules or automatically fetch application data.

### 11. Does declarative mean that I never use browser APIs?

No. Browser APIs are still useful. The main rule is to avoid manually changing DOM that React owns. Use a bounded imperative integration when needed, such as focusing an input with a ref, and keep it coordinated with React.

## Components and JSX

### 12. What is a React component?

A component is a reusable unit of UI, usually a JavaScript function that returns React nodes. Components can be nested and composed into larger screens.

### 13. Why do component names begin with a capital letter?

JSX uses capitalization to distinguish user-defined components from built-in HTML elements. For example, `<CourseCard />` refers to a component, while `<article>` refers to an HTML element.

### 14. What is JSX?

JSX is a JavaScript syntax extension that lets you write markup-like UI descriptions in JavaScript. A project build tool transforms JSX; browsers do not run JSX syntax directly.

### 15. Is JSX the same thing as HTML?

No. It resembles HTML, but it is JavaScript syntax with its own rules and conventions. For example, React DOM uses `className` for a CSS class. The roadmap covers these details in the JSX chapters.

### 16. What does it mean to compose components?

Composition means putting components together to create a larger interface. In the lesson's example, `App` uses `CourseCard` more than once and gives each card different text.

### 17. What are props?

Props are inputs a parent provides to a child component. In `<CourseCard title="React" />`, the title is a prop. A child reads props; it should not mutate them. Later lessons cover props in detail.

### 18. What is a component tree?

A component tree is the nested parent-child structure of an interface. It helps explain how the screen is composed and where data or reusable UI pieces fit.

### 19. Can a component be reused with different content?

Yes. The parent can render the same component in multiple places and supply different props. The component keeps a consistent structure while the inputs determine its content.

### 20. Should every visual detail become its own component?

No. Create a component when it gives the UI a useful boundary, reusable behavior, or clearer responsibility. Too many tiny components can make a simple interface harder to follow.

## Inputs, rendering, and practical decisions

### 21. What is the basic relationship between component inputs and UI?

A component renders UI from its props and state. When relevant inputs change, React can render the component again and update what appears.

### 22. What is state, in simple terms?

State is information a component remembers that can change over time and affect the UI. This lesson only introduces the concept; a later module explains state and its update rules.

### 23. What does it mean for rendering to be pure?

Rendering should calculate UI from current inputs without changing external systems or mutating existing data. This makes rendering predictable and lets React manage update work.

### 24. Does React automatically fetch data from a server?

No. React renders UI from data supplied to it. Your application or framework needs to obtain data and decide how loading, success, and error states should work.

### 25. Does React provide routing?

React core does not define a universal router. A project can choose a router or use one supplied by its framework.

### 26. Can React build mobile applications too?

React skills can be used with other renderers and tools, including React Native, which renders native platform UI. React DOM is specifically the browser renderer. The web and native environments have different elements and APIs.

### 27. What should I choose for a page that only displays static text?

Use the simplest approach that meets the need. Plain HTML and CSS may be enough. React is an option when component reuse or future interaction makes it useful.

### 28. What is the main lesson to remember?

Describe UI as a result of current inputs, organize that UI into components, and let React apply updates. Keep the distinction between React's UI role and the other parts of a web application clear.

## Quick self-check

1. A heading's text changes when its input value changes. Should code manually find the heading and rewrite it, or should the component describe the heading from that value?
2. Does using React mean the page no longer uses HTML semantics or CSS?
3. Which is the component in `<CourseCard title="HTML" />`: `CourseCard` or `title`?
4. Does React core decide which URL routes to which page?
5. Why can the same component render two different course cards?

### Suggested answers

1. Describe it from the value and let React update the rendered result.
2. No. React works with browser HTML elements and CSS.
3. `CourseCard` is the component; `title` is one of its props.
4. No. Routing is supplied by a separate library or framework.
5. Each use can receive different props while sharing the same component structure.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td>Start of course</td><td><a href="../lessons/001-introduction-to-react-and-declarative-ui.md">Lesson</a></td><td><a href="../examples/001-introduction-to-react-and-declarative-ui.md">Example</a></td><td><a href="002-how-react-fits-into-the-web-platform.md">002 - Q&amp;A: How React fits into the web platform</a></td></tr>
  </tbody>
</table>
