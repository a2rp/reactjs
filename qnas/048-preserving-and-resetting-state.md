# Preserving and Resetting State — Q&A

## Core concepts

### 1. Where does React keep a component's state?

React retains state outside the component function and associates it with a component instance in the rendered tree. The function runs again during rendering, but React supplies that instance's current state values.

### 2. What determines whether state is preserved?

React preserves state when it matches the same component type at the same position in the rendered tree with the same key. If the component is removed, replaced by a different type, moved to a different position, or given a different key, React can create a new instance with fresh state.

### 3. Does a prop change by itself reset local state?

No. If the component's type, position, and key stay the same, a new prop value is normally received by the same instance, so its local state is preserved.

### 4. If a component receives a new record prop, does useState(record.name) reinitialize?

No. The useState argument supplies the initial value when that component instance is created. Later prop changes do not rerun the initializer for an existing instance. Decide whether the state should follow the prop, reset when identity changes, or be owned by a parent.

### 5. Why can changing the rendered tree reset a child even if the child JSX looks unchanged?

React compares the resulting tree, not the source-code location of JSX. If a parent or ancestor changes the child's type or position in the tree, React may remove the previous subtree and mount a new one.

### 6. What happens to state when a component is conditionally removed?

The component is unmounted and its local state is discarded. Rendering it again creates a new instance and initializes its state again.

### 7. Does CSS visibility unmount a component?

No. If the component remains rendered and CSS or the hidden attribute makes it invisible, it remains mounted and can keep its state. Conditional rendering that removes it from the tree does unmount it.

### 8. Do keys matter only inside map calls?

No. Keys are commonly used for list children, but a key can identify any child in a parent's rendered subtree. Changing a component's key can intentionally create a new instance and reset its state.

### 9. What does a key do to component state?

A key contributes to how React identifies a child among its siblings. If the key changes, React treats that child as a different identity, removes the old subtree, and creates a new one with fresh state.

### 10. Are keys globally unique?

No. Keys need to be unique among siblings under the same parent. The same key value can be used in separate sibling lists.

### 11. Does a component receive key as a prop?

No. React consumes key for matching elements. If a child needs the same identifier, pass it as a separate prop, such as userId.

### 12. Why should keys be stable?

React needs the same data item to have the same key across renders. A changing key makes the item look new, so React recreates its subtree and may lose state, focus, selection, and other DOM continuity.

## Lists and identity

### 13. Why can an array index be a bad key?

When items are inserted, deleted, or reordered, an item's index can point to a different item. React may then reuse a stateful row for the wrong data, making input text or expanded state appear on another item.

### 14. When is an array index key acceptable?

It can be acceptable for a static list whose items will never be reordered, inserted, or removed and whose identities do not change. For lists that can change, use a stable ID stored with each item.

### 15. Why should a key not use Math.random during rendering?

It produces a different identity on each render. React treats every child as new, recreates the subtree, and can discard all local state and input. Create a stable ID when the data item is created instead.

### 16. What happens to local state when a keyed list is reordered?

If each child has a stable key derived from its data identity, React can match each child to the same item after reordering. Its local state can follow that item rather than the old array position.

### 17. Does a key need to be copied into the child for that state tracking to work?

No. React uses the key internally. Pass a separate identifier prop only if the component itself needs the ID for its behavior or output.

## Intentional resets and preservation

### 18. How can a profile editor reset its draft when the selected user changes?

Give the editor a key based on that user's stable ID:

~~~jsx
<ProfileEditor key={user.id} user={user} />
~~~

When user.id changes, React creates a new ProfileEditor instance and initializes its draft from the new user.

### 19. Why is a key-based form reset often preferable to syncing props into state with an Effect?

The key makes the identity boundary explicit and resets the whole subtree during reconciliation. An Effect that copies props into state can produce an extra render and creates two sources of truth that can drift. Use an Effect only when there is a separate synchronization requirement.

### 20. When should you not use a key to reset a component?

Do not change the key when the same conceptual entity is receiving updated props and its local state should continue. Do not use random keys or change the key on every render. Resetting is a user-visible state change and can discard work.

### 21. How can a chat input be cleared when the recipient changes?

Give the chat subtree a key derived from the recipient's ID:

~~~jsx
<Chat key={recipient.id} recipient={recipient} />
~~~

This gives each recipient a distinct chat instance. Changing recipients discards the previous chat component's local draft. If drafts should be restored later, store them by recipient ID or keep each chat instance mounted.

### 22. How can drafts be preserved independently for many records?

Lift the drafts to a parent or another state owner and store them by stable record ID. When the selected record changes, read and update that record's draft instead of relying on one component's local state.

### 23. How can two same-type counters at the same visible location reset independently?

Give each counter a distinct key based on its conceptual identity:

~~~jsx
<Counter key={player.id} player={player} />
~~~

A different player ID creates a different counter instance even though the component type and visual position are unchanged.

### 24. If two counters should retain their own values while switching back and forth, is a changing key sufficient?

No. A key makes their identities distinct, but an unmounted instance's local state is discarded. To preserve both values, keep both instances mounted or store the values in a longer-lived owner keyed by player ID.

### 25. Can a changed key reset descendant state too?

Yes. Changing the key recreates that component's subtree, including state held by its descendants. Use this when the entire subtree belongs to the changing identity.

## Component definitions and common bugs

### 26. Why should component functions be declared at module scope?

A nested component function is recreated whenever its parent function runs. React sees the new function as a different component type and may replace the subtree, resetting its state. A module-level function has a stable identity.

### 27. What is wrong with this component?

~~~jsx
function Parent() {
  function Field() {
    const [value, setValue] = useState("");
    return <input value={value} onChange={(event) => setValue(event.target.value)} />;
  }

  return <Field />;
}
~~~

Field is defined again every time Parent renders. A parent update can therefore make React see a different component type and discard the input state. Move Field outside Parent.

### 28. If an if statement returns different JSX branches, does state always reset?

No. The source-code branches do not determine identity by themselves. If both branches produce the same component type at the same tree position with the same key, React can preserve state. If their resulting tree structure differs, state can reset.

### 29. Does wrapping a component in a different element affect its identity?

It can. Changing an ancestor element or component can replace that part of the rendered tree and its descendants. Preserve the same tree structure when the child state should persist.

### 30. What does a state initializer mean when props later change?

It initializes state for a newly created component instance. It is not a rule that continually mirrors a prop. If the state should always equal a prop, derive it during rendering; if it is an editable draft, choose an explicit owner and reset policy.

## Apply the ideas

### 31. A tab panel is conditionally rendered when active. What happens to its local state when switching tabs?

If switching away removes the panel from the tree, it unmounts and loses its local state. To preserve its state, keep the panel mounted and hide inactive panels, or lift the important state to a longer-lived owner. Consider the cost of keeping all panel trees mounted.

### 32. An editable list changes order, and each row uses key={index}. What is the likely issue?

React identifies children by their positions. After reordering, a row instance and its local state may be associated with different item data. Use each item's stable ID as the key.

### 33. A user changes from one selected account to another, but the editor draft should be cleared. What should identify the editor?

Use a stable account ID as its key. Pass the account itself or its ID separately as props if the editor needs it.

### 34. A user changes a display preference, but a timer component should keep counting. Should the timer get a key based on the preference?

No, if the timer remains the same conceptual instance. Keep its component type, position, and key stable; let the preference arrive as a prop.

### 35. A form key changes after every keystroke. What user-visible problems can occur?

React recreates the form subtree on every keystroke. The input can lose its value, focus, selection, and other child state. Use a stable identity key that changes only when the entity should change.

### 36. How should an application decide whether a reset is appropriate?

Identify what the local state represents and who owns it. Reset when the identity genuinely changes and stale state would be misleading or dangerous. Preserve or lift it when people should resume work, switch between entities, or keep drafts.

## Short recap

- State follows React's matched component identity in the rendered tree.
- A new prop does not by itself imply a new component instance.
- Unmounting, changing type or position, and changing a key can reset state.
- Stable sibling keys let list state follow data identity as items move.
- Keys are not global IDs or component props.
- Use a reset boundary only when the state should belong to the new identity.
