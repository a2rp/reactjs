# 024. Component identity, position, and keys - Q&A

## State and component identity

### 1. Where does React keep component state?

React keeps state and associates it with a component's identity in the rendered UI tree. State is not stored in the JSX tag or in a local variable that names the component.

### 2. What helps React decide which rendered component is the same one as before?

Its position in the resulting tree, its element or component type, and any explicit key contribute to its identity. This describes the rendered tree, not the line number or branch of JavaScript that produced the JSX.

### 3. When is state usually preserved?

When the same component type remains at the same tree position with the same key, React generally preserves its state across renders. Changes to ordinary props do not by themselves reset it.

### 4. Does changing a prop reset local state?

Not automatically. For example, if the same `Counter` stays at the same position but its `player` prop changes, its score state is preserved unless its identity also changes or it is removed.

### 5. What happens when a component is removed from the tree?

React destroys its local state. If it is rendered again later, it is a new instance and starts from its initial state.

### 6. What happens when a different component type appears at the same position?

React removes the old component subtree and creates the new one. State in the old subtree is reset rather than transferred to the new component type.

### 7. Do two conditional branches always create different identities?

No. If both branches produce the same component type at the same position, React can preserve its state. React matches the resulting tree; it does not care which conditional branch or return statement produced it.

### 8. Does wrapping a component in a different parent type affect its identity?

It can. Changing the rendered structure above a component can change its tree position or replace an ancestor, causing the component subtree to be removed and recreated. Inspect the full rendered tree when state resets unexpectedly.

## Keys in collections

### 9. What is a key used for?

A key identifies an element among its siblings so React can match it with the same data item when list items are inserted, removed, or reordered. A stable key helps React preserve the correct row's state and update the intended DOM.

### 10. What makes a good key?

A key should be stable for the data item and unique among its siblings. A database ID or another stable identifier from the data is usually appropriate.

### 11. Do keys have to be unique across the whole application?

No. Keys only need to be unique among siblings under the same parent. Separate lists can reuse the same key values.

### 12. Why can an array index be a bad key?

The index identifies the current position, not the underlying record. If an item is inserted, deleted, or moved, the same index can refer to a different record, so React may associate existing component state or DOM with the wrong item.

### 13. Can I use an index as a key at all?

It can be acceptable for a genuinely static list that will not be reordered, inserted into, or deleted from, and where position reliably represents identity. A stable data ID is usually safer as requirements evolve.

### 14. Why shouldn't I use `Math.random()` as a key?

It generates a different identity on each render. React then recreates the component and DOM instead of matching the existing item, which can lose local state, focus, or typed input and do unnecessary work.

### 15. Why isn't a key passed into the component as a prop?

`key` is metadata React uses to match elements. It is not part of the component's props. Pass a separate value such as `contactId={contact.id}` when the component needs to read that identifier.

### 16. Where should I put the key when mapping data?

Put it on the outermost element or component returned directly for each mapped item:

~~~jsx
{people.map((person) => (
  <PersonRow key={person.id} person={person} />
))}
~~~

A key deeper inside `PersonRow` does not identify the list item to its parent.

### 17. How do I key several sibling elements for one item?

Use the explicit `Fragment` component and put the key on it:

~~~jsx
import { Fragment } from 'react';

{groups.map((group) => (
  <Fragment key={group.id}>
    <h2>{group.name}</h2>
    <GroupItems items={group.items} />
  </Fragment>
))}
~~~

The shorthand fragment syntax `<>...</>` cannot receive a key.

### 18. Can two different lists use the same key values?

Yes. A key only distinguishes items among siblings with the same parent. It is not a global ID.

## Using keys to preserve or reset state

### 19. Can a key be used outside a list?

Yes. A key can give a component an explicit identity in any position. It is useful when the same component type renders different conceptual records and each record should start with separate local state.

### 20. How can a key intentionally reset a form?

Use the record identifier as the form component's key, for example `<ProfileForm key={profile.id} profile={profile} />`. When the ID changes, React treats it as a new form instance and resets the form subtree's local state.

### 21. What is the cost of changing a key?

React discards the old component subtree's local state and recreates it, including DOM elements. This can discard drafts, focus, and other local UI state. Change a key only when starting a fresh instance matches the intended behavior.

### 22. What if a draft must survive switching records?

Store the draft in an owner that remains mounted, such as parent state keyed by the record ID, or use another appropriate persistence layer. A local state value inside a removed component cannot survive its removal.

### 23. Should I change a key whenever a prop changes?

No. A prop change is normally an update to the same component. Change the key only when the value represents a different conceptual instance and resetting its subtree is desired.

### 24. How can a component be hidden without resetting its state?

Keep it in the rendered tree and hide it in a way that fits the interface, or move the important state to an owner that remains rendered. Conditionally removing the component destroys its local state.

### 25. Is keeping every hidden screen mounted always a good idea?

No. Keeping large hidden subtrees rendered can add unnecessary work and DOM. Choose based on the interface: preserve important state in a stable owner, or allow a subtree to reset when it represents a new task.

## Component definitions and debugging

### 26. Why can a nested component definition reset state?

A component function declared inside another component is recreated each time the outer component renders. React can treat the new function as a different component type and replace its subtree. Define component functions at module scope and pass changing values through props.

### 27. How do I debug an unexpected state reset?

Check whether the component was removed, whether its type changed, whether its ancestor structure or position changed, whether its key changed, and whether the component function was defined inside another component. Then compare the intended lifetime of the state with the rendered tree.

### 28. Does a stable key move a component to a different parent while preserving state?

No. Keys distinguish siblings under a particular parent. Moving an element to another parent changes its tree location, so its state is not preserved merely because the key text is the same.

### 29. What should I use for an ID if the item has no natural stable identifier?

If the items are created by the application, assign a stable ID when the item is created and store it with the data. Avoid generating a fresh ID during rendering. For fixed, non-changing items, a carefully chosen stable value can also identify them.

### 30. What is the main rule to remember?

Match component lifetime to the user's task. Keep type, position, and key stable when state should continue; remove the component or change its key when a fresh instance should reset; lift or persist state when it must outlive that component.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="023-thinking-in-react-and-component-trees.md">023 - Thinking in React: from a design to a component tree - Q&amp;A</a></td><td><a href="../lessons/024-component-identity-position-and-keys.md">Lesson</a></td><td><a href="../examples/024-component-identity-position-and-keys.md">Example</a></td><td><a href="025-event-handlers-and-user-interactions.md">025 - Event handlers and user interactions - Q&amp;A</a></td></tr>
  </tbody>
</table>
