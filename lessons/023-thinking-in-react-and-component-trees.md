# 023. Thinking in React: from a design to a component tree

## Learning goals

By the end of this lesson, you should be able to:

- Turn a visual design and data shape into a sensible component hierarchy.
- Separate components by responsibility and meaningful reuse.
- Build the static interface from props before adding interactions.
- Distinguish props, state, and values that can be calculated during rendering.
- Put shared state in the closest parent that coordinates its consumers.
- Trace data down through props and interaction requests back through callbacks.

## Start with the interface and the data

Before writing components, understand what the screen displays and what users can do. Look at the visual design, the data available to the page, and the expected behavior. Do not begin by making a component for every element in the mockup. A heading or `div` is not automatically a useful component boundary.

For a product inventory table, write down the requirements first:

- Show products grouped by category.
- Show each product's name, price, and stock status.
- Let the user search by name.
- Let the user show only products that are in stock.

Then inspect the shape of the data. A small data record might look like this:

~~~js
{
  id: 'apple',
  name: 'Apple',
  category: 'Fruit',
  price: '$1.00',
  stocked: true,
}
~~~

The `id` identifies the product for rendering a list. The other properties describe what the interface displays or uses to filter. Keeping the data shape in mind helps you decide which components need which inputs.

## Break the design into components

Look for meaningful visual and behavioral regions. Ask:

- Does this region have its own responsibility or useful name?
- Is it repeated for several records?
- Does it need different inputs or behavior from its parent?
- Will another part of the interface need to coordinate with it?

One possible hierarchy for the inventory screen is:

~~~text
App
+-- FilterableProductTable
    +-- SearchBar
    +-- ProductTable
        +-- ProductCategory
            +-- ProductRow
~~~

Each component has a focused role:

- `App` supplies the product data.
- `FilterableProductTable` coordinates the filter inputs and the visible results.
- `SearchBar` displays the search and stock controls.
- `ProductTable` presents the matching products grouped in a table.
- `ProductCategory` renders a category heading and its product rows.
- `ProductRow` displays one product.

This is one reasonable decomposition, not a rule that every screen must follow exactly. Combine components when a split adds indirection without clarifying responsibility. Extract one when a repeated region or distinct behavior benefits from a named boundary.

## Build a static version first

Start by rendering the design without interactions. Put the data in the top-level owner that has it, then pass the required values down as props. At this stage, a search field can display a fixed value and the table can show every product. The objective is to check the structure and data mapping before adding state.

A top-down data flow makes dependencies visible. `ProductTable` receives a product list; each `ProductCategory` receives one category and its products; each `ProductRow` receives one product. Children should not reach into unrelated components to find data.

For repeated elements, map over data and use stable keys such as a product's `id` or a category name. Avoid inventing array positions as IDs when records already have stable identifiers. Stable keys help React match each rendered item with the corresponding data record across updates.

## Add behavior by identifying the minimum state

Once the static interface is clear, list what can change because of user interaction. In this example, the user can change:

- The search query.
- Whether to limit results to in-stock products.

Those are state candidates. The products themselves are supplied data, not state owned by the table. The filtered products are calculated from the product data and current filter values, so they should be derived during rendering instead of stored as another state variable.

| Value | Where it comes from | How to represent it |
| --- | --- | --- |
| Product records | `App` or a data source | Pass as props. |
| Search query | User input | State in the component coordinating the input and results. |
| In-stock-only choice | User input | State in the same coordinating component. |
| Visible products | Products plus the current filters | Calculate during render. |
| Product category heading | Each product's category | Derive from the supplied records. |

Keeping derived values out of state avoids duplicate data that can go out of sync. If `visibleProducts` were stored separately, every query or stock change would need to update both the filter state and the result state correctly. Calculating the list from current inputs gives one consistent result.

## Put state where coordination happens

`SearchBar` needs the current query and stock choice to display the controls. `ProductTable` needs those values indirectly through the filtered product list. Their common parent, `FilterableProductTable`, coordinates both regions, so it is a natural owner for the filter state.

The data flow is:

1. `FilterableProductTable` passes the current values and callback props down to `SearchBar`.
2. `SearchBar` calls a callback when the user changes a control.
3. `FilterableProductTable` updates its state.
4. It derives a new visible product list and passes it to `ProductTable`.

This keeps the current value in one place. The input is controlled by the value prop; it does not keep a separate copy that could disagree with the table. The state owner need not be the application's root. Place it in the lowest common ancestor that needs to coordinate the changing value.

## Keep the first implementation simple

A useful build sequence is:

1. Describe the screen's content and user actions.
2. Inspect the data shape and identify stable IDs.
3. Sketch component regions and their responsibilities.
4. Build the static component tree with props.
5. Render lists and verify the content and grouping.
6. Identify state candidates; remove props, constants, and derived values from that list.
7. Move each shared state value to the closest coordinating parent.
8. Pass current values down and callbacks up.
9. Check each interaction and refine component boundaries when the code reveals a real need.

Do not add an Effect to keep filtered data synchronized with search state. Filtering is a calculation from the current data and inputs, so derive it during rendering. Effects are for synchronizing with external systems; they are not needed to connect values that can be calculated inside the component tree.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Creating a component for every tag in the design | The tree becomes fragmented without clarifying responsibility. | Split at meaningful regions, repeated patterns, or distinct behavior. |
| Leaving the entire page in one component | Data flow and responsibilities become difficult to follow. | Extract components when a named boundary makes ownership or reuse clearer. |
| Adding interactivity before the static layout is clear | Structure and state problems get mixed together. | Render the data-driven static version first, then add interactions. |
| Keeping both filters and filtered results in state | The duplicate result can become stale when one input changes. | Derive visible results from source data and current filters. |
| Putting state in a distant ancestor that does not coordinate it | Unrelated parts of the app become coupled to local UI details. | Use the closest common parent that needs the value. |
| Letting each consumer keep its own copy of shared state | The controls and displayed results can disagree. | Lift the shared value to one owner and pass it down. |
| Passing the entire app data object to every child | Components depend on more data than they need. | Pass the focused values each component uses. |
| Using an Effect to calculate filtered data | It adds another update and another value to keep consistent. | Calculate derived data during render. |

## Key takeaways

- Begin with the interface, user actions, and data shape�not a component for every DOM element.
- Choose component boundaries that clarify responsibility, repeated content, or behavior.
- Build a static tree with props before adding interactions.
- Use state for values users can change; derive values that can be calculated from props and state.
- Put shared state in the closest parent that coordinates all consumers.
- Pass current values down and callbacks up through the component tree.
- Refine the hierarchy as real requirements become clear.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="022-designing-reusable-component-apis.md">022 - Designing reusable component APIs</a></td><td><a href="../qnas/023-thinking-in-react-and-component-trees.md">Q&amp;A</a></td><td><a href="../examples/023-thinking-in-react-and-component-trees.md">Example</a></td><td><a href="024-component-identity-position-and-keys.md">024 - Component identity, position, and keys</a></td></tr>
  </tbody>
</table>
