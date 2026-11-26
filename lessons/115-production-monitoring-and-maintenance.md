# Production Errors, Monitoring, and Maintenance

A React application is not finished when it works on a developer's laptop. Production users can encounter a failed request, a browser-specific issue, a stale deployment asset, or a bug that only appears with a particular set of data. This lesson explains how to give users a useful recovery path, capture enough safe diagnostic information to investigate, and keep a deployed application healthy over time.

## Learning goals

After this lesson, you should be able to:

- distinguish rendering errors from errors in event handlers and asynchronous work;
- add an Error Boundary around an appropriate part of a React interface;
- represent expected request failures as ordinary interface state;
- report useful, privacy-conscious diagnostics;
- connect client errors to releases and source maps;
- choose basic signals and alerts for an application; and
- plan dependency updates, verification, and rollback.

## 1. Production errors are different from development errors

During development, an error overlay and a detailed stack trace help locate a bug. A visitor needs different things:

1. A clear explanation that the current action did not succeed.
2. A safe way to try again, return, or continue using the rest of the application.
3. A support path when recovery does not work.

The developer needs diagnostic context too. Production JavaScript is usually bundled and minified, so a stack trace alone may be hard to understand. A useful system connects the error to the application release, route, browser, relevant interaction history, and (when available) the React component stack.

Do not show stack traces, internal paths, request headers, database messages, or raw server responses in the visitor-facing interface. These can expose implementation details or private data.

## 2. Use an Error Boundary for rendering failures

An Error Boundary is a React component that catches errors thrown while React renders its descendant tree and displays a fallback instead of allowing that part of the interface to disappear.

A boundary is useful around a page region, route, or complex widget. A page-level boundary can preserve the application shell; a smaller boundary can preserve the rest of the page when one widget fails. Choose a boundary size that gives users a sensible recovery option.

A class component can act as an Error Boundary by implementing `getDerivedStateFromError`, `componentDidCatch`, or both:

```jsx
import { Component } from "react";

export class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    // This method must stay pure. It prepares the next render.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Send diagnostics through a privacy-conscious reporting adapter.
    this.props.onError?.(error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <section role="alert">
          <h2>This section could not be loaded.</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </section>
      );
    }

    return this.props.children;
  }
}
```

Use it around a part of the interface:

```jsx
<ErrorBoundary fallback={<PageProblem onRetry={reloadPageData} />}>
  <AccountDashboard />
</ErrorBoundary>
```

The fallback should offer a real recovery action. Resetting a boundary only asks React to render its descendants again; if the underlying condition remains, the error will happen again. A retry should refresh or reset the relevant data, route, or component state when appropriate. Avoid an automatic retry loop.

### What an Error Boundary does not catch

An Error Boundary does not replace normal error handling. It does not catch:

- errors in event handlers such as a click handler;
- errors from timers, animation callbacks, or other asynchronous callbacks;
- rejected Promises handled later outside the render process;
- errors thrown by the boundary itself; or
- errors during server rendering.

Catch errors at the source when possible. For example, a submit handler should catch its own rejected request and update the form's error state. A global browser event listener is only a last-resort signal; it cannot reliably provide the local context needed to recover the particular operation.

## 3. Handle expected failures where they happen

A network request can fail even when the JavaScript code is correct. The user may be offline, the server may return an error status, or the request may be cancelled because the user left the page. Treat these as explicit UI states.

The Fetch API rejects for some network failures, but an HTTP response such as 404 or 500 still resolves to a Response. Check `response.ok` before treating the body as successful data.

```jsx
async function loadProfile(signal) {
  const response = await fetch("/api/profile", { signal });

  if (!response.ok) {
    throw new Error("Profile request failed with status " + response.status);
  }

  return response.json();
}
```

Then catch the operation in the component or in the data-fetching layer:

```jsx
async function handleRefresh() {
  setStatus("loading");
  setMessage("");

  try {
    const profile = await loadProfile();
    setProfile(profile);
    setStatus("success");
  } catch (error) {
    setStatus("error");
    setMessage("Your profile could not be refreshed. Please try again.");
    reportClientError(error, { operation: "profile-refresh" });
  }
}
```

The visitor-facing message should be useful without exposing the raw exception. Keep details for diagnostics, and sanitize them before sending.

### Cancellation is usually not a production failure

If a request is cancelled because the user navigated away or because a newer request replaced it, that is often expected. With `AbortController`, a cancellation rejects with an `AbortError`. Do not show a scary error banner or count every cancellation as a service outage. Handle cancellation separately when the UI needs to distinguish it.

### Retry with care

A retry can help with a temporary network failure, but it can also repeat an action. Retry reads and other idempotent operations more safely than a payment, message send, or other mutation. For transient failures, use a small retry limit and increasing delay. Stop when the user cancels or leaves. Make a manual retry available when it gives the user control.

Do not retry authentication errors, validation errors, or every server error without considering what the status means. Do not automatically repeat a non-idempotent request unless the server supports an idempotency mechanism.

## 4. Browser-level handlers are a backstop

The browser emits different events for uncaught synchronous script errors and unhandled Promise rejections. They can help detect failures that escaped local handling, but they do not know what action the user should take and are not substitutes for Error Boundaries or `try`/`catch`.

```js
window.addEventListener("error", (event) => {
  reportClientError(event.error ?? new Error(event.message), {
    source: "window-error",
  });
});

window.addEventListener("unhandledrejection", (event) => {
  reportClientError(event.reason, {
    source: "unhandled-rejection",
  });
});
```

Install these listeners once near application startup, not on every render. Avoid calling `preventDefault()` unless you deliberately want to suppress the browser's default handling. Do not assume every cross-origin rejection will be visible to the page.

A listener can receive values that are not Error objects. Your reporting adapter should normalize unknown values carefully and avoid serializing an arbitrary object wholesale.

## 5. Send useful diagnostics without collecting secrets

An error report should help answer: what failed, in which release, for which part of the product, and how often?

A practical report can include:

- a normalized error name and message;
- a sanitized stack trace, when available;
- the Error Boundary component stack, when available;
- a release or build identifier;
- environment, such as production or staging;
- timestamp and route template (for example, `/users/:id`);
- browser and operating system versions when available;
- a non-identifying session or user reference, if your privacy policy permits it; and
- a short list of safe breadcrumbs, such as “opened settings” or “submitted profile form.”

Avoid sending:

- passwords, access tokens, cookies, or authorization headers;
- form contents, payment details, or private message text;
- full URLs containing user identifiers or query-string secrets;
- raw API response bodies; or
- arbitrary local storage values.

Prefer route templates over full paths. Review stack traces and breadcrumbs for data that could contain personal information. Follow your organization's retention, consent, and privacy requirements. Limit report size, deduplicate repeated events, and sample high-volume reports if needed. A client-side monitoring endpoint is public to the browser; never put a secret API key in frontend code.

A reporting adapter should have one small interface so the application can change monitoring providers without scattering provider calls throughout components:

```js
export function reportClientError(error, context = {}) {
  const normalized = normalizeError(error);

  const report = {
    name: normalized.name,
    message: sanitizeMessage(normalized.message),
    stack: sanitizeStack(normalized.stack),
    release: import.meta.env.VITE_APP_VERSION ?? "unknown",
    environment: import.meta.env.MODE,
    route: routeTemplate(window.location.pathname),
    source: context.source ?? "handled-error",
    operation: context.operation,
    occurredAt: new Date().toISOString(),
  };

  // Send only approved fields to your monitoring provider or same-origin
  // reporting endpoint. Add deduplication and consent policy here.
  monitoringClient.capture(report);
}
```

The functions above describe the boundary of the adapter; implement them according to the chosen provider and your privacy rules. Keep the provider's secret credentials on the server. A public project identifier or ingest endpoint is not a secret, but it still needs appropriate rate limiting and data controls.

## 6. Make production stack traces actionable

A production bundle may use short variable names and transformed line numbers. Source maps map generated files back to the source used to build them.

For dependable diagnosis:

1. Generate source maps for the production build if your monitoring workflow needs them.
2. Upload them to your monitoring service as part of the build or release process.
3. Associate each upload with the exact release identifier.
4. Restrict access to source maps if they expose source code or paths.
5. Verify that a real test error resolves to the expected source file.
6. Avoid accidentally publishing source maps to a public static directory when that is not intended.

A release identifier must be consistent across the frontend build and its uploaded source maps. If the deployed JavaScript and uploaded maps come from different builds, decoded stack traces can point to the wrong lines.

## 7. Monitor more than uncaught exceptions

Errors tell you that something failed. Other signals help you see whether the application is becoming slow or unavailable before users report it.

Useful signals include:

- client-side exceptions and rejected operations;
- API request failure rate, grouped by endpoint and status;
- error-free sessions or users;
- route and interaction latency;
- page-load performance and asset loading failures;
- backend availability, latency, saturation, and error rate; and
- deployment health after a release.

Choose alerts that lead to action. Set thresholds based on normal traffic, group duplicate events, and route alerts to a person or team that can respond. An alert without ownership or a response process creates noise.

Check for changes after deployment. A sudden increase in errors for one release, route, browser, or endpoint can reveal a regression more quickly than looking at a single error report.

## 8. Design recovery states

A complete data-driven screen should consider at least these states:

- **Loading:** show progress when the wait is noticeable.
- **Success:** render the data.
- **Empty:** explain that there is no data and offer the next useful action.
- **Error:** explain what failed in plain language and provide an appropriate recovery action.

Keep the rest of the application usable when one independent region fails. Preserve form input after a server error if it is safe to do so. Make retry controls keyboard-accessible and give changing status an appropriate accessible announcement.

Do not treat every failure as the same. A validation error belongs next to the field; an expired session may require signing in again; a temporary service error may allow retry; a missing resource may need a not-found page.

## 9. Maintain dependencies and deployments

Production reliability also depends on the software around the application.

- Keep the package lockfile under version control so installs resolve reproducibly.
- Review dependency updates and security advisories regularly.
- Run `npm audit` as one input to security review. Check the affected package, severity, reachable code, available fix, and potential compatibility impact.
- Apply updates in a controlled change and run the project's build and relevant checks afterward.
- Do not blindly use a force upgrade to resolve every audit finding. A breaking major update can introduce new failures.
- Remove unused dependencies and keep supported versions of the runtime and build tools.
- Test in the browsers and devices that your users need.
- Use staged rollout or feature flags when they reduce the impact of a risky change.
- Keep a rollback path. Confirm that the previous application build and compatible server behavior can be restored.
- Monitor old and new releases during rollout, and record what changed.

A lockfile makes dependency resolution repeatable; it does not guarantee every dependency is safe or correct. A successful build does not guarantee the deployed environment is configured correctly.

## 10. A practical maintenance routine

### For every release

- Confirm the intended commit and release identifier.
- Build the production bundle and inspect build errors and warnings.
- Verify configuration without exposing secrets.
- Upload source maps if the monitoring workflow uses them.
- Smoke-test the main user journeys in the deployed environment.
- Check error rate, latency, and failed requests after rollout.
- Keep a clear rollback decision and owner.

### Regularly

- Review error trends and remove duplicate or obsolete alerts.
- Update dependencies in small, reviewable changes.
- Check supported browser behavior and accessibility of fallbacks.
- Verify that reports are still sanitized and retention settings are appropriate.
- Exercise recovery and rollback procedures.
- Record known issues and planned follow-up work.

## Common mistakes

- **Using an Error Boundary for every kind of failure:** it only covers errors in React's descendant render work; catch event and async failures at their source.
- **Showing `error.message` directly to visitors:** exceptions and server responses may expose internals or sensitive data.
- **Logging the entire request or application state:** these can contain credentials or personal information.
- **Treating HTTP 500 as a successful Fetch result:** check `response.ok`.
- **Counting cancellation as a service outage:** distinguish intentional aborts from failures.
- **Retrying every failed mutation:** repeated actions may charge, send, or create duplicate data.
- **Uploading source maps without matching the release:** mismatched maps decode stacks incorrectly.
- **Turning on alerts without an owner:** alerts need a response path and actionable threshold.
- **Using `npm audit fix --force` without review:** forced updates can make breaking changes.
- **Assuming a healthy local build proves production is healthy:** configuration, assets, APIs, and browser behavior differ after deployment.

## Key takeaways

- Place Error Boundaries where their fallback can preserve a useful part of the application.
- Handle expected request, event, and asynchronous errors where they occur.
- Check HTTP status, distinguish cancellation, and retry only when it is safe.
- Treat browser-level error events as a diagnostic backstop.
- Send normalized, sanitized reports tied to a release; never expose secrets in client code.
- Monitor failures, performance, releases, and backend health with clear ownership.
- Maintain dependencies deliberately and keep a verified rollback path.

## Next steps

Continue to the chapter Q&A to review common production error and monitoring decisions, then work through the example to put a boundary and explicit async error state into a small React interface.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="114-production-builds-and-deployment.md">114 - Production Builds, Configuration, and Deployment</a></td><td><a href="../qnas/115-production-monitoring-and-maintenance.md">Q&amp;A</a></td><td><a href="../examples/115-production-monitoring-and-maintenance.md">Example</a></td><td><a href="116-react-capstone-project.md">116 - Build and Ship a Complete Accessible React Application</a></td></tr>
  </tbody>
</table>
