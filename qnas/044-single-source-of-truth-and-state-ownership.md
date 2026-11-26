# Single Source of Truth and State Ownership - Q&A

## Core concepts

### 1. What does “single source of truth” mean in a React component tree?

For one shared fact, there is one authoritative state value owned by one component. Other components that need the fact receive it from that owner instead of keeping separate copies.

### 2. Does it mean an application has only one state variable?

No. An application can have many state values. The principle is that each shared fact should have a clear authoritative owner.

### 3. Does it mean all state must live in the root component?

No. Put each value in the closest component that can coordinate all of its consumers. Independent values can have different owners at different levels of the tree.

### 4. What is state ownership?

State ownership is the decision about which component controls the authoritative value and applies the rules for updating it.

### 5. What is the difference between an owner and a consumer?

The owner keeps the state and decides how it changes. A consumer reads the value to render UI. A consumer can ask the owner to change it by calling a callback prop.

### 6. How do consumers receive a shared value?

Usually the owner passes the current value down through props. For widely separated descendants, Context may help distribute it without passing props through every intermediary component.

### 7. Does a consumer directly modify a prop?

No. Props are inputs received from the parent. A child can call a callback prop to report an interaction; the owner decides whether and how to update state.

### 8. How do you choose a state owner?

Identify what fact changes, which components read it, which interactions can change it, and which components must stay consistent. Then choose the closest common parent that can coordinate them.

### 9. Can a component own some state and consume other state?

Yes. A component can own local state for one concern and receive another value from a parent. Ownership is decided separately for each piece of state.

### 10. Why is one shared value safer than separate sibling copies?

Separate copies can diverge when one component updates and another does not. A shared value gives all consumers the same current source.

## Derived values and redundant state

### 11. What is redundant state?

It is state that duplicates information already represented by other current state or props, so that it can be calculated rather than stored separately.

### 12. Why might storing both a selected product ID and selected product object be risky?

If the ID changes but the stored object does not, they can describe different selections. When the object can be found from the product list and ID, deriving it avoids that synchronization problem.

### 13. How can the selected product be derived from a product list and selected ID?

Use a lookup during rendering:

~~~jsx
const selectedProduct = products.find(
  (product) => product.id === selectedProductId
);
~~~

The UI should also handle the case where no product matches, such as after the list changes.

### 14. What are common examples of derived values?

A cart total from cart items, a filtered list from items and a search query, a character count from text, or an isSelected boolean derived from an active ID.

### 15. Should a cart total usually be stored separately from the cart items?

Usually not if it can be calculated reliably from the items. Store the items and calculate the total for rendering. Store a total separately only when it represents a distinct authoritative value, such as a server-calculated amount with specific business rules.

### 16. Should a filtered list usually be stored in state separately from the source list and query?

Usually not. Filter the source list from the current query during rendering. This keeps the filtered result consistent with both inputs.

### 17. Should an Effect copy one piece of React state into another to keep them synchronized?

Usually no, if the second value can be derived. Derive it from the current state and props instead. Effects are meant for synchronization with external systems, not routine duplication inside React state.

### 18. Is calculating a derived value on every render always a performance problem?

No. Start with a direct calculation. If profiling shows that a calculation is meaningfully expensive, consider memoization as a performance optimization. Correctness should not depend on memoization.

### 19. Can a derived boolean be stored as state?

It can be, but if it is fully determined by other current state or props, deriving it is usually simpler and avoids disagreement. For example, derive isSelected by comparing item IDs.

## Shared values and local values

### 20. If the same selection appears in a dropdown, heading, and results, who should own it?

The closest common parent that coordinates those views should own one selected value and pass it to each consumer.

### 21. If only a search picker uses its query, should the page own that query?

Not unless another component needs it or the page must coordinate it. Keep the query local to the picker when it is independent.

### 22. Can one part of the interface use shared state while another uses local state?

Yes. For example, the page may own a selected product ID shared by several components, while the picker owns a local search query used only to filter its options.

### 23. Does every consumer need the entire state object?

No. Pass only the values a component needs. Smaller props make component responsibilities easier to understand.

### 24. Should the owner pass a state setter directly or an intent-based callback?

Either can work. A direct setter can be concise for simple components. A callback such as onSelectProduct communicates the supported action and keeps the parent's state representation private, which is often better for reusable components.

### 25. Can children receive the same object reference in props?

Yes. Multiple consumers can receive the same value. They should treat props as read-only and request changes through callbacks rather than mutating the object.

### 26. What does “closest common parent” have to do with single source of truth?

It is often the natural owner when multiple descendants need one fact. It can pass the value down and handle update requests from those descendants without moving unrelated state to the root.

### 27. How is Context related to state ownership?

Context is a way to distribute values to descendants. It does not itself decide which component owns or updates the value. You still need a clear owner.

## Intentional separate values

### 28. Are a saved profile and an edit draft duplicate state?

Not necessarily. A saved profile and a draft represent different stages: persisted data and unsaved changes. They can both be needed by the UI.

### 29. How can you tell whether two values are duplicates or distinct facts?

Ask whether they represent the same information at the same stage. If the user can change one without changing the other, and the difference matters to the UI, they may be distinct facts.

### 30. What is another example where two similar values can be legitimate?

A server result and an editable local copy can be different values. So can original text and a draft, or the current route and a proposed route being edited.

### 31. Should the state names reveal these different roles?

Yes. Names such as savedProfile and draftProfile make their distinction and update behavior clearer than two generic names like profile and copy.

### 32. What is a server-side source of truth?

For persisted data, a server or database may be authoritative across sessions and users. React state can be a client-side representation or draft of that data. The phrase “single source of truth” in a component tree describes ownership of the current client UI value; it does not replace server-side authority.

## Common failure modes

### 33. What happens if each sibling stores its own selected category?

They can display conflicting categories. Lift the shared selection to a common parent and pass the same value to both.

### 34. Why can copying a prop into local state make a child stale?

The prop value passed to useState is only the initial state. Later prop changes do not automatically replace that local state.

### 35. When is local state initialized from a prop valid?

When the component intentionally creates a separate local value, such as an editable draft initialized from an incoming value. The component then needs a clear rule for what happens if the incoming value changes.

### 36. Why should a child avoid mutating an object it receives in props?

Mutation bypasses the owner and can leave consumers without a clear update signal. Treat the value as read-only and ask the owner to create the next state through a callback.

### 37. What can go wrong when every state value is moved into one parent?

The parent becomes coupled to unrelated concerns, values travel farther than necessary, and ownership becomes harder to reason about. Keep independent state near its use.

### 38. What if the stored selected ID no longer exists in the current list?

The derived lookup may return undefined. Render an appropriate empty or fallback state, or update the selection as part of the list-changing action if the product behavior requires it. Avoid an Effect that blindly copies a potentially missing object into duplicate state.

### 39. Should a component store both isSelected and selectedId if it can calculate one from the other?

Usually no. Derive the boolean from the ID and the item's own ID to avoid conflicting values.

### 40. Is state ownership the same as where data is displayed?

No. A component may display a value without owning it. The owner is where the authoritative state and update rules live; consumers render that value.

## Apply the concepts

### 41. A heading and a filtered list both depend on the chosen category. What should change when the user selects another category?

The owner updates the one selected-category value. It passes that new value to both consumers, which render the matching heading and results.

### 42. A parent stores cart items. Two children need the total. Should they each store their own total?

No. Derive the total from the shared cart items, or calculate it once at a suitable level and pass the result. Do not maintain independent copies that can disagree.

### 43. A reusable picker needs to report an item choice. What might its callback look like?

It could call onSelectProduct(product.id). The parent receives the ID and updates whichever state representation it owns.

### 44. Does a common parent have to own all state within its descendants?

No. It owns only the facts that need coordination there. Descendants can keep unrelated local state.

### 45. What is the practical test for whether a value should be stored?

Store it if it represents an independently changing fact the UI must remember. If it can be calculated from current state and props, prefer deriving it.

## Quick review

### 46. What does a single source of truth prevent?

It prevents multiple components from maintaining competing copies of the same shared fact.

### 47. How should a value reach multiple components?

The owner passes it down to each consumer, usually through props, and receives requested changes through callbacks.

### 48. What is the central ownership rule?

Give each shared fact one clear owner at the nearest level that can coordinate its consumers; derive redundant values and keep independent state local.

## Key takeaway

One shared fact should have one clear client-side owner. Consumers receive that value and report requested changes. Store distinct facts separately, derive what can be calculated, and keep unrelated state close to its use.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="043-lifting-state-up-and-sharing-data.md">043 - Lifting State Up and Sharing Data - Q&amp;A</a></td><td><a href="../lessons/044-single-source-of-truth-and-state-ownership.md">Lesson</a></td><td><a href="../examples/044-single-source-of-truth-and-state-ownership.md">Example</a></td><td><a href="045-controlled-and-uncontrolled-component-apis.md">045 - Controlled and Uncontrolled Component APIs - Q&amp;A</a></td></tr>
  </tbody>
</table>
