# 042. Derived values and avoiding duplicated state - Example

## What this example demonstrates

This product browser keeps only the source data and user choices in state:

- The product records.
- Search text.
- Selected category.
- Whether to show only available products.
- The selected product ID.

The visible product list, result count, available count, and selected product object are calculated during rendering. There is no duplicate filtered-list state and no Effect to synchronize derived values.

## Project files

Add these files to an existing React app created with Vite:

- src/App.jsx
- src/App.css
- src/main.jsx
- src/index.css

Run the development server with npm run dev.

## src/App.jsx

~~~jsx
import { useState } from 'react';
import './App.css';

const initialProducts = [
  {
    id: 'book-react',
    name: 'React State Patterns',
    category: 'books',
    price: 28,
    inStock: true,
  },
  {
    id: 'book-accessibility',
    name: 'Accessible Interface Design',
    category: 'books',
    price: 32,
    inStock: false,
  },
  {
    id: 'tool-keyboard',
    name: 'Compact Keyboard',
    category: 'tools',
    price: 54,
    inStock: true,
  },
  {
    id: 'stationery-notebook',
    name: 'Grid Notebook',
    category: 'stationery',
    price: 8,
    inStock: true,
  },
  {
    id: 'tool-stand',
    name: 'Adjustable Laptop Stand',
    category: 'tools',
    price: 42,
    inStock: false,
  },
];

const categories = ['all', 'books', 'stationery', 'tools'];

function formatPrice(amount) {
  return '$' + amount.toFixed(2);
}

export default function App() {
  const [products, setProducts] = useState(initialProducts);
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');
  const [availableOnly, setAvailableOnly] = useState(false);
  const [selectedProductId, setSelectedProductId] = useState('book-react');

  const normalizedQuery = query.trim().toLowerCase();

  const visibleProducts = products.filter((product) => {
    const matchesName = product.name
      .toLowerCase()
      .includes(normalizedQuery);
    const matchesCategory =
      category === 'all' || product.category === category;
    const matchesAvailability =
      !availableOnly || product.inStock;

    return matchesName && matchesCategory && matchesAvailability;
  });

  const availableCount = products.filter(
    (product) => product.inStock
  ).length;

  const selectedProduct =
    products.find((product) => product.id === selectedProductId) ?? null;

  function handleToggleAvailability(productId, nextInStock) {
    setProducts((currentProducts) =>
      currentProducts.map((product) =>
        product.id === productId
          ? { ...product, inStock: nextInStock }
          : product
      )
    );
  }

  function handleRemoveProduct(productId) {
    setProducts((currentProducts) =>
      currentProducts.filter((product) => product.id !== productId)
    );
  }

  function handleReset() {
    setProducts(initialProducts);
    setQuery('');
    setCategory('all');
    setAvailableOnly(false);
    setSelectedProductId('book-react');
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React state</p>
        <h1>Derived values and avoiding duplicated state</h1>
        <p>
          Change the filters and product records. The visible list, counts, and
          selected details are derived from the current state on each render.
        </p>
      </header>

      <section className="summary-grid" aria-label="Inventory summary">
        <article className="summary-card">
          <span className="summary-value">{products.length}</span>
          <span className="summary-label">Products in inventory</span>
        </article>
        <article className="summary-card">
          <span className="summary-value">{availableCount}</span>
          <span className="summary-label">Available products</span>
        </article>
        <article className="summary-card">
          <span className="summary-value">{visibleProducts.length}</span>
          <span className="summary-label">Matching filters</span>
        </article>
      </section>

      <section className="browser-layout">
        <div className="panel" aria-labelledby="browse-heading">
          <div className="panel-heading">
            <div>
              <p className="eyebrow">Source data and controls</p>
              <h2 id="browse-heading">Browse products</h2>
            </div>
            <button
              className="secondary-button"
              type="button"
              onClick={handleReset}
            >
              Reset
            </button>
          </div>

          <div className="filters">
            <div className="field">
              <label htmlFor="search-products">Search by name</label>
              <input
                id="search-products"
                type="search"
                value={query}
                onChange={(event) => setQuery(event.target.value)}
                placeholder="For example, keyboard"
              />
            </div>

            <div className="field">
              <label htmlFor="category">Category</label>
              <select
                id="category"
                value={category}
                onChange={(event) => setCategory(event.target.value)}
              >
                {categories.map((option) => (
                  <option key={option} value={option}>
                    {option === 'all'
                      ? 'All categories'
                      : option[0].toUpperCase() + option.slice(1)}
                  </option>
                ))}
              </select>
            </div>

            <label className="checkbox-field" htmlFor="available-only">
              <input
                id="available-only"
                type="checkbox"
                checked={availableOnly}
                onChange={(event) =>
                  setAvailableOnly(event.target.checked)
                }
              />
              Show available only
            </label>
          </div>

          <p className="result-summary" aria-live="polite">
            Showing {visibleProducts.length} of {products.length} products.
          </p>

          {visibleProducts.length === 0 ? (
            <p className="empty-state">
              No products match these filters. Try changing the search or
              category.
            </p>
          ) : (
            <ul className="product-list">
              {visibleProducts.map((product) => (
                <li className="product-row" key={product.id}>
                  <div className="product-copy">
                    <h3>{product.name}</h3>
                    <p>
                      {product.category} · {formatPrice(product.price)}
                    </p>
                  </div>

                  <label
                    className="stock-control"
                    htmlFor={'stock-' + product.id}
                  >
                    <input
                      id={'stock-' + product.id}
                      type="checkbox"
                      checked={product.inStock}
                      onChange={(event) =>
                        handleToggleAvailability(
                          product.id,
                          event.target.checked
                        )
                      }
                    />
                    Available
                  </label>

                  <div className="product-actions">
                    <button
                      className={
                        product.id === selectedProductId
                          ? 'small-button selected-button'
                          : 'small-button secondary-button'
                      }
                      type="button"
                      onClick={() => setSelectedProductId(product.id)}
                      aria-pressed={product.id === selectedProductId}
                    >
                      Details
                    </button>
                    <button
                      className="small-button danger-button"
                      type="button"
                      onClick={() => handleRemoveProduct(product.id)}
                      aria-label={'Remove ' + product.name}
                    >
                      Remove
                    </button>
                  </div>
                </li>
              ))}
            </ul>
          )}
        </div>

        <aside className="panel detail-panel" aria-labelledby="details-heading">
          <p className="eyebrow">Derived selection</p>
          <h2 id="details-heading">Selected product</h2>

          {selectedProduct === null ? (
            <p className="empty-state">
              The selected product is no longer in the inventory. Choose a
              product from the list.
            </p>
          ) : (
            <div className="detail-content">
              <h3>{selectedProduct.name}</h3>
              <dl>
                <div>
                  <dt>Category</dt>
                  <dd>{selectedProduct.category}</dd>
                </div>
                <div>
                  <dt>Price</dt>
                  <dd>{formatPrice(selectedProduct.price)}</dd>
                </div>
                <div>
                  <dt>Availability</dt>
                  <dd>
                    {selectedProduct.inStock
                      ? 'Available'
                      : 'Currently unavailable'}
                  </dd>
                </div>
                <div>
                  <dt>Stored selection</dt>
                  <dd>{selectedProduct.id}</dd>
                </div>
              </dl>
              <p className="help-text">
                The selection is stored as an ID. This product object is found
                from the current products array, so its details follow updates
                to that array.
              </p>
            </div>
          )}
        </aside>
      </section>

      <section className="explanation-panel" aria-labelledby="state-heading">
        <p className="eyebrow">State design</p>
        <h2 id="state-heading">What is state, and what is derived?</h2>
        <div className="state-columns">
          <div>
            <h3>Stored state</h3>
            <ul>
              <li>The source products</li>
              <li>Search text and category filter</li>
              <li>Availability filter choice</li>
              <li>Selected product ID</li>
            </ul>
          </div>
          <div>
            <h3>Calculated during render</h3>
            <ul>
              <li>Visible products</li>
              <li>Available product count</li>
              <li>Matching result count</li>
              <li>The selected product object</li>
            </ul>
          </div>
        </div>
      </section>

      <footer className="page-footer">
        <p>
          There is one source of truth for each stored fact. Values that follow
          from it are calculated during rendering.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/App.css

~~~css
.page-shell {
  width: min(100% - 2rem, 1160px);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  max-width: 76ch;
  margin-bottom: 1.5rem;
}

.page-header h1 {
  margin-block: 0.35rem 0.75rem;
  font-size: clamp(2rem, 5vw, 3.2rem);
  line-height: 1.1;
}

.page-header > p:last-child,
.help-text,
.page-footer {
  line-height: 1.65;
  color: #4b5563;
}

.eyebrow {
  color: #4d3ab8;
  font-size: 0.8rem;
  font-weight: 750;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 0.85rem;
  margin-bottom: 1rem;
}

.summary-card,
.panel,
.explanation-panel {
  border: 1px solid #d5dae4;
  border-radius: 0.9rem;
  background: #ffffff;
  box-shadow: 0 0.5rem 1.5rem rgb(24 33 54 / 6%);
}

.summary-card {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding: 1rem 1.15rem;
}

.summary-value {
  font-size: 1.6rem;
  font-weight: 750;
  font-variant-numeric: tabular-nums;
}

.summary-label {
  color: #596273;
  font-size: 0.92rem;
}

.browser-layout {
  display: grid;
  grid-template-columns: minmax(0, 1.65fr) minmax(260px, 0.8fr);
  align-items: start;
  gap: 1rem;
}

.panel,
.explanation-panel {
  min-width: 0;
  padding: clamp(1rem, 3vw, 1.5rem);
}

.panel-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.panel h2,
.explanation-panel h2 {
  margin-top: 0.35rem;
}

.filters {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(9rem, 0.65fr);
  gap: 0.8rem 1rem;
  margin-block: 1.2rem 0.8rem;
}

.field {
  display: grid;
  gap: 0.4rem;
}

.field label {
  font-weight: 650;
}

.field input,
.field select {
  width: 100%;
  min-height: 2.7rem;
  padding: 0.55rem 0.7rem;
  border: 1px solid #aeb5c2;
  border-radius: 0.5rem;
  background: #ffffff;
  color: #202735;
}

.checkbox-field,
.stock-control {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-height: 2rem;
}

.checkbox-field {
  grid-column: 1 / -1;
}

.checkbox-field input,
.stock-control input {
  width: 1.05rem;
  height: 1.05rem;
  accent-color: #5b46d6;
}

input:focus-visible,
select:focus-visible,
button:focus-visible {
  outline: 3px solid #a99cff;
  outline-offset: 2px;
}

button,
input,
select {
  font: inherit;
}

button {
  min-height: 2.6rem;
  padding: 0.5rem 0.8rem;
  border: 1px solid #4d3ab8;
  border-radius: 0.5rem;
  background: #5b46d6;
  color: #ffffff;
  cursor: pointer;
}

button:hover {
  background: #4835b5;
}

.secondary-button {
  border-color: #abb3c0;
  background: #ffffff;
  color: #202735;
}

.secondary-button:hover {
  background: #f0f2f6;
}

.danger-button {
  border-color: #b42318;
  background: #ffffff;
  color: #9d1c13;
}

.danger-button:hover {
  background: #fff1f0;
}

.result-summary {
  margin-block: 0.85rem;
  color: #596273;
  font-size: 0.93rem;
}

.product-list {
  list-style: none;
}

.product-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto auto;
  align-items: center;
  gap: 0.9rem;
  padding-block: 0.9rem;
  border-top: 1px solid #e2e5eb;
}

.product-copy {
  min-width: 0;
}

.product-copy h3 {
  overflow-wrap: anywhere;
  font-size: 1rem;
}

.product-copy p {
  margin-top: 0.25rem;
  color: #596273;
  font-size: 0.9rem;
  text-transform: capitalize;
}

.product-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.45rem;
}

.small-button {
  min-height: 2.3rem;
  padding: 0.4rem 0.6rem;
  font-size: 0.9rem;
}

.selected-button {
  background: #34238f;
}

.detail-panel {
  position: sticky;
  top: 1rem;
}

.detail-content h3 {
  margin-block: 0.9rem;
  font-size: 1.25rem;
}

.detail-content dl {
  display: grid;
  gap: 0.6rem;
}

.detail-content dl > div {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  padding-bottom: 0.55rem;
  border-bottom: 1px solid #e2e5eb;
}

.detail-content dt {
  color: #596273;
}

.detail-content dd {
  text-align: right;
  overflow-wrap: anywhere;
}

.help-text {
  margin-top: 1rem;
  font-size: 0.92rem;
}

.empty-state {
  margin-block: 1rem;
  padding: 1rem;
  border-radius: 0.55rem;
  background: #f3f4f7;
  color: #596273;
  line-height: 1.5;
}

.explanation-panel {
  margin-top: 1rem;
}

.state-columns {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1.5rem;
  margin-top: 1rem;
}

.state-columns h3 {
  margin-bottom: 0.5rem;
}

.state-columns ul {
  padding-left: 1.2rem;
  color: #4b5563;
  line-height: 1.7;
}

.page-footer {
  margin-top: 1.5rem;
}

@media (max-width: 820px) {
  .page-shell {
    width: min(100% - 1.25rem, 1160px);
    padding-block: 1.5rem;
  }

  .browser-layout {
    grid-template-columns: 1fr;
  }

  .detail-panel {
    position: static;
  }
}

@media (max-width: 620px) {
  .summary-grid {
    grid-template-columns: 1fr;
  }

  .summary-card {
    flex-direction: row;
    align-items: baseline;
    justify-content: space-between;
  }

  .product-row {
    grid-template-columns: minmax(0, 1fr) auto;
  }

  .stock-control {
    grid-column: 1;
  }

  .product-actions {
    grid-column: 2;
    grid-row: 1 / span 2;
    justify-content: end;
  }
}

@media (max-width: 420px) {
  .filters,
  .state-columns {
    grid-template-columns: 1fr;
  }

  .checkbox-field {
    grid-column: auto;
  }

  .product-row {
    grid-template-columns: 1fr;
  }

  .stock-control,
  .product-actions {
    grid-column: auto;
    grid-row: auto;
  }

  .product-actions {
    justify-content: start;
  }
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.jsx';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #202735;
  background: #f6f7fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
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
~~~

## Try it

1. Search for “book” and notice the matching list and result count update immediately.
2. Choose a category, then enable Show available only. Each control filters the same source products.
3. Select a product's Details button. The selected product ID is stored, and its object is looked up from products.
4. Toggle that product's availability. Its list row and detail panel update from the replacement product object.
5. Remove the selected product. The lookup returns null, so the details panel reports that the selected record no longer exists instead of showing an old copy.
6. Change filters until no products match. The empty state is derived from the current filtered list.
7. Select Reset to restore the original products and filter choices.

## What to notice

- Products, controls, and selectedProductId are stored state.
- visibleProducts, availableCount, result count, and selectedProduct are calculated during render.
- No Effect copies source values into a second state variable.
- Updating product availability replaces the matching object in a new array, and the selected details are recalculated from that updated array.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="041-updating-array-state-immutably.md">041 - Updating array state immutably - Example</a></td><td><a href="../lessons/042-derived-values-and-duplicated-state.md">Lesson</a></td><td><a href="../qnas/042-derived-values-and-duplicated-state.md">Q&amp;A</a></td><td><a href="043-lifting-state-up-and-sharing-data.md">043 - Lifting State Up and Sharing Data - Example</a></td></tr>
  </tbody>
</table>
