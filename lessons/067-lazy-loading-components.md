# Lazy Loading Components with lazy

React's `lazy` API lets an application defer loading a component's JavaScript module until the component is rendered for the first time. When paired with dynamic imports and a build tool that supports code splitting, this can move code for less frequently used parts of an application into separate chunks.

Lazy loading is a delivery decision. It does not make a component invisible by itself, fetch data, or improve every page automatically. The application must decide when to render the component, where its loading state belongs, and how to handle a failed module load.

## Learning goals

After this lesson, you should be able to:

- explain how `lazy` and dynamic `import()` defer component code;
- declare lazy component types at module scope;
- use default exports and adapt named exports;
- render lazy components behind a suitable `Suspense` boundary;
- distinguish pending code from a failed import;
- choose which parts of an application are worth splitting;
- identify common state-reset, fallback, and loading mistakes.

## Static imports and dynamic imports

A static import is resolved as part of the module's normal dependency graph:

~~~jsx
import ReportsPanel from "./ReportsPanel.jsx";

function Dashboard() {
  return <ReportsPanel />;
}
~~~

A dynamic import returns a Promise and can give a bundler a split point:

~~~js
const reportsModule = import("./ReportsPanel.jsx");
~~~

Use `lazy` to turn a Promise-returning loader into a component that React can render:

~~~jsx
import { lazy } from "react";

const ReportsPanel = lazy(() => import("./ReportsPanel.jsx"));
~~~

The import path must refer to a module the project can resolve. Bundlers and frameworks decide how dynamic imports are emitted and split into assets. Check the build setup rather than assuming every environment will produce the same chunk layout.

## Export the component as default

The loader's Promise must resolve to an object whose `default` property is a valid React component type. A straightforward module uses a default export:

~~~jsx
function ReportsPanel() {
  return (
    <section>
      <h2>Reports</h2>
      <p>Report content is loaded when this panel is opened.</p>
    </section>
  );
}

export default ReportsPanel;
~~~

Then the importing module can use:

~~~jsx
const ReportsPanel = lazy(() => import("./ReportsPanel.jsx"));
~~~

A named export can be adapted to the required shape:

~~~jsx
const ReportsPanel = lazy(() =>
  import("./ReportsPanel.jsx").then((module) => ({
    default: module.ReportsPanel,
  })),
);
~~~

Prefer a normal default export when that module exists mainly to provide one lazy component. Use the adapter when a named export better fits an established module structure.

## Declare lazy component types at module scope

Declare each lazy component once, outside other components:

~~~jsx
import { lazy, Suspense } from "react";

const ReportsPanel = lazy(() => import("./ReportsPanel.jsx"));

function Dashboard() {
  return (
    <Suspense fallback={<p role="status">Loading reports...</p>}>
      <ReportsPanel />
    </Suspense>
  );
}
~~~

Do not call `lazy` inside a component or inside an event handler. A lazy declaration creates a component type. Recreating that type during a parent render can make React treat it as a different component and reset its state.

Bad:

~~~jsx
function Dashboard() {
  const ReportsPanel = lazy(() => import("./ReportsPanel.jsx"));
  return <ReportsPanel />;
}
~~~

Good:

~~~jsx
const ReportsPanel = lazy(() => import("./ReportsPanel.jsx"));

function Dashboard() {
  return <ReportsPanel />;
}
~~~

## Loading begins when the lazy component is rendered

Declaring the lazy component does not immediately call the loader. React calls it the first time the lazy component is rendered. If the parent renders that component only after a user opens a panel, the code can stay deferred until that interaction.

~~~jsx
const HelpPanel = lazy(() => import("./HelpPanel.jsx"));

function Dashboard() {
  const [showHelp, setShowHelp] = useState(false);

  return (
    <>
      <button type="button" onClick={() => setShowHelp(true)}>
        Open help
      </button>

      {showHelp && (
        <Suspense fallback={<p role="status">Loading help...</p>}>
          <HelpPanel />
        </Suspense>
      )}
    </>
  );
}
~~~

Once React has called a lazy loader, React caches the returned Promise and its resolved value. Hiding and rendering the same lazy component later normally reuses the loaded module; it does not turn the component into a fresh download each time.

Do not add artificial delays to real loaders just to make the fallback visible. A local or cached module can load too quickly for a person to notice the fallback. Verify loading behavior using the browser's network tools or a deliberately throttled development environment.

## Suspense handles the pending state

A lazy component suspends while its module is still loading. The nearest parent `Suspense` boundary that covers it displays its `fallback` until the component can render.

~~~jsx
<Suspense fallback={<div role="status">Loading the report panel...</div>}>
  <ReportsPanel />
</Suspense>
~~~

Place the boundary around a meaningful loading region. Keep surrounding controls, headings, navigation, or page context outside the boundary when they should remain usable or visible.

~~~jsx
function Dashboard() {
  return (
    <main>
      <h1>Analytics</h1>
      <DashboardNavigation />

      <section aria-labelledby="report-title">
        <h2 id="report-title">Reports</h2>
        <Suspense fallback={<ReportPanelSkeleton />}>
          <ReportsPanel />
        </Suspense>
      </section>
    </main>
  );
}
~~~

A fallback should communicate what is loading and fit the space being replaced. A small panel can use a local status message or skeleton; replacing an entire application shell with a large spinner is often unnecessarily disruptive.

The placement of the boundary defines the content React may replace with the fallback. If multiple lazy descendants share one boundary, they share one loading region. Nested boundaries can reveal useful content in stages, but do not add a boundary around every tiny component without a user-facing reason.

## Suspense is not general-purpose request handling

`Suspense` recognizes supported sources that suspend during rendering, including lazy component code. It does not automatically detect an ordinary network request started inside an Effect or click handler.

This does not become Suspense-enabled merely because it is wrapped in a boundary:

~~~jsx
useEffect(() => {
  fetch("/api/reports").then((response) => response.json());
}, []);
~~~

For ordinary Effect-based data fetching, represent pending and error states with application state. Use a data library or framework's documented Suspense integration when you specifically want data loading to suspend.

## Import failures need error handling

There are two different outcomes:

- **Pending module:** the Promise has not resolved yet. The closest `Suspense` boundary shows its fallback.
- **Rejected module load:** the Promise rejected, for example because a chunk could not be retrieved. React throws that rejection during rendering so an Error Boundary can handle it.

A `Suspense` fallback is not an error message. A typical arrangement is an Error Boundary outside the Suspense boundary:

~~~jsx
<PanelErrorBoundary>
  <Suspense fallback={<p role="status">Loading reports...</p>}>
    <ReportsPanel />
  </Suspense>
</PanelErrorBoundary>
~~~

The Error Boundary should explain that this region could not load and provide an appropriate next step for the application, such as checking the connection or refreshing. Do not claim that changing an unrelated state value will retry the same rejected lazy loader: React caches the loader Promise, including a rejection. Retry behavior depends on how the app and its bundler manage module loading.

## Decide what is worth loading lazily

A useful candidate is a substantial section that is not needed for the first view or is only opened in a particular flow, for example:

- an editor used only on an editing route;
- a charting or visualization area below the initial summary;
- an infrequently opened settings panel;
- an admin-only screen kept behind authorization.

Keep critical first-view UI eager when delaying it would make the page feel slower. Avoid splitting every small component. Each separate asset has loading and caching costs, and an overly fragmented component graph can add complexity or make a user wait for code that was always needed.

Look at production build output and real navigation behavior. The goal is to defer code that is not needed yet while keeping the common path responsive.

## Common mistakes

- Calling `lazy` inside a component, causing the component type to be recreated and its local state to reset.
- Omitting a `Suspense` boundary around a lazy component.
- Expecting an Effect-based fetch or any pending Promise to activate Suspense automatically.
- Using a named export without adapting the module to provide a `default` component.
- Expecting the fallback to handle a rejected import.
- Lazy-loading tiny components or essential first-view content without a clear benefit.
- Assuming a loading fallback must be visible for a fixed amount of time.
- Assuming dynamic imports always create the same chunks in every build tool.
- Assuming a rejected loader can be retried by simply rendering the same lazy component again.
- Putting all useful page context inside a boundary that replaces the whole region for one lazy child.

## Summary

- `lazy` creates a component whose loader runs when React first tries to render it.
- The loader must resolve to a module object with a valid `default` component; adapt a named export when needed.
- Declare lazy component types at module scope so their identity remains stable.
- A lazy component suspends while code is pending; `Suspense` controls the visible fallback.
- An Error Boundary handles a rejected import; a loading fallback represents pending content.
- Use lazy loading for meaningful code that is not needed on the common initial path, and confirm that the project build actually benefits.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="066-suspense-boundaries-and-fallbacks.md">066 - Suspense Boundaries and Fallbacks</a></td><td><a href="../qnas/067-lazy-loading-components.md">Q&amp;A</a></td><td><a href="../examples/067-lazy-loading-components.md">Example</a></td><td><a href="068-error-boundaries-and-recovery-ui.md">068 - Error Boundaries and Recovery UI</a></td></tr>
  </tbody>
</table>
