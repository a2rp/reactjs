# Frameworks, Client Rendering, SSR, and Static Generation: Questions and Answers

## Frameworks and rendering modes

### 1. What does React provide, and what does a framework add?
React provides component rendering and UI APIs. A framework can integrate routing, data loading, builds, server rendering, static generation, caching, and deployment conventions.

### 2. Does using a framework mean the app needs a server at runtime?
No. A framework may generate static files, use client rendering, render some routes on a server, or combine these modes. Its deployment needs depend on how the project is configured.

### 3. What is client-side rendering (CSR)?
The browser downloads and runs JavaScript, and React renders into a root container. An empty root may show little content until the JavaScript loads and executes.

### 4. What is server-side rendering (SSR)?
The server renders React to HTML for a request. The browser can display that markup before client JavaScript runs; hydration later attaches React behavior.

### 5. What is static site generation (SSG)?
A build process renders routes to HTML before visitors request them. The generated files can be served as static content, and interactive pages can hydrate them in the browser.

### 6. Is a single-page app the same thing as CSR?
No. CSR describes where initial UI rendering happens. An SPA describes a navigation model. They are related choices but not the same concept.

### 7. Can one application use multiple rendering modes?
Yes. A framework can use different modes for different routes or sections, subject to its supported APIs. Choose based on each route's data, privacy, freshness, and interaction needs.

### 8. When is CSR a reasonable choice?
For an embedded widget, a private application centered on browser interactions, or a project where an empty shell is acceptable while the app loads. Provide good loading, error, and accessibility behavior.

### 9. When is SSR useful?
When request-specific HTML or early content is valuable. It can improve the initial visible response, but adds server work, data coordination, caching concerns, and hydration requirements.

### 10. When is SSG useful?
When pages can be prepared at build time and served as static output. Plan how content updates: a new build, a supported revalidation mechanism, or a client-side fetch.

### 11. What is revalidation?
A framework or platform mechanism that refreshes previously generated output after a policy or event. It is not one universal React API; exact behavior depends on the framework and deployment.

### 12. Does SSR automatically guarantee better performance or search ranking?
No. Server response time, bundle size, data latency, metadata, content quality, caching, and the platform all matter. SSR only changes when and where initial HTML is generated.

## Hydration and React DOM APIs

### 13. What is hydration?
Hydration attaches React's client behavior to HTML produced by a matching React render on the server or during a build. It reuses the existing DOM as the initial UI.

### 14. When should I use `createRoot`?
Use `createRoot` to render React into an empty client-side root container.

### 15. When should I use `hydrateRoot`?
Use `hydrateRoot` when the root already contains HTML generated from the matching React tree. It attaches behavior to that markup.

### 16. What can happen if I call `createRoot` on server-rendered markup?
React can discard and recreate the existing DOM, wasting work and potentially resetting focus, scroll position, or user input. Use `hydrateRoot` for server HTML.

### 17. What must match during hydration?
The server output and the first client render should use the same initial component tree, props, data, locale, and other values that affect markup.

### 18. What causes hydration mismatch warnings?
Common causes include browser-only reads during render, random or current-time values, different server/client data, different locale or time zone, and invalid or differently nested HTML.

### 19. Can I use `window` or `localStorage` during a server render?
No; those browser APIs do not exist in a server environment. Read browser-only values after hydration or through a framework-supported client boundary, and keep the initial server/client output consistent.

### 20. Do `useEffect` callbacks run during server rendering?
No. They run in the browser after hydration. Data fetched only in an effect cannot be included in the initial server-rendered HTML.

### 21. How should I use a browser-only value without a mismatch?
Render a deterministic initial state that matches the server, then read the value after hydration. A framework may provide a browser-only boundary. Avoid a two-pass render unless the visual change is acceptable.

### 22. Is `suppressHydrationWarning` a general fix?
No. It is a narrow escape hatch for an unavoidable text or attribute mismatch. It does not fix a mismatched component tree or make the output consistent.

### 23. What is the difference between hydrating a root and hydrating a whole document?
Most apps hydrate a container such as `#root`. An app built to render the entire document can hydrate `document` itself, but that is an intentional architecture handled by the framework or app entry point.

## Static generation and server rendering APIs

### 24. Does static HTML need React JavaScript?
Not if the page is entirely static. If it needs React interactions, client code must hydrate the output or mount interactive sections through another deliberate approach.

### 25. Can a static page be interactive?
Yes. SSG can produce initial HTML at build time and include client code that hydrates it. The client needs the same initial data used to generate the markup.

### 26. What does `renderToString` do?
It renders a React tree to an HTML string. It is useful for a basic demonstration or constrained environment, but it does not stream and does not wait for suspended content.

### 27. Is `renderToString` the recommended production streaming API?
No. For streaming and Suspense-aware rendering, use the framework's supported server-rendering path or the appropriate React streaming API for the server runtime.

### 28. What is the difference between static rendering and server rendering?
Both can generate React HTML outside the browser. SSR usually generates HTML per request; SSG generates it ahead of requests during a build. Static output is then reused until rebuilt or revalidated.

### 29. Should an application component call low-level rendering APIs?
Usually not. Top-level framework or entry code should coordinate server rendering, hydration, and routing. Most components should describe UI and not manage how the entire document is delivered.

## Data, caching, and deployment

### 30. Where should data be loaded for SSG?
At build time or through a framework's supported generation process. Data that changes later can use revalidation or be loaded in the browser, depending on the product needs.

### 31. Where should request-specific data be loaded for SSR?
In the framework's request-aware loader, route handler, or server component mechanism. Follow its authentication, error, and caching rules.

### 32. How should a CSR page load data?
Load it in the browser through the app's data layer and render clear loading, success, empty, and error states. The initial HTML may not include that data unless it is supplied separately.

### 33. What is the danger of caching personalized SSR HTML?
A shared cache could serve one user's private data to another. Cache policy and keys must account for all user-specific inputs, or the personalized response should not be shared.

### 34. Can private data be included in a public static build?
Not safely. Build artifacts may be served to everyone. Generate only content intended to be public and handle private data through an authenticated request path.

### 35. Does a framework always run a server for every page request?
No. It may prebuild routes, serve static assets, use request-time rendering only for some routes, or deploy a mix. Inspect the framework's output and hosting model.

### 36. What should I evaluate before choosing a framework?
Routing, data loading, SSR and SSG support, streaming, caching, error handling, forms, authentication, deployment requirements, team familiarity, upgrade path, and operational support.

## Related concepts and practical review

### 37. Are SSR and React Server Components the same thing?
No. SSR describes generating HTML on a server. Server Components describe a component execution model. A server-rendered page may include client components, and the concepts can work together.

### 38. Does hydration mean the whole page immediately becomes interactive?
Hydration work must run and attach handlers, so interactivity may be progressive or delayed depending on the framework and page. Server HTML provides visible markup but does not itself make React event handlers active.

### 39. What is a good approach for a public article page?
If the article is known at build time and changes infrequently, SSG may be appropriate. If it must reflect request-time data, SSR may fit better. Include accessible markup and metadata either way.

### 40. What is a good approach for a signed-in dashboard?
Use a framework or client-rendered route that respects authentication and privacy. Decide whether data is loaded on the server or browser and ensure personalized output is never shared through an unsafe cache.

### 41. How do I avoid different initial dates or locale output?
Pass a stable timestamp, locale, and time-zone decision to the client from the server. Do not independently format “now” with potentially different settings on both sides during the initial render.

### 42. Can a route switch from SSG to SSR without changing its components?
Often the same component tree can be rendered in different modes, but the data APIs, caching behavior, request context, and framework configuration may change. Verify the framework's model.

### 43. What should the user see if JavaScript is slow or unavailable?
For SSR/SSG, meaningful initial HTML can remain readable before hydration. For CSR, provide an appropriate shell or loading state. Critical content and actions should not be hidden behind endless loading.

### 44. What is the main rule for hydration?
Render the same initial UI from the same inputs on both sides. Let React hydrate that exact snapshot, then make browser-only or user-specific changes through an intentional client update.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="094-internationalization-and-localization.md">094 - Internationalization, Localization, and Text Direction: Questions and Answers</a></td><td><a href="../lessons/095-frameworks-and-rendering-strategies.md">Lesson</a></td><td><a href="../examples/095-frameworks-and-rendering-strategies.md">Example</a></td><td><a href="096-react-native-and-shared-react-concepts.md">096 - React Native and the Shared React Mental Model: Questions and Answers</a></td></tr>
  </tbody>
</table>
