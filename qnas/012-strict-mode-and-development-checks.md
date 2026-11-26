# 012. Strict Mode and development checks - Q&A

## Enabling Strict Mode

### 1. What is Strict Mode?

Strict Mode is a React development aid that enables extra checks on a component tree. It can reveal impure rendering, missing cleanup, and deprecated API usage earlier.

### 2. How do I enable Strict Mode for an app?

Import StrictMode from React and wrap the app component when rendering it with the root:

~~~jsx
import { StrictMode } from 'react';

root.render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

### 3. Does Strict Mode add a DOM element?

No. StrictMode is a React component that controls development checks; it does not create an extra browser DOM wrapper.

### 4. Does StrictMode accept props?

No. StrictMode does not accept props.

### 5. Should a new app use Strict Mode?

React recommends enabling it for new apps. It helps reveal problems during development before they are harder to reproduce.

### 6. Can I enable it for only part of the component tree?

Yes. Wrap a subtree to apply the checks to that subtree and its descendants. Components outside the boundary do not receive those checks.

## Extra render calls

### 7. Why does Strict Mode call rendering logic an extra time?

React expects rendering and certain related functions to be pure. An extra call can expose code that mutates inputs or causes a side effect while calculating the UI.

### 8. What does it mean for rendering to be pure?

Given the same props, state, and context, rendering should calculate the same UI without changing external values or performing actions. For example, do not append to an array received through props.

### 9. Does Strict Mode call every function twice?

No. It calls functions that should be pure for additional development checks, including component render logic and certain initializer or updater functions. Event handlers are not called extra just because Strict Mode is enabled.

### 10. Why do I see a console message twice?

The message may be inside a component's render logic, which Strict Mode may call an extra time in development. Check where it is logged and keep render logic free of side effects.

### 11. Does an extra render mean the user sees duplicate UI?

No. The extra call is a development check to reveal impure logic. It does not mean the intended production UI should be duplicated.

## Effects and refs

### 12. Why does an Effect appear to start twice in development?

When Strict Mode wraps the root, React runs an extra Effect setup-and-cleanup cycle on initial mount in development. The sequence checks that setup can be undone before it runs again.

### 13. What should I do when an Effect starts a connection or subscription?

Return cleanup from the Effect that disconnects or unsubscribes. After the development setup-cleanup-setup sequence, only the current connection or subscription should remain active.

### 14. Does Strict Mode run every Effect twice on every update?

No. The extra initial setup-and-cleanup cycle checks mount behavior in development. Normal Effect updates still follow their dependencies and cleanup rules. The complete Effect lifecycle is covered later.

### 15. Why might a callback ref appear to run more than once?

Strict Mode performs an extra callback-ref setup and cleanup check in development. Make sure the callback removes or releases any node it registered when cleanup occurs.

### 16. What if Strict Mode wraps only a nested subtree?

Nested Strict Mode checks the subtree. React avoids an initial Effect replay that would re-run only child Effects without their parents, since that ordering cannot happen in production.

## Diagnosing and fixing issues

### 17. Should I remove Strict Mode when it reveals duplicate behavior?

Usually no. Removing it can hide the diagnostic while leaving the impurity or missing cleanup in the code. Fix the underlying problem.

### 18. Should an Effect submit a form or purchase because a component mounted?

A user-triggered action should happen in the event handler for that action. Effects are for synchronizing with external systems based on the component being present or its inputs changing.

### 19. Does Strict Mode change production behavior?

The extra checks are development-only and do not impact the production build. Strict Mode does not promise that all bugs will be found.

### 20. Can Strict Mode fix a deprecated API automatically?

No. It can report diagnostics for deprecated API usage. Update the code or dependency that causes the warning.

### 21. Does Strict Mode replace testing?

No. It checks for a focused set of common problems during development. Continue to use tests, debugging, and code review.

### 22. How should I investigate a Strict Mode warning?

Read the warning, find the component or dependency that triggered it, and correct the underlying behavior. Check for data mutation during rendering, missing Effect or ref cleanup, and deprecated APIs.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="011-create-a-root-and-render-an-application.md">011 - Create a root and render a React application - Q&amp;A</a></td><td><a href="../lessons/012-strict-mode-and-development-checks.md">Lesson</a></td><td><a href="../examples/012-strict-mode-and-development-checks.md">Example</a></td><td><a href="013-function-components-and-naming.md">013 - Function components and component naming - Q&amp;A</a></td></tr>
  </tbody>
</table>
