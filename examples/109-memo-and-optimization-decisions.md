# Memoization with memo — Complete Example

This example compares a plain React implementation with a measured manual optimization. It uses a product search with enough rows to make profiling meaningful, plus a theme toggle and selection state that update the parent independently of the list.

The code is an exercise in deciding where memoization helps. It does not guarantee that every device or data set will become faster.

## 1. Create stable sample data

Put the following at the top of src/App.jsx, outside the components. The data is created once when the module loads, so it does not become a new array every time App renders.

    const products = Array.from({ length: 5000 }, (_, index) => ({
      id: index + 1,
      name: 'Product ' + (index + 1),
      category: ['Books', 'Home', 'Office'][index % 3],
    }));

    function filterProducts(items, query, category) {
      const normalizedQuery = query.trim().toLowerCase();

      return items.filter((product) => {
        const matchesQuery = product.name
          .toLowerCase()
          .includes(normalizedQuery);
        const matchesCategory =
          category === 'all' || product.category === category;

        return matchesQuery && matchesCategory;
      });
    }

filterProducts is a pure calculation: it reads its arguments and returns a result without changing shared data or performing side effects.

## 2. Baseline: ordinary React without manual memoization

Start with this complete component implementation in src/App.jsx, after the data and filterProducts declarations:

    import { useState } from 'react';

    function ProductResults({ items, onSelect }) {
      return (
        <ul>
          {items.map((product) => (
            <li key={product.id}>
              <button onClick={() => onSelect(product)}>
                {product.name} — {product.category}
              </button>
            </li>
          ))}
        </ul>
      );
    }

    export default function App() {
      const [query, setQuery] = useState('');
      const [category, setCategory] = useState('all');
      const [darkTheme, setDarkTheme] = useState(false);
      const [selectedName, setSelectedName] = useState('');

      const visibleProducts = filterProducts(products, query, category);

      return (
        <main className={darkTheme ? 'dark' : 'light'}>
          <h1>Product search</h1>

          <label>
            Search
            {' '}
            <input
              value={query}
              onChange={(event) => setQuery(event.target.value)}
              placeholder="Try Product 42"
            />
          </label>

          <label>
            {' '}
            Category
            {' '}
            <select
              value={category}
              onChange={(event) => setCategory(event.target.value)}
            >
              <option value="all">All</option>
              <option value="Books">Books</option>
              <option value="Home">Home</option>
              <option value="Office">Office</option>
            </select>
          </label>

          <button onClick={() => setDarkTheme((current) => !current)}>
            Toggle theme
          </button>

          <p>
            Showing {visibleProducts.length} of {products.length} products
          </p>

          {selectedName && <p>Selected: {selectedName}</p>}

          <ProductResults
            items={visibleProducts}
            onSelect={(product) => setSelectedName(product.name)}
          />
        </main>
      );
    }

Use this baseline to observe the normal behavior. When theme or selection state changes, App renders again, recomputes the filtered array, and gives ProductResults a new array and callback. ProductResults renders again too.

Nothing here is incorrect. For a small list or an already-fast interface, this may be the best version because it is the simplest.

## 3. Manual optimization after identifying repeated work

For a manual memoization comparison, replace the import and component implementation with this version. Keep the products and filterProducts declarations unchanged.

    import { memo, useCallback, useMemo, useState } from 'react';

    const ProductResults = memo(function ProductResults({ items, onSelect }) {
      return (
        <ul>
          {items.map((product) => (
            <li key={product.id}>
              <button onClick={() => onSelect(product)}>
                {product.name} — {product.category}
              </button>
            </li>
          ))}
        </ul>
      );
    });

    export default function App() {
      const [query, setQuery] = useState('');
      const [category, setCategory] = useState('all');
      const [darkTheme, setDarkTheme] = useState(false);
      const [selectedName, setSelectedName] = useState('');

      const visibleProducts = useMemo(
        () => filterProducts(products, query, category),
        [query, category]
      );

      const handleSelect = useCallback((product) => {
        setSelectedName(product.name);
      }, []);

      return (
        <main className={darkTheme ? 'dark' : 'light'}>
          <h1>Product search</h1>

          <label>
            Search
            {' '}
            <input
              value={query}
              onChange={(event) => setQuery(event.target.value)}
              placeholder="Try Product 42"
            />
          </label>

          <label>
            {' '}
            Category
            {' '}
            <select
              value={category}
              onChange={(event) => setCategory(event.target.value)}
            >
              <option value="all">All</option>
              <option value="Books">Books</option>
              <option value="Home">Home</option>
              <option value="Office">Office</option>
            </select>
          </label>

          <button onClick={() => setDarkTheme((current) => !current)}>
            Toggle theme
          </button>

          <p>
            Showing {visibleProducts.length} of {products.length} products
          </p>

          {selectedName && <p>Selected: {selectedName}</p>}

          <ProductResults
            items={visibleProducts}
            onSelect={handleSelect}
          />
        </main>
      );
    });

The two versions demonstrate three separate choices:

- memo lets ProductResults skip a parent-caused render when both props compare equal.
- useMemo keeps the filtered array reference stable while query and category stay the same.
- useCallback keeps the selection callback reference stable while its dependencies stay the same.

The callback has no changing captured values: setSelectedName is a stable state setter, and the callback receives the selected product as an argument.

## 4. Predict the expected behavior

With the manual version:

| Interaction | Parent updates? | ProductResults expected to render? | Reason |
| --- | --- | --- | --- |
| Toggle theme | Yes | Usually no | The filtered array and callback references remain unchanged |
| Select a product | Yes | Usually no | The selected name changes, but the list props stay unchanged |
| Type a different search | Yes | Yes | The filtered product list changes |
| Choose a different category | Yes | Yes | The filtered product list changes |
| Type the same query value again | State may not change | No additional useful list work | React can skip an unchanged state update |

These are expectations for this code and its current React setup, not a promise about every render in every situation. A component can still render for its own state, context, or other React reasons.

## 5. Profile instead of guessing

1. Open React Developer Tools and select the Profiler.
2. Record a baseline with the ordinary version.
3. Toggle the theme and select a product; note whether ProductResults renders and how much time the interaction takes.
4. Switch to the manual version and repeat the same interactions with the same data.
5. Type a search query and confirm that the results update correctly.
6. Keep the manual optimization only if it reduces meaningful work without making the implementation harder to maintain.

A skipped list render may reduce React work, but it does not prove that the total interface is faster on every device. Use the same interactions and data for a useful comparison.

## 6. Try a React Compiler version

If the project uses the supported React Compiler setup from the previous lesson, start with the plain baseline implementation and profile it. The compiler can automatically memoize eligible components, calculations, and functions.

Do not add manual memoization everywhere just because the app has a compiler. Existing manual memoization remains supported; simplify it only after checking the compiler diagnostics, behavior, and measurements.

## 7. What this example teaches

- A memoized component needs stable props for the default comparison to skip a parent-caused render.
- useMemo caches a calculation result; useCallback caches a function identity.
- Search and category changes should update the list because its actual inputs changed.
- Theme and selection changes do not need to recompute the filtered list in this example.
- Profiling tells you whether a more complex implementation is worthwhile.
- The plain version is already correct; memoization is only a performance optimization.
