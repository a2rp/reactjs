# Preserving and Resetting State with Tree Position and Keys

## Why state sometimes stays and sometimes resets

A component can receive new props and keep local state. A change to the rendered tree can instead make that state start over. Understanding the difference helps prevent lost form input, misplaced list state, and confusing conditional UI.

React stores state outside the component function. During rendering, it matches the new component tree with the previous one. State is preserved when React identifies the same component instance in the same place. When its identity or position changes, React removes the old instance and creates a new one with fresh initial state.

## Learning goals

After this lesson, you should be able to:

- explain how React associates state with a component's identity and place in the rendered tree;
- predict when state is preserved or reset;
- distinguish a prop change from a component identity change;
- use stable keys to reset a subtree intentionally and identify list children reliably;
- recognize when conditional rendering unmounts a component;
- choose whether state should reset, persist per identity, or move to a parent.

## State belongs to a rendered position

A component function describes UI. Calling a state Hook does not store its value in a local variable that survives calls. React retains the value and associates it with a component instance in the rendered tree.

A useful mental model is:

~~~text
parent component
└── child position
    └── component identity
        └── Hook state
~~~

On a later render, React compares the new tree with the previous tree. If the same component type remains at the same position with the same key, React can reuse its instance and preserve its state.

~~~jsx
import { useState } from "react";

function Counter({ label }) {
  const [count, setCount] = useState(0);

  return (
    <section>
      <h2>{label}: {count}</h2>
      <button onClick={() => setCount((current) => current + 1)}>
        Add one
      </button>
    </section>
  );
}
~~~

If a parent changes the label prop but renders Counter in the same place, the count remains. The label is new input to the same component instance.

~~~jsx
function App({ showScore }) {
  return <Counter label={showScore ? "Score" : "Count"} />;
}
~~~

Changing showScore changes the displayed label. It does not by itself create a new Counter instance.

## Same type and position preserve state

A parent can choose between different props while keeping the same component type in one position:

~~~jsx
{isFirstUser ? (
  <Profile user={firstUser} />
) : (
  <Profile user={secondUser} />
)}
~~~

React sees a Profile in that position in both renders. If Profile has local state, such as an editable draft, the state is preserved when the user prop changes.

That behavior is often right. A display setting can change while a counter remains the same counter. But it may be wrong when the new prop represents a different identity. A draft for one account should usually not become the draft for another account.

## A different type or position resets state

React resets a component's state when it removes that component from the tree or replaces it with a different component type at that position.

~~~jsx
{isEditing ? <Editor /> : <Preview />}
~~~

Editor and Preview are different types. Switching between them removes one subtree and adds the other. When Editor is rendered again, its local state begins from its initial value.

The same can happen when an ancestor changes:

~~~jsx
return isCompact
  ? <div><Counter /></div>
  : <section><Counter /></section>;
~~~

The Counter appears below a different tree structure. The change to its ancestor can cause the old subtree to be removed and a new one created.

What matters is the returned tree, not where a JSX expression appears in the source file. Two branches of an if statement can preserve state if they produce the same component at the same tree position.

### Conditional rendering and unmounting

When a condition stops rendering a component, React removes it:

~~~jsx
{isOpen && <SearchPanel />}
~~~

Closing the panel unmounts SearchPanel. If it is rendered later, its local state is initialized again. CSS that merely hides an element does not remove it from the React tree:

~~~jsx
<div hidden={!isOpen}>
  <SearchPanel />
</div>
~~~

Here SearchPanel remains mounted, so its state can be preserved while hidden. Keeping large hidden subtrees mounted can have performance and accessibility costs, so choose this only when the behavior is useful.

## Keep component types stable

Do not define a component function inside another component's render function:

~~~jsx
function Parent() {
  function TextField() {
    const [text, setText] = useState("");

    return (
      <input
        value={text}
        onChange={(event) => setText(event.target.value)}
      />
    );
  }

  return <TextField />;
}
~~~

Every Parent render creates a new TextField function. React treats that as a different component type, so the input subtree can be replaced and its state lost.

Define components at the module level instead:

~~~jsx
function TextField() {
  const [text, setText] = useState("");

  return (
    <input
      value={text}
      onChange={(event) => setText(event.target.value)}
    />
  );
}

function Parent() {
  const [updates, setUpdates] = useState(0);

  return (
    <>
      <TextField />
      <button onClick={() => setUpdates((count) => count + 1)}>
        Parent updates: {updates}
      </button>
    </>
  );
}
~~~

The TextField function identity is now stable between Parent renders.

## Keys give a component an explicit identity

A key is part of how React identifies a child among siblings. It is not only for arrays. A different key tells React that a component in the same place represents a different instance.

~~~jsx
<ProfileEditor key={user.id} user={user} />
~~~

When user.id changes, React resets ProfileEditor and its entire descendant subtree. The editor can initialize a local draft from the selected user's saved value:

~~~jsx
function ProfileEditor({ user }) {
  const [draftName, setDraftName] = useState(user.name);

  return (
    <label>
      Display name
      <input
        value={draftName}
        onChange={(event) => setDraftName(event.target.value)}
      />
    </label>
  );
}
~~~

The key is not passed as a normal prop. If the component needs the identifier, pass it separately:

~~~jsx
<ProfileEditor key={user.id} userId={user.id} user={user} />
~~~

Use a key when the change in identity should reset the whole subtree. Avoid changing a key just to refresh an input on every render. That can destroy focus, selection, user input, and child state.

## Stable keys in lists

When mapping data to elements, a stable key helps React match each rendered child to its data item:

~~~jsx
{people.map((person) => (
  <PersonCard key={person.id} person={person} />
))}
~~~

If an item is inserted, removed, or reordered, React can use the key to follow that item instead of treating its current array position as its identity. This matters when a list item owns local state, such as an expanded panel or input draft.

An array index is safe only when the list is static: items never reorder, get inserted, or get removed, and item identity never changes. For a list that can change, index keys may make state appear on the wrong row after an update.

Do not generate a new key during render:

~~~jsx
// Avoid: every render makes a new identity and resets the subtree.
{people.map((person) => (
  <PersonCard key={Math.random()} person={person} />
))}
~~~

Keys need to be stable and unique among siblings. They do not need to be unique across the application. If a component needs an identifier, pass it as another prop.

## Choosing a state strategy

When identity changes, first decide what should happen to the existing state.

### Reset the subtree

Use a key based on the identity when local state should start fresh for each entity:

~~~jsx
<Chat key={recipient.id} recipient={recipient} />
~~~

This is appropriate when switching recipients should clear a message draft so it cannot accidentally be sent to the wrong person.

### Keep state for each identity

If each entity's local state should survive while the user switches away and back, render each instance and hide inactive ones, or store per-entity values in a parent or another state store.

A parent can own drafts by recipient ID:

~~~jsx
const [drafts, setDrafts] = useState({});

function saveDraft(recipientId, text) {
  setDrafts((current) => ({
    ...current,
    [recipientId]: text,
  }));
}
~~~

Lifting state is usually a better fit when there are many entities or hidden subtrees would be large.

### Preserve state while props change

If the new data is still the same conceptual component, keep the same type, position, and key. A changing prop alone should not be used as an excuse to reset state.

## Common mistakes

### Assuming state is attached to a JSX tag

JSX is an instruction for producing a tree. React associates state with the component instance it identifies in that tree, not with a line of JSX source.

### Copying changing props into local state without reset rules

A state initializer runs when the component instance is created; it does not run again just because a prop changes. If identity changed and local state should reset, use a deliberate boundary such as a key. If the state should always reflect the prop, derive the value during render or make the parent its owner instead of keeping competing copies.

### Using an array index for changing item identity

After a reorder or deletion, index-based identity can associate an existing stateful component with another data item. Use a stable item ID.

### Generating keys while rendering

Random or changing keys force React to recreate children, losing state and DOM continuity. Create IDs when data items are created and keep them with the data.

### Expecting hidden CSS to reset React state

An element hidden with CSS still exists in the rendered tree. Conditional removal unmounts it; visual hiding does not.

### Defining components inside components

A new nested function on every parent render is a new component type. Move the component definition to module scope.

### Using a key as an ordinary prop

React consumes key for reconciliation. Pass a separate prop when the child needs the same ID.

## Accessibility and user experience

Resetting state can discard text a person has typed. That may be correct when the entity changes, but it should be intentional. If the reset would lose meaningful work, consider saving drafts per identity, asking before discarding, or keeping the relevant state in a longer-lived owner.

Do not rely on a reset alone to communicate a context change. Give the page a clear heading or label so keyboard and screen-reader users can tell which record or recipient is active.

## Practice

1. A Counter receives a new title prop but stays at the same tree position. Does its local count necessarily reset?
2. What happens to local state when a condition removes a component and later renders it again?
3. Why can switching from Editor to Preview reset the editor's descendant state?
4. How does a key change component identity?
5. Why does a keyed ProfileEditor also receive userId as a separate prop when it needs that ID?
6. What bug can an index key cause after deleting the first row of an editable list?
7. Why should a key not be generated with Math.random during rendering?
8. Are keys required to be globally unique across the application?
9. Does hiding a still-rendered component with CSS unmount it?
10. Why can a component declared inside a parent unexpectedly lose its state?
11. When should state be lifted instead of resetting the subtree with a key?
12. What user-experience concern should be considered before changing a key on a form?

## Recap

- React preserves state while it matches the same component identity at the same position in the rendered tree.
- A prop change alone does not necessarily reset local state.
- Removing a component, changing its type or tree position, or changing its key can create a fresh instance.
- Stable keys identify list children and can intentionally reset a subtree when identity changes.
- Keys are sibling-local, are not passed as props, and must not change on every render.
- Define component functions at module scope.
- Choose deliberately whether state should reset, persist per identity, or be owned by a parent.
