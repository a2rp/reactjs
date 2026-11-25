# React Compiler — Complete Example

This example shows how to add the React Compiler to a small React application, write ordinary React code without manual memoization, and check whether compilation is active.

The compiler changes performance opportunities; it should not change the app's intended behavior. Keep the app correct even if a component is not compiled.

## 1. Start with a React project

Use an existing React application or create a Vite React project. This setup example uses the current Vite React integration for projects on a compatible version of @vitejs/plugin-react.

Install the compiler as a development dependency:

    npm install -D babel-plugin-react-compiler

For React 19, the compiler does not need a separate compiler runtime package. React 17 and 18 need the compatibility runtime and an appropriate compiler target; use the instructions that match the project's React version.

## 2. Configure the Vite build

For a compatible @vitejs/plugin-react v6 setup, add the compiler preset through the Babel integration:

    import { defineConfig } from 'vite';
    import react, { reactCompilerPreset } from '@vitejs/plugin-react';
    import babel from '@rolldown/plugin-babel';

    export default defineConfig({
      plugins: [
        react(),
        babel({
          presets: [reactCompilerPreset()],
        }),
      ],
    });

This integration is version-specific. If the project uses an older Vite React plugin or a different framework, follow the supported compiler setup for that toolchain instead of copying this configuration unchanged.

For a direct Babel setup, add babel-plugin-react-compiler before other Babel plugins that transform React code. Do not configure two competing compiler integrations in the same build.

## 3. Write a component without manual memoization

Create or replace src/App.jsx with:

    import { useState } from 'react';

    const products = [
      { id: 1, name: 'Notebook', category: 'Stationery', inStock: true },
      { id: 2, name: 'Desk lamp', category: 'Home', inStock: false },
      { id: 3, name: 'Pen set', category: 'Stationery', inStock: true },
      { id: 4, name: 'Water bottle', category: 'Travel', inStock: true },
    ];

    function ProductRow({ product }) {
      return (
        <li>
          <strong>{product.name}</strong>
          {' — '}
          {product.category}
          {' — '}
          {product.inStock ? 'In stock' : 'Out of stock'}
        </li>
      );
    }

    function ProductList({ items }) {
      if (items.length === 0) {
        return <p>No products match these filters.</p>;
      }

      return (
        <ul>
          {items.map((product) => (
            <ProductRow key={product.id} product={product} />
          ))}
        </ul>
      );
    }

    export default function App() {
      const [query, setQuery] = useState('');
      const [inStockOnly, setInStockOnly] = useState(false);

      const visibleProducts = products.filter((product) => {
        const matchesQuery = product.name
          .toLowerCase()
          .includes(query.trim().toLowerCase());

        return matchesQuery && (!inStockOnly || product.inStock);
      });

      return (
        <main>
          <h1>Product finder</h1>

          <label>
            Search products
            {' '}
            <input
              value={query}
              onChange={(event) => setQuery(event.target.value)}
              placeholder="Try stationery"
            />
          </label>

          <label>
            {' '}
            <input
              type="checkbox"
              checked={inStockOnly}
              onChange={(event) => setInStockOnly(event.target.checked)}
            />
            {' '}
            In stock only
          </label>

          <p aria-live="polite">
            Showing {visibleProducts.length} of {products.length} products
          </p>

          <ProductList items={visibleProducts} />
        </main>
      );
    }

The components use normal state, filtering, and props. There is no memo, useMemo, or useCallback call. With a supported compiler integration, the build can optimize eligible work automatically.

## 4. Run and inspect the app

Start the development server using the project's existing script:

    npm run dev

Try these interactions:

1. Type part of a product name.
2. Toggle the In stock only checkbox.
3. Clear the search and switch the checkbox repeatedly.
4. Confirm the count, list, and empty state remain correct.

Open React Developer Tools and inspect the Components panel. The compiler can mark compiled components with its memoization indicator. Check the compiler-aware ESLint diagnostics as well; they can explain code that is skipped or patterns that need attention.

To assess performance, profile a representative interaction before and after enabling the compiler. An indicator confirms compilation, but does not by itself prove that a real bottleneck improved.

## 5. Add compiler-aware lint diagnostics

Use the version of eslint-plugin-react-hooks supported by the project, and merge its compiler-aware recommended configuration into the existing ESLint setup. Do not replace unrelated project lint rules with a copied configuration.

Run the project's lint command and review compiler diagnostics. Fix actual Rules of React violations, such as mutating values during render or calling Hooks conditionally. Safe code that the compiler cannot analyze may remain uncompiled.

## 6. Debug a suspected interaction

If the app's behavior changes after enabling the compiler:

1. Reproduce the issue and identify the affected component.
2. Check lint diagnostics and the Rules of React.
3. Look for render-time mutation, impure calculations, conditional Hook calls, or assumptions that a new object or callback is created on every render.
4. If the configured compiler supports directives, temporarily put the following as the first statement inside the affected component:

       'use no memo';

5. Compare behavior with and without compilation, then fix the underlying assumption and remove the temporary directive.

Do not keep a directive as a substitute for fixing code that violates React's rules.

## 7. What this example demonstrates

- The build pipeline, rather than application code, enables the compiler.
- Components can be written without manually adding memoization everywhere.
- The compiler only optimizes code it can analyze safely.
- Correctness does not depend on whether a component is compiled.
- Lint diagnostics, Developer Tools, and profiling answer different questions: code eligibility, compilation status, and measured performance.

## Adaptation notes

- Match compiler configuration to the React, framework, bundler, Babel, and plugin versions already in the project.
- For React 17 or 18, configure the compatibility runtime and compiler target as required.
- Keep manual memoization when it is needed for a clear, measured, or documented reason; the compiler does not make memoization a correctness guarantee.
- This small list does not need optimization. The example demonstrates setup and verification, not a claim that memoization improves this particular interface.
