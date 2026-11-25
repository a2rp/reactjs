# Async UI, Suspense, and Framework Data Loading — Example

This React 19 + Vite demo reads one shared Promise from two regions. Each region has a local loading fallback. A button forces the next request to fail so you can see the Error Boundary and retry path.

The small in-memory cache is for learning only. It does not provide production freshness, invalidation, cancellation, or user scoping.

## Create the project

    npm create vite@latest async-suspense-demo -- --template react
    cd async-suspense-demo
    npm install
    npm run dev

Replace the generated files with these.

## File structure

    src/
      App.jsx
      App.css
      ErrorBoundary.jsx
      data.js
      main.jsx

## src/data.js

    const report = {
      title: 'Weekly activity',
      totalVisits: 1842,
      change: '+8.4%',
      recentEvents: [
        { id: 1, label: 'New account created', time: '2 minutes ago' },
        { id: 2, label: 'Invoice downloaded', time: '11 minutes ago' },
        { id: 3, label: 'Profile updated', time: '24 minutes ago' },
      ],
    };

    let reportPromise = null;
    let failNextRequest = false;

    export function getReport() {
      if (reportPromise === null) {
        const shouldFail = failNextRequest;
        failNextRequest = false;

        const request = new Promise((resolve, reject) => {
          window.setTimeout(() => {
            if (shouldFail) {
              reject(new Error('The report could not be loaded.'));
              return;
            }

            resolve(report);
          }, 900);
        });

        let cachedRequest;
        cachedRequest = request.catch(error => {
          if (reportPromise === cachedRequest) {
            reportPromise = null;
          }

          throw error;
        });

        reportPromise = cachedRequest;
      }

      return reportPromise;
    }

    export function failNextReportRequest() {
      failNextRequest = true;
      reportPromise = null;
    }

Both regions receive the same Promise. If it rejects, the rejected Promise is removed so the retry can create a new request.

## src/ErrorBoundary.jsx

    import { Component } from 'react';

    export class ErrorBoundary extends Component {
      state = { error: null };

      static getDerivedStateFromError(error) {
        return { error };
      }

      render() {
        if (this.state.error) {
          return (
            <section className="error-panel" role="alert">
              <h2>Report unavailable</h2>
              <p>{this.state.error.message}</p>
              <button
                type="button"
                onClick={() => this.setState({ error: null })}
              >
                Try again
              </button>
            </section>
          );
        }

        return this.props.children;
      }
    }

The boundary catches render errors from its descendants. Its retry button clears the error; on the new render, the data module returns a fresh Promise after a failure.

## src/App.jsx

    import { Suspense, use, useState } from 'react';
    import { ErrorBoundary } from './ErrorBoundary.jsx';
    import { failNextReportRequest, getReport } from './data.js';
    import './App.css';

    function SummaryCard() {
      const report = use(getReport());

      return (
        <section className="card" aria-labelledby="summary-title">
          <p className="eyebrow">Overview</p>
          <h2 id="summary-title">{report.title}</h2>
          <p className="metric">{report.totalVisits.toLocaleString()}</p>
          <p className="positive">{report.change} compared with last week</p>
        </section>
      );
    }

    function RecentEvents() {
      const report = use(getReport());

      return (
        <section className="card" aria-labelledby="events-title">
          <p className="eyebrow">Latest updates</p>
          <h2 id="events-title">Recent activity</h2>
          <ul className="events">
            {report.recentEvents.map(event => (
              <li key={event.id}>
                <span>{event.label}</span>
                <time>{event.time}</time>
              </li>
            ))}
          </ul>
        </section>
      );
    }

    function LoadingCard({ label }) {
      return (
        <section className="card loading" role="status" aria-live="polite">
          <span className="spinner" aria-hidden="true" />
          <span>Loading {label}…</span>
        </section>
      );
    }

    export default function App() {
      const [attempt, setAttempt] = useState(0);

      function failNextRequest() {
        failNextReportRequest();
        setAttempt(current => current + 1);
      }

      return (
        <main className="page">
          <header className="page-header">
            <div>
              <p className="eyebrow">Suspense demonstration</p>
              <h1>Activity dashboard</h1>
              <p>Both regions read one shared asynchronous resource.</p>
            </div>

            <button
              className="secondary-button"
              type="button"
              onClick={failNextRequest}
            >
              Fail the next request
            </button>
          </header>

          <div className="grid" data-request-attempt={attempt}>
            <ErrorBoundary>
              <Suspense fallback={<LoadingCard label="summary" />}>
                <SummaryCard />
              </Suspense>
            </ErrorBoundary>

            <ErrorBoundary>
              <Suspense fallback={<LoadingCard label="activity" />}>
                <RecentEvents />
              </Suspense>
            </ErrorBoundary>
          </div>

          <p className="note">
            Reload the page to reset the in-memory cache and failure flag.
          </p>
        </main>
      );
    }

Clicking the button clears the cached successful value and schedules a re-render. Both panels suspend on the new request. It rejects once, each Error Boundary displays its retry action, and a retry succeeds because the one-shot failure has been consumed and the rejected Promise was removed.

## src/App.css

    :root {
      font-family: Inter, ui-sans-serif, system-ui, sans-serif;
      color: #e8edf5;
      background: #0d1117;
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
      border: 0;
      border-radius: 0.65rem;
      padding: 0.75rem 1rem;
      color: #08111f;
      background: #a5d6ff;
      font: inherit;
      font-weight: 700;
      cursor: pointer;
    }

    button:hover {
      background: #c2e3ff;
    }

    button:focus-visible {
      outline: 3px solid #f0b429;
      outline-offset: 3px;
    }

    .page {
      width: min(100% - 2rem, 960px);
      margin: 0 auto;
      padding: 4rem 0;
    }

    .page-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 2rem;
      margin-bottom: 2rem;
    }

    .page-header h1 {
      margin: 0.25rem 0 0.5rem;
      font-size: clamp(2rem, 5vw, 3rem);
    }

    .page-header p {
      margin: 0;
      color: #aebbd0;
      line-height: 1.6;
    }

    .eyebrow {
      color: #8bceff !important;
      font-size: 0.8rem;
      font-weight: 700;
      letter-spacing: 0.1em;
      text-transform: uppercase;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 1rem;
    }

    .card {
      min-height: 220px;
      border: 1px solid #303b4b;
      border-radius: 1rem;
      padding: 1.5rem;
      background: #151c27;
    }

    .card h2 {
      margin: 0.5rem 0 1rem;
      font-size: 1.2rem;
    }

    .metric {
      margin: 0;
      font-size: 3rem;
      font-weight: 800;
      letter-spacing: -0.04em;
    }

    .positive {
      color: #7ee0ae;
    }

    .events {
      display: grid;
      gap: 1rem;
      margin: 1.25rem 0 0;
      padding: 0;
      list-style: none;
    }

    .events li {
      display: flex;
      justify-content: space-between;
      gap: 1rem;
      border-top: 1px solid #303b4b;
      padding-top: 0.75rem;
    }

    .events time,
    .note {
      color: #aebbd0;
      font-size: 0.9rem;
    }

    .loading {
      display: flex;
      align-items: center;
      gap: 0.75rem;
      color: #aebbd0;
    }

    .spinner {
      width: 1rem;
      height: 1rem;
      border: 2px solid #536174;
      border-top-color: #a5d6ff;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
    }

    .error-panel {
      min-height: 220px;
      border: 1px solid #a64545;
      border-radius: 1rem;
      padding: 1.5rem;
      background: #291719;
    }

    .error-panel p {
      color: #ffc5c5;
    }

    .secondary-button {
      color: #e8edf5;
      background: #273449;
    }

    .secondary-button:hover {
      background: #35465f;
    }

    .note {
      margin-top: 1.5rem;
      line-height: 1.5;
    }

    @keyframes spin {
      to {
        transform: rotate(360deg);
      }
    }

    @media (prefers-reduced-motion: reduce) {
      *,
      *::before,
      *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
      }
    }

    @media (max-width: 680px) {
      .page {
        padding: 2rem 0;
      }

      .page-header {
        align-items: flex-start;
        flex-direction: column;
      }

      .grid {
        grid-template-columns: 1fr;
      }
    }

## src/main.jsx

    import { StrictMode } from 'react';
    import { createRoot } from 'react-dom/client';
    import App from './App.jsx';

    createRoot(document.getElementById('root')).render(
      <StrictMode>
        <App />
      </StrictMode>,
    );

## Observe the behavior

1. Initially, both Suspense fallbacks appear while the shared request is pending.
2. The request resolves and both regions render from the shared result.
3. Press “Fail the next request.” Both regions suspend again, then their Error Boundaries show an error state.
4. Press “Try again” in either panel. That panel requests a fresh Promise and succeeds.
5. Reload to reset the module-level cache and failure flag.

## What this demonstrates

- Suspense boundaries control fallback UI for supported pending resources.
- Both panels reuse one Promise rather than starting duplicate requests.
- Error Boundaries handle rejected Promise reads.
- A retry must create a new request instead of reusing a rejected Promise.
- The sample cache intentionally omits production concerns such as freshness, cancellation, invalidation, and user isolation.
