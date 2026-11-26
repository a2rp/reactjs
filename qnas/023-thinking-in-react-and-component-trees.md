# 023. Thinking in React: from a design to a component tree - Q&A

## Starting from a design

### 1. What does �Thinking in React� mean?

It means using the interface, its data, and its interactions to plan the component tree and data flow. Instead of starting with arbitrary files or markup, identify what each part displays, what can change, and which component should own each changing value.

### 2. What should I inspect before writing components?

Understand the visual design, the available data shape, and the user actions the interface must support. These three inputs help identify component responsibilities and the values the UI needs.

### 3. Should every visible region become a component?

No. A component is useful when its boundary clarifies responsibility, repetition, behavior, or data ownership. Making a component for every `div` or heading can add indirection without improving the design.

### 4. Should I build the whole screen in one component?

Not automatically. A small screen may be clear in one component, but a growing screen often benefits from named regions and repeated-item components. Split when doing so makes responsibilities or data flow easier to understand.

### 5. How do I choose component boundaries?

Look for regions with a distinct responsibility, repeated content, independent behavior, or a clear set of inputs. Then check whether the resulting component has a focused job and whether its call site is easier to read.

### 6. Is there one correct component hierarchy for a screen?

Usually not. Multiple trees can work. Choose a hierarchy that makes responsibilities, data flow, and future changes understandable without splitting components unnecessarily.

## Building the component tree

### 7. Why start with a static version?

It separates layout and data-mapping work from interaction and state decisions. Once the static tree displays the expected content, adding changing values is easier to reason about.

### 8. What does �static� mean in this workflow?

The interface renders its structure and supplied data, but the controls do not yet change the results. It is fine to pass fixed example values while checking the layout and component boundaries.

### 9. Where should initial data live?

Keep it in the component or data layer that already owns it, then pass the values required by child components as props. The important rule is that each child receives the inputs it needs through its interface instead of reaching into unrelated code.

### 10. Why should child components receive focused props?

Focused props make a component's dependencies visible and reduce unnecessary coupling. If a row displays a product name and price, it should not need the whole application model when it can receive that product record or just those fields.

### 11. How should a component render repeated records?

Map over the data and render one component or element per record. Use a stable identifier from the data as the key when available, rather than relying on an array position that may change meaning after insertion or filtering.

### 12. Should I split a repeated row into its own component?

Often, if naming the row clarifies its role, it receives a focused set of props, or the markup is reused. For a tiny one-off mapping, inline JSX may be simpler. Let clarity guide the split.

### 13. What does top-down data flow mean?

A parent passes current values to its children through props. A child that handles an interaction can call a callback prop to report the user's request back to a parent. This makes the path of information explicit.

## Deciding what is state

### 14. Which values should become state?

A value is a state candidate when it changes over time in response to an interaction or another update and the rendered UI needs to reflect that change. In a searchable table, the query and stock-only choice are user-controlled values.

### 15. Should product data itself be state in a filter component?

Not necessarily. If the component only receives a product list and displays a filtered view, that list can remain a prop. Use state when this component owns changes to the underlying product data, such as adding or editing products.

### 16. Should I store filtered results in state?

Usually no. If the visible results can be calculated from the product list and current filters, derive them during rendering. Storing both the filters and the result list creates duplicate information that can get out of sync.

### 17. What is derived data?

It is a value calculated from current props or state. For example, `visibleProducts` can be computed from `products`, `query`, and `onlyInStock` every render. It does not need a separate state variable when that calculation is straightforward.

### 18. Does deriving a filtered list mean React recalculates it on every render?

Yes, the calculation runs as part of rendering. For ordinary lists, this is usually the simplest and clearest approach. Performance optimizations should respond to measured needs and should not replace a sound state model.

### 19. Should I use an Effect to keep a derived list updated?

No, if the list is just a calculation from current props and state. Calculate it during rendering. Effects are for synchronizing with systems outside React, not for copying one React value into another derived value.

### 20. Is a constant the same as state?

No. A constant is a value calculated or declared for a render and does not itself persist as changing information. State is a value React preserves between renders and updates through its state setter.

## Choosing the state owner

### 21. Where should shared state live?

In the closest common parent that needs to coordinate the consumers. For a search field and a results table, their common parent can own the query and pass the current value and callback to the search field, then pass filtered results to the table.

### 22. Does all state belong in the root component?

No. Root-level state can make unrelated parts of the application depend on a local interaction. Keep a value near the components that need to coordinate it; lift it only as far as necessary.

### 23. What does lifting state up mean?

Move a shared value into a common ancestor of the components that use or change it. That ancestor passes the current value down and provides callback props so children can request changes.

### 24. Should a child own its own copy of a shared value?

Not when multiple components must stay coordinated. Give the shared value one owner and pass it to each consumer. Independent copies can disagree.

### 25. What is the difference between passing a callback up and directly changing parent state?

A child should not reach into its parent's implementation. It calls a callback prop that communicates the requested interaction. The parent decides how that request changes its state or application behavior.

### 26. Is every text input required to be controlled?

No. A native input can be uncontrolled. In an interface where React state drives filtering as the user types, a controlled input is useful because the current query is state in the coordinating component and is passed to the input with an update handler.

## Applying the workflow

### 27. How can I tell whether a state value is redundant?

Ask whether it can be calculated from existing props and state during rendering. If so, storing it separately adds synchronization work and may create inconsistencies.

### 28. When should I revise the component tree?

Revise it when implementation reveals unclear responsibilities, duplicated logic, excessive prop dependencies, or a component that is difficult to change. The first sketch is a useful starting point, not a permanent contract.

### 29. Does the component tree have to match the visual DOM nesting exactly?

No. Component boundaries are about responsibilities and composition. They often align with visible regions, but a component can return several elements or delegate to components without representing a one-to-one DOM wrapper.

### 30. What is the workflow to remember?

Understand the interface and data, sketch meaningful components, build the static tree with props, identify the minimum state, put shared state in the closest coordinating parent, derive the rest, and pass values down with callbacks for interactions.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="022-designing-reusable-component-apis.md">022 - Designing reusable component APIs - Q&amp;A</a></td><td><a href="../lessons/023-thinking-in-react-and-component-trees.md">Lesson</a></td><td><a href="../examples/023-thinking-in-react-and-component-trees.md">Example</a></td><td><a href="024-component-identity-position-and-keys.md">024 - Component identity, position, and keys - Q&amp;A</a></td></tr>
  </tbody>
</table>
