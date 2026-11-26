# Virtualizing a Large Responsive List - Complete Example

This example uses the current react-window v2 List API to display 10,000 product records while mounting only rows near the scroll viewport. The row has a fixed height so the virtualizer can calculate positions without measuring every item.

This is a library-based example, not a React built-in API. The older react-window v1 API is different.

## 1. Install the library

In an existing React project, install react-window:

    npm install react-window

## 2. Create src/App.jsx

    import { useCallback, useMemo, useState } from 'react';
    import { List } from 'react-window';
    import './App.css';

    const products = Array.from({ length: 10000 }, (_, index) => ({
      id: 'product-' + (index + 1),
      name: 'Product ' + (index + 1),
      category: ['Books', 'Home', 'Office'][index % 3],
    }));

    function ProductRow({ ariaAttributes, index, products, style }) {
      const product = products[index];

      return (
        <li
          {...ariaAttributes}
          className="product-row"
          style={style}
        >
          <strong className="product-name">{product.name}</strong>
          <span className="product-category">
            Category: {product.category}
          </span>
          <span className="product-id">ID: {product.id}</span>
        </li>
      );
    }

    export default function App() {
      const [query, setQuery] = useState('');
      const [category, setCategory] = useState('all');

      const visibleProducts = useMemo(() => {
        const normalizedQuery = query.trim().toLowerCase();

        return products.filter((product) => {
          const matchesName = product.name
            .toLowerCase()
            .includes(normalizedQuery);
          const matchesCategory =
            category === 'all' || product.category === category;

          return matchesName && matchesCategory;
        });
      }, [query, category]);

      const rowProps = useMemo(
        () => ({ products: visibleProducts }),
        [visibleProducts]
      );

      const rowKey = useCallback(
        (index, data) => data.products[index].id,
        []
      );

      return (
        <main className="page">
          <header className="page-header">
            <div>
              <h1>Product directory</h1>
              <p>Search and filter a large product list.</p>
            </div>

            <div className="filters">
              <label>
                Search products
                <input
                  type="search"
                  value={query}
                  onChange={(event) => setQuery(event.target.value)}
                  placeholder="For example, Product 42"
                />
              </label>

              <label>
                Category
                <select
                  value={category}
                  onChange={(event) => setCategory(event.target.value)}
                >
                  <option value="all">All categories</option>
                  <option value="Books">Books</option>
                  <option value="Home">Home</option>
                  <option value="Office">Office</option>
                </select>
              </label>
            </div>
          </header>

          <p className="result-count">
            Showing {visibleProducts.length} of {products.length} products
          </p>

          {visibleProducts.length === 0 ? (
            <p>No products match the current filters.</p>
          ) : (
            <List
              aria-label="Products"
              tagName="ul"
              rowComponent={ProductRow}
              rowCount={visibleProducts.length}
              rowHeight={72}
              rowProps={rowProps}
              rowKey={rowKey}
              defaultHeight={432}
              overscanCount={6}
              style={{ height: 432, width: '100%' }}
            />
          )}
        </main>
      );
    }

The row spreads ariaAttributes supplied by the virtualizer onto the native list item. rowKey returns the product's stable ID, so a filter change does not make the row's identity depend on its current position.

The search and category filters run over the source array. Virtualization reduces mounted rows; it does not make filtering 10,000 records free. For much larger remote datasets, move appropriate filtering and paging to the server.

## 3. Create src/App.css

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      color: #172033;
      background: #f4f6fa;
      font-family: system-ui, sans-serif;
    }

    .page {
      width: min(100% - 2rem, 70rem);
      margin-inline: auto;
      padding-block: 2rem;
    }

    .page-header {
      display: flex;
      align-items: end;
      justify-content: space-between;
      gap: 1.5rem;
      margin-bottom: 1rem;
    }

    .page-header h1 {
      margin: 0;
    }

    .page-header p {
      margin: 0.4rem 0 0;
      color: #596579;
    }

    .filters {
      display: flex;
      flex-wrap: wrap;
      gap: 0.75rem;
    }

    .filters label {
      display: grid;
      gap: 0.3rem;
      font-size: 0.9rem;
    }

    .filters input,
    .filters select {
      min-height: 2.5rem;
      max-width: 100%;
      padding: 0.4rem 0.65rem;
      font: inherit;
    }

    .result-count {
      color: #46546a;
    }

    .product-row {
      display: grid;
      grid-template-columns: minmax(0, 1fr) 12rem 12rem;
      align-items: center;
      gap: 1rem;
      width: 100%;
      padding: 0.75rem 1rem;
      overflow: hidden;
      border-bottom: 1px solid #d9dfeb;
      background: white;
    }

    .product-name,
    .product-category,
    .product-id {
      min-width: 0;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .product-category,
    .product-id {
      color: #596579;
    }

    @media (max-width: 42rem) {
      .page-header {
        align-items: stretch;
        flex-direction: column;
      }

      .filters {
        display: grid;
        grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
      }

      .product-row {
        grid-template-columns: minmax(0, 1fr) auto;
        grid-template-rows: 1fr 1fr;
        gap: 0 0.75rem;
        padding-inline: 0.75rem;
      }

      .product-name {
        grid-column: 1;
        grid-row: 1;
      }

      .product-category {
        grid-column: 1;
        grid-row: 2;
      }

      .product-id {
        grid-column: 2;
        grid-row: 1 / span 2;
      }
    }

    @media (max-width: 24rem) {
      .filters {
        grid-template-columns: minmax(0, 1fr);
      }
    }

The row remains 72 pixels tall at every breakpoint. CSS changes its columns and spacing, while text truncation prevents content from changing the virtualizer's configured row height.

## 4. Try the list

1. Start the project using its existing development command.
2. Scroll quickly through the directory and notice that the scroll surface represents the full result set.
3. Search for Product 42 and verify that the result count and list update.
4. Select a category and confirm that each visible row still represents the correct product.
5. Resize the browser below and above 42rem; the list layout should change while the row height stays fixed.
6. Clear the search and confirm the empty-result message appears for a query with no matches.
7. Use a screen reader to confirm that the list reports item positions and the total count where supported by the virtualizer's provided ARIA attributes.

The filter input and select remain normal keyboard controls. Rows in this example are informational list items. If you make rows interactive, add and test a focus strategy so moving or filtering the list does not strand keyboard focus on an unmounted item.

## 5. Understand the virtualizer props

- rowCount is the number of filtered products.
- rowHeight gives the fixed height in pixels for each row.
- defaultHeight supplies an initial viewport height, including for the initial render before the list has measured itself.
- overscanCount renders a few additional rows near the visible range to help fast scrolling.
- rowProps sends the data required by the row component.
- rowKey returns a stable product ID rather than relying on a row's changing index.
- style gives the list a definite height and width so it has a scroll viewport.

## 6. When to use a different strategy

- For a small collection, render a normal ul and li list.
- For results that should be split into navigable pages, use pagination.
- For remote data that loads as someone scrolls, add incremental loading and a clear loading state.
- For content where every item must be in the DOM for print or browsing, do not rely on a virtualized view as the only presentation.
- For rows whose height changes unpredictably, use a virtualizer with measured dynamic sizes and test the impact of resizing.

## Key takeaway

Virtualization keeps a long scrollable list while limiting mounted rows. Stable data keys, correct row dimensions, responsive CSS, and accessibility checks are part of the implementation-not optional finishing touches.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="109-memo-and-optimization-decisions.md">109 - Memoization with memo - Complete Example</a></td><td><a href="../lessons/110-large-lists-and-virtualization.md">Lesson</a></td><td><a href="../qnas/110-large-lists-and-virtualization.md">Q&amp;A</a></td><td><a href="111-code-splitting-and-asset-loading.md">111 - Code Splitting and Asset Loading - Complete Example</a></td></tr>
  </tbody>
</table>
