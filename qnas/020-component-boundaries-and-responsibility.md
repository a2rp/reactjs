# 020. Component boundaries and responsibility - Q&A

## Planning the component hierarchy

### 1. What is a component boundary?

It is the point where a meaningful part of the UI is represented by its own component. The component has a name, an input interface such as props, and JSX describing that part of the screen.

### 2. How do I find component boundaries in a design?

Start with the whole screen, then identify major regions, repeated patterns, and parts with their own meaning or behavior. Sketch how those pieces contain or arrange one another. Keep small details together when splitting them would not make the structure clearer.

### 3. Should every HTML element become a component?

No. Components should help explain, reuse, or independently change part of the interface. An ordinary heading, wrapper, or paragraph can stay in its parent when it has no separate responsibility.

### 4. Does every component need to be reused?

No. A component can be useful even if it appears once, for example when it represents a large, meaningful page section that is easier to understand separately. Reuse is one reason to extract a component, not the only reason.

### 5. Is there a correct number of components for a page?

No. The right hierarchy depends on the interface and how it is expected to change. The goal is a clear structure with useful responsibilities, not the largest possible component count.

### 6. Should I split a component based only on its line count?

No. Line count is only a clue that a component might be difficult to scan. A long component can still have one coherent job, while several tiny components can make simple UI harder to follow. Split by meaning, behavior, reuse, and clarity.

## Defining responsibilities

### 7. What does it mean for a component to have one responsibility?

It means the component has a coherent purpose that can be described clearly, such as rendering one product row or arranging a catalog section. It does not mean that the component can contain only one element or perform only one tiny step.

### 8. Can one component render several related pieces of information?

Yes. A `ProductRow` can render a product's name, price, and availability because they form one row about one product. Split one of those details out only when it gains its own meaning, behavior, reuse, or useful boundary.

### 9. Can a parent component arrange children without rendering all their details itself?

Yes. A page-level component often composes a heading, navigation, and content component. The parent describes how the page fits together; each child renders its own focused part.

### 10. How should data get to the component that renders it?

Pass it through props. A parent that owns a product collection can pass the collection to a table, and the table can pass one record to each row:

~~~jsx
function App() {
  return <ProductTable products={products} />;
}

function ProductTable({ products }) {
  return (
    <table>
      <tbody>
        {products.map((product) => (
          <ProductRow key={product.id} product={product} />
        ))}
      </tbody>
    </table>
  );
}
~~~

The component that needs the value receives it explicitly.

### 11. Can a child component read a variable declared inside its parent component?

Not unless the value is passed to it or otherwise made available. Component functions have their own scope. Pass the needed value as a prop rather than relying on a hidden connection to the parent's local variables.

### 12. What if an intermediate component only arranges content?

It may not need to receive page-specific data. Use the composition pattern from the previous chapter: let the component that knows the data create the consumer element and pass it as children to the layout. An intermediate component should receive data when it uses or transforms that data.

## Component definitions and organization

### 13. Where should component functions be defined?

Usually at the top level of a module, outside other component functions. This gives each component a stable definition and makes it easy to reuse and inspect.

### 14. Why should I not define one component inside another component?

The nested component function is created again whenever its parent function runs. React can then treat it as a different component type, which may cause unnecessary work and reset state in its subtree. Define it at the module level and pass its data through props.

### 15. Should every component have a separate file?

No. Keep small, tightly related components together while the file remains easy to scan. Move a component into its own file when it is reused elsewhere, grows substantially, or makes the original file hard to navigate.

### 16. Do file boundaries change the component hierarchy?

No. Files organize source code. The hierarchy is determined by which components render other components, and data still flows through the same props and children relationships.

### 17. How should I name a component?

Choose a capitalized name that describes its role or the UI it represents, such as `ProductRow`, `StockStatus`, or `ProductCatalog`. Avoid generic names that provide no context, such as `Wrapper2`.

## Practical decisions

### 18. When should I extract a repeated row or card?

Extract it when the repeated structure has a clear role or its own display rules. Then pass one data record to it and give the mapped element a stable key. The parent list can focus on arranging the collection while the row describes one item.

### 19. Should I build the whole interface as one component before splitting it?

You can start with a static version of the screen and then extract meaningful pieces as the hierarchy becomes clear. Building the data-driven structure before adding interactions helps keep responsibilities visible. You do not need to invent the final component architecture before seeing the UI.

### 20. Can a component have several props and still have a clear responsibility?

Yes. A component can need several values for one coherent job. However, a prop interface with many unrelated flags or data values can signal that the component combines unrelated roles. Consider named child components or composition if the API has become hard to understand.

### 21. Does extracting a component automatically make it reusable or better?

No. Extraction introduces a new name and boundary. It is useful when that boundary makes the code easier to understand, change, or reuse. If the caller must jump through a component that adds no meaning, keeping the JSX in place may be clearer.

### 22. How is a component boundary related to pure rendering?

A component boundary organizes the UI into named parts. Pure rendering is about a component producing predictable output from its inputs. They are related design concerns but different topics; the next chapter covers predictable rendering.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="019-rendering-collections-and-stable-keys.md">019 - Rendering collections and choosing stable keys - Q&amp;A</a></td><td><a href="../lessons/020-component-boundaries-and-responsibility.md">Lesson</a></td><td><a href="../examples/020-component-boundaries-and-responsibility.md">Example</a></td><td><a href="021-pure-components-and-predictable-rendering.md">021 - Pure components and predictable rendering - Q&amp;A</a></td></tr>
  </tbody>
</table>
