# Code Splitting and Asset Loading — Complete Example

This example uses Vite and React to defer an optional Reports panel and a CSV exporter until they are used. It also shows a useful Suspense fallback, an Error Boundary for a failed component chunk, image sizing, and a production-build inspection workflow.

The code-splitting feature is runnable without adding image assets. The image section is an optional snippet; add a real image at the shown path before using it.

## 1. Install the list of files

In an existing Vite React project, keep the existing React setup and package scripts. This example uses React and Vite's normal production build.

The feature code will be placed in separate modules:

    src/App.jsx
    src/App.css
    src/ReportsPanel.jsx
    src/ReportsPanel.css
    src/report-exporter.js

## 2. Create src/App.jsx

    import { Component, lazy, Suspense, useState } from 'react';
    import './App.css';

    const ReportsPanel = lazy(() => import('./ReportsPanel.jsx'));

    class FeatureErrorBoundary extends Component {
      state = { failed: false };

      static getDerivedStateFromError() {
        return { failed: true };
      }

      render() {
        if (this.state.failed) {
          return (
            <section className="feature-error" role="alert">
              <h2>Reports could not be loaded</h2>
              <p>
                Check your connection. If this page was open during a new
                deployment, reload to get the current application files.
              </p>
              <button
                type="button"
                onClick={() => window.location.reload()}
              >
                Reload application
              </button>
            </section>
          );
        }

        return this.props.children;
      }
    }

    function ReportsLoading() {
      return (
        <div className="reports-loading" role="status" aria-live="polite">
          <span className="loading-indicator" aria-hidden="true" />
          <span>Loading reports…</span>
        </div>
      );
    }

    export default function App() {
      const [showReports, setShowReports] = useState(false);

      return (
        <main className="app-shell">
          <header className="app-header">
            <div>
              <p className="eyebrow">Workspace</p>
              <h1>Project dashboard</h1>
              <p>Open the reports feature only when you need it.</p>
            </div>

            <button
              type="button"
              aria-expanded={showReports}
              aria-controls="reports-region"
              onClick={() => setShowReports((open) => !open)}
            >
              {showReports ? 'Close reports' : 'Open reports'}
            </button>
          </header>

          <section className="dashboard-card" aria-labelledby="overview-title">
            <h2 id="overview-title">Overview</h2>
            <p>
              This dashboard content is statically imported and remains
              available while the optional reports module loads.
            </p>
          </section>

          {showReports && (
            <section id="reports-region" aria-label="Reports">
              <FeatureErrorBoundary>
                <Suspense fallback={<ReportsLoading />}>
                  <ReportsPanel />
                </Suspense>
              </FeatureErrorBoundary>
            </section>
          )}
        </main>
      );
    }

ReportsPanel is declared with lazy at module scope. The reports import starts when the panel is first rendered. Suspense handles its pending state; FeatureErrorBoundary handles a failed import.

## 3. Create src/ReportsPanel.jsx

    import { useEffect, useState } from 'react';
    import './ReportsPanel.css';

    const reports = [
      { id: 'r-101', name: 'Accessibility review', owner: 'Mina', status: 'Ready' },
      { id: 'r-102', name: 'Release checklist', owner: 'Dev', status: 'In progress' },
      { id: 'r-103', name: 'Usage summary', owner: 'Sam', status: 'Ready' },
      { id: 'r-104', name: 'Support themes', owner: 'Ravi', status: 'Draft' },
    ];

    export default function ReportsPanel() {
      const [message, setMessage] = useState('');
      const [downloadUrl, setDownloadUrl] = useState('');

      useEffect(() => {
        return () => {
          if (downloadUrl) {
            URL.revokeObjectURL(downloadUrl);
          }
        };
      }, [downloadUrl]);

      async function prepareCsv() {
        setMessage('Preparing the CSV exporter…');

        try {
          const { createCsvDownloadUrl } =
            await import('./report-exporter.js');
          const nextUrl = createCsvDownloadUrl(reports);

          setDownloadUrl(nextUrl);
          setMessage('The report is ready to download.');
        } catch {
          setMessage('The exporter could not load. Try again.');
        }
      }

      return (
        <section className="reports-panel" aria-labelledby="reports-title">
          <div className="reports-heading">
            <div>
              <p className="eyebrow">Optional feature</p>
              <h2 id="reports-title">Reports</h2>
            </div>

            <button type="button" onClick={prepareCsv}>
              Prepare CSV
            </button>
          </div>

          <ul className="report-list">
            {reports.map((report) => (
              <li key={report.id} className="report-row">
                <strong>{report.name}</strong>
                <span>Owner: {report.owner}</span>
                <span>Status: {report.status}</span>
              </li>
            ))}
          </ul>

          <p role="status" aria-live="polite">{message}</p>

          {downloadUrl && (
            <a href={downloadUrl} download="reports.csv">
              Download CSV
            </a>
          )}
        </section>
      );
    }

The CSV module is not imported at the top of the file. It is requested only after the user selects Prepare CSV. After it loads, the app offers a normal download link, so the user explicitly starts the download.

## 4. Create src/report-exporter.js

    function escapeCsv(value) {
      return '"' + String(value).replaceAll('"', '""') + '"';
    }

    export function createCsvDownloadUrl(rows) {
      const header = ['Report', 'Owner', 'Status']
        .map(escapeCsv)
        .join(',');

      const body = rows.map((row) =>
        [row.name, row.owner, row.status]
          .map(escapeCsv)
          .join(',')
      );

      const csv = [header, ...body].join('
');
      const blob = new Blob([csv], {
        type: 'text/csv;charset=utf-8',
      });

      return URL.createObjectURL(blob);
    }

The browser-created object URL is revoked when the panel unmounts or a new URL replaces it. This avoids keeping generated downloads alive indefinitely.

## 5. Create src/App.css

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      color: #172033;
      background: #f4f6fa;
      font-family: system-ui, sans-serif;
    }

    button,
    a {
      font: inherit;
    }

    button {
      min-height: 2.6rem;
      padding: 0.55rem 0.9rem;
      border: 0;
      border-radius: 0.5rem;
      color: white;
      background: #2458c6;
      cursor: pointer;
    }

    button:focus-visible,
    a:focus-visible {
      outline: 3px solid #f2a900;
      outline-offset: 3px;
    }

    .app-shell {
      width: min(100% - 2rem, 68rem);
      margin-inline: auto;
      padding-block: 2rem;
    }

    .app-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 1.5rem;
      margin-bottom: 1.5rem;
    }

    .app-header h1,
    .app-header p {
      margin-block: 0.35rem;
    }

    .eyebrow {
      color: #53647e;
      font-size: 0.8rem;
      font-weight: 700;
      letter-spacing: 0.08em;
      text-transform: uppercase;
    }

    .dashboard-card,
    .feature-error {
      padding: 1.25rem;
      border: 1px solid #d9dfeb;
      border-radius: 0.75rem;
      background: white;
    }

    .dashboard-card h2 {
      margin-top: 0;
    }

    .reports-loading {
      display: flex;
      align-items: center;
      gap: 0.75rem;
      min-height: 10rem;
      margin-top: 1rem;
      padding: 1.25rem;
      border: 1px solid #d9dfeb;
      border-radius: 0.75rem;
      background: white;
    }

    .loading-indicator {
      width: 1rem;
      height: 1rem;
      border: 2px solid #c7d3ea;
      border-top-color: #2458c6;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
    }

    @keyframes spin {
      to {
        transform: rotate(360deg);
      }
    }

    .feature-error {
      margin-top: 1rem;
      border-color: #c84b4b;
    }

    @media (max-width: 36rem) {
      .app-header {
        align-items: flex-start;
        flex-direction: column;
      }
    }

## 6. Create src/ReportsPanel.css

    .reports-panel {
      margin-top: 1rem;
      padding: 1.25rem;
      border: 1px solid #d9dfeb;
      border-radius: 0.75rem;
      background: white;
    }

    .reports-heading {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 1rem;
    }

    .reports-heading h2 {
      margin-block: 0.25rem 0;
    }

    .report-list {
      display: grid;
      gap: 0.5rem;
      margin: 1rem 0;
      padding: 0;
      list-style: none;
    }

    .report-row {
      display: grid;
      grid-template-columns: minmax(0, 1fr) 8rem 10rem;
      gap: 0.75rem;
      padding: 0.85rem;
      border: 1px solid #e2e7f0;
      border-radius: 0.5rem;
    }

    .report-row > * {
      min-width: 0;
      overflow-wrap: anywhere;
    }

    @media (max-width: 38rem) {
      .report-row {
        grid-template-columns: minmax(0, 1fr);
      }
    }

The CSS file is imported by the lazy ReportsPanel module. Vite can associate feature styles with the asynchronous module in a production build. Keep the fallback styled from App.css so it is available before ReportsPanel.css loads.

## 7. Optional image-loading example

To try Vite's source asset handling, add a real image at src/assets/product-cover.webp and import it from a component:

    import productCover from './assets/product-cover.webp';

    <img
      src={productCover}
      width="960"
      height="540"
      alt="A project report cover"
      fetchPriority="high"
    />

For an image far below the first screen, use loading="lazy" instead of fetchPriority="high". Do not use both priorities for the same image. The intrinsic dimensions should match the source aspect ratio.

For a public file, place an image at public/images/gallery-cover.webp and use:

    <img
      src="/images/gallery-cover.webp"
      width="960"
      height="540"
      loading="lazy"
      alt="A gallery cover"
    />

The public file keeps its specified path and is copied as-is. Imported assets are resolved through the build's module graph.

## 8. Build and inspect

Use the project's production build and preview scripts:

    npm run build
    npm run preview

Open the preview in a browser with the Network panel visible:

1. Reload the page with the network log cleared.
2. Confirm that the dashboard appears without requesting the ReportsPanel chunk.
3. Select Open reports and observe the lazy component chunk and its CSS load.
4. Select Prepare CSV and observe the exporter chunk load only after that action.
5. Select Download CSV and confirm the generated file contains the report rows.
6. Resize to a narrow viewport and check that report rows remain readable.
7. Simulate a failed chunk request and confirm the error state is understandable.

Inspect the build output for entry files, asynchronous JavaScript, CSS, fonts, and images. Compare transferred sizes and use browser Performance tools to look at parse and execution work. For dependency-level reports, choose a bundle analyzer compatible with the project's current bundler version.

## 9. What this example demonstrates

- lazy and dynamic import create useful feature loading boundaries in a supported production build.
- Suspense provides the pending UI, while an Error Boundary handles a rejected component import.
- A second dynamic import delays the CSV exporter until the user asks to prepare a download.
- Feature-specific CSS can be split with an async module by the build tool.
- The first screen remains usable while optional code loads.
- Production Network and Performance tools show what the visitor actually downloads and executes.

## Key takeaway

Split around user-visible features, not around arbitrary files. Keep critical assets prompt, make pending and error states clear, and verify the production build rather than assuming a source-level import changed the user experience.
