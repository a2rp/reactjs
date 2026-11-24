# 017. Composition, containment, and reusable layouts

## Learning goals

By the end of this lesson, you should be able to:

- Compose a page by nesting focused components.
- Use containment when a component should wrap caller-supplied content.
- Build a specific component by configuring a more general component.
- Design layouts with children and named content areas.
- Keep data props close to the components that use them.
- Choose useful component boundaries without over-splitting the UI.

## Build interfaces by composition

A React interface is a tree of components. A page component can compose smaller pieces by placing them in its JSX:

~~~jsx
function CourseHeader() {
  return <header><h1>React course</h1></header>;
}

function CourseSummary() {
  return <p>Learn to build interfaces from reusable components.</p>;
}

function CoursePage() {
  return (
    <main>
      <CourseHeader />
      <CourseSummary />
    </main>
  );
}
~~~

Each component has a clear role, and `CoursePage` assembles them into a page. The component tree describes how the interface is composed; it does not require every HTML element to have its own component.

Composition is a common way to reuse UI. Instead of copying a large block of JSX or building a class hierarchy, render a component inside another component and configure it with props and children.

## Containment: let the caller supply content

A component uses containment when it provides a wrapper or visual structure for content it does not need to know in advance. The caller supplies that content through `children`:

~~~jsx
function Notice({ title, children }) {
  return (
    <aside>
      <h2>{title}</h2>
      <div>{children}</div>
    </aside>
  );
}

function LessonPage() {
  return (
    <Notice title="Study tip">
      <p>Try the example before reading the explanation.</p>
      <a href="#practice">Go to practice</a>
    </Notice>
  );
}
~~~

`Notice` controls the wrapper and heading. `LessonPage` controls the paragraph and link inside it. The wrapper can therefore be reused with different content without knowing the specific components or markup its caller will provide.

Use `children` when there is one general content area. Use named JSX props such as `header`, `actions`, or `footer` when a component has separate content regions.

## Specialization: configure a general component

A specific component can reuse a more general component by passing it the props and content that define the specialized case:

~~~jsx
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
    <Notice title="Important" tone="warning">
      {children}
    </Notice>
  );
}
~~~

The caller can use `WarningNotice` when it needs the warning style and title. `WarningNotice` composes `Notice` rather than inheriting its implementation.

Use a specialized component when it gives a repeated UI pattern a clearer name or a fixed configuration. If it only forwards every prop without adding a useful meaning, the extra component may not help.

## Build a reusable layout with slots

A layout can arrange shared page regions and let its caller provide what belongs in them:

~~~jsx
function PageLayout({ header, sidebar, children, footer }) {
  return (
    <div className="page-layout">
      <header>{header}</header>

      <div className="page-layout__body">
        <aside>{sidebar}</aside>
        <main>{children}</main>
      </div>

      <footer>{footer}</footer>
    </div>
  );
}
~~~

The `header`, `sidebar`, and `footer` props are named JSX slots. The nested content is the main page content. The layout decides where each region appears without owning the details of the content.

A page can provide different content in those regions:

~~~jsx
function CoursePage({ courses }) {
  return (
    <PageLayout
      header={<h1>React course</h1>}
      sidebar={<CourseNavigation />}
      footer={<p>Personal learning notes</p>}
    >
      <CourseCatalog courses={courses} />
    </PageLayout>
  );
}
~~~

`CoursePage` passes `courses` directly to `CourseCatalog`, the component that uses that data. `PageLayout` receives the catalog element as its children and does not need a `courses` prop.

## Use composition to keep data near its consumer

A page can accidentally pass data to a visual wrapper even though only the nested content needs it:

~~~jsx
function App({ courses }) {
  return (
    <PageFrame courses={courses}>
      <CourseCatalog courses={courses} />
    </PageFrame>
  );
}

function PageFrame({ courses, children }) {
  return <div className="page-frame">{children}</div>;
}
~~~

`PageFrame` receives `courses` but never uses it. If intermediate components keep forwarding such values, their props become coupled to page-specific data. Composition can remove that unnecessary prop:

~~~jsx
function App({ courses }) {
  return (
    <PageLayout>
      <CourseCatalog courses={courses} />
    </PageLayout>
  );
}

function PageLayout({ children }) {
  return <main>{children}</main>;
}
~~~

The parent that knows both the layout and the course data composes the catalog element and passes it as content. The layout only places that content. This is useful when intermediary components exist mainly to position UI.

Explicit prop passing is still a good choice when an intermediate component actually needs the value or when the data path is short and clear. Composition is one design option; it does not mean every prop should be hidden in children.

## Composition and context solve different problems

Composition lets one component supply already-created JSX to another component for placement. Props pass data to components that need it. Context can make a value available to distant components across a subtree.

Start with explicit props and composition when the relationships are clear. Consider context when many distant components need the same information and passing it through intermediate components becomes inconvenient. Context is not required just because a value passes through more than one component.

## Choose component boundaries with purpose

A component boundary helps when it:

- Names a meaningful piece of the interface.
- Keeps one responsibility understandable.
- Is reused or needs a stable place for its own behavior.
- Gives a layout a clear area to arrange.
- Separates content from a reusable wrapper.

Avoid creating a component for every HTML tag or tiny fragment. Too many small components can make the component tree harder to follow. Also avoid a single giant page component that mixes unrelated sections. Split where names and responsibilities make the structure clearer.

## Composition is not inheritance

Composition means one component renders other components and passes them props or children. A specialized component can wrap and configure a general one. React component reuse usually works well through these relationships rather than by extending one component class to inherit its UI.

This is a design approach, not a rule that every component must wrap another. Sometimes a direct component with clear props is the simplest design.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Passing data through a layout that never uses it | The layout becomes coupled to page-specific data. | Compose the data-consuming component as the layout's children. |
| Making a wrapper depend on exact child markup | The wrapper is harder to reuse with different content. | Let the caller supply content through `children` or named slots. |
| Using `children` for several unrelated page regions | It is unclear which content belongs in each region. | Use named JSX props for distinct regions such as header, sidebar, and footer. |
| Creating a specialized component that adds no meaning | It adds another layer without clarifying the API. | Keep the generic component when its props are already clear. |
| Making a component for every small tag | The hierarchy gets noisy and harder to scan. | Create boundaries around meaningful UI and responsibilities. |
| Using context for every shared value | Data flow becomes less explicit than necessary. | Start with props and composition; add context when distant consumers need shared data. |
| Assuming composition prevents props from being passed | Data still needs to reach the component that uses it. | Pass the value directly to its consumer while using children to arrange its UI. |

## Key takeaways

- Compose interfaces by placing components inside other components' JSX.
- Containment lets a wrapper render content supplied by its caller.
- Specialized components can configure general components with fixed props and children.
- Layouts can use `children` and named JSX slots for separate regions.
- Pass data directly to the component that uses it when composition can avoid needless forwarding.
- Use context for genuinely shared, distant data rather than as a default replacement for props.
- Choose component boundaries that make names and responsibilities clear.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [016 - The children prop and nested content](016-children-prop-and-nested-content.md) | [Open Q&A](../qnas/017-composition-containment-and-layouts.md) | [Open Example](../examples/017-composition-containment-and-layouts.md) | [018 - Conditional rendering patterns](018-conditional-rendering-patterns.md) |
