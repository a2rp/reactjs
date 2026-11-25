# Production Builds, Configuration, and Deployment

## Learning objectives

After this lesson, you should be able to:

- Explain what a production build creates and what a static host serves.
- Distinguish development mode, build mode, and deployment environment.
- Keep public frontend configuration separate from private secrets.
- Configure a public base path for root or subdirectory hosting.
- Explain SPA route fallback and static asset cache behavior.
- Set up a repeatable build from a lockfile and deploy the correct artifact.
- Verify and recover a production release.

## 1. What production deployment means

React is a UI library. A production application also needs a way to compile source code, serve its files, route requests, provide API behavior, and manage configuration.

A client-rendered React application can often be built as static files and served by a static host or content delivery network. A server-rendered or full-stack React application also needs the runtime and deployment setup required by its framework.

This lesson uses a React single-page app built with Vite. The same principles apply to other build tools, but commands, output directories, and configuration names differ.

## 2. Development server versus production build

A development server is designed for editing: it provides fast feedback and development diagnostics. It is not the production application artifact.

For a typical Vite project, the scripts may look like this:

    {
      "scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview"
      }
    }

Run the development server with npm run dev. Create a production bundle with npm run build. Vite's default output directory is dist, unless the project config changes it.

The preview command serves the built output locally so you can inspect the production bundle before deployment. It is a preview tool, not a production server. Deploy the built files to the hosting service or use the production runtime required by your framework.

## 3. Build from a lockfile

Commit the package manager lockfile, such as package-lock.json, and use the matching package manager in the build pipeline. With npm, npm ci installs the versions recorded in the lockfile and fails if the manifest and lockfile are inconsistent.

A typical build sequence is:

    npm ci
    npm run build
    npm run preview

Do not deploy a local node_modules directory. Build in a clean, repeatable environment and publish the generated artifact, not the development source tree.

## 4. Understand what goes into the build

The build tool resolves imports, transforms JSX and supported syntax, processes CSS and assets, and emits browser-ready files. Production builds commonly minify code and create fingerprints in asset filenames so changed files can be cached independently.

The output might include:

    dist/
      index.html
      assets/
        index-<fingerprint>.js
        index-<fingerprint>.css

The exact files and names depend on the project and build tool. Do not hard-code generated filenames. The generated HTML references the files emitted by that build.

A build that succeeds proves the build tool produced output. It does not prove that routes, API calls, assets, environment values, browser support, or hosting configuration work after deployment.

## 5. Development, modes, and deployment environments

These terms describe different things:

- Development or production describes how the app or build is optimized and instrumented.
- A build mode selects build-time configuration, such as Vite's production or staging mode.
- A deployment environment is where a release runs, such as preview, staging, or production.

Vite's default dev command uses development mode. Vite's build command uses production mode by default. A custom mode can load a matching environment file, but the build remains a production build unless configured otherwise.

Choose an explicit environment strategy. A team can build a separate artifact for each environment, or build once and load a public runtime configuration when the app starts. Either way, document which source supplies each value.

## 6. Build-time configuration is public

Vite exposes selected values through import.meta.env. Variables prefixed with VITE_ are bundled into client-side code at build time and can be read by anyone who downloads the application.

Example:

    VITE_PUBLIC_API_BASE=/api
    VITE_PUBLIC_APP_NAME=Learning workspace

    const apiBase = import.meta.env.VITE_PUBLIC_API_BASE;
    const appName = import.meta.env.VITE_PUBLIC_APP_NAME;

Environment values exposed to browser code are public, even if they came from a private CI variable or an ignored .env file. Never put database passwords, private API keys, signing secrets, or server credentials in VITE_ variables.

Keep secrets on a server or in a serverless or edge function. The browser should call your backend, which can authenticate the user and make any secret-bearing request privately.

Vite exposes environment values as strings. Convert and validate them before using them as booleans, numbers, URLs, or other structured values.

## 7. Environment files and runtime configuration

Vite can load generic and mode-specific environment files. Local-only files such as .env.local should be excluded from version control. Check the repository's ignore rules; do not assume every environment file is ignored automatically.

A checked-in example file can document the public variables a build needs:

    VITE_PUBLIC_API_BASE=/api
    VITE_PUBLIC_APP_NAME=Learning workspace

A mode-specific file can provide public defaults, while the deployment pipeline can supply values at build time. After changing environment files, restart the dev server or rebuild so the values are loaded.

Build-time configuration is fixed in the generated assets. If the same artifact must move unchanged from staging to production, use a deliberate runtime configuration mechanism or have the backend provide environment-specific data. Do not expect changing a CI variable after the build to rewrite values already embedded in JavaScript.

## 8. Choose the correct public base path

A Vite app served from the domain root can use the default base path /. An app published below a repository or subdirectory needs a matching base path.

For a site hosted at /react-demo/, a config can declare:

    import { defineConfig } from 'vite';
    import react from '@vitejs/plugin-react';

    export default defineConfig({
      plugins: [react()],
      base: '/react-demo/',
    });

Use the actual deployment path. If a build is published at a different path than the configured base, the generated HTML may request JavaScript, CSS, or images from the wrong URLs.

For dynamic references to public assets, use Vite's base value:

    const logoUrl = import.meta.env.BASE_URL + 'images/logo.svg';

Prefer imported assets when they belong in the module graph. Use the public directory for files that need a stable URL and are copied as-is.

## 9. Client-side routes need host support

A single-page app can render different views for paths such as /settings or /courses/intro. The browser router handles navigation after the app loads, but the web host receives the initial request first.

If someone opens or refreshes a nested client-side route directly, the host must serve the app's index.html for that route. Configure the host's SPA fallback or rewrite. Keep real static files and API routes out of that fallback so a missing image or API request does not receive HTML by mistake.

A server-rendered framework, a multi-page application, and an API server have different routing requirements. Configure the behavior for the architecture actually being deployed.

## 10. Cache HTML and fingerprinted assets differently

Generated JavaScript, CSS, and images with content fingerprints can usually use long-lived immutable caching: if their content changes, the build emits a new filename. The HTML entry point must refresh often enough to point to the current filenames.

If HTML is cached too long, a visitor may keep receiving an old page that requests assets removed by a later deployment. If fingerprinted assets are cached too briefly, users download unchanged files again. Set cache headers according to the host and build strategy.

Deploy each build's HTML and referenced assets as a consistent release. Avoid deleting old assets while users may still have a page open that references them, or provide a recovery path for stale chunks.

## 11. Browser support and polyfills

The build tool's target setting controls which language features it transforms. A syntax transform does not necessarily add polyfills for every browser API. Choose supported browsers based on product requirements, check the build tool's current defaults, and add polyfills only for features the app needs.

Test on the browsers and devices the product supports. A successful build on the developer's machine does not prove that a browser has a required API.

## 12. Source maps and error visibility

Source maps can make production errors easier to trace to authored code. They can also expose source details if published publicly. Decide whether to publish them, restrict access, or upload them to an error-monitoring service during release.

Never rely on minification to keep secrets safe. Anything included in client assets can be inspected. Use privacy-aware error reporting, avoid logging tokens or personal data, and verify that release identifiers match the deployed build.

## 13. Continuous integration and release flow

A basic release pipeline should:

1. Check out the intended commit.
2. Install dependencies from the committed lockfile.
3. Run the project's required checks.
4. Build the production artifact.
5. Publish the correct output directory.
6. Wait for deployment and inspect the hosting result.
7. Keep the previous known-good release available for rollback.

Preview deployments are useful for reviewing a candidate build before production. Protect production credentials and deployment permissions in the hosting platform. Do not store secrets in repository files or browser environment variables.

## 14. Verify the deployed application

After deployment, check the actual deployed URL:

- The root page loads without console errors.
- JavaScript, CSS, fonts, and images return successfully.
- Asset requests use the expected base path.
- A nested client route works when opened directly and refreshed.
- API requests use the intended environment and return expected responses.
- The production build does not expose secrets or local development values.
- Cache headers and content types are appropriate.
- The page works in the supported browsers and on a narrow viewport.
- A failed release can be rolled back without losing data.

Use browser developer tools and hosting logs to distinguish a build problem from an asset path, routing, cache, or API problem.

## Common mistakes

- Deploying the development server or source folder instead of the built artifact.
- Treating vite preview as a production server.
- Publishing the wrong directory instead of dist.
- Forgetting the subdirectory base path.
- Assuming a router can handle a direct request before the host returns index.html.
- Adding a fallback that also catches missing assets or API endpoints.
- Putting secrets in variables exposed to browser code.
- Expecting build-time variables to change after the artifact is built.
- Forgetting that environment variables are usually strings.
- Caching index.html as aggressively as fingerprinted assets.
- Treating a successful build as proof that deployment works.
- Assuming transpilation automatically polyfills every browser API.

## Key takeaways

- Build a production artifact from a clean install and committed lockfile.
- Deploy the generated output with hosting behavior suited to the app architecture.
- Vite's VITE_ variables are public build-time values; keep secrets on the server.
- Set the correct base path and SPA fallback for the deployed URL structure.
- Refresh HTML promptly and cache fingerprinted assets according to their filenames.
- Verify the live deployment, not only the local build.
