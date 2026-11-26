# Q&A: Production Errors, Monitoring, and Maintenance

## 1. What should a production error strategy accomplish?

It should help a visitor understand what happened and recover, while giving the team enough safe information to diagnose the problem. That usually means a useful fallback or error state, a recovery action, sanitized diagnostics, and monitoring tied to the deployed release.

## 2. What is an Error Boundary?

An Error Boundary is a React component that catches errors thrown while rendering its descendant tree and displays a fallback interface. It can keep one failed page region from taking down the rest of the React interface.

## 3. How does a class component become an Error Boundary?

It implements at least one of the lifecycle methods React uses for boundaries: `static getDerivedStateFromError(error)` to update state for the fallback, and/or `componentDidCatch(error, info)` to report the error and component stack. In practice, implementations commonly use both.

## 4. Why should `getDerivedStateFromError` stay pure?

React uses it to calculate the next state from an error. It should return state for rendering the fallback and avoid side effects. Do reporting or other side effects in `componentDidCatch`.

## 5. Do Error Boundaries catch click-handler errors?

No. An error thrown by an event handler occurs outside React's descendant rendering work. Handle it in the handler itself, update the relevant UI state, and report it if it is unexpected.

## 6. Do Error Boundaries catch rejected Promises and timer errors?

Generally, no. A rejection from asynchronous work or an error thrown later by a timer is not caught by a boundary just because it was started by a component. Use `try`/`catch` around awaited work, handle a Promise's rejection, or use the API's supported error mechanism.

## 7. Do Error Boundaries catch server-rendering errors?

No. Error Boundaries handle errors in the client-side React tree; server rendering has its own error handling and reporting path.

## 8. What errors can escape an Error Boundary?

Errors thrown by the boundary itself cannot be caught by that same boundary. Errors in event handlers, later asynchronous callbacks, and server rendering also require other handling. A higher boundary may catch an error thrown while rendering a lower boundary's fallback.

## 9. Is a global `window.error` listener a replacement for an Error Boundary?

No. The global listener is a diagnostic backstop for certain uncaught browser errors. It does not know the right local recovery action and does not catch all categories of failures.

## 10. Which browser event is used for an unhandled Promise rejection?

The browser dispatches `unhandledrejection` on the relevant global object when a Promise rejection has no rejection handler. It can help reveal missed handling, but expected failures should be caught at the operation that can explain or recover from them.

## 11. Why can a cross-origin rejection be missing from global reports?

Browsers may suppress cross-origin script rejection details to avoid exposing information across origins. Global listeners also depend on the failure reaching that page's global scope. They are useful as a backstop, not a complete audit trail.

## 12. Does Fetch reject when the server returns HTTP 404 or 500?

No. Fetch normally resolves to a `Response` for HTTP error statuses. Check `response.ok` or `response.status` and handle unsuccessful responses explicitly. Network failures can reject the Fetch Promise.

## 13. Where should an expected request failure be handled?

In the request's caller or data layer, where the application knows what the user was trying to do. Set an appropriate error state, show a clear message, and provide a relevant action. A global handler lacks that context.

## 14. Should the application display `error.message` to the visitor?

Usually not. An exception message can contain internal details, paths, response text, or sensitive values. Show a safe, task-specific message to the user and keep sanitized diagnostics for the monitoring system.

## 15. Should every failed request be reported as an application bug?

No. Some failures are expected states: validation responses, missing data, unauthenticated access, rate limits, offline use, or a request cancelled because the user navigated away. Handle each according to its meaning. Report unexpected failures and service-level trends with appropriate grouping.

## 16. Is an `AbortError` always a problem?

No. It often means a request was intentionally cancelled, for example when a component unmounts or a newer request replaces an old one. Suppress visitor-facing failure for expected cancellation and avoid counting it as an outage. If cancellation was unexpected, investigate the call site.

## 17. When is retrying a request safe?

A bounded retry can help with transient failures, especially for reads or idempotent operations. Consider status and operation type, stop after a small number of attempts, use increasing delays, and allow cancellation. Do not blindly retry a mutation such as a payment or message send; use server-supported idempotency when appropriate.

## 18. What context makes a client error report useful?

A normalized error name and message, sanitized stack, React component stack when available, release identifier, environment, timestamp, route template, relevant browser information, and a small set of safe breadcrumbs. Include only context that is permitted by the product's privacy rules.

## 19. What should a client error report avoid?

Credentials, access tokens, cookies, authorization headers, form contents, payment details, private message text, raw API bodies, arbitrary local storage values, and full URLs that may include identifiers or secrets. Sanitize both stack traces and breadcrumbs.

## 20. Why use a route template instead of a full URL?

A full path or query string may contain a person's name, account identifier, search term, or token. A template such as `/users/:id` usually retains useful debugging context while reducing exposure of user-specific values.

## 21. Can a frontend application keep a monitoring secret key?

No. Anything bundled into browser JavaScript can be inspected by visitors. Keep secret credentials on the server. A provider may use a public project identifier or ingestion endpoint, but protect the endpoint with appropriate data controls and abuse prevention.

## 22. What are source maps for?

They map transformed or minified production code back to the original source. A monitoring tool can use matching maps to show a meaningful source file and line for a production stack trace.

## 23. Why must source maps match the release exactly?

A source map describes a specific generated bundle. If the map and deployed JavaScript came from different builds, the decoded location can be wrong. Associate both with the same release or build identifier.

## 24. Should source maps always be publicly served?

Not necessarily. They can reveal source code, paths, or other implementation details. Decide whether to upload maps privately to a monitoring provider or serve them publicly based on the application's security and debugging requirements.

## 25. What does `npm audit` tell a team?

It compares the dependency tree with known vulnerability advisories and reports findings and available remediation information. It is an input to review, not a guarantee that the application is secure or vulnerable in exactly the same way as every report suggests. Assess affected versions, severity, exposure, and fix compatibility.

## 26. Why avoid blindly running `npm audit fix --force`?

A forced fix may install a breaking major version or otherwise change dependency behavior. Review the proposed changes, update deliberately, and run the project's build and relevant checks before release.

## 27. Does a lockfile prove dependencies are safe?

No. A lockfile helps make dependency installation reproducible. It does not guarantee that every package is secure, maintained, or appropriate.

## 28. What should a useful Error Boundary fallback include?

A short, calm explanation and a recovery action that makes sense for the affected region, such as retrying a data load, returning to a safe route, or contacting support. It should be accessible, avoid stack traces, and preserve unrelated parts of the interface where possible.

## 29. Why might resetting a boundary fail to recover the page?

Resetting the boundary allows React to try rendering its descendants again. If the underlying bad data, failed request, or faulty component state has not changed, it may throw again. A retry needs to refresh or reset the underlying condition, not only clear the boundary's flag.

## 30. Which operational signals are useful besides JavaScript exceptions?

API failure rates, error-free sessions, route and interaction latency, page-load performance, asset loading failures, service availability, backend latency, saturation, and deployment health. Group and correlate signals by release, route, browser, and endpoint when that helps identify a regression.

## 31. What makes an alert actionable?

It has a meaningful threshold, enough context to assess impact, a clear owner, and a response path. Deduplication and grouping reduce noise. An alert that nobody owns or cannot lead to action should be revised.

## 32. What should be checked after a deployment?

Confirm the new release identifier, exercise the main user journeys, check required configuration and asset loading, verify that source maps match if used, and watch errors, latency, and failed requests. Keep a documented rollback path and know who can invoke it.

## 33. What are the four common states for a data-driven view?

Loading, success, empty, and error. An empty result is not necessarily a failure. Each state should explain what is happening and offer the next useful action when one exists.

## 34. How should validation errors differ from service errors?

Validation errors belong near the affected input and should explain what needs correction. A temporary service error should explain that the operation did not finish and offer a safe retry. An expired session may require signing in again. The message and action should match the cause.

## 35. What does maintenance look like between releases?

Review error trends and alerts, update dependencies in small reviewed changes, verify supported browser behavior, audit data collection and retention, exercise recovery and rollback, and record known issues with follow-up owners.

## 36. How do Error Boundaries and local error state complement each other?

Local state handles failures the application expects and understands, such as a request that can be retried or a field that needs correction. An Error Boundary catches unexpected rendering failures in its descendants and replaces the broken region with a safe fallback. Both are needed because they solve different problems.

## 37. What is the main principle of production error handling?

Handle a failure at the narrowest layer that has enough context to respond correctly. Use boundaries and global listeners as safeguards, keep user messages safe and useful, and make diagnostic reporting privacy-conscious and tied to a release.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="114-production-builds-and-deployment.md">114 - Production Builds, Configuration, and Deployment - Q&amp;A</a></td><td><a href="../lessons/115-production-monitoring-and-maintenance.md">Lesson</a></td><td><a href="../examples/115-production-monitoring-and-maintenance.md">Example</a></td><td><a href="116-react-capstone-project.md">116 - Q&amp;A: Build and Ship a Complete Accessible React Application</a></td></tr>
  </tbody>
</table>
