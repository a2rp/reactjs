# Caching, Deduplication, Invalidation, and Revalidation - Example

This Vite demo uses a small in-memory mock API and a shared query cache to demonstrate:

- two React components requesting the same query share one in-flight request;
- a successful result is reused while it is fresh;
- stale entries are loaded again on access;
- a mutation invalidates product queries and triggers revalidation;
- old data can remain visible while a refresh is running.

The mock API lives in the browser module, so changes disappear on a full page reload. This keeps the example self-contained; it is not a production API or cache implementation.

## Create the project

~~~sh
npm create vite@latest query-cache-demo -- --template react
cd query-cache-demo
npm install
~~~

Keep Vite's generated configuration and add or replace these files:

~~~text
query-cache-demo/
├── index.html
└── src/
    ├── App.jsx
    ├── main.jsx
    ├── mockProductsApi.js
    ├── productCache.js
    ├── styles.css
    └── useProducts.js
~~~

## `src/mockProductsApi.js`

This module stands in for a server. It delays list reads so request sharing is easy to see in the console, and it stores created products in memory:

~~~js
function wait(milliseconds) {
  return new Promise((resolve) => {
    setTimeout(resolve, milliseconds);
  });
}

let serverProducts = [
  { id: 1, name: "Notebook", category: "Stationery" },
  { id: 2, name: "Desk lamp", category: "Workspace" },
  { id: 3, name: "Water bottle", category: "Accessories" },
  { id: 4, name: "Pen set", category: "Stationery" },
  { id: 5, name: "Keyboard", category: "Workspace" },
  { id: 6, name: "Backpack", category: "Accessories" },
];

let nextId = 7;
let listRequestCount = 0;

export async function listProducts(query) {
  listRequestCount += 1;
  console.info(
    "[mock API] list request " + listRequestCount + " for query:",
    query,
  );

  await wait(700);

  const normalizedQuery = query.trim().toLowerCase();

  if (!normalizedQuery) {
    return serverProducts.map((product) => ({ ...product }));
  }

  return serverProducts
    .filter((product) => {
      const text = product.name + " " + product.category;
      return text.toLowerCase().includes(normalizedQuery);
    })
    .map((product) => ({ ...product }));
}

export async function createProduct(name) {
  await wait(300);

  const product = {
    id: nextId,
    name,
    category: "Added in demo",
  };

  nextId += 1;
  serverProducts = [product, ...serverProducts];

  return { ...product };
}
~~~

## `src/productCache.js`

This shared module caches successful results and in-flight Promises by normalized query. A result is fresh for eight seconds. Invalidating increments a generation so a request started before the invalidation cannot replace newer cache data when it finishes late:

~~~js
const FRESH_FOR_MS = 8000;
const entries = new Map();

export function productsKey(query) {
  return "products:" + query.trim().toLowerCase();
}

function getEntry(key) {
  let entry = entries.get(key);

  if (!entry) {
    entry = {
      data: undefined,
      updatedAt: 0,
      promise: null,
      generation: 0,
    };
    entries.set(key, entry);
  }

  return entry;
}

export function readProductsCache(key) {
  const entry = entries.get(key);
  const hasData = entry !== undefined && entry.data !== undefined;

  return {
    hasData,
    data: hasData ? entry.data : undefined,
    updatedAt: hasData ? entry.updatedAt : 0,
    isFresh:
      hasData && Date.now() - entry.updatedAt < FRESH_FOR_MS,
  };
}

export function loadCachedProducts(key, loader) {
  const entry = getEntry(key);

  if (entry.promise) {
    return entry.promise;
  }

  if (
    entry.data !== undefined &&
    Date.now() - entry.updatedAt < FRESH_FOR_MS
  ) {
    return Promise.resolve(entry.data);
  }

  const generation = entry.generation;

  const promise = Promise.resolve()
    .then(loader)
    .then((data) => {
      if (entry.generation === generation) {
        entry.data = data;
        entry.updatedAt = Date.now();
      }

      return data;
    })
    .finally(() => {
      if (
        entry.generation === generation &&
        entry.promise === promise
      ) {
        entry.promise = null;
      }
    });

  entry.promise = promise;
  return promise;
}

export function invalidateAllProducts() {
  for (const entry of entries.values()) {
    entry.updatedAt = 0;
    entry.generation += 1;
    entry.promise = null;
  }
}
~~~

The generation counter matters when an old request is still pending at the moment of invalidation. A later completion from that old generation may return to its original caller, but it cannot overwrite the cache's newer generation.

This teaching cache has no memory limit, persistence, account scoping, retry policy, or subscriber tracking. A production application should generally use its framework or a query library for these responsibilities.

## `src/useProducts.js`

Each panel subscribes through this hook. It keeps its view state local, while the shared cache provides data and in-flight request deduplication:

~~~jsx
import { useEffect, useState } from "react";
import { listProducts } from "./mockProductsApi.js";
import {
  loadCachedProducts,
  productsKey,
  readProductsCache,
} from "./productCache.js";

function stateFromCache(key) {
  const cached = readProductsCache(key);

  if (!cached.hasData) {
    return { key, status: "loading", data: [] };
  }

  return {
    key,
    status: cached.isFresh ? "success" : "refreshing",
    data: cached.data,
    updatedAt: cached.updatedAt,
  };
}

export function useProducts(query, refreshVersion) {
  const key = productsKey(query);
  const [state, setState] = useState(() => stateFromCache(key));

  useEffect(() => {
    let ignore = false;
    const cachedState = stateFromCache(key);

    setState(cachedState);

    loadCachedProducts(key, () => listProducts(query))
      .then((data) => {
        if (!ignore) {
          setState({
            key,
            status: "success",
            data,
            updatedAt: Date.now(),
          });
        }
      })
      .catch((error) => {
        if (ignore) {
          return;
        }

        const latestCache = readProductsCache(key);

        if (latestCache.hasData) {
          setState({
            key,
            status: "refresh-error",
            data: latestCache.data,
            error,
            updatedAt: latestCache.updatedAt,
          });
        } else {
          setState({ key, status: "error", data: [], error });
        }
      });

    return () => {
      ignore = true;
    };
  }, [key, query, refreshVersion]);

  // Never present data for an old query as data for the current query.
  return state.key === key
    ? state
    : { key, status: "loading", data: [] };
}
~~~

If two mounted panels ask for the same key, the first hook call starts a request and the second receives the same pending Promise. Each component still has its own display state.

## `src/App.jsx`

~~~jsx
import { useState } from "react";
import { createProduct } from "./mockProductsApi.js";
import { invalidateAllProducts } from "./productCache.js";
import { useProducts } from "./useProducts.js";

function ProductList({ products }) {
  return (
    <ul className="products">
      {products.map((product) => (
        <li className="product" key={product.id}>
          <strong>{product.name}</strong>
          <span>{product.category}</span>
        </li>
      ))}
    </ul>
  );
}

function ProductPanel({
  title,
  query,
  refreshVersion,
  onRefresh,
  compact = false,
}) {
  const state = useProducts(query, refreshVersion);

  return (
    <section className={compact ? "panel panel-compact" : "panel"}>
      <h2>{title}</h2>

      {state.status === "loading" && (
        <p className="loading" role="status">
          Loading products…
        </p>
      )}

      {state.status === "error" && (
        <div className="error" role="alert">
          <p>Products could not be loaded: {state.error.message}</p>
          <button type="button" onClick={onRefresh}>
            Retry
          </button>
        </div>
      )}

      {state.status === "refreshing" && (
        <p className="refreshing" role="status">
          Refreshing; showing cached results…
        </p>
      )}

      {state.status === "refresh-error" && (
        <div className="refresh-error" role="alert">
          Refresh failed. Showing the previous cached results.{" "}
          {state.error.message}
          <button type="button" onClick={onRefresh}>
            Retry
          </button>
        </div>
      )}

      {(state.status === "success" ||
        state.status === "refreshing" ||
        state.status === "refresh-error") &&
        (state.data.length === 0 ? (
          <p className="empty">No products match this query.</p>
        ) : compact ? (
          <div className="summary">
            <p>{state.data.length} matching products</p>
            <ul>
              {state.data.slice(0, 3).map((product) => (
                <li key={product.id}>{product.name}</li>
              ))}
            </ul>
          </div>
        ) : (
          <ProductList products={state.data} />
        ))}

      {state.status === "success" && (
        <p className="freshness">Using cached data while it is fresh.</p>
      )}
    </section>
  );
}

export default function App() {
  const [query, setQuery] = useState("");
  const [productName, setProductName] = useState("");
  const [refreshVersion, setRefreshVersion] = useState(0);
  const [mutationState, setMutationState] = useState({
    status: "idle",
    message: "",
  });

  function refreshProducts() {
    invalidateAllProducts();
    setRefreshVersion((version) => version + 1);
  }

  async function handleCreateProduct(event) {
    event.preventDefault();

    const name = productName.trim();

    if (!name) {
      return;
    }

    setMutationState({ status: "saving", message: "" });

    try {
      await createProduct(name);
      setProductName("");
      setMutationState({
        status: "success",
        message: "Product created. Cached product queries were invalidated.",
      });
      refreshProducts();
    } catch (error) {
      setMutationState({
        status: "error",
        message: "Could not create the product: " + error.message,
      });
    }
  }

  return (
    <main className="app">
      <p className="eyebrow">React query cache</p>
      <h1>Shared product data</h1>
      <p className="intro">
        Both panels use the same search key. The cache shares the first
        request, retains its result briefly, and refreshes it after changes.
      </p>

      <div className="toolbar">
        <label className="search">
          <span>Filter products</span>
          <input
            type="search"
            value={query}
            onChange={(event) => setQuery(event.target.value)}
            placeholder="Try “stationery” or “lamp”"
          />
        </label>
        <button className="secondary" type="button" onClick={refreshProducts}>
          Invalidate and refresh
        </button>
      </div>

      <form className="create-form" onSubmit={handleCreateProduct}>
        <label>
          <span>New product</span>
          <input
            value={productName}
            onChange={(event) => setProductName(event.target.value)}
            placeholder="For example, Drawing tablet"
          />
        </label>
        <button type="submit" disabled={mutationState.status === "saving"}>
          {mutationState.status === "saving" ? "Saving…" : "Create product"}
        </button>
      </form>

      {mutationState.message && (
        <p
          className={
            mutationState.status === "error"
              ? "mutation-message error"
              : "mutation-message"
          }
          role={mutationState.status === "error" ? "alert" : "status"}
        >
          {mutationState.message}
        </p>
      )}

      <div className="panels">
        <ProductPanel
          title="Catalog"
          query={query}
          refreshVersion={refreshVersion}
          onRefresh={refreshProducts}
        />
        <ProductPanel
          title="Second component using the same query"
          query={query}
          refreshVersion={refreshVersion}
          onRefresh={refreshProducts}
          compact
        />
      </div>

      <aside className="notes">
        <h2>Explore the cache</h2>
        <ol>
          <li>
            Open the browser console. Both panels use the same key, so they
            share one mock list request.
          </li>
          <li>
            Change the query, then return to a query you already used. A fresh
            cached result can be reused.
          </li>
          <li>
            Wait longer than eight seconds and revisit a query. Its next read
            revalidates the stale cache entry.
          </li>
          <li>
            Create a product. The mock API changes, the cache is invalidated,
            and active panels load the updated result.
          </li>
          <li>
            Search for “does-not-exist” to see that a successful empty result
            is also cached.
          </li>
        </ol>
        <p>
          The cache is in memory and resets on page reload. The mock API is also
          in memory, so created products reset too.
        </p>
      </aside>
    </main>
  );
}
~~~

The second component uses the same query key as the catalog. On an initial request, inspect the console: there should be one call to `listProducts` for that key, even though two panels use the result.

The manual refresh button invalidates known product keys and increments a version that makes each panel read again. The cache retains old data while the new request is pending, so the panel can label it as refreshing. A successful create mutation also invalidates and revalidates the product queries.

## `src/main.jsx`

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The #root container was not found.");
}

createRoot(container).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

Strict Mode is enabled. The shared in-flight Promise allows the development setup-and-cleanup check to reuse pending work for the same key.

## `src/styles.css`

~~~css
:root {
  font-family: Inter, ui-sans-serif, system-ui, sans-serif;
  color: #182337;
  background: #f3f6fb;
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

button,
input {
  font: inherit;
}

button {
  padding: 0.65rem 0.9rem;
  border: 0;
  border-radius: 0.55rem;
  color: white;
  background: #2857c5;
  font-weight: 650;
  cursor: pointer;
}

button:hover {
  background: #1e439d;
}

button:disabled {
  cursor: wait;
  opacity: 0.7;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #e3a700;
  outline-offset: 3px;
}

.app {
  width: min(100% - 2rem, 72rem);
  margin-inline: auto;
  padding-block: clamp(2rem, 6vw, 4.5rem);
}

.eyebrow {
  margin: 0 0 0.4rem;
  color: #52637d;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

h1 {
  margin: 0;
  font-size: clamp(2rem, 5vw, 3.2rem);
  letter-spacing: -0.04em;
}

.intro {
  max-width: 52rem;
  color: #526079;
  line-height: 1.6;
}

.toolbar,
.create-form {
  display: flex;
  align-items: end;
  flex-wrap: wrap;
  gap: 0.8rem;
  margin-top: 1.4rem;
}

.search,
.create-form label {
  display: grid;
  flex: 1;
  gap: 0.45rem;
  max-width: 36rem;
  font-weight: 700;
}

.search input,
.create-form input {
  width: 100%;
  padding: 0.72rem 0.85rem;
  border: 1px solid #b8c3d5;
  border-radius: 0.55rem;
  color: inherit;
  background: white;
}

.secondary {
  color: #254caa;
  border: 1px solid #b8c3d5;
  background: white;
}

.secondary:hover {
  background: #edf3ff;
}

.mutation-message {
  color: #176743;
}

.error,
.refresh-error {
  color: #8b1e2d;
}

.refresh-error {
  margin-block: 0.6rem;
  padding: 0.7rem;
  border-radius: 0.5rem;
  background: #fff0f1;
  line-height: 1.5;
}

.refresh-error button {
  display: block;
  margin-top: 0.5rem;
}

.panels {
  display: grid;
  grid-template-columns: 1.4fr 1fr;
  gap: 1rem;
  align-items: start;
  margin-top: 1.5rem;
}

.panel {
  min-height: 20rem;
  padding: clamp(1rem, 3vw, 1.4rem);
  border: 1px solid #d4dce8;
  border-radius: 0.85rem;
  background: white;
  box-shadow: 0 0.7rem 2rem rgb(24 35 55 / 6%);
}

.panel h2 {
  margin-top: 0;
  font-size: 1.2rem;
}

.panel-compact {
  background: #fbfcfe;
}

.loading,
.refreshing,
.empty {
  padding: 0.8rem 0.9rem;
  border-radius: 0.5rem;
  line-height: 1.5;
}

.loading {
  color: #254caa;
  background: #edf3ff;
}

.refreshing {
  color: #77520a;
  background: #fff6dc;
}

.empty {
  color: #526079;
  background: #f3f5f8;
}

.products {
  display: grid;
  gap: 0.55rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.product {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.75rem;
  border: 1px solid #e0e5ed;
  border-radius: 0.5rem;
}

.product span {
  color: #526079;
  font-size: 0.9rem;
}

.summary ul {
  padding-left: 1.2rem;
  line-height: 1.7;
}

.freshness {
  margin-bottom: 0;
  color: #526079;
  font-size: 0.85rem;
}

.notes {
  max-width: 62rem;
  margin-top: 1.5rem;
  color: #526079;
  line-height: 1.6;
}

.notes h2 {
  color: #182337;
  font-size: 1.1rem;
}

@media (max-width: 720px) {
  .panels {
    grid-template-columns: 1fr;
  }

  .toolbar,
  .create-form {
    align-items: stretch;
    flex-direction: column;
  }

  .toolbar > button,
  .create-form > button {
    align-self: start;
  }

  .panel {
    min-height: 0;
  }
}
~~~

## `index.html`

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="Explore shared query caching, deduplication, invalidation, and revalidation in React."
    />
    <title>React query cache</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## Run and explore

Start Vite:

~~~sh
npm run dev
~~~

Use the steps under **Explore the cache**. You can inspect the console to see how many mock list calls are made for the current key. Change the query quickly; each distinct key has its own entry. Return to a recent key to observe a fresh cache hit, then wait longer than eight seconds before revisiting it to see revalidation.

The **Create product** action updates the mock data in memory, invalidates product entries, and refreshes the active query. Search for the new product by name if the current filter does not match it.

## Common mistakes

- Assuming the two mounted panels have independent requests; they share one key and one cache entry.
- Forgetting a query input in the cache key.
- Expecting invalidation to fetch by itself without a cache policy or a subsequent read.
- Treating the eight-second freshness period as the memory-retention period.
- Assuming this map is persistent or safe for authenticated production data.
- Believing a browser reload preserves this JavaScript cache or mock product data.
- Letting a request from before invalidation overwrite newer cache data.
- Confusing HTTP response caching with this in-memory query cache.

## Key takeaway

A shared query cache needs complete keys, in-flight request deduplication, successful data storage, freshness rules, and mutation invalidation. Revalidation can keep useful old data on screen while it refreshes. This demo makes those mechanics visible; use a production data library or framework to handle the many lifecycle and security details a real application needs.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="075-loading-empty-success-and-error-states.md">075 - Loading, Empty, Success, and Error States - Example</a></td><td><a href="../lessons/076-caching-and-revalidation.md">Lesson</a></td><td><a href="../qnas/076-caching-and-revalidation.md">Q&amp;A</a></td><td><a href="077-async-ui-and-framework-data-loading.md">077 - Async UI, Suspense, and Framework Data Loading - Example</a></td></tr>
  </tbody>
</table>
