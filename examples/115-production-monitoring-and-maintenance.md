# Example: Production Errors, Monitoring, and Maintenance

This small Vite + React example demonstrates two different failure paths:

1. A render error in one widget is caught by an Error Boundary, so the rest of the page remains available.
2. An asynchronous operation handles its own failure, shows a useful message, and sends a privacy-conscious diagnostic through one adapter.

The reporting adapter deliberately omits raw exception messages, stack traces, form contents, and full URLs. Without a configured endpoint it logs only its safe payload in development. In production, connect it to a same-origin server endpoint or replace the transport with a monitoring provider SDK that has been configured to scrub sensitive data.

## Suggested files

Add these files to a React application. The snippets assume a Vite project with the React plugin.

### `src/error-reporting.js`

```js
const endpoint = import.meta.env.VITE_ERROR_REPORT_ENDPOINT;

const allowedSources = new Set([
  "react-boundary",
  "handled-request",
  "window-error",
  "unhandled-rejection",
]);

const allowedCategories = new Set([
  "widget-render",
  "profile-refresh",
  "uncaught",
]);

const recentlySent = new Map();

function safeErrorName(value) {
  if (!value || typeof value !== "object" || typeof value.name !== "string") {
    return "UnknownError";
  }

  return value.name.replace(/[^a-zA-Z0-9_-]/g, "").slice(0, 60) || "UnknownError";
}

function safeTag(value, allowedValues) {
  return allowedValues.has(value) ? value : "other";
}

export function reportClientError(error, context = {}) {
  const source = safeTag(context.source, allowedSources);
  const category = safeTag(context.category, allowedCategories);
  const name = safeErrorName(error);
  const dedupeKey = [source, category, name].join(":");
  const now = Date.now();
  const lastSentAt = recentlySent.get(dedupeKey);

  // Avoid flooding the endpoint with the same error in a short interval.
  if (lastSentAt && now - lastSentAt < 60_000) {
    return;
  }
  recentlySent.set(dedupeKey, now);

  // Use a route template or another reviewed route label. Do not send the
  // complete URL, which can contain identifiers or private query values.
  const report = {
    name,
    source,
    category,
    release: import.meta.env.VITE_APP_VERSION || "local",
    environment: import.meta.env.MODE,
    route: "/",
    occurredAt: new Date(now).toISOString(),
  };

  if (!endpoint) {
    if (import.meta.env.DEV) {
      console.info("[safe error report preview]", report);
    }
    return;
  }

  // The endpoint is public to the browser. Keep its server-side rate limits
  // and data-retention rules in place. Never put a secret key in this bundle.
  void fetch(endpoint, {
    method: "POST",
    credentials: "same-origin",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(report),
  })
    .then((response) => {
      if (!response.ok) {
        throw new Error("The reporting endpoint returned an unsuccessful status.");
      }
    })
    .catch(() => {
      // Do not create another unhandled rejection while trying to report one.
      // A monitoring transport failure should not break the visitor's UI.
      if (import.meta.env.DEV) {
        console.warn("The safe error report could not be delivered.");
      }
    });
}
```

The allow-lists keep this example's event fields predictable. Update them only with static, reviewed labels. Do not pass user input, error text, or arbitrary application state as a tag. The one-minute deduplication is intentionally small; a real monitoring integration may provide grouping, sampling, and rate limits.

### `src/ErrorBoundary.jsx`

```jsx
import { Component } from "react";

export class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error) {
    // The reporter accepts the Error, but sends only its approved fields.
    this.props.onError?.(error);
  }

  reset = () => {
    this.setState({ hasError: false });
  };

  render() {
    if (this.state.hasError) {
      const fallback = this.props.fallback;

      if (typeof fallback === "function") {
        return fallback({ reset: this.reset });
      }

      return fallback ?? (
        <section role="alert">
          <h2>This section could not be displayed.</h2>
          <button onClick={this.reset}>Try again</button>
        </section>
      );
    }

    return this.props.children;
  }
}
```

The reset button gives React another render attempt. For real data, make the retry refresh the failed data or reset the underlying state too.

### `src/App.jsx`

```jsx
import { useEffect, useState } from "react";
import { ErrorBoundary } from "./ErrorBoundary.jsx";
import { reportClientError } from "./error-reporting.js";
import "./App.css";

function SectionFallback({ reset }) {
  return (
    <section className="notice notice-error" role="alert">
      <h2>The demo widget stopped working.</h2>
      <p>The rest of the page is still available. Reset the widget to try again.</p>
      <button onClick={reset}>Reset widget</button>
    </section>
  );
}

function CrashingWidget() {
  const [shouldCrash, setShouldCrash] = useState(false);

  if (shouldCrash) {
    // This render-time error is caught by the nearest ErrorBoundary.
    throw new Error("Widget render failed in this local demo.");
  }

  return (
    <div className="widget">
      <p>This widget is inside its own Error Boundary.</p>
      <button onClick={() => setShouldCrash(true)}>
        Simulate a render error
      </button>
    </div>
  );
}

function wait(milliseconds) {
  return new Promise((resolve) => window.setTimeout(resolve, milliseconds));
}

async function requestDemoProfile(shouldFail) {
  await wait(500);

  if (shouldFail) {
    throw new Error("The simulated profile request failed.");
  }

  return { displayName: "React learner" };
}

function ProfileRequestDemo() {
  const [status, setStatus] = useState("idle");
  const [profile, setProfile] = useState(null);

  async function refreshProfile(shouldFail) {
    setStatus("loading");

    try {
      const nextProfile = await requestDemoProfile(shouldFail);
      setProfile(nextProfile);
      setStatus("success");
    } catch (error) {
      setStatus("error");

      // This asynchronous error is handled here; an Error Boundary would
      // not catch a rejection from this operation.
      reportClientError(error, {
        source: "handled-request",
        category: "profile-refresh",
      });
    }
  }

  return (
    <section className="card" aria-labelledby="request-title">
      <h2 id="request-title">Handle an asynchronous failure</h2>
      <p>
        These buttons simulate a request so the example works without a backend.
      </p>

      <div className="actions">
        <button
          disabled={status === "loading"}
          onClick={() => refreshProfile(false)}
        >
          Simulate success
        </button>
        <button
          disabled={status === "loading"}
          onClick={() => refreshProfile(true)}
        >
          Simulate request failure
        </button>
      </div>

      <p className="status" aria-live="polite">
        {status === "loading" && "Loading profile…"}
        {status === "success" && "Profile loaded for " + profile.displayName + "."}
        {status === "error" && "The profile could not be refreshed. Please try again."}
        {status === "idle" && "Choose an action to start."}
      </p>
    </section>
  );
}

export default function App() {
  useEffect(() => {
    function handleWindowError(event) {
      reportClientError(event.error, {
        source: "window-error",
        category: "uncaught",
      });
    }

    function handleUnhandledRejection(event) {
      reportClientError(event.reason, {
        source: "unhandled-rejection",
        category: "uncaught",
      });
    }

    window.addEventListener("error", handleWindowError);
    window.addEventListener("unhandledrejection", handleUnhandledRejection);

    return () => {
      window.removeEventListener("error", handleWindowError);
      window.removeEventListener("unhandledrejection", handleUnhandledRejection);
    };
  }, []);

  return (
    <main className="app">
      <header className="intro">
        <p className="eyebrow">React production patterns</p>
        <h1>Errors with a recovery path</h1>
        <p>
          Try the render error and the request failure. Each is handled at the
          layer that has the right context.
        </p>
      </header>

      <section className="card" aria-labelledby="boundary-title">
        <h2 id="boundary-title">Keep a render failure inside its section</h2>
        <ErrorBoundary
          onError={(error) =>
            reportClientError(error, {
              source: "react-boundary",
              category: "widget-render",
            })
          }
          fallback={({ reset }) => <SectionFallback reset={reset} />}
        >
          <CrashingWidget />
        </ErrorBoundary>
      </section>

      <ProfileRequestDemo />

      <footer className="footnote">
        <p>
          The demo reporter sends only an error name, reviewed labels, release,
          environment, a fixed route label, and a timestamp.
        </p>
      </footer>
    </main>
  );
}
```

### `src/App.css`

```css
:root {
  color: #172033;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button {
  min-height: 2.75rem;
  padding: 0.65rem 1rem;
  border: 0;
  border-radius: 0.5rem;
  color: white;
  background: #234db4;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button:hover {
  background: #173a91;
}

button:focus-visible {
  outline: 3px solid #ed9d24;
  outline-offset: 3px;
}

button:disabled {
  cursor: wait;
  opacity: 0.6;
}

.app {
  width: min(100% - 2rem, 48rem);
  margin: 0 auto;
  padding: 3rem 0;
}

.intro,
.card,
.footnote {
  margin-bottom: 1rem;
}

.intro h1 {
  margin: 0.25rem 0 0.75rem;
  font-size: clamp(2rem, 6vw, 3rem);
  line-height: 1.1;
}

.intro > p:last-child,
.card p,
.footnote {
  line-height: 1.6;
}

.eyebrow {
  color: #234db4;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.card,
.notice {
  padding: 1.25rem;
  border: 1px solid #d6deeb;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 0.25rem 1rem rgb(24 44 82 / 6%);
}

.card h2,
.notice h2 {
  margin-top: 0;
  font-size: 1.2rem;
}

.widget {
  padding-top: 0.25rem;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin: 1rem 0;
}

.status {
  min-height: 1.6em;
}

.notice-error {
  border-color: #cf6671;
  background: #fff7f7;
}

.footnote {
  color: #4a566d;
  font-size: 0.9rem;
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### `src/main.jsx`

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

## Configure a real reporting destination

Create an environment file for the local project when a backend endpoint is available:

```text
VITE_ERROR_REPORT_ENDPOINT=/api/client-errors
VITE_APP_VERSION=local
```

Vite exposes `VITE_*` values to browser code, so use these only for public configuration. The endpoint above must be implemented by the application's backend or provided by a monitoring integration. The backend should validate the payload, enforce rate limits, apply retention rules, and avoid trusting client-supplied identity or release data without validation.

For a production build, set `VITE_APP_VERSION` to the same release identifier used for deployment and source-map upload. Do not set a secret in a `VITE_*` variable.

## Try the example

1. Start the React application and open the page.
2. Select **Simulate a render error**. The widget fallback appears, while the request demo remains usable.
3. Select **Reset widget**. The boundary renders its child again.
4. Select **Simulate request failure**. The request demo shows a safe message and calls the reporting adapter from its `catch` block.
5. Select **Simulate success**. The request demo returns to a successful state.
6. With no endpoint configured in development, inspect the browser console for the sanitized report preview.
7. With a reviewed same-origin endpoint configured, verify that it accepts only the expected fields and does not store private data.

## Replace the demo request with Fetch

The example uses a deterministic Promise so it can run without a server. In an application with an API, replace `requestDemoProfile` with a Fetch operation that checks the HTTP status:

```js
async function requestProfile(signal) {
  const response = await fetch("/api/profile", { signal });

  if (!response.ok) {
    throw new Error("Profile request failed with status " + response.status);
  }

  return response.json();
}
```

Keep the request in the same local `try`/`catch` flow. If you use `AbortController`, handle an expected `AbortError` separately instead of showing it as a service failure.

## Important production notes

- The example's reporter sends no raw exception message or stack. A real provider may capture more detail; configure and verify its privacy scrubbing first.
- The same-origin endpoint is a placeholder contract, not a server implementation.
- The boundary is deliberately limited to the widget. Add boundaries at other meaningful recovery points rather than wrapping every tiny component.
- The global browser listeners are backstops. Keep local handling for requests, event handlers, and other expected failures.
- A retry should repeat the underlying operation or reset its state. Clearing the boundary alone may simply reproduce the same failure.
- Never include passwords, access tokens, cookies, form contents, or arbitrary application state in an error report.
