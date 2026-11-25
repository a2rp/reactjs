# Production Builds, Configuration, and Deployment — Q&A

## Builds and artifacts

### 1. What is a production build?

It is the generated application output prepared for the deployment architecture. A Vite client app is bundled into browser assets and HTML; a server-rendered app also needs its server runtime and framework-specific output.

### 2. Is the development server the production app?

No. The development server is for local editing and fast feedback. Build the production artifact and serve it with the production hosting or framework setup.

### 3. What does npm ci do?

It performs a clean dependency installation for automated environments. It requires a lockfile, removes an existing node_modules directory, and fails if the lockfile and package manifest do not agree. It does not update the package manifest or lockfile.

### 4. Why should a project commit its lockfile?

The lockfile records the resolved dependency tree so local and automated installations can reproduce it. Use the same package manager and compatible install settings in development and CI.

### 5. What does npm run build do in a typical Vite project?

It runs the configured build script, commonly vite build. Vite transforms and bundles the app for production and writes output to dist by default unless configured otherwise.

### 6. What should be deployed from a Vite static app?

Deploy the generated output directory, usually dist, to the static host's publish directory. Do not deploy node_modules or assume the host should compile a development source tree unless that host is configured to run the build command.

### 7. Is a build that succeeds ready to publish?

Not by itself. It proves that the configured build completed. It does not prove routes, API endpoints, asset paths, environment values, browser support, or hosting rules work.

### 8. What is vite preview for?

It serves the built output locally so the developer can inspect a production build before publishing. It is not designed to be used as the production server.

### 9. Does a production build automatically add every needed polyfill?

No. Vite transforms syntax for its configured target, but does not automatically provide polyfills for every browser API. Check the supported browser range and add only the polyfills the app needs.

### 10. Are generated asset filenames stable?

Not necessarily. Build tools may add content fingerprints or otherwise change filenames when content changes. Use the generated HTML or manifest references rather than hard-coding output filenames.

## Configuration and environments

### 11. What is the difference between build mode and deployment environment?

A build mode selects configuration used during a build. A deployment environment is where a release runs, such as preview, staging, or production. They are related but are not the same concept.

### 12. Which mode does Vite use for its default commands?

The dev command uses development mode. The build command uses production mode by default. A custom mode can load different mode-specific configuration while still producing a production build.

### 13. What does VITE_ mean for a variable?

A variable with the VITE_ prefix can be exposed to client-side code through import.meta.env and embedded in the build. Treat its value as public.

### 14. Can I put a private API key in VITE_API_KEY?

No. The value would be bundled for the browser and could be inspected by visitors. Keep private credentials on a server or serverless function and have the browser call that backend.

### 15. Are .env files automatically secret?

No. Files can be committed accidentally, and values intended for the browser become public when exposed by the build. Check ignore rules and never include credentials in client-exposed variables.

### 16. Are Vite environment values strings?

Yes. Convert and validate them before using them as booleans, numbers, URLs, or structured configuration.

### 17. When does Vite read environment files?

Vite loads them when the command starts. Restart the development server after changing the files, and rebuild to change values embedded in a production artifact.

### 18. Can changing a CI variable update an already-built artifact?

Not if the value was substituted at build time. Build a new artifact or use an intentional runtime configuration mechanism.

### 19. What is runtime configuration?

It is public configuration delivered when the app starts or requested from an endpoint at runtime, instead of being fixed in the compiled assets. It can let one artifact run in multiple environments, but it still must not expose secrets.

### 20. How should I choose between build-time and runtime config?

Choose based on release policy. Build-time config is simple when each environment gets its own build. Runtime config can support moving one artifact between environments but requires a clear loading and validation mechanism.

## Paths, routes, and hosting

### 21. What is Vite's base path?

It is the public path prefix where the built app is served. The default is the domain root. Configure a subdirectory base when the app is hosted below a path such as /project/.

### 22. What can go wrong if the base path is incorrect?

The HTML can request scripts, CSS, and images from the wrong URLs. The page may load without styling or JavaScript, or assets may return 404 errors.

### 23. How should a dynamic public asset path use Vite's base?

Use import.meta.env.BASE_URL when constructing a dynamic asset URL. Build tools can adjust statically referenced assets automatically, but manually constructed URLs need to account for the deployment path.

### 24. What is an SPA fallback?

It is a host rule that serves the app's index.html for client-side routes so the browser router can render the requested view after the app loads.

### 25. Why can a nested route fail after refresh?

The browser requests that route directly from the host. If the host looks only for a physical file at that path and has no SPA fallback, it can return 404 before React starts.

### 26. Should every missing request be rewritten to index.html?

No. Keep API endpoints and real static assets out of the SPA fallback. Otherwise missing JavaScript or image requests can incorrectly receive HTML and fail with confusing content-type or parsing errors.

### 27. Does a Vite SPA and a server-rendered React app deploy the same way?

No. A static client-rendered app can often be hosted as files. A server-rendered or full-stack framework may require a runtime, server routes, or framework-specific hosting output. Follow that framework's deployment model.

## Caching, monitoring, and releases

### 28. Why are fingerprinted assets good candidates for long-lived caching?

Their filenames change when their content changes, so a new build can use new URLs while unchanged files remain cached. Configure cache headers based on the host and the generated asset strategy.

### 29. Should index.html receive the same long cache lifetime as fingerprinted assets?

Usually not. HTML points to the current asset filenames and should refresh often enough to discover a new release. If it stays stale, users may request old assets that are no longer available.

### 30. What is a stale chunk failure?

A user may have an older app open when a deployment removes files that the old app later requests, such as a lazily loaded chunk. Keep deployment and asset retention behavior in mind and provide a recovery path that does not erase user work.

### 31. Should source maps be published publicly?

Choose deliberately. Source maps help trace errors to authored source but can reveal source details. Restrict them or upload them to a monitoring service if that better fits the app's needs.

### 32. Should minification be used to hide secrets?

No. Minification is not confidentiality. Anything embedded in browser assets can be inspected.

### 33. What should a CI deployment pipeline do?

Use a clean checkout, install from the lockfile, run the project's required checks, build the artifact, publish the correct directory, and report the deployment result. Protect deployment credentials and permissions in the CI platform.

### 34. Should a deployment command run against a moving branch head?

A release should correspond to a known commit and artifact. Record which commit was built so the deployment can be traced and rolled back.

### 35. What should I verify after publishing?

Open the deployed URL and check the page, console, network requests, asset paths, nested-route refresh, API behavior, and supported browsers. Confirm the actual production configuration rather than relying only on local output.

### 36. How do I distinguish a build failure from a deployment failure?

Check the pipeline logs first. If the build produced the expected artifact, inspect the deployed response, asset requests, hosting rewrites, cache headers, and API configuration to locate the failure.

### 37. What is a safe rollback?

Restore a previously known-good release or artifact using the host's release controls. Verify the restored app and its corresponding assets; avoid rolling back in a way that leaves HTML pointing at files that were deleted.

### 38. Should preview deployments use production secrets?

No. Give each environment only the credentials it needs. Browser-exposed config is public in every environment, and CI secrets should be restricted to the workflows and environments that require them.

### 39. How should a GitHub Pages repository subpath be configured?

Set Vite's base path to the repository's published subdirectory, such as /repository-name/. A custom domain or username root page generally uses the root path instead. Confirm the final deployed URL.

### 40. What is the main release principle?

Build a traceable artifact from a clean, locked dependency install; configure it for the intended public path and environment; deploy it using the correct hosting model; then verify the live release and keep a rollback path.
