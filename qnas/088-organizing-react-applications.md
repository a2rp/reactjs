# Organizing Components, Features, and Application Code - Q&A

## Structure and boundaries

### 1. Does React require a particular folder structure?

No. React components can be organized in different ways. Choose a structure that makes ownership, dependencies, and change locations easy to understand, and follow any conventions required by the framework or repository.

### 2. What does “feature-oriented” organization mean?

It groups code by the product behavior it supports. For example, task-related pages, components, state logic, styles, and API helpers live near one another under `features/tasks/`.

### 3. What is layer-oriented organization?

It groups files by technical role, such as `components/`, `hooks/`, `pages/`, and `services/`. This can be straightforward for a small app, but as features grow, one feature's files may become spread across many folders.

### 4. Should every app use feature folders?

No. A very small application may be clearer with a few top-level components. Add feature folders when related files become difficult to find or change together. The structure should respond to the code that exists.

### 5. What belongs in an `app/` folder?

Typically, application composition: the root component, route setup, top-level layout, global providers, and global styles. Keep feature-specific business behavior in its feature.

### 6. What is a feature?

A cohesive area of product behavior such as tasks, search, accounts, or billing. A feature can own its page, components, hooks, styles, data helpers, and state logic.

### 7. What belongs in `shared/`?

Code that has a useful general interface and is genuinely used across unrelated features. Examples include a general-purpose button, a common date formatter, or an application-wide authentication provider.

### 8. If two features use a component, should it immediately move to `shared/`?

Not necessarily. First check whether it really has the same responsibility and interface in both places. If the behavior is still specific to one feature, keep that ownership clear. Extract the common stable part when actual reuse provides value.

### 9. Is a folder tree itself an architecture?

No. The tree is only a filing system. A useful architecture also makes dependencies, state ownership, and responsibilities clear.

## Components and modules

### 10. When should I extract a component?

Extract one when it has a distinct responsibility, is reused, needs a clear interface, or can be understood and changed independently. Avoid extracting every single HTML element into a component without a useful boundary.

### 11. Should a page component own all of its UI state?

It can own state that coordinates the page's sections. State used only by one input or widget can remain local to that component. Split the owner only when the responsibility or data flow becomes difficult to follow.

### 12. What makes a component interface clear?

It receives the data it needs through props and reports user actions through callbacks. Its parent can use it without knowing internal implementation details such as which Hook or reducer stores the state.

### 13. How can I avoid passing many props through unrelated intermediate components?

First check whether the component hierarchy reflects the actual UI. If many distant descendants need the same value, context may be appropriate. You can also compose children or move a component closer to where its data is available. Avoid adding context just to avoid one or two clear prop boundaries.

### 14. When is composition better than boolean props?

Composition is often clearer when a component should provide structure but callers need to choose its contents. It can avoid many boolean combinations that create different, hard-to-reason-about modes.

### 15. Should I split a very long component even if nothing is reused?

Possibly. Reuse is only one reason to extract code. A distinct form, list, or interaction with a meaningful interface may be easier to understand as a separate component even if it appears once.

### 16. What should a presentational component know about data storage?

Usually only the props and callbacks in its interface. It should not need to know whether a parent uses local state, a reducer, context, a server cache, or another store.

## State and Hooks

### 17. What is a single source of truth?

Each unique mutable value has one clear owner. Other components read it through props, context, or a store rather than keeping unsynchronized copies.

### 18. Where should state shared by two siblings live?

Usually in their closest common parent. The parent owns the value and passes it and the relevant event handlers down. This keeps the siblings synchronized.

### 19. Does calling the same custom Hook in two components share its state?

No. Each component call gets its own Hook state instance. Custom Hooks share reusable logic; they do not automatically share one state value. To share state, give it a common owner or use an appropriate shared-state mechanism.

### 20. When should I use context?

Use context when many components at different depths need the same value, such as locale or a feature-level state. A provider should have a clear scope and owner. Context is not automatically the best choice for every value used by more than one component.

### 21. When should state move into a reducer?

A reducer can help when related state updates have multiple action types, the transition rules are becoming hard to follow across handlers, or several updates should be expressed as named actions. A few simple independent values can remain in `useState`.

### 22. Should all application state live in one global store?

Usually not. Keep local state close to its use. Move state higher or into shared infrastructure when there is a real coordination need. A single global store can make ownership and update paths harder to understand if used without a reason.

### 23. Should I make a custom Hook for every repeated line?

No. Extract a Hook when it represents meaningful reusable stateful behavior or synchronization. A tiny wrapper around a single state call can add indirection without improving the interface.

### 24. Should Effects be grouped in a generic lifecycle Hook?

Avoid generic wrappers such as a `useMount` hook that hide an Effect's purpose and dependencies. Keep the Effect visible or extract a purpose-specific Hook, such as `useOnlineStatus()` or `useChatRoom(options)`.

### 25. Should derived values be stored in state?

Usually no. If a value can be calculated from current props or state during render, calculate it there. Duplicating derived state creates synchronization work and can introduce stale values.

## Dependencies, files, and growth

### 26. Which direction should imports flow?

A simple starting rule is: the application composes features; features may depend on shared utilities and components; shared code should not depend on a specific feature. Keep exceptions intentional and avoid circular imports.

### 27. How should I name files?

Use one consistent convention. Component filenames commonly use PascalCase, Hook filenames start with `use`, and utility or reducer files commonly use camelCase. Specific names such as `taskReducer.js` communicate intent better than `helpers.js`.

### 28. Should every folder have an `index.js` barrel?

No. A barrel can provide a deliberate public interface, but widespread barrels can hide dependencies, create circular imports, or make navigation harder. Use one when it improves a clear module boundary.

### 29. Where should styles and assets go?

Keep feature-specific styles and assets near the code that owns them. Keep global styles and truly shared assets at a suitable app-level location. Consistent placement matters more than a universal directory name.

### 30. Where should tests go?

Follow the repository's established convention. Tests for a feature or module can live beside that code or in a corresponding test directory. End-to-end tests may have a separate top-level location. Keep the relationship easy to discover.

### 31. When should I create a generic service or utility layer?

When multiple concrete use cases have revealed a stable shared responsibility. Do not build a generic abstraction simply because it might be useful later. Unneeded layers make code navigation and change more complicated.

### 32. What should I do when a feature gets large?

Look for cohesive subfeatures and responsibilities. Separate components, state logic, API boundaries, and styles where those boundaries help people change code independently. Keep the overall feature ownership visible rather than scattering files across generic directories.

### 33. Should I preserve the same folder structure across every project?

Consistency within one repository is useful, but a small learning project and a large multi-feature app may need different structures. Also follow framework conventions for route and server code where applicable.

### 34. How can I tell if my structure needs refactoring?

Refactor when people repeatedly search many directories for one feature, unrelated changes keep colliding in the same file, module ownership is unclear, dependencies point in confusing directions, or shared state has several competing copies.

### 35. What is a good way to start organizing a new app?

Start with a small entry point, a root app component, and a few focused components. Add a feature folder when related behavior becomes sizable. Extract shared code when its purpose and interface are stable. Refactor based on real navigation and change patterns.

## Quick review

- React does not mandate a single folder layout.
- Feature folders group code that changes together.
- Keep app composition, feature ownership, and shared code distinct.
- Put each state value under one clear owner.
- Custom Hooks reuse stateful logic, not a shared state instance.
- Use component interfaces, composition, and dependency direction to make data flow clear.
- Add abstractions when they solve an observed problem.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="087-url-state-and-navigation.md">087 - URL State, Search Parameters, and Navigation - Q&amp;A</a></td><td><a href="../lessons/088-organizing-react-applications.md">Lesson</a></td><td><a href="../examples/088-organizing-react-applications.md">Example</a></td><td><a href="089-design-systems-and-component-libraries.md">089 - Design Systems, Shared Components, and Component Libraries - Q&amp;A</a></td></tr>
  </tbody>
</table>
