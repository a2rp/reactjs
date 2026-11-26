# Organizing Components, Features, and Application Code

## Learning goals

After this lesson, you should be able to:

- Choose a folder structure that makes an application easier to understand.
- Separate application setup, routes, feature code, and genuinely shared code.
- Decide when code belongs together and when it deserves its own module.
- Set component boundaries based on responsibility and data flow.
- Keep each piece of state with one clear owner.
- Refactor toward a structure that fits the app instead of creating layers too early.

## 1. Organization should make change easier

A React application starts with a few components. As it grows, one file may accumulate unrelated screens, state updates, data access, and styling. Splitting files helps only when the boundaries clarify what each part owns.

A useful structure lets a developer answer questions such as:

- Where is this screen assembled?
- Which feature owns this component or state?
- Is this code shared across features?
- Where does this action update data?
- Which files should change when this feature changes?

React does not require one universal folder layout. Treat the examples in this lesson as starting points, then use the conventions of the project's framework and build tools.

## 2. Start from the application tree

A typical client-rendered application has a small entry point that mounts the app, an application shell that composes routes and global providers, and feature code that renders the actual screens.

```text
src/
  main.jsx
  app/
    App.jsx
    app.css
  features/
    tasks/
      ...
  shared/
    ...
```

The entry point should stay small. It commonly imports global styles, creates the React root, and renders the top-level application component.

The application shell can own cross-cutting composition such as routing, top-level layout, and providers that truly apply across the app. Avoid turning it into a container for every business rule.

## 3. Organize around features

A feature is a cohesive area of product behavior, such as tasks, accounts, search, or billing. A feature may contain the components, state logic, hooks, styles, and data helpers that mainly serve that behavior.

```text
features/
  tasks/
    components/
    hooks/
    taskReducer.js
    tasks.css
    TaskPage.jsx
  projects/
    components/
    hooks/
    projectApi.js
    ProjectsPage.jsx
```

Feature-oriented organization keeps related code close. A change to one feature is less likely to require searching several broad folders.

Keep a feature small at first. A feature with one page and two components may need only those files; add subfolders when the number or role of files makes the boundary useful.

## 4. Compare common organization styles

### Layer-oriented structure

```text
components/
hooks/
pages/
services/
```

This can be easy to understand in a very small application. As features grow, related code spreads across folders: a task page, its form, hook, API calls, and styles may all live far apart.

### Feature-oriented structure

```text
features/
  tasks/
  projects/
shared/
app/
```

This groups code by the product area it supports. It often works well for applications with multiple meaningful features.

### A practical combination

Use a small number of layers at the top, then organize feature-owned code by responsibility:

```text
src/
  app/
  features/
    tasks/
      components/
      hooks/
  shared/
    components/
    hooks/
```

Avoid choosing a large architecture before the application has enough code to justify it. Start with clear, local boundaries and extract shared layers when actual reuse or ownership makes them helpful.

## 5. Separate app, feature, and shared responsibilities

### Application code

The `app/` area composes the application: root layout, route declarations, global providers, and global styles. It may import features, but a feature should not need to import the application's root just to render itself.

### Feature code

A feature owns code whose meaning is specific to that feature:

- A task form used only by task management.
- A task reducer and task-specific state.
- A task API adapter.
- A task page and its local components.

Keeping these files together makes it easier to see what the feature depends on.

### Shared code

Put code in `shared/` when it is genuinely reusable across unrelated features and has an appropriate general interface:

- A button with general visual and interaction behavior.
- A date-formatting utility with a clear input/output contract.
- A common authentication provider.

Do not move a file into `shared/` simply because it is used in two places today. If its behavior is still specific to one feature, a clear feature-owned interface may be better. When real reuse emerges, extract the stable common part.

## 6. Define clear module boundaries

A module should have a reason to change and a clear interface. Components should receive data and callbacks through props when their parent owns the state.

```jsx
function TaskList({ tasks, onToggle, onDelete }) {
  return tasks.map((task) => (
    <TaskItem
      key={task.id}
      task={task}
      onToggle={onToggle}
      onDelete={onDelete}
    />
  ));
}
```

This component knows how to render a list and report user actions. It does not need to know whether the parent stores tasks in `useState`, a reducer, a context, or a server cache.

Use composition when a component should provide structure while its caller decides what content goes inside:

```jsx
function Card({ title, children }) {
  return (
    <section className="card">
      <h2>{title}</h2>
      {children}
    </section>
  );
}
```

Avoid components with many unrelated boolean props that create hard-to-understand combinations. Prefer a small number of focused components, explicit variants, or composition.

## 7. Keep state with a clear owner

For each piece of state, choose one component or system as its source of truth.

- Local input state can stay inside the input or form.
- State coordinating a page's siblings can live in their closest common parent.
- State used by distant parts of one feature may belong in a feature-level owner or provider.
- Cross-feature state may need a higher-level owner only when those features truly coordinate around it.
- Shareable navigation state may belong in the URL.
- Remote data needs a data-fetching and caching strategy appropriate to the app.

Do not duplicate the same mutable value in multiple components and try to keep the copies synchronized. Pass the value and event handlers down, or use an appropriate shared owner.

A custom Hook can reuse stateful logic, but calling a custom Hook separately in two components does not make them share one state instance. The Hook runs as part of each calling component. To share a value, give it a common owner and expose it through props, context, or a store.

## 8. Choose component boundaries by responsibility

A component is worth extracting when it has a distinct job, is reused, has a meaningful interface, or can be understood and changed independently.

Common responsibilities include:

- **Route or page components**: coordinate a screen and its page-level state.
- **Feature components**: implement a specific product behavior.
- **Presentational components**: render given props and report interactions.
- **Layout components**: arrange sections and provide shared page structure.
- **Form components**: own input details and report validated values.
- **Hooks**: package reusable stateful behavior or synchronization with an external system.

Do not create a component for every HTML element. Excessive splitting can make simple data flow harder to follow. Conversely, split a component when its render logic, event handling, or state responsibilities become difficult to reason about together.

## 9. Keep effects and external integrations at clear boundaries

Effects synchronize React with systems outside React, such as browser APIs, subscriptions, and third-party widgets. Keep that synchronization near the component or feature that owns the relationship.

If several components need the same external-system behavior, a custom Hook can express the shared use case, such as `useOnlineStatus()` or `useChatRoom(options)`. Prefer purpose-specific Hook names over generic lifecycle wrappers.

Do not add an Effect merely to copy one piece of React state into another. Derive values during render when they can be calculated from existing props or state.

## 10. Decide where data access belongs

Keep the data flow explicit. For a small feature, a page or feature hook can make a request. If request handling grows, separate the API boundary into a feature-owned module such as `tasksApi.js`.

A UI component should not need to know transport details such as request URLs, response parsing, or retry behavior. Conversely, avoid building a generic service layer before you have multiple concrete use cases that justify it.

Keep server-only code out of browser bundles, and follow the conventions of the framework that owns routing and data loading.

## 11. Name files and exports consistently

Pick conventions and apply them throughout the repository:

- Component files can use PascalCase, such as `TaskList.jsx`.
- Hooks start with `use` and use camelCase, such as `useTasks.js`.
- Utility and reducer modules commonly use camelCase, such as `taskReducer.js`.
- Use explicit named exports for modules with multiple public items.
- Choose default exports or named exports consistently with the project.

Avoid vague names such as `helpers.js`, `common.js`, or `misc.js` when the file's purpose can be named more clearly. A well-named file communicates why its code exists.

## 12. Use barrel files deliberately

A barrel file re-exports items from other modules, often through an `index.js`. It can provide a convenient public interface, but too many barrels can hide where code comes from, create circular imports, or make dependencies less obvious.

For a small feature, direct imports are often clearest:

```js
import TaskList from "./components/TaskList.jsx";
```

Use a barrel when it represents a deliberate module boundary and improves the consumer's imports. Keep internal imports direct where that makes ownership clearer.

## 13. Keep styles, tests, and assets near their owner

Place styles and assets near the feature or component that owns them unless they are global or truly shared. This makes cleanup and changes easier to scope.

Tests can follow the same principle: a test for a task reducer can live near that reducer, while end-to-end tests may live in a top-level test area according to the repository convention. Consistency is more useful than a particular universal directory name.

## 14. Refactor as the application grows

A safe evolution might look like this:

1. Start with a small `App.jsx` and a few focused components.
2. Extract a file when a component or module has a distinct responsibility.
3. Group related files into a feature when navigating between them becomes repetitive.
4. Move stable, cross-feature behavior into shared code only when the interface is clear.
5. Introduce context, reducers, or a store when the state ownership or update logic calls for them.
6. Revisit boundaries when a feature's files repeatedly change together or have unclear dependencies.

A folder tree is not an architecture by itself. The important part is that ownership and dependencies remain understandable.

## 15. Common mistakes

- Keeping every component in one file after responsibilities have diverged.
- Creating a large folder hierarchy before it solves a real navigation problem.
- Moving feature-specific code to `shared/` too early.
- Treating custom Hooks as shared state containers.
- Duplicating mutable state in sibling components.
- Building a generic service or utility layer without concrete use cases.
- Using a catch-all `utils.js` for unrelated behavior.
- Letting shared modules import feature modules and form circular dependencies.
- Adding components that only wrap one element without a useful boundary.
- Repeating business rules in multiple UI components.
- Adding Effects to derive values that can be calculated during render.
- Assuming a folder convention replaces a clear data flow.

## Key takeaways

- React does not impose one folder structure; select one that fits the app and its framework.
- Feature-oriented folders keep behavior that changes together close to each other.
- Keep app composition, feature-owned code, and genuinely shared code distinct.
- Give each piece of mutable state one clear owner.
- Use component props and composition to keep dependencies explicit.
- Extract hooks for meaningful reusable stateful behavior, not as a way to make separate state instances magically shared.
- Add architecture when it makes code easier to change; refactor as the application gives you evidence about its boundaries.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="087-url-state-and-navigation.md">087 - URL State, Search Parameters, and Navigation</a></td><td><a href="../qnas/088-organizing-react-applications.md">Q&amp;A</a></td><td><a href="../examples/088-organizing-react-applications.md">Example</a></td><td><a href="089-design-systems-and-component-libraries.md">089 - Design Systems, Shared Components, and Component Libraries</a></td></tr>
  </tbody>
</table>
