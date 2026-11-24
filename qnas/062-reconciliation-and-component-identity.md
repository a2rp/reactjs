# Reconciliation, Component Identity, and Keys — Q&A

## Reconciliation and component identity

### 1. What does reconciliation mean in React?

It is React’s process of comparing the next UI tree with the previous one and deciding which components and DOM nodes can be preserved, updated, removed, or created.

### 2. Does React always recreate the whole DOM during reconciliation?

No. React applies the DOM changes needed for the new UI. Unchanged nodes can be reused.

### 3. Where does React keep a component’s state?

React associates state with a component’s identity in the render tree, not with a temporary JavaScript function call.

### 4. What influences component identity?

The component’s parent position, element/component type, and key are important parts of its identity. The exact reconciliation algorithm is an implementation detail.

### 5. When is state usually preserved?

When React continues to render the same component type with the same identity at the same position in the tree.

### 6. When is state reset?

When the component is removed, a different type replaces it, or its identity changes, such as when its key changes. React creates a new instance with fresh local state.

### 7. Do two instances of the same component share state?

No. Two instances at different positions each have their own state, even if they use the same component function.

### 8. Does changing a component’s props automatically reset local state?

Usually no. If the type and identity remain the same, React preserves local state while supplying the new props. Use a changed key only when a full reset is intended.

### 9. What happens when two different component types appear at the same position on alternating renders?

React removes the old component and its subtree, then creates the new type. State from the removed type is discarded.

### 10. Why should component functions be declared outside other components?

A nested declaration creates a new function identity whenever the parent renders. React sees a different component type and may remount it, resetting its state and DOM state such as focus.

## List keys

### 11. What is a React key?

A key is a value React uses to match an element with its sibling identity across renders, especially when list items are inserted, deleted, or reordered.

### 12. Where should a key come from?

Prefer a stable ID already stored with the item’s data, such as a database ID or a persistent ID assigned when a local item is created.

### 13. Where should the key go when mapping items?

Put it on the top-level JSX element returned for that item from **map()**. If that element is a component, put the key on the component instance.

### 14. How unique must keys be?

They must be unique among the siblings in that rendered collection. They do not need to be globally unique across the whole app.

### 15. Can two separate lists use the same key values?

Yes. Keys are scoped to sibling elements under a parent, so unrelated lists can reuse values.

### 16. Why can array indexes be unsafe keys?

An index represents the item’s current position, not its identity. After an insertion, deletion, or reorder, React may reuse a component instance for a different item and carry its local state or DOM value to the wrong row.

### 17. Is it ever okay to use an array index as a key?

It can be acceptable for a truly static list that never reorders, inserts, or removes items and has no item-specific state or DOM state to preserve. Stable IDs remain the safer default.

### 18. Why should I not use Math.random() as a key?

It produces a new key on each render. React treats every item as new, remounts it, and can lose local state, focus, uncontrolled input values, and other DOM state.

### 19. Can I generate a missing key inline while rendering?

No. Generate or assign the ID when creating the data item, store it with the item, and reuse it on later renders.

### 20. Can useId generate keys for a list?

No. **useId** creates IDs for accessibility relationships, while keys identify data items among siblings. Use a stable ID from the data for keys.

### 21. Does a component receive key in its props?

No. React consumes **key** internally. If the component needs the same ID as data, pass it separately as a normal prop such as **personId**.

### 22. Does key appear as an HTML attribute in the DOM?

No. It is a React hint and is not rendered as a DOM attribute.

### 23. What if two siblings have duplicate keys?

React cannot reliably distinguish those siblings. Duplicate keys can produce warnings and incorrect preservation or updates. Ensure each sibling item has a unique stable key.

### 24. What if an item’s key changes because its data ID changed?

React treats it as a different identity. The previous component is removed, its local state is discarded, and a new component is mounted.

### 25. How can a list item return several sibling DOM elements and still have one key?

Use an explicit **<Fragment key={item.id}>** around the elements. The short **<>...</>** syntax does not accept a key.

## Keys and state resets

### 26. Can a key be used outside a list?

Yes. A key can distinguish component identities anywhere among siblings. Giving a form a key based on the selected record can intentionally reset its entire local state when the record changes.

### 27. Does changing a key only reset state variables?

No. React removes the old subtree and creates a new one. Its local state and refs reset, and Effects clean up as part of unmounting.

### 28. Does a keyed component preserve its state after it is removed and later shown again?

No. The key does not cache removed component instances. Once a component is removed, its local state is destroyed. Store drafts higher in the tree or keep instances mounted if they must survive switching.

### 29. Should I use a key to reset a form when switching users?

If switching users should discard all form state, rendering the editor with **key={user.id}** is a clear option. If each user’s draft should be restored, store drafts by user ID in a parent or another suitable state store.

### 30. Can the same key preserve a component when its parent changes?

No. A key only helps distinguish siblings under the same parent. Moving an element to a different parent or changing surrounding structure can remove the old instance and create a new one.

### 31. If the same component appears at two positions, can React move its state between them?

Not just because the component types match. The positions are separate identities. Keys help match siblings within one parent’s child collection; they are not a global state address.

### 32. Do keys control state for every Hook separately?

Keys affect the identity of the component instance. If the instance is preserved, its Hook state is preserved in the normal Hook order. If the instance remounts, all of its local Hook state starts fresh.

## Common pitfalls

### 33. Why might an input lose focus after an unrelated parent update?

The input or a containing component may be remounting because a component type is being redefined inside the parent, a key is changing, or the tree position/type is changing.

### 34. Why might text entered in one row appear under another row after sorting?

An index key may have caused React to associate component-local state or an uncontrolled input with the position rather than the data item. Use stable item IDs.

### 35. Does wrapping a component in a new DOM element always preserve state?

No. Changing the tree structure can change the position or parent of descendants. A wrapper added or removed conditionally may cause a subtree to be removed and recreated.

### 36. Are keys a way to force React to update a DOM attribute?

No. Keys are used to match element identity. Pass the value as an ordinary prop or DOM attribute if the rendered output needs it.

### 37. Should I use a key as an element’s accessible ID?

No. Keys are not rendered and are not available to the child. Use **id** and accessibility relationships such as **htmlFor** or **aria-describedby** for accessible DOM connections.

### 38. How can I preserve per-item draft text when only one item is visible at a time?

Lift the draft state to a parent and store it by stable item ID, or keep all relevant editors mounted. A key reset is for discarding prior local state, not saving it.

### 39. What does a key on a child at a different parent do?

It identifies that child only among its new siblings. It does not link it to an old instance under a different parent.

### 40. What is the quickest key-selection rule?

Use a stable data ID that identifies the item across renders. Avoid indexes for changing lists and never generate keys during rendering.

### 41. What should I inspect if state unexpectedly resets?

Check whether the component was removed, its type changed, its key changed, its parent or position changed, or its function was recreated inside another component.

### 42. What should I inspect if state appears on the wrong list item?

Check whether the list uses indexes, duplicate keys, or keys that change between renders. Replace them with unique stable IDs from the item data.
