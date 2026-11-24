# Example: Memoization and useId

This runnable React 19 app is a small searchable catalog. It demonstrates:

- **useId** to connect search and category labels with their inputs and help text;
- **useMemo** to keep the filtered product array stable until the query or category changes;
- **useCallback** to keep the selection handler stable;
- **memo** to let the results grid and product cards skip renders when their props are unchanged;
- state for the selected item and visual theme, because both affect the visible interface.

The catalog is intentionally small. Memoization here demonstrates identity and API behavior; it does not claim that a small filter needs optimization in a real app. The example makes no external requests.

## Create the project

~~~sh
npm create vite@latest memoization-and-ids-demo -- --template react
cd memoization-and-ids-demo
npm install
npm install react@19 react-dom@19
~~~

Replace the generated files with the following, then start the development server:

~~~sh
npm run dev
~~~

## Project files

~~~text
memoization-and-ids-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f6fb" />
    <title>React Memoization and useId Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## src/main.jsx

~~~jsx
import React from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
~~~

## src/App.jsx

~~~jsx
import { memo, useCallback, useId, useMemo, useState } from "react";

const PRODUCTS = [
  { id: "book-01", name: "The Quiet Forest", category: "Books", detail: "A field guide to woodland plants", price: 18 },
  { id: "book-02", name: "Small Habits", category: "Books", detail: "A practical journal for daily notes", price: 14 },
  { id: "book-03", name: "Lines and Forms", category: "Books", detail: "An introduction to visual composition", price: 22 },
  { id: "tool-01", name: "Focus Timer", category: "Tools", detail: "A simple desk timer for focused work", price: 12 },
  { id: "tool-02", name: "Pocket Notebook", category: "Tools", detail: "A compact notebook for quick ideas", price: 8 },
  { id: "tool-03", name: "Desk Organizer", category: "Tools", detail: "A tray for everyday writing tools", price: 26 },
  { id: "design-01", name: "Shape Study", category: "Design", detail: "A set of geometric practice cards", price: 16 },
  { id: "design-02", name: "Color Notes", category: "Design", detail: "A workbook for building color palettes", price: 19 },
  { id: "design-03", name: "Grid Pad", category: "Design", detail: "A reusable pad for layout sketches", price: 11 },
];

const CATEGORIES = ["All", "Books", "Design", "Tools"];

function filterProducts(products, query, category) {
  const normalizedQuery = query.trim().toLowerCase();

  return products.filter((product) => {
    const matchesQuery =
      normalizedQuery.length === 0 ||
      `${product.name} ${product.detail}`.toLowerCase().includes(normalizedQuery);
    const matchesCategory = category === "All" || product.category === category;

    return matchesQuery && matchesCategory;
  });
}

function SearchField({ query, onQueryChange }) {
  const id = useId();
  const inputId = `${id}-input`;
  const helpId = `${id}-help`;

  return (
    <div className="field">
      <label className="field__label" htmlFor={inputId}>
        Search products
      </label>
      <input
        className="field__control"
        id={inputId}
        type="search"
        value={query}
        onChange={(event) => onQueryChange(event.target.value)}
        placeholder="Try “notebook” or “color”"
        aria-describedby={helpId}
      />
      <p className="field__help" id={helpId}>
        Search by product name or description.
      </p>
    </div>
  );
}

function CategorySelect({ category, onCategoryChange }) {
  const id = useId();
  const selectId = `${id}-select`;
  const helpId = `${id}-help`;

  return (
    <div className="field">
      <label className="field__label" htmlFor={selectId}>
        Category
      </label>
      <select
        className="field__control"
        id={selectId}
        value={category}
        onChange={(event) => onCategoryChange(event.target.value)}
        aria-describedby={helpId}
      >
        {CATEGORIES.map((name) => (
          <option key={name} value={name}>
            {name}
          </option>
        ))}
      </select>
      <p className="field__help" id={helpId}>
        Choose one category or show every item.
      </p>
    </div>
  );
}

const ProductCard = memo(function ProductCard({ product, isSelected, onSelect }) {
  return (
    <article className={`product-card${isSelected ? " product-card--selected" : ""}`}>
      <div className="product-card__topline">
        <span className="category-tag">{product.category}</span>
        <span className="product-price">${product.price}</span>
      </div>
      <h3>{product.name}</h3>
      <p className="product-card__detail">{product.detail}</p>
      <button
        className={isSelected ? "button button--selected" : "button button--card"}
        type="button"
        aria-pressed={isSelected}
        onClick={() => onSelect(product.id)}
      >
        {isSelected ? "Selected" : "Select item"}
      </button>
    </article>
  );
});

const ProductGrid = memo(function ProductGrid({ products, selectedId, onSelect }) {
  if (products.length === 0) {
    return (
      <p className="empty-state">
        No products match those filters. Try a different search or category.
      </p>
    );
  }

  return (
    <div className="product-grid">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          isSelected={product.id === selectedId}
          onSelect={onSelect}
        />
      ))}
    </div>
  );
});

export default function App() {
  const [query, setQuery] = useState("");
  const [category, setCategory] = useState("All");
  const [selectedId, setSelectedId] = useState(null);
  const [isDark, setIsDark] = useState(false);

  const visibleProducts = useMemo(
    () => filterProducts(PRODUCTS, query, category),
    [query, category],
  );

  const handleSelect = useCallback((productId) => {
    setSelectedId(productId);
  }, []);

  const selectedProduct = PRODUCTS.find((product) => product.id === selectedId);

  return (
    <main className={`app-shell${isDark ? " app-shell--dark" : ""}`}>
      <div className="app-content">
        <header className="page-header">
          <div>
            <p className="eyebrow">React Hooks and performance</p>
            <h1>Mini catalog</h1>
            <p className="page-intro">
              Filter a small catalog, select an item, and toggle the theme. The
              controls use accessible IDs; memoized results receive stable props.
            </p>
          </div>
          <button
            className="button button--theme"
            type="button"
            aria-pressed={isDark}
            onClick={() => setIsDark((current) => !current)}
          >
            {isDark ? "Use light theme" : "Use dark theme"}
          </button>
        </header>

        <section className="filter-panel" aria-label="Catalog filters">
          <SearchField query={query} onQueryChange={setQuery} />
          <CategorySelect category={category} onCategoryChange={setCategory} />
        </section>

        <section className="results-section" aria-labelledby="results-heading">
          <div className="results-heading">
            <div>
              <p className="eyebrow">{visibleProducts.length} matching items</p>
              <h2 id="results-heading">Browse products</h2>
            </div>
            <p className="selection-status" role="status" aria-live="polite">
              {selectedProduct
                ? `Selected: ${selectedProduct.name}`
                : "No item selected"}
            </p>
          </div>

          <ProductGrid
            products={visibleProducts}
            selectedId={selectedId}
            onSelect={handleSelect}
          />
        </section>

        <aside className="learning-note" aria-labelledby="why-heading">
          <h2 id="why-heading">What to notice</h2>
          <ul>
            <li>
              Search and category labels use IDs created with <code>useId</code>.
            </li>
            <li>
              <code>useMemo</code> keeps the filtered array identity when only the
              theme changes.
            </li>
            <li>
              <code>useCallback</code> keeps the selection callback stable for the
              memoized grid and cards.
            </li>
            <li>
              Selection is state because it changes visible buttons and status
              text. Product keys come from each product’s data ID.
            </li>
          </ul>
        </aside>

        <footer className="page-footer">
          This tiny catalog does not need memoization for speed. In a real app,
          profile first and optimize only work that matters.
        </footer>
      </div>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #1a2435;
  background: #f4f6fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-weight: 400;
}

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input,
select {
  font: inherit;
}

button,
select {
  cursor: pointer;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #8ab7ff;
  outline-offset: 3px;
}

.app-shell {
  min-height: 100vh;
  padding: 52px 18px 40px;
  color: #1a2435;
  background: #f4f6fb;
  transition: color 160ms ease, background-color 160ms ease;
}

.app-content {
  width: min(100%, 1040px);
  margin: 0 auto;
}

.page-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 22px;
  margin-bottom: 28px;
}

.eyebrow {
  margin-bottom: 7px;
  color: #60708a;
  font-size: 0.75rem;
  font-weight: 800;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2,
h3,
p {
  overflow-wrap: anywhere;
}

h1 {
  font-size: clamp(2.1rem, 6vw, 3.2rem);
  line-height: 1.05;
  letter-spacing: -0.05em;
}

h2 {
  font-size: 1.35rem;
  line-height: 1.2;
}

h3 {
  font-size: 1.05rem;
  line-height: 1.3;
}

.page-intro {
  max-width: 680px;
  margin-top: 12px;
  color: #5a6880;
  font-size: 1rem;
  line-height: 1.65;
}

.button {
  min-height: 42px;
  padding: 9px 13px;
  border: 1px solid #c8d2e0;
  border-radius: 10px;
  color: #26354e;
  background: #fff;
  font-weight: 700;
  transition: background-color 120ms ease, border-color 120ms ease,
    transform 120ms ease;
}

.button:hover {
  border-color: #9aabc2;
  background: #f6f8fc;
}

.button:active {
  transform: translateY(1px);
}

.button--theme {
  flex: 0 0 auto;
}

.filter-panel {
  display: grid;
  grid-template-columns: minmax(0, 2fr) minmax(190px, 1fr);
  gap: 18px;
  padding: 20px;
  border: 1px solid #dce3ed;
  border-radius: 16px;
  background: #fff;
  box-shadow: 0 9px 28px rgb(34 52 79 / 5%);
}

.field__label {
  display: block;
  margin-bottom: 7px;
  color: #344158;
  font-size: 0.9rem;
  font-weight: 750;
}

.field__control {
  width: 100%;
  min-height: 44px;
  padding: 10px 12px;
  border: 1px solid #cbd5e2;
  border-radius: 10px;
  color: #1a2435;
  background: #fff;
}

.field__control::placeholder {
  color: #8591a3;
}

.field__help {
  margin-top: 6px;
  color: #6b7890;
  font-size: 0.82rem;
  line-height: 1.4;
}

.results-section {
  margin-top: 32px;
}

.results-heading {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 16px;
}

.results-heading .eyebrow {
  margin-bottom: 5px;
}

.selection-status {
  color: #596880;
  font-size: 0.9rem;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 14px;
}

.product-card {
  display: flex;
  min-height: 215px;
  flex-direction: column;
  align-items: flex-start;
  padding: 17px;
  border: 1px solid #dce3ed;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 6px 18px rgb(34 52 79 / 4%);
}

.product-card--selected {
  border-color: #5a84d6;
  box-shadow: 0 0 0 2px rgb(90 132 214 / 18%);
}

.product-card__topline {
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  margin-bottom: 17px;
}

.category-tag {
  padding: 5px 8px;
  border-radius: 999px;
  color: #30548c;
  background: #edf4ff;
  font-size: 0.73rem;
  font-weight: 750;
}

.product-price {
  color: #48566d;
  font-size: 0.85rem;
  font-weight: 750;
}

.product-card__detail {
  margin-top: 7px;
  color: #627089;
  font-size: 0.9rem;
  line-height: 1.45;
}

.button--card,
.button--selected {
  margin-top: auto;
  padding: 8px 11px;
  font-size: 0.83rem;
}

.button--selected {
  border-color: #285cb5;
  color: #fff;
  background: #285cb5;
}

.button--selected:hover {
  border-color: #194b9f;
  background: #194b9f;
}

.empty-state {
  padding: 22px;
  border: 1px dashed #b9c6d8;
  border-radius: 14px;
  color: #56647c;
  background: rgb(255 255 255 / 70%);
  line-height: 1.5;
}

.learning-note {
  margin-top: 24px;
  padding: 20px;
  border: 1px solid #dce3ed;
  border-radius: 14px;
  background: #fff;
}

.learning-note ul {
  display: grid;
  gap: 8px;
  margin: 13px 0 0 20px;
  color: #53627a;
  line-height: 1.5;
}

code {
  padding: 1px 4px;
  border-radius: 4px;
  color: #244b87;
  background: #edf4ff;
  font-size: 0.92em;
}

.page-footer {
  margin-top: 20px;
  color: #63718a;
  font-size: 0.87rem;
  line-height: 1.5;
}

.app-shell--dark {
  color: #ecf1fa;
  background: #121925;
}

.app-shell--dark .eyebrow,
.app-shell--dark .page-intro,
.app-shell--dark .selection-status,
.app-shell--dark .field__help,
.app-shell--dark .page-footer {
  color: #a6b3c8;
}

.app-shell--dark .filter-panel,
.app-shell--dark .product-card,
.app-shell--dark .learning-note {
  border-color: #344155;
  background: #1c2737;
}

.app-shell--dark .field__label,
.app-shell--dark .product-price {
  color: #e3eaf5;
}

.app-shell--dark .field__control,
.app-shell--dark .button {
  border-color: #46546a;
  color: #edf2fa;
  background: #202d40;
}

.app-shell--dark .field__control::placeholder {
  color: #a1aec1;
}

.app-shell--dark .product-card__detail,
.app-shell--dark .learning-note ul {
  color: #b4c0d2;
}

.app-shell--dark .category-tag {
  color: #c4d9ff;
  background: #2b4264;
}

.app-shell--dark .button--selected {
  border-color: #6b9bea;
  color: #0f1a2a;
  background: #9bbcff;
}

.app-shell--dark .empty-state {
  border-color: #52627a;
  color: #c1cce0;
  background: #1c2737;
}

@media (max-width: 760px) {
  .product-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 560px) {
  .app-shell {
    padding: 32px 12px;
  }

  .page-header {
    align-items: stretch;
    flex-direction: column;
  }

  .button--theme {
    align-self: flex-start;
  }

  .filter-panel {
    grid-template-columns: 1fr;
    gap: 14px;
    padding: 16px;
  }

  .product-grid {
    grid-template-columns: 1fr;
  }

  .product-card {
    min-height: 190px;
  }
}
~~~

## What each API is doing

### useId: connect the controls

**SearchField** and **CategorySelect** each call **useId**. Their labels use **htmlFor** to identify the matching control, and **aria-describedby** points to the explanatory text. Each component instance gets its own IDs.

The product cards use **product.id** as the list key because it is stable data identity. They do not use **useId** for keys.

### useMemo: keep the derived array stable

The visible products depend on **query** and **category**, so those are the dependencies. A theme change does not change either value; the array identity therefore stays stable and can help **ProductGrid** skip a render.

The calculation itself is deliberately small. In a real application, do not assume that memoizing a tiny filter makes the interface faster. Profile the actual interaction first.

### useCallback and memo: demonstrate prop identity

**handleSelect** depends only on React’s stable state setter, so its dependency array is empty. The stable callback and stable **visibleProducts** array are passed to the memoized grid. When only the theme changes, those props and **selectedId** stay the same, so React can reuse the grid’s prior result.

When a product is selected, **selectedId** changes. The grid must render to pass the new selection state. Each memoized card compares its own props; the previously selected and newly selected cards receive changed **isSelected** values, while other cards can skip rendering.

### State: values that affect what is visible

The theme and selected product are React state because they change the visible interface. The search query and category are also state because they control visible results. Memoization does not replace these state updates.

## useId with server rendering or multiple roots

This demo has one client-rendered React root, so it uses the default ID prefix. If one page has multiple independent roots, give each root a different prefix:

~~~jsx
const root = createRoot(container, { identifierPrefix: "catalog-" });
root.render(<App />);
~~~

For server-rendered content, the server and hydration call must use matching prefixes. The server and client must also render the same component tree so generated IDs line up.

## React Compiler note

This project uses explicit **memo**, **useMemo**, and **useCallback** to teach manual identity-based optimization. It does not configure React Compiler. When a project has the Compiler configured, it may automatically memoize many components, values, and functions. Check the project’s actual compiler setup and diagnostics rather than assuming it is enabled.

## Try these changes

1. Remove **useMemo** and check whether the app still works. It should; the filter is not required for correctness.
2. Remove **useCallback** and check that selection still works. It should; the difference is callback identity.
3. Remove **memo** from **ProductGrid** and **ProductCard**, then compare behavior and profile an interaction in a realistic larger app.
4. Add a second **SearchField** in a separate panel and inspect that its label and help IDs remain distinct.
5. Add a product and give it a stable data ID. Use that ID for **key**, not a Hook-generated ID or array position.
6. Make the filter calculation intentionally expensive only in a local experiment, measure before and after, then remove the artificial slowdown.
