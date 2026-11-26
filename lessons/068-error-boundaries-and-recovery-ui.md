# Error Boundaries and Recovery UI

A rendering error in a React subtree can prevent that part of the interface from working. An **Error Boundary** catches eligible errors thrown while rendering its descendant components and replaces the failed region with fallback UI. This lets the rest of the application remain available when the boundary is placed around a useful part of the interface.

Error Boundaries are for failures, not ordinary loading states. A pending lazy component belongs to a `Suspense` fallback; a rejected lazy import can be handled by an Error Boundary.

## Learning goals

After this lesson, you should be able to:

- describe which errors an Error Boundary catches and which it does not;
- implement a boundary with `static getDerivedStateFromError`;
- use `componentDidCatch` for diagnostics and error reporting;
- place boundaries around meaningful parts of an interface;
- design recovery UI that does not expose internal error details;
- reset or remount a failed region when recovery is possible;
- distinguish a user-facing fallback from logging and monitoring.

## What an Error Boundary does

When a descendant throws an eligible error during rendering, React searches upward for the nearest Error Boundary. That boundary can render fallback UI in place of its failed descendants. Content outside the boundary can remain on screen.

~~~jsx
<ErrorBoundary>
  <AccountSummary />
</ErrorBoundary>
~~~

An Error Boundary is a React component with special class methods. Function components cannot currently become Error Boundaries using Hooks alone. A class boundary can be written once and reused around function components.

## The two boundary methods

Use `static getDerivedStateFromError` to update the boundary's state so its next render displays a fallback. Keep this method pure: calculate and return state only.

Use `componentDidCatch` for side effects such as sending an error report. It receives the thrown value and an object with a React component stack. Do not use it as the main place to set the fallback state.

~~~jsx
import { Component } from "react";

class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    reportError(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return <p role="alert">This section could not be displayed.</p>;
    }

    return this.props.children;
  }
}
~~~

In this example, `reportError` stands for an application-specific reporting function. Do not add an unconfigured service call to a boundary. If the application has no reporting service, logging locally during development can help investigate the failure.

The value thrown is not guaranteed to be an `Error` object. JavaScript permits throwing other values too. Diagnostics should handle that possibility, and the UI should not assume it can safely display `error.message`.

## What errors Error Boundaries catch

An Error Boundary catches errors thrown by descendants while React is rendering them. This includes errors during render and errors caused by lifecycle work managed by React for those descendants. It can also catch a rejected lazy component loader when that rejection is thrown while React attempts to render the lazy component.

The closest boundary handles the error. If that boundary itself throws while rendering its fallback, the same boundary cannot catch its own error; an outer boundary may catch it.

## What they do not catch

Error Boundaries do not catch every JavaScript error. In particular, they do not catch:

- errors thrown directly by event handlers;
- errors thrown later by ordinary asynchronous callbacks, such as `setTimeout` or `requestAnimationFrame`;
- errors thrown while rendering on the server;
- errors thrown by the boundary itself;
- errors outside the React tree.

Handle event failures in the event handler or route them into application state that the UI can render. Handle asynchronous work with the relevant Promise rejection path, request state, or data library. A boundary is not a replacement for `try`/`catch` around imperative code.

There is a React exception: errors thrown inside the function passed to React's `startTransition` can be handled by an Error Boundary. Do not generalize this exception to all asynchronous code.

## Suspense and Error Boundaries solve different states

A lazy import has a pending state and can also fail:

~~~jsx
<ErrorBoundary>
  <Suspense fallback={<p role="status">Loading settings...</p>}>
    <SettingsPanel />
  </Suspense>
</ErrorBoundary>
~~~

- While the module is pending, the nearest Suspense boundary shows its loading fallback.
- If the module import rejects, the Error Boundary handles the thrown failure.

The loading fallback should describe waiting. The error fallback should describe that the region failed and give a practical next step. Do not show a spinner forever for an error.

## Place boundaries around meaningful regions

Choose a boundary based on the part of the product that can fail independently and the part of the page you want to preserve.

~~~jsx
function Dashboard() {
  return (
    <main>
      <h1>Project dashboard</h1>
      <DashboardNavigation />

      <ErrorBoundary>
        <ProjectReports />
      </ErrorBoundary>
    </main>
  );
}
~~~

The heading and navigation are outside the boundary, so a failure in reports does not replace them. If reports contain several independent panels, smaller nested boundaries may let one panel fail while its siblings remain useful.

Do not wrap every tiny component by default. Too many boundaries can make the product's failure behavior inconsistent and harder to understand. A boundary should correspond to a useful recovery or isolation point: for example, a route, an editor, a reports panel, or a message list.

At the other extreme, a single boundary around an entire application may replace all app content when one isolated region fails. It can still be useful as a last-resort top-level fallback, but local boundaries usually provide better recovery for independent parts.

## Design useful recovery UI

A good fallback should:

- say which feature is unavailable in plain language;
- keep unrelated page controls and navigation visible;
- avoid exposing stack traces, internal paths, sensitive values, or raw server messages;
- offer a recovery action that makes sense, such as retrying after correcting state or navigating away;
- preserve enough context for the person to continue their task;
- have accessible semantics, such as `role="alert"` for an important failure message.

For a transient failure, an explicit retry can be useful. For a deterministic programming error, retrying the same render without changing anything will usually fail again. A retry button should reset the boundary and, when necessary, also change the state or inputs that caused the failure.

## Resetting a failed boundary

After an error, a boundary remains in its failed state until the application resets it. One approach is to provide a retry callback that resets the boundary's state and repairs or refreshes the child input.

~~~jsx
class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  handleRetry = () => {
    this.props.onRetry?.();
    this.setState({ hasError: false });
  };

  render() {
    if (this.state.hasError) {
      return (
        <section role="alert">
          <p>This panel could not be shown.</p>
          <button type="button" onClick={this.handleRetry}>
            Try again
          </button>
        </section>
      );
    }

    return this.props.children;
  }
}
~~~

The parent can change the value that caused the failure in `onRetry`. If it only clears the boundary state while the child still throws, React will encounter the same error again.

Another option is to change the boundary's `key` so React unmounts the failed boundary and mounts a fresh one. A key reset clears that boundary's state and remounts its children, but it does not repair broken code or guarantee that a network or module-loading failure will succeed on the next attempt.

For lazy imports in particular, React caches the loader Promise for a lazy component type. A rejected Promise may remain rejected when the same lazy type is rendered again. Explain the real recovery path for the app rather than promising that a boundary reset always retries the import.

## Logging and privacy

Use `componentDidCatch` to capture diagnostics such as the thrown value and component stack. A production reporting system can attach useful context like a route or feature name, subject to the application's privacy rules.

Keep the diagnostic payload intentional:

- collect only data needed to investigate the failure;
- do not send passwords, tokens, personal form contents, or other sensitive values;
- avoid displaying internal diagnostics to ordinary users;
- do not let failure of the logging service break the fallback UI.

A fallback is user interface. An error report is diagnostic data. They have different audiences and should be handled separately.

## Error boundaries and the component tree

A boundary catches descendant failures, not errors from its own render. If the fallback UI can fail, place an outer boundary above it or make the fallback deliberately small and dependable.

If no boundary catches a render error, React may remove the affected application UI rather than leave a potentially corrupted tree on screen. A top-level boundary can provide a final recovery screen, while nested boundaries keep isolated failures local.

Error boundaries are not a substitute for validating props, handling expected empty states, or checking normal request outcomes. Use regular conditional rendering for expected states such as no results or no permissions; reserve error fallbacks for unexpected failures.

## Common mistakes

- Expecting a boundary to catch an event-handler exception.
- Expecting it to catch arbitrary asynchronous callbacks or server-rendering errors.
- Using a Suspense loading message for a permanent failure.
- Putting the whole page inside a boundary when an isolated panel can be protected.
- Wrapping every small component in its own boundary without a recovery reason.
- Rendering raw error messages or stack traces to users.
- Calling external logging code from `getDerivedStateFromError` instead of a side-effect method.
- Trying to catch an error thrown by the boundary's own fallback with that same boundary.
- Resetting boundary state while leaving the failing child input unchanged.
- Treating a boundary reset as a guaranteed retry for a cached rejected lazy import.
- Ignoring failures in the monitoring code itself.

## Summary

- An Error Boundary replaces a failed descendant region with fallback UI for eligible render errors.
- `static getDerivedStateFromError` updates state for the fallback; `componentDidCatch` can record diagnostics.
- Boundaries do not catch ordinary event-handler, asynchronous callback, server-rendering, or self-errors.
- Use Suspense for pending supported resources and Error Boundaries for failures.
- Place boundaries around meaningful regions so unrelated interface remains available.
- Make recovery actions honest: reset or retry only when the application can change the condition that failed.
- Keep user-facing recovery text separate from private diagnostic details.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="067-lazy-loading-components.md">067 - Lazy Loading Components with lazy</a></td><td><a href="../qnas/068-error-boundaries-and-recovery-ui.md">Q&amp;A</a></td><td><a href="../examples/068-error-boundaries-and-recovery-ui.md">Example</a></td><td><a href="069-portals.md">069 - Portals and Rendering Outside the Parent DOM Node</a></td></tr>
  </tbody>
</table>
