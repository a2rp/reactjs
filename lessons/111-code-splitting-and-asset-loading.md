# Code Splitting, Asset Loading, and Bundle Analysis

## Learning objectives

After this lesson, you should be able to:

- Explain the difference between tree shaking, code splitting, and lazy loading.
- Split rarely used React features into asynchronous chunks with lazy and Suspense.
- Choose loading boundaries that preserve useful visible content.
- Load images, fonts, and other assets with appropriate priority and dimensions.
- Inspect the production output and identify what contributes to initial load.
- Handle chunk-loading failures and avoid creating too many tiny chunks.

## 1. What affects the first visit?

A visitor waits for more than the JavaScript entry file. The browser may need to download, parse, and execute JavaScript, load CSS and fonts, fetch images, and render the initial UI.

Large or unnecessary files on the critical path can delay interaction. A useful goal is to keep the first screen responsive while loading other features when the user needs them.

Measure real loads on representative devices and network conditions. A file's compressed download size is only one part of its cost; parsing, execution, layout, and image decoding also matter.

## 2. Tree shaking, splitting, and lazy loading

These ideas are related but distinct:

- Tree shaking removes unused exports from production output when the module and build setup allow it.
- Code splitting divides output into separate files or chunks.
- Lazy loading delays requesting some code or assets until they are needed.

A project can tree-shake one entry bundle without splitting it. A split chunk can still be downloaded immediately if the application requests it immediately. The goal is not simply to create more files; it is to defer nonessential work and load the right parts when needed.

## 3. Split a React feature with lazy and Suspense

React lazy loads a component from a dynamic import when the component is first rendered. The imported module must resolve to a default component export.

    import { lazy, Suspense, useState } from 'react';

    const ReportsPanel = lazy(() => import('./ReportsPanel.jsx'));

    export default function Dashboard() {
      const [showReports, setShowReports] = useState(false);

      return (
        <main>
          <h1>Dashboard</h1>

          <button onClick={() => setShowReports(true)}>
            Open reports
          </button>

          {showReports && (
            <Suspense fallback={<p role="status">Loading reports…</p>}>
              <ReportsPanel />
            </Suspense>
          )}
        </main>
      );
    }

The dashboard remains available while the reports feature loads. Because the lazy component is rendered only after the user asks for it, its code can stay out of the initial route's required work.

Declare lazy components outside component bodies. Each Suspense boundary should cover a meaningful portion of the interface, and its fallback should fit the space that is waiting.

### Choosing a Suspense boundary

- Keep important page content outside a boundary if it can appear immediately.
- Use a local skeleton or message for a panel that loads independently.
- Use a broader fallback only when the whole region truly cannot be used yet.
- Avoid nesting many tiny boundaries without a clear reveal design.

Suspense handles the pending load for a lazy component. If its import fails, React sends the rejection to the nearest Error Boundary. Provide an error state that lets the user retry or reload where appropriate.

## 4. Split routes and optional features

Route-level splitting is often a good first boundary because a visitor usually does not need every page's code before seeing the current route. A router may provide its own lazy route API; follow that router's approach.

Other candidates include:

- A report editor opened only by some users.
- A map or chart below the initial viewport.
- An administration panel behind a permission check.
- A rich-text editor used only on one workflow.
- A large export library used only after an export action.

Do not split tiny components only to increase the chunk count. Each boundary can introduce loading states, requests, caching behavior, and more deployment failure cases. Group code according to user-visible features and actual usage.

## 5. Load non-component code on demand

Dynamic import can defer a non-React module until an action needs it:

    async function exportReport(report) {
      const { createPdf } = await import('./pdf-exporter.js');
      return createPdf(report);
    }

Keep the loading and failure experience in the feature that starts the import. If the module is large or shared across features, inspect how the build groups its dependencies.

A dynamic import can produce a separate chunk in a production build when the bundler supports it. The exact chunk layout is a build-tool decision; do not assume one source import always maps to exactly one output file.

## 6. Static assets and image priority

In a Vite project, assets imported by source code are processed by the build and receive a resolved production URL. Files placed in the public directory are copied as-is and are typically referenced by a root-relative public path. Choose the approach that fits whether the file is part of the module graph or needs a fixed public name.

For images:

- Include real intrinsic width and height to reserve layout space.
- Use descriptive alt text for meaningful images and empty alt text for decorative images.
- Use responsive sources and sizes when devices need different image dimensions.
- Use loading="lazy" for images well below the initial viewport.
- Do not lazy-load the main image that is needed to show the first screen; let it load promptly.
- Avoid sending a very large source image to a small card when smaller variants are available.

Example:

    <img
      src="/images/course-cover-640.webp"
      srcSet="/images/course-cover-320.webp 320w,
              /images/course-cover-640.webp 640w,
              /images/course-cover-960.webp 960w"
      sizes="(max-width: 40rem) 100vw, 40rem"
      width="960"
      height="540"
      alt="A learner following a web development lesson"
    />

For a below-the-fold gallery image, add loading="lazy". Do not add it to every image without considering whether the image is needed immediately.

## 7. Fonts, CSS, and other assets

- Prefer efficient font formats and only the weights and character subsets the interface needs.
- Use font-display: swap or another deliberate display strategy so text remains readable while a font loads.
- Preload only a resource that is genuinely critical and known early. Preloading too much competes with the page's most important resources.
- Keep CSS for the first screen available promptly. Split feature-specific styles only when the build setup supports it and the loading behavior remains correct.
- Use explicit media, dimensions, or responsive sources for video and audio assets when relevant.
- Avoid inlining large assets as data URLs without measuring the resulting JavaScript and caching cost.

## 8. Inspect the production build

Development mode is not a reliable picture of production output. Use the project's production build command and inspect its output:

1. Record the generated entry chunks, asynchronous chunks, CSS, fonts, and images.
2. Note raw sizes and compressed transfer sizes when available.
3. Identify which files are required before the initial route becomes usable.
4. Confirm that opening a lazy feature requests its chunk then, not at initial page load.
5. Check for repeated dependencies, unexpectedly large libraries, and assets with excessive dimensions.
6. Use the browser Network panel to inspect request timing and transfer size.
7. Use browser Performance and Coverage tools to inspect execution and unused code.
8. If a visual bundle report is useful, use a build analyzer that supports the project's current bundler and version.

Set size budgets around what the product needs and the devices it supports. Review the report after adding a large dependency, changing routes, or moving code across split boundaries.

## 9. Read bundle results carefully

Ask these questions when a chunk looks large:

- Is the dependency needed on the first screen?
- Can it be imported by only the route or feature that uses it?
- Is the package importing more code than the feature needs?
- Are several versions of the same dependency included?
- Could an image or font, rather than JavaScript, be the actual transfer problem?
- Is parse or execution time high even when the file is compressed well?
- Does splitting the code add more requests or duplicate shared modules?

A smaller initial JavaScript chunk can improve initial loading while a later interaction takes longer to open. Measure both the first screen and the delayed feature.

## 10. Chunk caching and deployment failures

Production builds often give changed assets new filenames so browsers can cache unchanged files longer. The HTML entry should still refresh to point at the current chunk names.

A visitor can have an older application open when a new deployment removes its old chunks. A later dynamic import can then fail. Keep deployment and cache behavior in mind, show a useful failure state, and consider a controlled recovery such as asking the visitor to reload.

Do not blindly reload on every import error. The error might be an offline connection or a temporary server issue. Avoid loops and preserve user work when possible.

## 11. Common mistakes

- Moving every component into a dynamic import without a user-facing reason.
- Putting all page content behind one large loading fallback.
- Assuming Suspense catches network or data work started in an Effect.
- Forgetting that a lazy import rejection needs an Error Boundary.
- Lazy-loading an image needed for the initial visible screen.
- Omitting image dimensions and causing layout shifts.
- Preloading many fonts, scripts, and images at once.
- Measuring only compressed download size while ignoring parsing and execution.
- Checking only development output instead of a production build.
- Adding too many tiny chunks or duplicating common dependencies.
- Treating a chunk-loading failure as if it always means the visitor should reload.

## 12. A practical workflow

1. Measure initial loading and identify the largest critical resources.
2. Keep the initial route's essential UI and code on the critical path.
3. Split a route or feature that users do not need immediately.
4. Add a contextual Suspense fallback and an Error Boundary for load failures.
5. Optimize image dimensions, font selection, and loading priority.
6. Build for production and inspect the entry and async chunks.
7. Test first load, feature opening, offline or failed chunk behavior, and deployment updates.
8. Compare user-visible performance again and keep only useful splits.

## Key takeaways

- Tree shaking removes unused code; code splitting separates output; lazy loading delays requesting part of it.
- React lazy and Suspense support loading component code at a feature boundary.
- Error Boundaries handle failed lazy imports; Suspense shows the pending state.
- Prioritize the first screen's code and assets, then load optional work when needed.
- Measure the production build and browser work, not just file count or raw bundle size.
- Split around useful features, keep loading states clear, and plan for stale chunk failures.
