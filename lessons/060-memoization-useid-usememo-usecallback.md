# Memoization, useId, useMemo, and useCallback

## Keep work and identities stable only when it helps

React components render again when their state or relevant inputs change. Usually that is expected and inexpensive. **Memoization** is an optimization that can reuse a calculation, function identity, or component result when its inputs have not changed.

React provides **useMemo**, **useCallback**, and **memo** for targeted manual optimization. They solve related but different problems:

| API | What it can reuse | Typical reason |
| --- | --- | --- |
| **useMemo** | A calculated value | Skip a measured expensive calculation or preserve a value identity passed to a memoized consumer |
| **useCallback** | A function identity | Keep a callback prop stable for a memoized consumer or a Hook dependency |
| **memo** | A component’s previous rendered result when props are unchanged | Skip some parent-caused renders of an unchanged child |

Memoization is not needed for every component or value. Keep rendering pure, keep transient state close to where it is used, and measure before adding optimizations. If React Compiler is configured for a project, it can automatically memoize many values, functions, and components, reducing the need for manual memoization.

**useId** has a different purpose. It generates stable IDs for relationships such as a label and its input or an input and its help text. It is not a memoization API and must not generate list keys.

## Learning goals

After this lesson, you should be able to:

- distinguish **useMemo**, **useCallback**, **memo**, and **useId**;
- explain how Hook dependency arrays are compared;
- keep calculations pure and use complete dependencies;
- use memoization for an identified performance reason rather than as a correctness requirement;
- understand how object and function identity affect shallow prop comparisons;
- connect labels, inputs, and descriptions with **useId**;
- keep data-derived list keys separate from accessibility IDs;
- account for React Compiler and avoid unnecessary manual optimization.

## What memoization means

Memoization remembers a result associated with a set of inputs. In React, dependency values are compared with **Object.is**. If all dependencies match the previous render, a memoized value or callback can be reused. If one changes, React recalculates or returns a new function.

This can skip work, but it adds code and dependencies to reason about. It does not make an incorrect calculation correct, and it does not replace state.

## useMemo caches a calculation result

Call **useMemo** at the top level of a component or custom Hook:

~~~jsx
import { useMemo } from "react";

function ProductList({ products, query }) {
  const matchingProducts = useMemo(
    () => products.filter((product) => product.name.includes(query)),
    [products, query],
  );

  return <Results products={matchingProducts} />;
}
~~~

The first argument is a pure calculation function. React calls it during the initial render. On later renders, React returns the cached result while every dependency remains equal by **Object.is**; otherwise, it runs the calculation again.

The dependency array must include every reactive value used by the calculation: props, state, and variables or functions declared in the component. Write the array inline with a constant number of entries. The Hooks linter can help detect missing dependencies.

### When useMemo may help

Consider it when:

- a calculation is measurably expensive and its inputs often stay the same;
- you pass a calculated object or array to a child wrapped in **memo**, and stable identity lets that child skip work;
- a memoized value is needed as a dependency of another Hook, after considering simpler alternatives.

Most ordinary filtering of a small array is fast enough without it. **useMemo** does not make the initial calculation faster; it can skip repeating that calculation on later renders.

### Keep the calculation pure

A calculation passed to **useMemo** must not mutate props, state, or other external values. Do not use it to perform side effects or update state:

~~~jsx
// Bad: this mutates an array received from a parent.
const sortedProducts = useMemo(() => products.sort(compareProducts), [products]);

// Good: copy before sorting.
const sortedProducts = useMemo(
  () => [...products].sort(compareProducts),
  [products],
);
~~~

In development Strict Mode, React may call the calculation twice to reveal accidental impurity. One result is ignored. A pure calculation returns the same result for the same inputs and does not modify its inputs.

### Do not rely on the cache for correctness

React treats the memoized value as a performance cache. It may discard that cache in specific situations, so program behavior must remain correct if the calculation runs again. If code only works while a cached object exists, solve the underlying state or Effect design instead.

## useCallback caches a function identity

**useCallback** returns the same function object between renders while its dependencies remain equal:

~~~jsx
import { useCallback } from "react";

const handleSelect = useCallback((productId) => {
  setSelectedProductId(productId);
}, []);
~~~

React does not call this function when **useCallback** runs. It returns the function for later use, such as an event handler. Every reactive value captured by that function belongs in the dependency array.

This can help when passing a callback to a child wrapped in **memo**, or when a function itself is a dependency of another Hook. Usually, creating a new function during render is harmless; use **useCallback** only when stable function identity has a concrete benefit.

Conceptually, **useCallback(fn, dependencies)** is similar to **useMemo(() => fn, dependencies)**. The difference is that **useCallback** avoids the extra nested function syntax.

## memo can skip some child renders

Wrap a component in **memo** to let React reuse its previous rendered result when its props are unchanged:

~~~jsx
import { memo } from "react";

const ProductCard = memo(function ProductCard({ product, isSelected, onSelect }) {
  return (
    <button
      type="button"
      aria-pressed={isSelected}
      onClick={() => onSelect(product.id)}
    >
      {product.name}
    </button>
  );
});
~~~

By default, React compares each prop with its previous value using **Object.is**. A memoized component can still render again because its own state changed or because it consumed context that changed. **memo** is a performance optimization, not a promise that the component will never render again.

A newly created object, array, or function prop has a new identity. That can cause a memoized child to render again even if its contents look the same:

~~~jsx
// A new object is created every time the parent renders.
<ProductCard options={{ compact: true }} />
~~~

If stable identity is needed for a measured reason, calculate the object with **useMemo** or create the function with **useCallback**. First consider whether the child needs that value or whether the object can be created inside an Effect or outside the component.

A custom comparison function for **memo** is an advanced tool. It must compare every prop that affects output, including function props. Ignoring a changed function can preserve a closure with stale values.

## useMemo, useCallback, and memo work together

A common manual optimization has these pieces:

~~~jsx
const visibleProducts = useMemo(
  () => filterProducts(products, query),
  [products, query],
);

const handleSelect = useCallback((id) => {
  setSelectedId(id);
}, []);

return <MemoizedProductList products={visibleProducts} onSelect={handleSelect} />;
~~~

When the parent re-renders for a reason unrelated to **products** or **query**, **visibleProducts** can keep its identity. The callback can also keep its identity. If the child’s other props have not changed, **memo** may skip rendering it.

These APIs do not block updates that the child itself needs. If a displayed prop changes, the child must render to show the new value.

## Object dependencies and Effect design

A new object created in a component body has a new identity on every render:

~~~jsx
function ChatRoom({ roomId }) {
  const options = { roomId };

  useEffect(() => {
    const connection = connect(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);
}
~~~

Since **options** is new on every render, the Effect sees a changed dependency each time. Prefer creating the object inside the Effect and depending on the primitive value it needs:

~~~jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const options = { roomId };
    const connection = connect(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
}
~~~

Sometimes a stable value is genuinely useful, but **useMemo** should not be used to make an Effect’s correctness depend on a cache. Make dependencies complete and restructure the code where possible.

## useId creates accessibility IDs

Call **useId** at the top level of a component to generate a unique ID string for that component instance:

~~~jsx
import { useId } from "react";

function PasswordField() {
  const id = useId();
  const inputId = `${id}-input`;
  const hintId = `${id}-hint`;

  return (
    <div>
      <label htmlFor={inputId}>Password</label>
      <input id={inputId} type="password" aria-describedby={hintId} />
      <p id={hintId}>Use at least twelve characters.</p>
    </div>
  );
}
~~~

The generated ID connects an input to its label and description. If the component appears more than once, each instance gets IDs that do not collide within the React root.

You may use the returned value as a prefix for a few related IDs. Do not call **useId** inside a loop or condition; extract a component when each repeated item needs its own accessibility IDs.

### Do not use useId for list keys

Keys identify data items among siblings so React can preserve the correct item when a list is inserted, reordered, or removed. Use a stable ID from the data:

~~~jsx
products.map((product) => <ProductCard key={product.id} product={product} />)
~~~

Do not generate those keys with **useId**, an array index when order can change, or a random value during rendering. An accessibility ID and a data key solve different problems.

### Server rendering and multiple roots

**useId** is designed to produce matching IDs when the server and client render the same component tree. If a page has multiple independent React roots, pass a distinct **identifierPrefix** to each **createRoot** call so generated IDs from separate roots do not collide. For server-rendered roots, use the matching prefix for server rendering and **hydrateRoot**.

For a page with one React root, the default prefix is usually enough.

## React Compiler and manual memoization

When React Compiler is configured and compiling a component, it can automatically memoize many values, functions, and components. This reduces the need to manually add **useMemo**, **useCallback**, or **memo** throughout the codebase.

The Compiler is not automatically active in every React project. Check the project’s build setup and compiler diagnostics before assuming code has been optimized. Manual memoization remains useful for understanding identity and for projects that do not use the Compiler, but still add it for a specific reason and measure its effect.

The Compiler does not change the purpose of **useId**: use it for related accessibility IDs, not item keys.

## Common mistakes

- Calling **useMemo**, **useCallback**, or **useId** inside a loop, condition, or nested function.
- Leaving out a reactive dependency because adding it causes recalculation.
- Mutating an array or object inside a **useMemo** calculation.
- Using a ref or memoization to avoid representing visible UI with state.
- Wrapping every component and callback in memoization without evidence of a problem.
- Assuming **memo** prevents updates from the component’s own state or context.
- Passing fresh object and function props to a memoized child and expecting shallow comparison to treat them as equal.
- Depending on a **useMemo** cache for correctness or resource lifetime.
- Using **useId** to generate list keys or application data identifiers.

## Summary

- **useMemo** caches a pure calculation result; **useCallback** caches a function identity; **memo** may skip a child render when props are unchanged.
- Dependency arrays contain every reactive value used by the calculation or callback and are compared with **Object.is**.
- Memoization is an optimization, not a correctness guarantee. Keep calculations pure and measure before optimizing.
- Object and function identity matters when shallow prop comparison is expected to skip work.
- **useId** generates IDs for accessibility relationships such as **htmlFor** and **aria-describedby**.
- Generate list keys from stable data IDs, never from **useId**.
- React Compiler can reduce manual memoization when it is configured for the project.
