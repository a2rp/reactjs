# 017. Composition, containment, and reusable layouts - Example

## Goal

Build a course page by composing small components. The example demonstrates:

- A reusable `PageLayout` that owns the page regions.
- Named JSX props for the header, sidebar, and footer.
- `children` for the page's main content.
- A warning component specialized from a general notice component.
- Passing course data directly to the catalog that uses it.

## Files

In an existing Vite React project, create or update these files:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required. The browser's default styles are enough to inspect the rendered structure.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
const courses = [
  {
    id: 'components',
    title: 'Components and JSX',
    description: 'Build interfaces from clear, reusable pieces.',
  },
  {
    id: 'composition',
    title: 'Composition and layouts',
    description: 'Arrange components and provide content through props.',
  },
  {
    id: 'state',
    title: 'State and interaction',
    description: 'Make an interface respond to user input.',
  },
];

function PageLayout({ header, sidebar, children, footer }) {
  return (
    <div className="page-layout">
      <header className="page-layout__header">{header}</header>

      <div className="page-layout__body">
        <aside className="page-layout__sidebar" aria-label="Course navigation">
          {sidebar}
        </aside>

        <main className="page-layout__main" id="main-content">
          {children}
        </main>
      </div>

      <footer className="page-layout__footer">{footer}</footer>
    </div>
  );
}

function CourseHeading() {
  return (
    <div>
      <p>Learning notes</p>
      <h1>React course</h1>
    </div>
  );
}

function CourseNavigation({ courses }) {
  return (
    <nav aria-label="Lessons">
      <h2>Lessons</h2>
      <ul>
        {courses.map((course) => (
          <li key={course.id}>
            <a href={`#${course.id}`}>{course.title}</a>
          </li>
        ))}
      </ul>
    </nav>
  );
}

function Notice({ title, tone = 'info', children }) {
  return (
    <aside className={`notice notice--${tone}`}>
      <h2>{title}</h2>
      {children}
    </aside>
  );
}

function WarningNotice({ children }) {
  return (
    <Notice title="Before you continue" tone="warning">
      {children}
    </Notice>
  );
}

function CourseCatalog({ courses }) {
  return (
    <section aria-labelledby="course-catalog-title">
      <h2 id="course-catalog-title">Course lessons</h2>
      <ul>
        {courses.map((course) => (
          <li id={course.id} key={course.id}>
            <article>
              <h3>{course.title}</h3>
              <p>{course.description}</p>
            </article>
          </li>
        ))}
      </ul>
    </section>
  );
}

function App() {
  return (
    <PageLayout
      header={<CourseHeading />}
      sidebar={<CourseNavigation courses={courses} />}
      footer={<p>Personal React learning notes</p>}
    >
      <WarningNotice>
        <p>Read each lesson, then try its example in your own project.</p>
      </WarningNotice>

      <CourseCatalog courses={courses} />
    </PageLayout>
  );
}

export default App;
~~~

## What the example shows

`PageLayout` owns the shared page structure. It places the JSX passed through `header`, `sidebar`, and `footer` in named regions, then places `children` in the main region. It does not need to know how the course heading, navigation, notice, or catalog work internally.

`App` supplies the page-specific pieces. `CourseHeading`, `CourseNavigation`, and the footer are passed as JSX props. The warning and catalog are nested as children. Because `CourseCatalog` is the component that reads the `courses` data, `App` passes the array directly to it; the layout does not receive or forward that data.

`WarningNotice` specializes the general `Notice` by fixing its title and tone. Its caller only supplies the message content.

## Expected result

The page shows a course heading, lesson navigation, a warning message, a course lesson list, and a footer. Selecting a lesson in the navigation moves to the matching lesson item.

## Practice

1. Add a `CourseSearch` component and place it above `CourseCatalog` inside the layout's children.
2. Add a `header` prop to `WarningNotice` only if callers should be able to customize its title; otherwise keep its fixed title.
3. Create a second page that reuses `PageLayout` but supplies a different heading and sidebar.
4. Remove one of the named layout regions and decide whether it belongs as a named prop or as part of `children`.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [016 - The children prop and nested content](../lessons/016-children-prop-and-nested-content.md) | [017 - Composition, containment, and reusable layouts](../lessons/017-composition-containment-and-layouts.md) | [Open Q&A](../qnas/017-composition-containment-and-layouts.md) | [018 - Conditional rendering patterns](../lessons/018-conditional-rendering-patterns.md) |