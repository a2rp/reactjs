# Build and Deploy a Vite React App - Complete Example

This example configures an existing Vite React app for a static host that publishes the app below the path /react-production-demo/. It includes:

- A Vite base path for a repository subdirectory.
- Public build-time configuration.
- A React page that displays the active build mode and paths.
- A public SVG asset to verify base-path handling.
- Commands to create and preview the production artifact.
- A deployment checklist for a static host.

The example uses public configuration only. It contains no secrets and does not configure a production server.

## 1. Start with an existing Vite React project

Keep the React and Vite dependencies created by your scaffold. Ensure the package scripts include:

    {
      "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview"
      }
    }

Keep the package lockfile in version control. This example assumes npm.

## 2. Configure the public base path

Create or update vite.config.js:

    import { defineConfig } from 'vite';
    import react from '@vitejs/plugin-react';

    export default defineConfig({
      plugins: [react()],
      base: '/react-production-demo/',
    });

Replace react-production-demo with the exact path where the host publishes the app. For a site served from the domain root, use the default base path / instead.

The base path is part of the production build. If the app is deployed under a different path later, rebuild it with the matching base.

## 3. Add public build configuration

Create .env.production.example:

    VITE_PUBLIC_APP_NAME=React production demo
    VITE_PUBLIC_API_BASE=/api

For a local production-mode build, copy this example to .env.production and change only the public values needed for the deployment. The production build reads .env.production. A hosting pipeline can provide the same values through its build environment.

Anything with the VITE_ prefix is exposed in the browser bundle. Do not place private API keys, database passwords, signing keys, or user credentials in this file or in CI variables that Vite exposes to the client.

## 4. Create src/config.js

    const appName =
      import.meta.env.VITE_PUBLIC_APP_NAME || 'React production demo';

    const apiBase =
      import.meta.env.VITE_PUBLIC_API_BASE || '/api';

    export const appConfig = Object.freeze({
      appName,
      apiBase,
      basePath: import.meta.env.BASE_URL,
      mode: import.meta.env.MODE,
      isProduction: import.meta.env.PROD,
    });

These values are compiled into the browser application. They are useful for public settings such as an API origin, display name, or feature label, but they are not a place for secrets.

## 5. Add a public asset

Create public/react-mark.svg:

    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 96 96" role="img" aria-labelledby="title">
      <title id="title">React deployment demo</title>
      <rect width="96" height="96" rx="20" fill="#172554"/>
      <circle cx="48" cy="48" r="7" fill="#61dafb"/>
      <g fill="none" stroke="#61dafb" stroke-width="4">
        <ellipse cx="48" cy="48" rx="34" ry="13"/>
        <ellipse cx="48" cy="48" rx="34" ry="13" transform="rotate(60 48 48)"/>
        <ellipse cx="48" cy="48" rx="34" ry="13" transform="rotate(120 48 48)"/>
      </g>
    </svg>

Files in public are copied as-is. Reference this asset with the configured base path so it works both at the domain root and under the repository subdirectory.

## 6. Create src/App.jsx

    import './App.css';
    import { appConfig } from './config.js';

    const logoUrl = import.meta.env.BASE_URL + 'react-mark.svg';

    export default function App() {
      return (
        <main className="page-shell">
          <header className="hero">
            <img
              className="logo"
              src={logoUrl}
              width="96"
              height="96"
              alt=""
            />
            <div>
              <p className="eyebrow">Production deployment example</p>
              <h1>{appConfig.appName}</h1>
              <p>
                This page shows the public configuration embedded in this
                build.
              </p>
            </div>
          </header>

          <section className="panel" aria-labelledby="build-title">
            <h2 id="build-title">Build details</h2>
            <dl className="details">
              <div>
                <dt>Mode</dt>
                <dd>{appConfig.mode}</dd>
              </div>
              <div>
                <dt>Production build</dt>
                <dd>{appConfig.isProduction ? 'Yes' : 'No'}</dd>
              </div>
              <div>
                <dt>Public base path</dt>
                <dd>{appConfig.basePath}</dd>
              </div>
              <div>
                <dt>Public API base</dt>
                <dd>{appConfig.apiBase}</dd>
              </div>
            </dl>
          </section>

          <section className="panel" aria-labelledby="check-title">
            <h2 id="check-title">Deployment checks</h2>
            <ul>
              <li>The logo loads from the configured base path.</li>
              <li>The page shows the mode embedded during this build.</li>
              <li>The API path is public configuration, not a credential.</li>
            </ul>
            <a href={appConfig.basePath}>Open the app base path</a>
          </section>
        </main>
      );
    }

The image path uses Vite's BASE_URL. A leading root path such as /react-mark.svg would bypass the repository subdirectory and fail on many project pages.

## 7. Create src/App.css

    * {
      box-sizing: border-box;
    }

    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172033;
      background: #f2f5fa;
      font-synthesis: none;
      text-rendering: optimizeLegibility;
    }

    body {
      min-width: 20rem;
      margin: 0;
    }

    a {
      color: #174ea6;
      font-weight: 650;
      overflow-wrap: anywhere;
    }

    a:focus-visible {
      outline: 3px solid #f1a900;
      outline-offset: 3px;
    }

    .page-shell {
      width: min(100% - 2rem, 52rem);
      margin-inline: auto;
      padding-block: 3rem;
    }

    .hero {
      display: flex;
      align-items: center;
      gap: 1.25rem;
      margin-bottom: 1.5rem;
    }

    .hero h1 {
      margin-block: 0.25rem;
    }

    .logo {
      flex: 0 0 auto;
      width: 6rem;
      height: 6rem;
    }

    .eyebrow {
      margin: 0;
      color: #4b5c78;
      font-size: 0.8rem;
      font-weight: 700;
      letter-spacing: 0.08em;
      text-transform: uppercase;
    }

    .panel {
      margin-block: 1rem;
      padding: 1.25rem;
      border: 1px solid #d5dce8;
      border-radius: 0.8rem;
      background: white;
      box-shadow: 0 0.4rem 1.5rem rgb(20 40 80 / 6%);
    }

    .panel h2 {
      margin-top: 0;
      font-size: 1.2rem;
    }

    .details {
      display: grid;
      gap: 0.8rem;
      margin-bottom: 0;
    }

    .details div {
      display: grid;
      grid-template-columns: minmax(9rem, 0.6fr) 1fr;
      gap: 1rem;
      padding-bottom: 0.7rem;
      border-bottom: 1px solid #e5e9f0;
    }

    .details dt {
      color: #4b5c78;
      font-weight: 650;
    }

    .details dd {
      margin: 0;
      overflow-wrap: anywhere;
    }

    li + li {
      margin-top: 0.45rem;
    }

    @media (max-width: 36rem) {
      .hero {
        align-items: flex-start;
      }

      .logo {
        width: 4rem;
        height: 4rem;
      }

      .details div {
        grid-template-columns: 1fr;
        gap: 0.2rem;
      }
    }

## 8. Create index.html

    <!doctype html>
    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta
          name="description"
          content="A small React example for production build configuration and deployment."
        />
        <title>React production demo</title>
      </head>
      <body>
        <div id="root"></div>
        <script type="module" src="/src/main.jsx"></script>
      </body>
    </html>

Keep the existing Vite entry script in the app scaffold if it differs. Vite processes the HTML entry during the build.

## 9. Confirm the React entry point

A standard src/main.jsx for this app is:

    import { StrictMode } from 'react';
    import { createRoot } from 'react-dom/client';
    import App from './App.jsx';

    createRoot(document.getElementById('root')).render(
      <StrictMode>
        <App />
      </StrictMode>
    );

If the existing Vite scaffold already has this entry point, keep it.

## 10. Build and preview locally

Install the locked dependencies and create the production artifact:

    npm ci
    npm run build

Vite writes the output to dist by default. Preview that output locally:

    npm run preview

The preview server is for checking the built app locally; it is not the production server. Confirm the displayed mode is production, the logo loads, and the base path shown on the page matches the configured deployment path.

## 11. Publish the artifact

Configure the static host or CI provider to:

1. Check out the intended commit.
2. Install dependencies with npm ci.
3. Provide any required public build variables.
4. Run npm run build.
5. Publish the dist directory.
6. Configure an SPA fallback if the app uses client-side routes.
7. Keep API routes and missing static assets out of the SPA fallback.

For a domain-root deployment, use base /. For a repository subpath deployment, use the matching subpath in vite.config.js and build again. Do not publish the source directory as if it were the completed site.

## 12. Verify the live deployment

Open the deployed URL and check:

| Check | Expected result |
| --- | --- |
| App root | The React page renders |
| Public logo | The image request succeeds below the configured base path |
| Build mode | The page reports production |
| API configuration | The expected public path is displayed |
| Nested client route | Direct navigation and refresh work if the app has client routing |
| Network panel | JavaScript and CSS requests succeed with the expected content types |
| Browser console | No missing asset or initialization errors |

If the HTML loads but the app is blank, inspect the JavaScript request path and browser console. If only direct nested-route refresh fails, configure the host's SPA fallback. If an API request is wrong, verify the public build configuration and backend route separately.

## 13. Change this example for your deployment

- For a root domain, set base to / or remove the base option.
- For a repository project page, set base to /your-repository/.
- For a different API environment, provide the intended public value at build time or use a documented runtime configuration endpoint.
- For a server-rendered React framework, use its output and hosting instructions rather than publishing only Vite's dist folder.
- For private credentials, make requests through a backend or serverless function; never add them to VITE_ variables.

## Common mistakes

- Leaving the subdirectory base path in place after moving the app to a domain root.
- Setting a root-relative image URL that skips the configured base.
- Forgetting to rebuild after changing build-time variables or base configuration.
- Publishing the project root instead of dist.
- Using vite preview as a public production server.
- Putting a private token into VITE_PUBLIC_API_BASE or another browser variable.
- Forgetting the static host's SPA rewrite for a client-side route.
- Rewriting missing asset and API requests to index.html.
- Assuming the app's own React router can configure the web host.

## Key points

- Build from a lockfile and publish the generated artifact.
- Vite's base path must match the public URL path.
- VITE_ values are public and fixed into the build.
- Use BASE_URL for dynamic public asset references.
- Preview the production build locally, then verify the actual hosted app.
- Hosting fallbacks, API routes, cache behavior, and server runtime depend on the deployment architecture.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="113-html-injection-and-trusted-types.md">113 - Sanitized Rich Text with React and Trusted Types - Complete Example</a></td><td><a href="../lessons/114-production-builds-and-deployment.md">Lesson</a></td><td><a href="../qnas/114-production-builds-and-deployment.md">Q&amp;A</a></td><td><a href="115-production-monitoring-and-maintenance.md">115 - Example: Production Errors, Monitoring, and Maintenance</a></td></tr>
  </tbody>
</table>
