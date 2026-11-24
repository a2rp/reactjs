# 024. Component identity, position, and keys

## Learning goals

By the end of this lesson, you should be able to:

- Explain why component state can persist in one render and reset in another.
- Describe how component type, tree position, and `key` help React identify a component.
- Choose stable keys for changing lists.
- Use a changing key intentionally when a subtree should start fresh.
- Recognize accidental state resets caused by nested component definitions.
- Decide whether state should reset, remain mounted, or move to a parent.

## State belongs to a place in the UI tree

A component's state is kept by React and associated with that component's identity in the rendered UI tree. It is not stored inside the JSX tag or in the local variable that names a state setter. When React renders the same component at the same place in the tree, its state is generally preserved. When that identity changes or the component is removed, React discards that state.

Consider a score component:

~~~jsx
function Counter({ player }) {
  const [score, setScore] = useState(0);

  return (
    <section>
      <h2>{player}</h2>
      <p>Score: {score}</p>
      <button type="button" onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </section>
  );
}
~~~

If a parent renders `<Counter player="Ari" />` and later passes `player="Bea"` to the same `Counter` at the same tree position, React sees the same component identity with a changed prop. The `score` state is preserved. Props changing does not automatically reset local state.

A useful mental model is that React matches a component using its place in the returned tree, the component or element type at that place, and any explicit key. This is about the rendered tree, not the line number or branch of JavaScript that produced it.

## Position and type affect preservation

Imagine two conditional branches that each return a `Counter` as the first child under the same parent:

~~~jsx
return isFancy
  ? <div><Counter player="Ari" /></div>
  : <div><Counter player="Ari" />
    <p>Regular mode</p>
  </div>;
~~~

The two render results have a `div` at the same place and a `Counter` in the same child position. The counter's state can therefore be preserved as the condition changes. React compares the resulting tree; it does not track which `return` statement ran.

If the child component type changes at that position, React removes the old component and creates the new one. State in the removed subtree is discarded:

~~~jsx
return isEditing
  ? <EditProfile />
  : <ProfileSummary />;
~~~

Similarly, when a condition removes a component entirely, its state is destroyed. Rendering it again creates a fresh instance with its initial state. This behavior is often helpful, but it can surprise you when temporarily hiding a form or switching the selected record.

## Keys identify items among siblings

When rendering a collection, give each mapped item a stable key that identifies that data item among its siblings:

~~~jsx
{contacts.map((contact) => (
  <ContactRow key={contact.id} contact={contact} />
))}
~~~

Keys tell React which rendered item corresponds to which item in the data when list entries are inserted, removed, or reordered. With stable keys, React can preserve the right row's component state and update the intended DOM nodes. The key should come from the data, such as a database ID or another stable identifier.

Keys must be unique among items with the same parent. They do not need to be globally unique: two separate lists can use the same key values. A key should also remain stable for the lifetime of the item. Do not generate it while rendering with a random number or the current time.

### Why an array index can be the wrong key

An array index identifies a position in the current array, not the underlying item. If items are inserted, deleted, or reordered, the item at index `0` may be a different record than before. React may then preserve state or input values for the wrong row.

Use an index only when the collection is genuinely static: its items will not be reordered, inserted, or removed, and positions reliably represent identity. Even then, a stable data ID is usually easier to maintain as requirements change.

### Keys are not ordinary props

React consumes `key` while matching elements. It does not pass `key` to the component as a prop. If the child needs the identifier, pass it separately:

~~~jsx
<ContactRow key={contact.id} contactId={contact.id} contact={contact} />
~~~

The `key` helps React match the component. `contactId` is ordinary input the component can read.

If one list item needs to return several sibling elements, use a keyed `Fragment`:

~~~jsx
import { Fragment } from 'react';

{groups.map((group) => (
  <Fragment key={group.id}>
    <h2>{group.name}</h2>
    <GroupItems items={group.items} />
  </Fragment>
))}
~~~

The short fragment syntax `<>...</>` cannot receive a key.

## Use a key to intentionally reset state

A key can distinguish different conceptual instances even when they use the same component type in the same location. For example, a message editor may keep a draft in its local state. When the selected recipient changes, you may want a new editor with a fresh draft:

~~~jsx
<MessageEditor key={recipient.id} recipient={recipient} />
~~~

When `recipient.id` changes, React treats the editor subtree as a different identity and resets its local state. The key can therefore be used outside list rendering.

This is useful when the new item should begin as a separate editing session, such as changing from one profile record to another. It also discards the old subtree's local state and recreates its DOM. If a draft should survive switching away and back, keep drafts in a parent keyed by record ID or use another appropriate storage owner instead of resetting them.

Do not change a key on every render. A key such as `Math.random()` makes React recreate the component each time, which loses local state, user input, focus, and can do unnecessary DOM work.

## Keep component definitions stable

Declare component functions at the top level of a module or another stable module scope. Avoid defining a component function inside a rendering component:

~~~jsx
function ProfilePage() {
  function TextField() {
    const [text, setText] = useState('');
    return <input value={text} onChange={(event) => setText(event.target.value)} />;
  }

  return <TextField />;
}
~~~

Each time `ProfilePage` runs, that nested declaration creates a new function object. React can see it as a different component type and reset the input subtree. Move `TextField` outside `ProfilePage` and pass the values it needs through props. This also makes the component boundary easier to reuse and inspect.

## Decide what should happen when UI changes

When a component disappears or its identity changes, ask where its state belongs:

- If the next screen represents a new task or record, reset the local state intentionally, possibly with a key.
- If a panel is only temporarily hidden and its draft must remain, keep it rendered and hide it appropriately, or lift the draft state to a parent.
- If a value must survive navigation or a page refresh, component-local state alone may not be the right storage location.
- If switching between records should retain a separate draft for each one, store the drafts by record ID in an owner that remains mounted.

State reset is not a bug by itself. It is a consequence of the tree identity you render. Make the reset or preservation behavior match the user's task.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming state belongs to a JSX line | JSX source location does not determine the rendered component identity. | Reason about the component type, key, and position in the resulting tree. |
| Using array indexes for reorderable records | State or DOM can become associated with a different item after a change. | Use a stable identifier from the data. |
| Generating a new key every render | Every render creates a different identity and resets the subtree. | Keep the key stable while the item remains the same. |
| Expecting `key` to appear in component props | React uses it internally when matching elements. | Pass a separate ordinary ID prop if the child needs the value. |
| Changing a key to reset a component whose draft should persist | The subtree and local state are recreated. | Lift the draft to an owner that remains mounted or store it per record. |
| Nesting a component function inside another component | A new component type can be created on each parent render. | Define component functions at module scope and pass props. |
| Assuming different conditional branches always reset state | The returned tree may still have the same type at the same position. | Check the actual rendered tree; use distinct positions or keys when identities must differ. |

## Key takeaways

- React associates state with a component identity in the rendered tree.
- The same component type at the same position usually preserves state; removal or a different type resets it.
- Stable keys identify list items among siblings when their order or membership changes.
- Keys are local to a parent and are not passed as ordinary props.
- Changing a key intentionally resets that component subtree; changing it accidentally loses state and user input.
- Keep component function definitions stable by declaring them outside other components.
- Decide whether state should reset, stay mounted, move up, or live elsewhere based on the user's task.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [023 - Thinking in React: from a design to a component tree](023-thinking-in-react-and-component-trees.md) | [Open Q&A](../qnas/024-component-identity-position-and-keys.md) | [Open Example](../examples/024-component-identity-position-and-keys.md) | [025 - Event handlers and user interactions](025-event-handlers-and-user-interactions.md) |