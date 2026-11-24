# 020. Component boundaries and responsibility - Example

## Goal

Build a product catalog from small, related components. The example demonstrates:

- A page component that composes a catalog.
- A catalog section that separates its introduction from its table.
- A table that renders one row component for each product record.
- A focused status component used inside each row.
- Passing each component the data it needs through props.

## Component hierarchy

~~~text
App
+-- ProductCatalog
    +-- CatalogIntro
    +-- ProductTable
        +-- ProductRow (one per product)
            +-- StockStatus
~~~

The pieces are small and closely related, so this example keeps them together in one file. In a larger app, a row or status component could move to a separate file if it grows or is used elsewhere.

## Files

In an existing Vite React project, create or update these files:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required.

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
const products = [
  {
    id: 'notebook',
    name: 'Notebook',
    category: 'Stationery',
    price: '$4.00',
    inStock: true,
  },
  {
    id: 'desk-lamp',
    name: 'Desk lamp',
    category: 'Lighting',
    price: '$22.00',
    inStock: false,
  },
  {
    id: 'pen-set',
    name: 'Pen set',
    category: 'Stationery',
    price: '$8.50',
    inStock: true,
  },
];

function CatalogIntro({ productCount }) {
  return (
    <header>
      <h1 id="catalog-title">Product catalog</h1>
      <p>{productCount} products in this catalog.</p>
    </header>
  );
}

function StockStatus({ inStock }) {
  return <span>{inStock ? 'In stock' : 'Out of stock'}</span>;
}

function ProductRow({ product }) {
  return (
    <tr>
      <th scope="row">{product.name}</th>
      <td>{product.category}</td>
      <td>{product.price}</td>
      <td><StockStatus inStock={product.inStock} /></td>
    </tr>
  );
}

function ProductTable({ products }) {
  return (
    <table>
      <caption>Products, categories, prices, and availability</caption>
      <thead>
        <tr>
          <th scope="col">Product</th>
          <th scope="col">Category</th>
          <th scope="col">Price</th>
          <th scope="col">Availability</th>
        </tr>
      </thead>
      <tbody>
        {products.map((product) => (
          <ProductRow key={product.id} product={product} />
        ))}
      </tbody>
    </table>
  );
}

function ProductCatalog({ products }) {
  return (
    <section aria-labelledby="catalog-title">
      <CatalogIntro productCount={products.length} />
      <ProductTable products={products} />
    </section>
  );
}

export default function App() {
  return (
    <main>
      <ProductCatalog products={products} />
    </main>
  );
}
~~~

## What to notice

- `App` owns the example data and composes the page from `ProductCatalog`.
- `ProductCatalog` owns the section structure and composes the introduction and table.
- `ProductTable` owns the table markup and maps each product to a `ProductRow`.
- `ProductRow` presents one record. It delegates the availability label to `StockStatus` because that detail has a clear name and can be changed independently.
- The data moves down through explicit props. `ProductRow` receives one product; `StockStatus` receives the boolean it needs.
- The components are declared at the top level, so their definitions stay stable across renders.
- The table uses semantic headings and row headers so the relationships between values are clear.

## Expected result

The page displays a “Product catalog” heading, a product count, and a table with three product rows. Each row shows the product's category, price, and stock status.

## Practice

1. Add another product object with a unique ID and confirm the table displays another row.
2. Add a `ProductDescription` component only if the description becomes a meaningful part of a product row.
3. Move `StockStatus` into its own file and update the import; decide whether that improves navigation for this small example.
4. Change the product count label to show a different message when the array is empty.
5. Rename `ProductCatalog` to a clearer domain name and describe how that name changes the way `App` reads.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [019 - Rendering collections and choosing stable keys](../lessons/019-rendering-collections-and-stable-keys.md) | [020 - Component boundaries and responsibility](../lessons/020-component-boundaries-and-responsibility.md) | [Open Q&A](../qnas/020-component-boundaries-and-responsibility.md) | [021 - Pure components and predictable rendering](../lessons/021-pure-components-and-predictable-rendering.md) |