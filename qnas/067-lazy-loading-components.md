# Lazy Loading Components with lazy — Q&A

## Core concepts

### 1. What does React lazy loading mean?

It means deferring the JavaScript module for a component until React first tries to render that component.

### 2. What does `lazy` do?

`lazy` accepts a loader function and returns a React component type. React calls the loader when the lazy component is first rendered.

### 3. Does calling `lazy` immediately download the component?

No. Declaring the lazy component does not call its loader. Rendering the component for the first time triggers the loader.

### 4. Does `lazy` automatically hide the component?

No. It only defines how the component's code is loaded. The parent decides whether and when to render it.

### 5. What does dynamic `import()` return?

It returns a Promise that resolves to a module namespace object containing the module's exports.

### 6. What does code splitting mean?

It is a build-time arrangement that emits parts of an application as separate JavaScript assets instead of putting all code into one initial asset. Dynamic imports are a common split point.

### 7. Does every dynamic import create a separate chunk in every environment?

No. The bundler or framework controls how dynamic imports are transformed and grouped. Inspect the project's production build to see the actual output.

### 8. Is lazy loading the same as data fetching?

No. Lazy loading fetches code for a module on demand. A component may separately request data, but these are distinct concerns.

### 9. Does a lazy component have to be a function component?

It must resolve to a valid React component type. Valid component types can include function or class components and supported wrapped component types.

### 10. Does lazy loading guarantee a faster application?

No. It can reduce code needed on the initial path, but it also introduces loading work when a deferred section is first opened. Measure the common user path and the deferred interaction.

## Loader and exports

### 11. What should the lazy loader return?

A Promise or Promise-like value that eventually resolves to a module object whose `default` property is a valid React component type.

### 12. Why is a default export commonly used with `lazy`?

The resolved module is read from its `default` property, so a default-exported component works directly with the loader.

### 13. How do I lazy-load a default-exported component?

~~~jsx
const SettingsPanel = lazy(() => import("./SettingsPanel.jsx"));
~~~

The imported module must have a default export that is a React component.

### 14. Can `lazy` load a named export directly?

Not in the same shape. Adapt the module's named export to an object with a `default` property.

### 15. How do I adapt a named export?

~~~jsx
const SettingsPanel = lazy(() =>
  import("./SettingsPanel.jsx").then((module) => ({
    default: module.SettingsPanel,
  })),
);
~~~

### 16. Can the adapter select a non-component export?

No. The selected value must be a valid React component type.

### 17. Where should the import path point?

It should be a module path the project's module resolver and build tool can resolve from the file containing the loader.

### 18. Should I build a loader Promise during every render?

No. Declare the lazy component once at module scope. React then manages the loader and its cached result for that lazy component type.

### 19. What happens if the loader rejects?

React throws the rejection reason during rendering. The nearest Error Boundary can display an error state.

### 20. Does React cache a successful lazy load?

Yes. React caches the loader Promise and its resolved module value for that lazy component type.

### 21. Does React cache a rejected lazy load?

React caches the loader Promise as well. Re-rendering the same lazy component does not necessarily call the loader again, so do not promise that a simple state change will retry it.

## Component identity and state

### 22. Where should I declare a lazy component?

At the top level of a module, outside any component function.

### 23. Why should a lazy component not be declared inside another component?

Each parent render could create a new component type. React may treat it as a different component and reset state in its subtree.

### 24. Is this a safe declaration?

~~~jsx
function Page() {
  const Editor = lazy(() => import("./Editor.jsx"));
  return <Editor />;
}
~~~

No. Move the `lazy` call to the module scope and render the stable component type inside `Page`.

### 25. Does conditionally rendering a lazy component make its module load on demand?

Yes. If the parent does not render it until a user opens a section, the loader is not called until that first render attempt.

### 26. If I hide and show an already loaded lazy component, does it download the module again?

Normally no. React caches the resolved module for that lazy component type. Hiding the component can still unmount it and discard its local state, depending on the component tree.

### 27. Does caching the module preserve the component's state?

No. Module caching and component state are different. Unmounting a component may discard its local state even though its code remains loaded.

## Suspense and loading UI

### 28. What happens while a lazy component's code is pending?

The lazy component suspends. The closest parent `Suspense` boundary that covers it displays its fallback.

### 29. Is `Suspense` required around a lazy component?

In client-rendered UI, provide an appropriate boundary for any lazy component that may be rendered while its code is pending. Otherwise the application has no local loading fallback for that suspension.

### 30. What does the `fallback` prop do?

It is the React node the boundary displays while its children suspend.

### 31. Can the fallback be any React node?

Yes. Keep it lightweight and appropriate for the region being replaced, such as a status message or skeleton.

### 32. What determines how much of the interface is replaced by a fallback?

The placement of the `Suspense` boundary. Its fallback stands in for the boundary's suspended children.

### 33. How can I keep page navigation visible while a lazy panel loads?

Place the navigation outside the boundary and wrap only the panel region that may suspend.

### 34. What if multiple lazy components share one Suspense boundary?

They share a reveal group. The boundary shows one fallback for that region while the content needed for the reveal is pending.

### 35. When are nested Suspense boundaries useful?

Use them when one part of a page can be shown meaningfully before a slower inner part is ready. Each boundary can provide a fallback for its own region.

### 36. Should every small component have its own Suspense boundary?

Usually not. Use boundaries where a loading state makes sense to the person using the application, not simply around every component.

### 37. Does a Suspense fallback handle a failed import?

No. A fallback represents pending work. Use an Error Boundary for a rejected lazy import.

### 38. Can I use Suspense for a normal fetch inside `useEffect`?

No. Suspense does not automatically detect a request started inside an Effect or event handler. The data source must use a supported Suspense integration.

### 39. Can a regular Promise activate Suspense just by being pending?

No. A pending Promise by itself is not enough. It must be read through a supported React mechanism or a Suspense-enabled framework or data source.

### 40. What happens if the fallback itself suspends?

The nearest parent Suspense boundary can handle that fallback's suspension.

## Errors and recovery

### 41. What should the UI do when a lazy module fails to load?

An Error Boundary should replace the failed region with a clear message and an appropriate recovery action, such as checking the connection or refreshing the page.

### 42. Should the error boundary sit inside or outside Suspense?

A common arrangement is an Error Boundary outside Suspense: Suspense displays pending UI, while the outer Error Boundary handles a failed render or rejected import.

### 43. Does an Error Boundary catch event-handler errors?

No. Error Boundaries handle errors thrown while rendering descendants and related React lifecycle work. Errors thrown directly from event handlers need their own handling.

### 44. Can I always retry by resetting the Error Boundary state?

No. React caches the lazy loader Promise for that lazy component type, including a rejected Promise. Resetting an Error Boundary alone may render the same failure again.

### 45. What can affect retry behavior after a chunk load failure?

The browser, network, build tool, deployment setup, and the application's module-loading strategy can all matter. Avoid claiming a universal retry behavior.

## Performance and design choices

### 46. What are good candidates for lazy loading?

Substantial sections that are not needed for the initial view or are opened only in a particular flow, such as an editor, a charting workspace, or an admin route.

### 47. What should usually remain eager?

Small, frequently used, or essential first-view content when delaying it would make the common page slower or less useful.

### 48. Why should I avoid lazy-loading every component?

Too many split points add complexity and can defer code that the user needs immediately. Splitting should improve the real navigation path, not merely increase the number of chunks.

### 49. Why might I never notice the fallback?

The module may load very quickly, especially from local development or cache. Do not add an artificial production delay; inspect loading with browser tools or network throttling.

### 50. How should I decide whether a split is beneficial?

Inspect the production output and measure the initial path and the interaction that opens the deferred section. Keep the split when it improves a meaningful user experience without creating an avoidable wait.

### 51. Does lazy loading remove code from the application?

No. It changes when code is requested and evaluated; the component remains part of the application bundle graph and is loaded when needed.

### 52. Does a lazy component have to be rendered after a click?

No. It can be rendered based on any application condition. A click is one common way to defer a nonessential panel.

## Quick checks

### 53. What are the three parts of a basic lazy component setup?

Declare the component with `lazy` at module scope, render it when needed, and place it under an appropriate `Suspense` boundary.

### 54. Which tool represents a pending lazy import?

The nearest `Suspense` boundary.

### 55. Which tool represents a rejected lazy import?

An Error Boundary.

### 56. What is the main mistake that can reset state?

Creating the lazy component type inside another component instead of once at module scope.

### 57. What is the essential export requirement for the default loader pattern?

The dynamically imported module must resolve to an object with a valid component at `default`.

### 58. What is the practical rule for deciding what to split?

Defer meaningful code that is not needed on the common initial path; measure the production result instead of splitting every component by default.
