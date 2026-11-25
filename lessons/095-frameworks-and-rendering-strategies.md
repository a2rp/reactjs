# Frameworks, Client Rendering, SSR, and Static Generation in React

## Learning goals

After this lesson, you should be able to:

- Explain what a React framework adds around the React library.
- Distinguish client-side rendering (CSR), server-side rendering (SSR), and static site generation (SSG).
- Explain hydration and why the first client render must match server-generated HTML.
- Choose `createRoot` for an empty client-rendered root and `hydrateRoot` for existing React HTML.
- Compare the trade-offs of request-time and build-time rendering.
- Recognize what a production framework handles for routing, data, streaming, caching, and deployment.
- Select rendering per route instead of treating one mode as mandatory for an entire application.

## 1. React is a library; an app needs more pieces

React provides components, state, rendering, and UI-related APIs. A complete web application also needs choices for routing, asset builds, data loading, server requests, HTML generation, caching, error handling, and deployment.

A React framework integrates some or all of those pieces. It can also decide where each route or component renders. Frameworks do not all have the same capabilities or conventions, so check what the chosen tool supports.

For a production app, a framework can remove much of the manual setup. A small widget embedded in an existing page, a learning exercise, or an app with unusual constraints can still use React without adopting a full framework.

A framework does not necessarily mean that every request requires a long-running server. Many frameworks can build static files, render routes at request time, render some routes on the client, or combine these approaches.

## 2. Rendering is about when and where HTML is produced

These terms describe how initial HTML is produced and delivered. They are separate from the component model, router, and data APIs used to build the app.

### Client-side rendering (CSR)

The browser downloads an HTML shell and JavaScript. React runs in the browser and renders the app into an empty root:

```tsx
import { createRoot } from "react-dom/client";
import App from "./App";

const root = document.getElementById("root");
if (!root) throw new Error("Missing root element");

createRoot(root).render(<App />);
```

CSR can be a good fit for a private, highly interactive app or an embedded widget. A static host can serve its files. If the HTML shell is empty, visitors may see little content until the JavaScript arrives and runs. The app still needs loading, error, and no-JavaScript behavior appropriate to its purpose.

A single-page app often uses client routing, but CSR and SPA are not synonyms. Rendering location and navigation style are different design choices.

### Server-side rendering (SSR)

A server renders React to HTML for a request. The response can show meaningful content before the browser downloads and runs the client JavaScript. The browser then hydrates that HTML to attach React behavior.

SSR can be useful for request-specific pages or when early HTML matters. It adds server runtime work, data coordination, caching decisions, and potential time-to-first-byte costs. The HTML is not automatically personalized or cache-safe; the app must handle user-specific data carefully.

React provides server rendering APIs, but a framework usually coordinates routing, data loading, streaming, and the response shell. The simple `renderToString` API is useful for a basic demonstration, but it does not stream or wait for suspended content. Production setups often use framework-managed streaming APIs.

### Static site generation (SSG)

A build process renders a route to HTML before visitors request it. The generated file can be served from a static host or content delivery network. If the page is interactive, the browser hydrates the generated HTML; a purely static document may not need React running in the browser.

SSG is a strong fit for content that can be prepared ahead of time. A changed article or product page may require another build, unless the platform supports revalidation or another regeneration strategy.

Static generation avoids rendering that route on every request, but personalization and fresh request-time data need a separate strategy. Never bake one user's private content into a shared public artifact.

### Mixed rendering

A single app can use static output for public documentation, request-time rendering for frequently changing public pages, and client fetching for a personalized dashboard. The route and data requirements should decide the mode.

Revalidation, streaming, partial rendering, and route-level client/server boundaries are framework-specific mechanisms. They build on these rendering concepts but are not interchangeable across frameworks.

## 3. Hydration connects server HTML to React behavior

Server-rendered or statically generated React HTML is initially a snapshot. Hydration attaches React's event handling and resumes normal client updates while reusing that existing DOM:

```tsx
import { hydrateRoot } from "react-dom/client";
import App from "./App";

const root = document.getElementById("root");
if (!root) throw new Error("Missing root element");

hydrateRoot(root, <App />);
```

Use `hydrateRoot` when the root already contains HTML produced by the matching React tree. Calling `createRoot` on that pre-rendered root can replace the server HTML, waste work, reset user state, and cause a flash.

The first client render must match the server-rendered markup. React expects both sides to start from the same component tree, initial data, locale, and relevant rendering inputs.

### Common hydration mismatch causes

- Reading `window`, `document`, local storage, or other browser-only values during render.
- Calling `Date.now()`, generating random values, or formatting with different time zones/locales on each side.
- Fetching different data for server and client initial renders.
- Using different feature flags or authentication state.
- Producing invalid or differently nested markup.
- Rendering a different initial locale or direction in the browser.

Pass the server's initial data to the client and render from that same snapshot. If a value truly is browser-only, render a deterministic initial state and read the browser value after hydration in an effect, or use the framework's supported browser-only boundary. A two-pass update can cause a second render and visual change, so use it deliberately.

`suppressHydrationWarning` is a narrow escape hatch for an unavoidable text or attribute difference. It does not make mismatched trees correct and should not be used to hide general bugs.

## 4. Rendering and data loading belong together

The rendering mode determines when a component's initial HTML is generated, but the data-loading strategy determines which data is available at that time.

- For SSG, data must be available at build time or handled by a later client request.
- For SSR, request-dependent data can be loaded for that request, with appropriate authentication and caching rules.
- For CSR, the browser can load data after the app starts and should provide a loading state.

A data request in `useEffect` only runs in the browser after hydration. It cannot provide the initial HTML for SSR or SSG. Framework loaders, server components, server routes, or build scripts may provide data before rendering; their APIs depend on the framework.

Do not accidentally share a cached response containing private data. Cache keys and response headers must reflect which inputs affect the output.

## 5. Framework responsibilities to inspect

Before choosing a framework or rendering mode, check:

- How routes and layouts are declared.
- Where data loading and mutations run.
- How client JavaScript is divided and loaded.
- Whether SSR, static generation, streaming, and revalidation are supported.
- How errors, not-found states, redirects, and loading UI work.
- How forms, authentication, and secrets are handled.
- What can be cached and whether personalized HTML is protected.
- How the app is built, deployed, monitored, and upgraded.

A framework can manage React DOM APIs internally. Most application components should not call low-level server-rendering or hydration APIs themselves.

React Server Components are a separate component execution model that can work alongside server rendering and client rendering. “Server-rendered HTML” does not by itself mean that a component is a Server Component; see the dedicated Server Components lesson for that distinction.

## 6. A practical selection guide

| Page or product need | A likely starting point |
| --- | --- |
| Embedded widget inside a non-React site | Client rendering into the widget root |
| Private tool whose main content requires signed-in browser state | Client rendering or a framework route with client interactivity |
| Public content that changes infrequently | Static generation, with a rebuild or revalidation plan |
| Public or request-specific content that changes often | Server rendering or a framework's mixed rendering mode |
| Content should appear before client JavaScript | Server rendering or static generation, followed by hydration when interactive |
| Highly interactive page with public and private sections | Choose per route or boundary based on data, cache, and interaction needs |

These are starting points, not fixed rules. Caching, privacy, latency, data freshness, and deployment constraints may change the right choice.

## 7. Progressive enhancement and loading behavior

A rendered HTML response can provide readable content before JavaScript runs. Hydration then adds interactivity. This can improve the initial experience, but SSR does not automatically make a page fast: large bundles, slow server responses, blocking data, and excessive hydration work still matter.

Consider what visitors see while code loads, when a request fails, and when JavaScript is unavailable. Use meaningful HTML and metadata, give controls proper labels, and provide loading or error states. Do not assume that because a page is server-rendered all interactions work without client code.

## 8. Common mistakes

- **Calling `createRoot` on server HTML:** use `hydrateRoot` for an existing React-rendered root.
- **Assuming a framework always needs a server:** frameworks may generate static files or mix modes.
- **Treating CSR as identical to an SPA:** rendering and navigation are independent choices.
- **Expecting `useEffect` data to appear in server HTML:** effects run on the client after hydration.
- **Using different first-render inputs:** serialize or otherwise share the same initial data and settings.
- **Hiding hydration warnings instead of fixing the mismatch:** determine why output differs.
- **Using `renderToString` for streaming behavior:** it returns a string and does not wait for suspended content; use the framework's supported streaming path when needed.
- **Caching personalized HTML publicly:** isolate user-specific output and configure caching deliberately.
- **Assuming SSR guarantees search ranking or performance:** content, metadata, bundle size, server speed, and platform behavior still matter.
- **Generating private pages into public static files:** only build shared public data into public artifacts.
- **Treating Server Components and SSR as synonyms:** one describes a component execution model, the other describes when HTML is rendered.

## Summary

CSR renders in the browser; SSR renders HTML for a request; SSG generates HTML ahead of requests. SSR and interactive SSG typically hydrate that HTML with the same initial React tree and data. Choose a framework and rendering mode around each route's data freshness, interactivity, privacy, caching, and delivery needs. For production, let the framework coordinate routing, data, server rendering, streaming, hydration, and deployment.
