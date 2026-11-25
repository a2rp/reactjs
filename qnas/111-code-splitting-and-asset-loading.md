# Code Splitting, Asset Loading, and Bundle Analysis — Q&A

## Code splitting and loading behavior

### 1. What is code splitting?

Code splitting divides a production build into multiple files or chunks so the application can load some code separately from the initial entry.

### 2. Does creating another chunk automatically make the app faster?

No. A chunk helps only if it keeps nonessential work off the critical path or improves caching. Too many chunks can add requests, loading states, and coordination overhead.

### 3. What is the difference between tree shaking and code splitting?

Tree shaking removes unused exports when the module and build tool support it. Code splitting separates output into chunks. They can be used together but solve different problems.

### 4. What does lazy loading mean?

Lazy loading delays requesting code or an asset until it is needed. A split chunk that the app immediately requests is not meaningfully lazy for the initial experience.

### 5. What is React lazy used for?

React lazy defines a component whose module is loaded through a dynamic import when the component is first rendered. The resolved module must provide a default component export.

### 6. Where should I declare a lazy component?

Declare it outside the rendering component, usually at module scope. This keeps the lazy component identity stable across renders.

### 7. Does React.lazy work with a named export?

It expects the loaded module to resolve to an object with a default component. A named export can be adapted by returning an object whose default property is that export, but using a default export for a lazily loaded feature is often simpler.

### 8. What does Suspense do for a lazy component?

When the lazy component is pending, the nearest Suspense boundary displays its fallback. When the module is ready, React renders the component in that boundary.

### 9. Does Suspense catch a failed dynamic import?

No. Suspense handles the pending state. If a lazy import rejects, React sends the rejection to the nearest Error Boundary.

### 10. Does Suspense automatically wait for every fetch or image?

No. Suspense only responds to supported suspending sources, such as lazy component code and supported Suspense-enabled data or asset integrations. A fetch started in an Effect or event handler does not suspend by itself.

### 11. Where should I put a Suspense boundary?

Place it around a meaningful region that can wait independently. Keep usable content outside when possible, and choose a fallback that resembles the size and shape of the region being loaded.

### 12. Should the whole page use one fallback?

Only if the entire page genuinely cannot be used before that content is ready. Otherwise, a broad fallback hides already-available UI and makes loading feel slower.

### 13. Can I split a route instead of one component?

Yes. Route-level splitting is often a natural boundary because visitors generally need the current route before other pages. Use the router's recommended lazy route API when it has one.

### 14. Can I dynamically import a library from an event handler?

Yes. A dynamic import can defer optional non-component code, such as an export or editor library, until the relevant action. Show pending and error states for the action and inspect how the build groups shared dependencies.

## Assets and priority

### 15. How does Vite treat an imported image?

An image imported from source code becomes a build-managed asset URL. The production build can rename it and update references so caching and deployment work with the generated output.

### 16. When should a file go in Vite's public directory?

Use public for a file that needs a stable, known URL and should be copied as-is. Use source imports when the asset belongs in the module graph and should be resolved by the build.

### 17. Should every image use loading=lazy?

No. Lazy-load images that are below the initial viewport. The main image needed to show the initial screen should load promptly; delaying it can make the page appear slower.

### 18. Why should image width and height be specified?

Intrinsic dimensions let the browser reserve layout space before the image arrives, reducing layout shifts. They also make the intended aspect ratio clear.

### 19. What are srcSet and sizes for?

They let the browser choose an appropriate image source for the rendered size and device resolution. This can avoid transferring a large image when a smaller source is sufficient.

### 20. Should I preload every important image or font?

No. Preload only resources known to be critical and needed early. Too many preloads compete with the document's other important requests.

### 21. How should I handle fonts?

Ship only needed font formats, weights, and character subsets. Choose a deliberate display strategy so text remains usable while a font loads, and preload only a font that is genuinely critical.

### 22. Can I lazy-load CSS with a feature?

Some build tools can split styles associated with asynchronous modules. Confirm the current toolchain's behavior, and ensure the feature does not flash unstyled content or block unrelated content.

## Bundle analysis and performance

### 23. How do I inspect bundle size?

Build for production and inspect the output chunks and assets. Use the browser Network panel for transfer timing and sizes, Performance tools for parse and execution costs, and a compatible bundle visualizer when a dependency-level view is needed.

### 24. Why is compressed size not enough?

A compressed file's cost also includes decompression, parsing, compilation, execution, and its effects on rendering. A small but CPU-heavy dependency can still hurt responsiveness.

### 25. What should I check in a bundle report?

Look for large entry chunks, dependencies only used by optional features, duplicate package versions, unexpectedly large assets, and shared chunks that are pulled into the initial route.

### 26. What is a good first code-splitting boundary?

A route or a large optional feature that is not needed for the first screen is often a useful starting point. Measure before and after to confirm that the split improves the actual experience.

### 27. Can I split a tiny component?

Technically yes, but it is rarely useful by itself. Tiny chunks can add request and loading overhead without removing enough work from the initial path.

### 28. Why can my initial chunk remain large after splitting a feature?

The feature may still be statically imported elsewhere, shared dependencies may remain in the entry, or the build may group modules differently than expected. Inspect the generated chunks and import graph.

### 29. Does Vite make async chunks and assets load automatically?

Vite handles supported asset references and production chunk loading as part of its build pipeline. The exact emitted files and optimizations depend on the project and build tool version; inspect the production output instead of assuming a particular file layout.

### 30. Does code splitting improve caching?

It can. If stable code is separated from frequently changing code, an unchanged chunk can remain cached. But split decisions can also cause shared chunk invalidation; check the emitted filenames and deployment cache policy.

## Failure handling and deployment

### 31. What happens if a lazy import fails?

The lazy component's loading Promise rejects, and React sends the error to the nearest Error Boundary. Show a useful error message and provide an appropriate recovery action.

### 32. Why might a chunk load fail after a deployment?

A visitor can keep an older application open while a deployment removes its old hashed chunks. If that application later requests an old chunk URL, the request may fail.

### 33. Should an app reload whenever a chunk fails?

Not blindly. The cause could be offline connectivity, a temporary server problem, or a stale deployment. Provide a controlled recovery that avoids reload loops and protects user work where possible.

### 34. How should HTML and hashed assets be cached?

HTML needs to refresh often enough to point at the current asset names. Fingerprinted assets can generally be cached longer because changed content gets a different URL. Match cache headers to the hosting and deployment strategy.

### 35. How do I test the lazy loading boundary?

Use the browser Network panel with cache disabled or a clean session. Confirm the optional feature's chunk is requested when that feature is opened, observe the pending fallback, and simulate a failed request to verify the Error Boundary.

### 36. What if the import works in development but not production?

Inspect the production build, case-sensitive file paths, dynamic import patterns supported by the bundler, base paths, and deployed asset URLs. Development servers and production output can resolve code differently.

### 37. Does Vite's dynamic import variable support arbitrary paths?

No. The bundler needs to determine which files could be imported. Follow its supported path pattern rules; for a finite feature set, an explicit import map or supported glob import is safer than an unconstrained variable path.

## Decision workflow

### 38. Should I split code before measuring?

Usually not. Establish a baseline, identify the critical resource or expensive execution, then split a meaningful feature and compare. The split should improve loading or interaction without adding unjustified complexity.

### 39. What is the safest optimization workflow?

1. Measure the initial route and important interactions.
2. Inspect JavaScript, CSS, image, and font costs.
3. Split a route or optional feature with a clear loading boundary.
4. Optimize critical images and fonts.
5. Build for production and inspect emitted chunks.
6. Test slow, failed, and stale asset loads.
7. Compare the same user experience and keep the change only if it helps.

### 40. What is the central rule for bundle optimization?

Optimize what users wait for. Keep essential code and assets ready for the initial screen, load optional work at a useful boundary, and validate the production build with real measurements.
