# 017. Composition, containment, and reusable layouts - Q&A

## Composition basics

### 1. What does composition mean in React?

Composition means building a larger interface by rendering smaller components inside other components. A page component can arrange a header, navigation, and content component in its JSX, then pass each piece the props it needs.

~~~jsx
function CoursePage() {
  return (
    <main>
      <CourseHeader />
      <CourseSummary />
      <CourseCatalog />
    </main>
  );
}
~~~

### 2. How does composition relate to the component tree?

Whenever one component renders another, the rendered component becomes part of that UI tree. The tree shows how the interface is nested and which components are responsible for each region. It does not mean that every HTML element needs a separate component.

### 3. Why is composition useful for reuse?

A component can reuse another component by rendering it and providing props or nested content. This lets the same wrapper or layout work with different content without copying its structure or relying on a class hierarchy.

### 4. Should every small JSX fragment become a component?

No. Create a component when it gives a meaningful name or responsibility, needs its own behavior, is reused, or makes a large parent easier to understand. Extracting every tag or tiny fragment can make the component tree harder to follow.

## Containment and children

### 5. What does containment mean?

Containment is a pattern where a component provides a wrapper or region and its caller supplies the content to put inside it. The wrapper commonly receives that content through `children`:

~~~jsx
function Notice({ children }) {
  return <aside className="notice">{children}</aside>;
}

function LessonPage() {
  return (
    <Notice>
      <p>Read the explanation, then try the example.</p>
    </Notice>
  );
}
~~~

### 6. Does a component automatically display its children?

No. The receiving component must include `{children}` in its returned JSX. It controls where the nested content appears, and it can choose not to render it.

### 7. When should a component use `children`?

Use `children` when the component has one general content area, such as a panel, notice, dialog body, or page's main content. The caller can provide text, elements, or other components there.

### 8. When are named JSX props better than `children`?

Use named props when a component has several distinct content areas whose roles should be clear, such as a `header`, `sidebar`, `actions`, or `footer`:

~~~jsx
<PageLayout
  header={<CourseHeader />}
  sidebar={<CourseNavigation />}
  footer={<CourseFooter />}
>
  <CourseCatalog />
</PageLayout>
~~~

The names make it clear which content belongs in each region. A prop that contains JSX is still an ordinary prop; it is not a special slot feature.

## Specialization and layouts

### 9. What is a specialized component?

A specialized component wraps or configures a more general component for a common case. It can supply fixed props or arrange children so callers use a clearer, more focused interface:

~~~jsx
function WarningNotice({ children }) {
  return (
    <Notice title="Important" tone="warning">
      {children}
    </Notice>
  );
}
~~~

### 10. When is a specialized component worth creating?

Create one when the repeated configuration has a useful meaning or a stable name, such as `WarningNotice`. If the new component only forwards every prop without clarifying behavior or usage, keeping the general component may be simpler.

### 11. What should a reusable layout own?

A layout should own shared structure and placement: where its header, navigation, main content, and footer appear. Its caller supplies the page-specific content. This keeps the layout useful across pages with different content.

### 12. Should a layout know the details of every child it places?

Usually not. A layout should be able to place supplied JSX without depending on its internal markup or data model. This separation lets callers change the content without rewriting the layout.

### 13. Can one component receive both `children` and named content props?

Yes. For example, a page layout may use `header`, `sidebar`, and `footer` props for named regions and `children` for the central content. Choose the API that makes the roles understandable.

## Data flow, context, and design

### 14. How can composition reduce prop drilling?

If intermediate components only arrange content, the component that knows the data can create the data-consuming element and pass it as children. The layout then places that element without receiving or forwarding the data itself:

~~~jsx
function App({ courses }) {
  return (
    <PageLayout>
      <CourseCatalog courses={courses} />
    </PageLayout>
  );
}
~~~

`PageLayout` does not need a `courses` prop because `CourseCatalog` is the consumer.

### 15. Does composition replace props?

No. The component that needs a value still needs to receive it, usually as a prop. Composition helps arrange components and can avoid passing data through components that do not use it; it does not remove the data flow itself.

### 16. When should an intermediate component receive a prop?

Pass a prop to an intermediate component when that component uses the value, makes a decision based on it, or transforms it for a child. If it only passes the value along for another component, consider composing the consumer as children or otherwise simplifying the path.

### 17. When should I use context instead of composition or props?

Start with props and composition when the relationships are clear. Context can help when many distant components in a subtree need the same value and forwarding it through intermediate components becomes inconvenient. Context is not necessary just because a value crosses more than one component.

### 18. Is composition the same as inheritance?

No. Composition describes components rendering and configuring other components. Inheritance describes a class extending another class. React UI reuse commonly uses composition, but the practical goal is clear component responsibilities, not wrapping everything in another component.

### 19. How do I decide whether to extract a component boundary?

Extract a component when the boundary makes the UI easier to name, reuse, test independently, or maintain with its own behavior. Keep a small section inline if extracting it would add indirection without making its purpose clearer.

### 20. Is a React element passed through a prop rendered automatically?

No. A JSX element passed as `header`, `sidebar`, or another prop is a value. The receiving component must render that prop in its JSX, just as it must render `children`.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="016-children-prop-and-nested-content.md">016 - The children prop and nested content - Q&amp;A</a></td><td><a href="../lessons/017-composition-containment-and-layouts.md">Lesson</a></td><td><a href="../examples/017-composition-containment-and-layouts.md">Example</a></td><td><a href="018-conditional-rendering-patterns.md">018 - Conditional rendering patterns - Q&amp;A</a></td></tr>
  </tbody>
</table>
