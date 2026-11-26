# React Rendering Modes: One Article Page as CSR, SSR, and SSG

This example uses one interactive article component in three delivery modes:

- **CSR:** the browser renders into an empty root with `createRoot`.
- **SSR:** a request handler renders initial HTML with React, embeds the same article data, and the browser hydrates with `hydrateRoot`.
- **SSG:** a build script generates the same HTML ahead of requests and the browser hydrates it with the same data.

The small SSR and SSG demonstrations use `renderToString` to make the distinction visible. It is not a streaming renderer and does not wait for suspended content; production apps should use their framework's supported rendering and asset-manifest pipeline.

## File structure

```text
src/
├── ArticlePage.tsx
├── article.ts
├── client-only.tsx
├── hydrate.tsx
└── styles.css
server/
└── render-document.tsx
scripts/
└── generate-static.ts
```

## `src/article.ts`

```ts
export type Article = {
  id: string;
  title: string;
  summary: string;
  updatedAt: string;
  updatedAtLabel: string;
};

// Public sample content used by both server and client builds.
export const article: Article = {
  id: "react-rendering",
  title: "Where React renders",
  summary:
    "The same component can start in the browser, render for a request, or be generated during a build.",
  updatedAt: "2026-11-25",
  updatedAtLabel: "November 25, 2026",
};
```

## `src/ArticlePage.tsx`

```tsx
import { useState } from "react";
import type { Article } from "./article";

export function ArticlePage({ article }: { article: Article }) {
  const [isSaved, setIsSaved] = useState(false);

  return (
    <main className="page-shell">
      <article className="article-card">
        <p className="eyebrow">Field Notes · React</p>
        <h1>{article.title}</h1>
        <p className="summary">{article.summary}</p>
        <p className="updated">
          Updated <time dateTime={article.updatedAt}>{article.updatedAtLabel}</time>
        </p>
        <button
          className="save-button"
          type="button"
          aria-pressed={isSaved}
          onClick={() => setIsSaved((saved) => !saved)}
        >
          {isSaved ? "Saved" : "Save article"}
        </button>
      </article>
    </main>
  );
}
```

The component has no environment checks and no time-dependent rendering. Its first output is the same on the server and in the browser. The saved state also begins as `false` in both places; the button becomes interactive after CSR rendering or hydration.

## Option A: client-side rendering

### `index.html` for CSR

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Field Notes</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/client-only.tsx"></script>
  </body>
</html>
```

### `src/client-only.tsx`

```tsx
import { createRoot } from "react-dom/client";
import { ArticlePage } from "./ArticlePage";
import { article } from "./article";
import "./styles.css";

const rootElement = document.getElementById("root");
if (!rootElement) {
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(<ArticlePage article={article} />);
```

The root is empty, so `createRoot` renders the first UI in the browser. The module-script path is an example for a development build tool; use the built asset path in production.

## Shared document renderer for SSR and SSG

### `server/render-document.tsx`

```tsx
import { renderToString } from "react-dom/server";
import type { Article } from "../src/article";
import { ArticlePage } from "../src/ArticlePage";

function serializeForScript(value: unknown) {
  // Escaping `<` prevents data from ending an inline script element.
  return JSON.stringify(value).replace(/</g, "\\u003c");
}

export function renderArticleDocument(
  article: Article,
  clientEntry: string,
) {
  const markup = renderToString(<ArticlePage article={article} />);
  const serializedArticle = serializeForScript(article);

  return `<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Field Notes · React</title>
    <link rel="stylesheet" href="/assets/styles.css" />
  </head>
  <body>
    <div id="root">${markup}</div>
    <script id="initial-article" type="application/json">${serializedArticle}</script>
    <script type="module" src="${clientEntry}"></script>
  </body>
</html>`;
}
```

React escapes article text while rendering markup. The JSON payload is separately escaped for its inline script context. In a real build, use the framework's HTML and asset-manifest handling rather than hand-building a document shell.

## Option B: server-side rendering per request

A request handler loads the correct article, calls `renderArticleDocument`, and returns the resulting HTML. The following is the core of a Node HTTP handler; the project must compile the server-side TSX and serve the referenced browser assets:

```tsx
import { createServer } from "node:http";
import { article } from "../src/article";
import { renderArticleDocument } from "./render-document";

const server = createServer((_request, response) => {
  const html = renderArticleDocument(article, "/assets/hydrate.js");
  response.writeHead(200, { "content-type": "text/html; charset=utf-8" });
  response.end(html);
});

server.listen(3000);
```

A real route would load the article using request context and handle not-found, error, authentication, and cache behavior. Do not cache user-specific HTML as shared public output.

## `src/hydrate.tsx`

The SSR response includes the same article data used to produce its HTML. The client reads and validates that payload, then hydrates the existing root:

```tsx
import { hydrateRoot } from "react-dom/client";
import type { Article } from "./article";
import { ArticlePage } from "./ArticlePage";
import "./styles.css";

function isArticle(value: unknown): value is Article {
  if (typeof value !== "object" || value === null) return false;

  const record = value as Record<string, unknown>;
  return (
    typeof record.id === "string" &&
    typeof record.title === "string" &&
    typeof record.summary === "string" &&
    typeof record.updatedAt === "string" &&
    typeof record.updatedAtLabel === "string"
  );
}

function readInitialArticle(): Article {
  const dataElement = document.getElementById("initial-article");
  if (!dataElement?.textContent) {
    throw new Error("The initial article data was not found.");
  }

  const value: unknown = JSON.parse(dataElement.textContent);
  if (!isArticle(value)) {
    throw new Error("The initial article data has an unexpected shape.");
  }
  return value;
}

const rootElement = document.getElementById("root");
if (!rootElement) {
  throw new Error("The root element was not found.");
}

const initialArticle = readInitialArticle();
hydrateRoot(rootElement, <ArticlePage article={initialArticle} />);
```

Do not replace `hydrateRoot` with `createRoot` here. The server already filled the root. Hydration expects the same article data and initial `isSaved` state that created the markup.

## Option C: static generation at build time

The same render function can generate a static article document before deployment. A build script can write the HTML to the route's output path:

### `scripts/generate-static.ts`

```ts
import { mkdir, writeFile } from "node:fs/promises";
import { article } from "../src/article";
import { renderArticleDocument } from "../server/render-document";

const outputDirectory = "dist/notes/react-rendering";
await mkdir(outputDirectory, { recursive: true });

const html = renderArticleDocument(article, "/assets/hydrate.js");
await writeFile(`${outputDirectory}/index.html`, html, "utf8");
```

The output contains article HTML and a client entry, so it is interactive after hydration. To make it a noninteractive static document, omit the bootstrap data and client script. For several routes, run the generator for each public article and write each result to its route path. In a real build, resolve the stylesheet and JavaScript paths from the bundler's generated asset manifest.

## `src/styles.css`

```css
:root {
  color: #182033;
  background: #f3f6fb;
  font-family: Inter, ui-sans-serif, system-ui, sans-serif;
  line-height: 1.6;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button {
  font: inherit;
}

.page-shell {
  display: grid;
  min-height: 100vh;
  place-items: center;
  padding: 1.5rem;
}

.article-card {
  width: min(100%, 680px);
  padding: clamp(1.25rem, 5vw, 2.5rem);
  border: 1px solid #d9e0eb;
  border-radius: 1.1rem;
  background: #fff;
  box-shadow: 0 18px 50px rgb(24 42 82 / 8%);
}

.eyebrow {
  margin: 0 0 0.7rem;
  color: #3157d5;
  font-size: 0.75rem;
  font-weight: 850;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.article-card h1 {
  margin: 0;
  font-size: clamp(2rem, 7vw, 3.5rem);
  letter-spacing: -0.055em;
  line-height: 1.08;
}

.summary {
  margin-block: 1rem;
  color: #596579;
  font-size: 1.1rem;
}

.updated {
  color: #667187;
  font-size: 0.9rem;
}

.save-button {
  min-height: 2.8rem;
  margin-block-start: 1.2rem;
  padding-inline: 1rem;
  border: 0;
  border-radius: 0.7rem;
  color: #fff;
  background: #3157d5;
  font-weight: 750;
  cursor: pointer;
}

.save-button:hover {
  background: #2445b5;
}

.save-button:focus-visible {
  outline: 3px solid #7898ff;
  outline-offset: 3px;
}
```

## Compare the three outputs

| Mode | When HTML is generated | Client entry | React API in this example |
| --- | --- | --- | --- |
| CSR | In the browser after JavaScript loads | Starts the app | `createRoot` |
| SSR | For each request | Hydrates the response | `renderToString` for the demo, then `hydrateRoot` |
| SSG | During the build | Hydrates the generated file | `renderToString` for this small synchronous demo, then `hydrateRoot` |

The same `ArticlePage` is used in all cases. Rendering mode changes when the initial markup is produced, not the component's behavior or its semantic HTML.

## Try it

1. Run the CSR entry with an empty root. The page appears after the client bundle runs.
2. Request the SSR output with JavaScript disabled. The article content should still be present in the HTML response.
3. Enable JavaScript and load the SSR response. The existing markup should remain while the save button becomes interactive.
4. Generate the SSG output, then serve the generated folder as static files. The article should be present before hydration.
5. Click **Save article** in each mode. It should toggle to **Saved** without replacing the entire page.
6. Change the server article title and make sure the same data is sent to hydration. Different server and client initial data should produce a hydration mismatch.
7. Inspect the generated output's client and stylesheet paths. In a production build, they must match the actual asset manifest.

## Production notes

- This example uses `renderToString` to keep the flow small. It does not stream or wait for suspended content; use the framework's server-rendering path for a production app.
- The SSG script generates one public route. A real build should enumerate routes, handle failures, and decide whether each output is public.
- Use the same initial props and state on the server and client. Avoid `Date.now()`, random values, browser storage, and environment-dependent formatting during the initial render.
- Keep the HTML shell and asset URLs under the framework or bundler's ownership when possible.
- Rendering HTML on the server does not by itself protect personalized data or make the page interactive. Hydration and safe cache policies are separate requirements.

## Key takeaways

- `createRoot` renders into an empty client root; `hydrateRoot` reuses matching server or build-time HTML.
- SSR renders for a request; SSG renders during a build; both can hydrate the same component tree.
- One React component can support different delivery modes when its initial inputs are deterministic.
- A framework usually handles streaming, routing, data, assets, and hydration details that this teaching example shows manually.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="094-internationalization-and-localization.md">094 - React Internationalization and RTL: Locale Dashboard Example</a></td><td><a href="../lessons/095-frameworks-and-rendering-strategies.md">Lesson</a></td><td><a href="../qnas/095-frameworks-and-rendering-strategies.md">Q&amp;A</a></td><td><a href="096-react-native-and-shared-react-concepts.md">096 - React Native Example: A Native Task List with Shared React Logic</a></td></tr>
  </tbody>
</table>
