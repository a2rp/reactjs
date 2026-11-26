# 020. Component boundaries and responsibility

## Learning goals

By the end of this lesson, you should be able to:

- Break a screen into a useful component hierarchy.
- Give components clear names and focused responsibilities.
- Decide when JSX should stay in its parent and when it should become a component.
- Pass data through props to the components that render it.
- Keep component definitions at the top level of a module.
- Choose whether related components belong together in one file or separate files.

## What is a component boundary?

A component boundary is the point where a piece of the interface is represented by its own React component. The component has a name, receives the values it needs, and returns the JSX for that part of the UI.

A useful boundary makes the interface easier to understand or change. It does not mean every HTML tag or every line of JSX needs a component. A component can contain several related elements when they form one understandable piece of the interface.

~~~jsx
function ProfileCard({ name, role }) {
  return (
    <article>
      <h2>{name}</h2>
      <p>{role}</p>
    </article>
  );
}
~~~

`ProfileCard` has a meaningful name and one coherent job: presenting a person's profile summary. Its heading and paragraph do not need separate components just because they are separate tags.

## Start from the screen and build a hierarchy

When planning a page, use the visual design and the data it displays to find useful pieces:

1. Identify the major page regions, such as a header, navigation, main content, or footer.
2. Notice repeated structures, such as rows, cards, or navigation links.
3. Look for a piece with its own meaning or behavior, such as a search control or status label.
4. Sketch which component contains or arranges the others.
5. Keep a small detail inside its parent if extracting it would not make the structure clearer.

For a product catalog, the hierarchy could be:

~~~text
App
+-- ProductCatalog
    +-- CatalogIntro
    +-- ProductTable
        +-- ProductRow (one per product)
            +-- StockStatus
~~~

This hierarchy is a guide, not a required number of components. For example, a table header can remain inside `ProductTable` while it is simple. If it gains sorting controls or other behavior, it may deserve a separate component.

## Give each component a coherent responsibility

A component should have a clear reason to exist. It might:

- Represent a meaningful region of the interface.
- Render a repeated record or visual pattern.
- Own a focused interaction or visual state.
- Arrange several child components into a page or section.
- Hide a complex implementation behind a clear prop interface.

A component's responsibility can include several related details. `ProductRow`, for example, can render the product name, price, category, and stock status because those details together describe one row. It does not need to be split into one component per cell unless a cell has separate meaning, reuse, or behavior.

## Build from data and pass props down

A parent can own the data and pass it to the component that needs to render it. Child components receive values through props:

~~~jsx
const products = [
  { id: 'p-101', name: 'Notebook', price: '$4', inStock: true },
  { id: 'p-102', name: 'Desk lamp', price: '$22', inStock: false },
];

function App() {
  return <ProductCatalog products={products} />;
}
~~~

`App` composes the page. `ProductCatalog` receives the collection and can pass it to the table that renders it. A product row receives one product record. This makes each component's inputs explicit and keeps the parent-child data path understandable.

A component should not need to reach into another component's local variables. Pass the data it needs through props, or use the composition patterns from the previous chapter when a wrapper only arranges supplied content.

## Split a component when it improves the design

Extract JSX into a child component when one or more of these are true:

- The markup represents a repeated item or pattern.
- The section has a clear name that helps explain the parent.
- It has behavior or display rules that can be understood separately.
- The parent has grown difficult to scan because it mixes unrelated sections.
- The piece is reused or is likely to be reused with different data.
- It needs to be changed independently from the surrounding interface.

Keep JSX in the parent when it is small, used once, and easier to understand in place. A component is not automatically better just because it has a short body.

## Avoid components that are too large or too small

A large component can become hard to reason about when it mixes unrelated UI, data formatting, and multiple interactions in one return tree. Extract coherent sections so the parent describes how the page fits together.

On the other hand, a component for every wrapper or text node adds indirection. Readers have to jump through many names to understand a simple page. A useful component boundary makes the structure clearer than the JSX it replaces.

Do not use line count as the only rule. A long component with one clear job can be easier to understand than several tiny components with vague names. Judge by the meaning and responsibilities in the code.

## Keep component definitions at the top level

Define components at the module's top level rather than inside another component:

~~~jsx
function ProductRow({ product }) {
  return <tr><th scope="row">{product.name}</th></tr>;
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

Avoid defining `ProductRow` inside `ProductTable`. A nested definition creates a new component function when the parent renders. React can treat it as a different component type, which may reset child state and cause unnecessary work. A top-level definition also makes the child easier to reuse and inspect.

## Choose file boundaries based on cohesion

Several small components can live in one file when they are tightly related and the file stays easy to scan. Move a component to its own file when it is reused elsewhere, has grown substantial, or makes the original file difficult to navigate.

A file boundary is an organizational choice. It does not change the component hierarchy or how props flow. Use clear component names and consistent exports so imports show where each piece comes from.

## Names describe the component's role

Use descriptive names that tell readers what UI or responsibility the component represents:

- `ProductRow` describes one product's row.
- `StockStatus` describes the availability label.
- `ProductCatalog` describes the larger section that organizes those pieces.

Avoid names that only describe implementation details, such as `Wrapper2`, or names so broad that they do not explain the component's role. React component names start with a capital letter so JSX can distinguish them from lowercase HTML tags.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Creating a component for every HTML tag | The component tree becomes noisy without adding meaning. | Keep simple markup together unless it has a useful responsibility. |
| Keeping an entire complex page in one component | Unrelated sections become difficult to scan and change. | Extract named regions or repeated structures. |
| Splitting components only by line count | A short component may add indirection without clarifying the UI. | Split by meaning, reuse, behavior, or independent change. |
| Making one component handle unrelated page sections | Its props and logic can grow into a difficult interface. | Separate responsibilities that have different purposes. |
| Passing data through hidden variables or reaching into a parent's scope | Component inputs are less clear and reuse becomes harder. | Pass required data through props or supply content through composition. |
| Defining a component function inside another component | The component identity can change between renders and child state may reset. | Define components at the top level of the module. |
| Moving every component into its own file immediately | Related code becomes harder to read across many files. | Keep closely related small components together until the file needs separation. |
| Naming a component after a generic wrapper | The name does not explain its purpose. | Prefer names that describe the UI role or data it represents. |

## Key takeaways

- A component boundary should make the UI easier to name, reuse, understand, or change.
- Start with the page hierarchy, then extract meaningful regions and repeated structures.
- Keep related markup together when it has one coherent responsibility.
- Pass data through props to the components that render it.
- Define components at the top level, not inside another component.
- Choose file boundaries based on reuse and readability, not as a rule that every component gets a file.
- Avoid both one giant component and a component for every tiny element.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="019-rendering-collections-and-stable-keys.md">019 - Rendering collections and choosing stable keys</a></td><td><a href="../qnas/020-component-boundaries-and-responsibility.md">Q&amp;A</a></td><td><a href="../examples/020-component-boundaries-and-responsibility.md">Example</a></td><td><a href="021-pure-components-and-predictable-rendering.md">021 - Pure components and predictable rendering</a></td></tr>
  </tbody>
</table>
