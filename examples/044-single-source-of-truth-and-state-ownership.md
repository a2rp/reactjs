# Single Source of Truth and State Ownership — Example

This example lets a user choose a product and shows that selection in three places:

- the picker marks the selected product;
- a details panel shows its information;
- a summary repeats the selected name and price.

ProductPage owns one selectedProductId. It derives the product from the ID and product list, then passes the same selection to every consumer. ProductPicker separately owns its search query because that value is only used inside the picker.

## Project files

In a Vite React project, use these files:

~~~text
index.html
src/
  App.jsx
  main.jsx
  styles.css
~~~

## src/App.jsx

~~~jsx
import { useState } from "react";

const products = [
  {
    id: "notebook",
    name: "Hardcover notebook",
    category: "Stationery",
    price: 12,
    description: "A durable notebook for notes, plans, and sketches.",
  },
  {
    id: "desk-lamp",
    name: "Adjustable desk lamp",
    category: "Workspace",
    price: 38,
    description: "A focused light with an adjustable arm for a desk.",
  },
  {
    id: "water-bottle",
    name: "Insulated water bottle",
    category: "Everyday",
    price: 24,
    description: "A reusable bottle that keeps drinks at a steady temperature.",
  },
  {
    id: "keyboard",
    name: "Compact keyboard",
    category: "Workspace",
    price: 64,
    description: "A compact keyboard that leaves more room on the desk.",
  },
];

const currency = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

function ProductPicker({
  products,
  selectedProductId,
  onSelectProduct,
}) {
  const [query, setQuery] = useState("");

  const normalizedQuery = query.trim().toLowerCase();
  const visibleProducts = products.filter((product) =>
    product.name.toLowerCase().includes(normalizedQuery)
  );

  return (
    <section className="card" aria-labelledby="picker-heading">
      <div className="card-heading">
        <div>
          <p className="eyebrow">Choose one</p>
          <h2 id="picker-heading">Products</h2>
        </div>
        <span className="result-count">
          {visibleProducts.length}{" "}
          {visibleProducts.length === 1 ? "result" : "results"}
        </span>
      </div>

      <label className="search-field">
        <span>Filter products by name</span>
        <input
          type="search"
          value={query}
          onChange={(event) => setQuery(event.target.value)}
          placeholder="Try “lamp”"
        />
      </label>

      {visibleProducts.length > 0 ? (
        <ul className="product-list">
          {visibleProducts.map((product) => {
            const isSelected = product.id === selectedProductId;

            return (
              <li key={product.id}>
                <button
                  className={
                    isSelected
                      ? "product-option product-option-selected"
                      : "product-option"
                  }
                  type="button"
                  aria-pressed={isSelected}
                  onClick={() => onSelectProduct(product.id)}
                >
                  <span>
                    <strong>{product.name}</strong>
                    <span className="option-meta">
                      {product.category} · {currency.format(product.price)}
                    </span>
                  </span>
                  <span className="selection-mark" aria-hidden="true">
                    {isSelected ? "Selected" : "Select"}
                  </span>
                </button>
              </li>
            );
          })}
        </ul>
      ) : (
        <p className="empty-state">
          No product names match “{query}”. Try another search.
        </p>
      )}

      <p className="card-note">
        The search is local to this picker. The selected product is controlled
        by ProductPage.
      </p>
    </section>
  );
}

function ProductDetails({ product }) {
  if (!product) {
    return (
      <section className="card" aria-labelledby="details-heading">
        <h2 id="details-heading">Product details</h2>
        <p className="empty-state">Choose an available product to see details.</p>
      </section>
    );
  }

  return (
    <section className="card" aria-labelledby="details-heading">
      <p className="eyebrow">Details</p>
      <h2 id="details-heading">{product.name}</h2>
      <p className="description">{product.description}</p>

      <dl className="details-list">
        <div>
          <dt>Category</dt>
          <dd>{product.category}</dd>
        </div>
        <div>
          <dt>Price</dt>
          <dd>{currency.format(product.price)}</dd>
        </div>
        <div>
          <dt>Product ID</dt>
          <dd>{product.id}</dd>
        </div>
      </dl>
    </section>
  );
}

function SelectionSummary({ product }) {
  return (
    <section className="summary" aria-labelledby="summary-heading">
      <div>
        <p className="eyebrow">Current selection</p>
        <h2 id="summary-heading">
          {product ? product.name : "No product selected"}
        </h2>
      </div>
      <p className="summary-price">
        {product ? currency.format(product.price) : "—"}
      </p>
    </section>
  );
}

export default function App() {
  const [selectedProductId, setSelectedProductId] = useState(products[0].id);

  const selectedProduct = products.find(
    (product) => product.id === selectedProductId
  );

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React state ownership</p>
        <h1>One selection, several views</h1>
        <p className="intro">
          Choose a product. The picker, details, and summary all render from
          one selection owned by their common parent.
        </p>
      </header>

      <SelectionSummary product={selectedProduct} />

      <div className="content-grid">
        <ProductPicker
          products={products}
          selectedProductId={selectedProductId}
          onSelectProduct={setSelectedProductId}
        />
        <ProductDetails product={selectedProduct} />
      </div>
    </main>
  );
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/styles.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  color: #18212f;
  background: #f3f6fb;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
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

button:focus-visible,
input:focus-visible {
  outline: 3px solid #82aaff;
  outline-offset: 3px;
}

.page {
  width: min(100% - 2rem, 1080px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  max-width: 680px;
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin: 0 0 0.45rem;
  color: #315db5;
  font-size: 0.76rem;
  font-weight: 750;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

h1,
h2,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 6vw, 3.2rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0;
  font-size: 1.2rem;
}

.intro {
  max-width: 62ch;
  margin-bottom: 0;
  color: #526071;
  line-height: 1.65;
}

.card,
.summary {
  border: 1px solid #d8e0eb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 35px rgb(32 53 84 / 7%);
}

.summary {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
  padding: 1.2rem 1.4rem;
}

.summary h2 {
  overflow-wrap: anywhere;
}

.summary-price {
  flex: 0 0 auto;
  margin: 0;
  color: #244578;
  font-size: 1.4rem;
  font-weight: 750;
}

.content-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 0.9fr);
  align-items: start;
  gap: 1rem;
}

.card {
  min-width: 0;
  padding: 1.25rem;
}

.card-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.1rem;
}

.result-count,
.option-meta,
.card-note {
  color: #637185;
  font-size: 0.88rem;
}

.search-field {
  display: grid;
  gap: 0.45rem;
  margin-bottom: 1rem;
  font-size: 0.9rem;
  font-weight: 650;
}

.search-field input {
  width: 100%;
  padding: 0.7rem 0.8rem;
  border: 1px solid #bac8dc;
  border-radius: 0.55rem;
  color: inherit;
  background: #fff;
}

.product-list {
  display: grid;
  gap: 0.55rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.product-option {
  display: flex;
  width: 100%;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
  padding: 0.8rem;
  border: 1px solid #dce4ef;
  border-radius: 0.65rem;
  color: inherit;
  background: #fff;
  text-align: left;
  cursor: pointer;
}

.product-option:hover {
  background: #f7f9fc;
}

.product-option-selected {
  border-color: #4f76c7;
  background: #f0f5ff;
}

.product-option > span:first-child,
.option-meta {
  display: grid;
  gap: 0.25rem;
}

.selection-mark {
  flex: 0 0 auto;
  color: #315db5;
  font-size: 0.8rem;
  font-weight: 700;
}

.card-note {
  margin: 1rem 0 0;
  line-height: 1.5;
}

.description,
.empty-state {
  margin: 0.9rem 0 0;
  color: #526071;
  line-height: 1.65;
}

.details-list {
  display: grid;
  gap: 0;
  margin: 1.4rem 0 0;
}

.details-list div {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  padding-block: 0.7rem;
  border-top: 1px solid #e4eaf2;
}

.details-list dt {
  color: #637185;
}

.details-list dd {
  margin: 0;
  font-weight: 650;
  text-align: right;
  overflow-wrap: anywhere;
}

@media (max-width: 720px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .content-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 440px) {
  .summary {
    align-items: flex-start;
    padding: 1rem;
  }

  .summary-price {
    font-size: 1.15rem;
  }

  .card {
    padding: 1rem;
  }

  .product-option {
    align-items: flex-start;
  }
}
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="A React example of one source of truth and clear state ownership."
    />
    <title>State Ownership — React Example</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How it works

### One selection value

ProductPage stores only selectedProductId. It derives selectedProduct from the product list. The ID is the source value; the selected object is calculated from it.

ProductPage passes the same selectedProduct to ProductDetails and SelectionSummary. It passes the selected ID to ProductPicker so that the picker can mark the selected option.

When a user activates a product button:

1. ProductPicker calls onSelectProduct with the product ID.
2. ProductPage updates selectedProductId.
3. ProductPage derives the corresponding product again.
4. The picker, details panel, and summary render from the updated value.

No child maintains its own competing selected-product state.

### A separate local value

ProductPicker owns query because only the picker uses it to filter the visible options. Changing the query does not change which product is selected. Selecting a product does not need to reset the query.

The product selection and picker query are both state, but they represent different facts and have different owners.

### What if a selection is missing?

The details panel handles an absent product, and the summary has an empty-state label. In this static example the initial ID always exists. If an application loads the product list asynchronously or allows items to be removed, define what selection should mean when its ID is no longer present.

## Run it

From the Vite project root, install the project's dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

Open the local address printed by Vite. Select different products and confirm that the picker mark, details, and summary update together. Filter the product list and confirm that the query remains local to the picker.

## What to notice

- ProductPage is the one owner of the shared product selection.
- ProductDetails and SelectionSummary are consumers; they do not own copies.
- The product object is derived from the ID instead of stored as a second state value.
- ProductPicker has independent local state for its search query.
- A single source of truth applies to a shared fact; it does not move unrelated state to one global component.
