# 023. Thinking in React: from a design to a component tree - Example

## Goal

Build a searchable product table from a small data set. The example demonstrates:

- Breaking a design into components with distinct responsibilities.
- Passing product data down through props.
- Keeping the search query and stock filter in their closest coordinating parent.
- Deriving visible products during rendering instead of duplicating them in state.
- Rendering semantic table sections and stable keys.

## Design and data

The page needs a heading, a search field, a stock-only checkbox, and a table grouped by category. Product records already include the fields needed to display and filter them:

~~~js
{
  id: 'apple',
  name: 'Apple',
  category: 'Fruit',
  price: '$1.00',
  stocked: true,
}
~~~

The planned component tree is:

~~~text
App
+-- FilterableProductTable
    +-- SearchBar
    +-- ProductTable
        +-- ProductCategory
            +-- ProductRow
~~~

`App` supplies the data. `FilterableProductTable` coordinates the controls and results. `SearchBar` handles the controls' presentation and reports changes. `ProductTable` groups the current results, while `ProductCategory` and `ProductRow` render the repeated table content.

When building this from scratch, first render a static `ProductTable` from the product prop. Once the data and hierarchy look right, add the controls and connect them to state. The completed code below shows the interactive result.

## Files

In an existing React project, create or update:

- `src/main.jsx`
- `src/App.jsx`

No extra packages or stylesheet are required.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
import { useState } from 'react';

const PRODUCTS = [
  { id: 'apple', name: 'Apple', category: 'Fruit', price: '$1.00', stocked: true },
  {
    id: 'dragonfruit',
    name: 'Dragonfruit',
    category: 'Fruit',
    price: '$4.00',
    stocked: true,
  },
  {
    id: 'passionfruit',
    name: 'Passionfruit',
    category: 'Fruit',
    price: '$2.00',
    stocked: false,
  },
  {
    id: 'spinach',
    name: 'Spinach',
    category: 'Vegetable',
    price: '$2.00',
    stocked: true,
  },
  {
    id: 'pumpkin',
    name: 'Pumpkin',
    category: 'Vegetable',
    price: '$4.00',
    stocked: false,
  },
  {
    id: 'peas',
    name: 'Peas',
    category: 'Vegetable',
    price: '$3.00',
    stocked: true,
  },
];

function SearchBar({
  query,
  onQueryChange,
  onlyInStock,
  onOnlyInStockChange,
}) {
  return (
    <form onSubmit={(event) => event.preventDefault()}>
      <div>
        <label htmlFor="product-search">Search products</label>
        <input
          id="product-search"
          type="search"
          value={query}
          onChange={(event) => onQueryChange(event.target.value)}
        />
      </div>
      <div>
        <input
          id="in-stock-only"
          type="checkbox"
          checked={onlyInStock}
          onChange={(event) => onOnlyInStockChange(event.target.checked)}
        />
        <label htmlFor="in-stock-only">Show only products in stock</label>
      </div>
    </form>
  );
}

function ProductRow({ product }) {
  return (
    <tr>
      <th scope="row">
        {product.name}
        {!product.stocked && <span> (out of stock)</span>}
      </th>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductCategory({ category, products }) {
  return (
    <tbody>
      <tr>
        <th scope="rowgroup" colSpan={2}>
          {category}
        </th>
      </tr>
      {products.map((product) => (
        <ProductRow key={product.id} product={product} />
      ))}
    </tbody>
  );
}

function ProductTable({ products }) {
  if (products.length === 0) {
    return <p role="status">No products match these filters.</p>;
  }

  const categories = [...new Set(products.map((product) => product.category))];

  return (
    <table>
      <caption>Products matching the current filters</caption>
      <thead>
        <tr>
          <th scope="col">Product</th>
          <th scope="col">Price</th>
        </tr>
      </thead>
      {categories.map((category) => {
        const categoryProducts = products.filter(
          (product) => product.category === category,
        );

        return (
          <ProductCategory
            key={category}
            category={category}
            products={categoryProducts}
          />
        );
      })}
    </table>
  );
}

function FilterableProductTable({ products }) {
  const [query, setQuery] = useState('');
  const [onlyInStock, setOnlyInStock] = useState(false);
  const normalizedQuery = query.trim().toLowerCase();

  const visibleProducts = products.filter((product) => {
    const matchesQuery = product.name.toLowerCase().includes(normalizedQuery);
    const matchesStockFilter = !onlyInStock || product.stocked;
    return matchesQuery && matchesStockFilter;
  });

  return (
    <section aria-labelledby="inventory-heading">
      <h2 id="inventory-heading">Inventory</h2>
      <SearchBar
        query={query}
        onQueryChange={setQuery}
        onlyInStock={onlyInStock}
        onOnlyInStockChange={setOnlyInStock}
      />
      <ProductTable products={visibleProducts} />
    </section>
  );
}

export default function App() {
  return (
    <main>
      <h1>Product inventory</h1>
      <p>Search by product name or limit the results to items in stock.</p>
      <FilterableProductTable products={PRODUCTS} />
    </main>
  );
}
~~~

## Follow the data flow

1. `App` passes the product records to `FilterableProductTable`.
2. `FilterableProductTable` owns `query` and `onlyInStock`, the two values users can change.
3. It passes each current value and its setter callback to `SearchBar`.
4. `SearchBar` calls those callbacks when an input changes.
5. `FilterableProductTable` calculates `visibleProducts` from the current products and filters.
6. `ProductTable` receives only the visible products and groups them by category.
7. `ProductCategory` and `ProductRow` render the individual table sections and records.

The state and derived data are intentionally different:

| Value | Stored in state? | Reason |
| --- | --- | --- |
| Search query | Yes | The user changes it by typing. |
| In-stock-only choice | Yes | The user changes it with the checkbox. |
| Original products | No, supplied as a prop | This component displays the provided data; it does not edit the catalog. |
| Visible products | No, derived | It is calculated from the products and current filters. |
| Category list | No, derived | It is calculated from the visible products. |

There is no second copy of the filtered product list to synchronize. Each render calculates the results from the current inputs.

## Try these changes

1. Search for `spinach`; only matching product names should remain.
2. Turn on the stock checkbox and search for `fruit`; the out-of-stock Passionfruit row should be excluded.
3. Search for a name that is not present and observe the status message.
4. Add another product with a unique `id`. Decide which category it belongs to and observe how the table groups it.
5. Add a `category` field to the search behavior only if the requirement calls for it. Keep the filtered list derived from the current query and stock choice.
6. Add a product-count sentence using `visibleProducts.length`. Do not create a separate count state variable.
7. Move a filter into another component only if the design gives that component a clear responsibility; keep shared state with the parent that coordinates the input and table.

## Common mistakes to avoid

- Do not store `visibleProducts` in state and update it separately from the query and checkbox.
- Do not place query state separately inside both `SearchBar` and `ProductTable`.
- Do not pass application-wide values to a row that only needs one product.
- Do not use the array index as a key when each product already has a stable `id`.
- Do not add a component boundary for every HTML element. Keep boundaries that make the design or data flow clearer.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="022-designing-reusable-component-apis.md">022 - Designing reusable component APIs - Example</a></td><td><a href="../lessons/023-thinking-in-react-and-component-trees.md">Lesson</a></td><td><a href="../qnas/023-thinking-in-react-and-component-trees.md">Q&amp;A</a></td><td><a href="024-component-identity-position-and-keys.md">024 - Component identity, position, and keys - Example</a></td></tr>
  </tbody>
</table>
