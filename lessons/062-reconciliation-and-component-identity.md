# Reconciliation, Component Identity, and Keys

## React matches the next tree with the previous tree

When a component renders again, React compares the new UI description with the previous one. This process is commonly called **reconciliation**. React uses the structure of the tree, element types, positions, and keys to decide what to preserve, update, remove, or create.

This decision affects more than DOM reuse. React state belongs to a component’s identity in the render tree. When React recognizes the same component at the same identity, it can preserve its state. When that identity is removed or changed, React destroys that state and creates a new instance if needed.

A useful model is:

~~~text
component identity ≈ parent position + element/component type + key
~~~

The exact reconciliation algorithm is an implementation detail. Use stable component structure and keys to express the identity you want; do not depend on internal algorithm details.

## Learning goals

After this lesson, you should be able to:

- describe reconciliation as matching the next UI tree with the previous one;
- explain why state belongs to a component’s position and identity in the tree;
- predict when state is preserved and when a component is remounted;
- choose stable data keys for lists that can change order or membership;
- avoid index and random keys when items can move or be inserted or removed;
- use a changed key intentionally to reset a component subtree;
- explain why keys are local to siblings and are not passed as props;
- avoid defining component functions inside another component’s render.

## State is associated with a tree position

A function component’s state does not live inside the JavaScript function invocation. React stores it and associates it with a component in the render tree.

Two instances of the same component at different positions have separate state:

~~~jsx
function App() {
  return (
    <section>
      <Counter />
      <Counter />
    </section>
  );
}
~~~

Each **Counter** has its own state because there are two distinct positions in the tree.

If the same component remains at the same position, changing its props does not normally reset its state:

~~~jsx
function Scoreboard({ player }) {
  return <Counter player={player} />;
}
~~~

When **player** changes, React still sees a **Counter** of the same type at that position. Its local state is preserved unless the element’s identity changes or it is removed.

## Component type matters

If React sees a different component type at the same position, it removes the old component and creates the new one. State in the old subtree is discarded:

~~~jsx
function App({ showCounter }) {
  return showCounter ? <Counter /> : <Message />;
}
~~~

Switching between **Counter** and **Message** changes the type, so the old component’s local state does not carry over.

Component type is based on the component function itself. Define component functions at module scope instead of creating them inside another component:

~~~jsx
function Parent() {
  function TextField() {
    const [text, setText] = useState("");
    return <input value={text} onChange={(event) => setText(event.target.value)} />;
  }

  return <TextField />;
}
~~~

This is a mistake. Every **Parent** render creates a new **TextField** function, which React sees as a different component type. The input can be removed and recreated, losing its state, focus, or entered value.

Define it outside instead:

~~~jsx
function TextField() {
  const [text, setText] = useState("");
  return <input value={text} onChange={(event) => setText(event.target.value)} />;
}

function Parent() {
  return <TextField />;
}
~~~

## Keys identify siblings

When rendering a collection, a **key** tells React which item corresponds to which sibling across updates:

~~~jsx
const rows = people.map((person) => (
  <PersonRow key={person.id} person={person} />
));
~~~

If an item is inserted, deleted, or reordered, its key lets React match that item to the right component instance. This keeps each row’s state and DOM associated with the correct data item.

Keys must be unique among siblings in the same collection. They do not need to be globally unique; two unrelated arrays can use the same key values.

A key must remain stable for the lifetime of that data item. Usually the best key is an ID already present in the data:

~~~jsx
const tasks = [
  { id: "task-17", title: "Write notes" },
  { id: "task-18", title: "Review examples" },
];

return tasks.map((task) => <TaskRow key={task.id} task={task} />);
~~~

For locally created and saved items, assign an ID when creating the item, then store it with the data. Do not generate a new key while rendering.

## Why array indexes often fail as keys

An index describes an item’s current position, not its identity:

~~~jsx
people.map((person, index) => (
  <PersonRow key={index} person={person} />
));
~~~

If the first row is removed, the former second row now has index **0**. React may reuse the first component instance for a different person, carrying the old row state or DOM value to the wrong item. Reordering and filtering can cause the same mismatch.

An index can be acceptable for a truly static list that never reorders, inserts, or removes items and has no item-specific state or DOM state to preserve. Stable data IDs are the safer default.

Never use **Math.random()**, the current time, or another changing value as a key during rendering. A new key makes React treat each item as new on every render, remounting the component and losing local state, focus, and uncontrolled input values.

## key is special and not a regular prop

React consumes **key** to match elements. The component cannot read it from props:

~~~jsx
function PersonRow({ key, person }) {
  // key is not provided here.
  return <p>{person.name}</p>;
}
~~~

Pass the same value separately if the component needs it as data:

~~~jsx
<PersonRow key={person.id} personId={person.id} person={person} />
~~~

The **key** helps React identify an element; **personId** is an ordinary prop the component can use.

## Keyed fragments

The short Fragment syntax **<>...</>** cannot receive a key. When each mapped item returns multiple adjacent elements, use an explicit **Fragment**:

~~~jsx
import { Fragment } from "react";

const entries = posts.map((post) => (
  <Fragment key={post.id}>
    <h2>{post.title}</h2>
    <p>{post.summary}</p>
  </Fragment>
));
~~~

Fragments group children without adding a wrapper DOM element.

## Changing a key intentionally resets state

A key is not only for lists. You can use it to tell React that two components at the same position represent different identities:

~~~jsx
<ProfileEditor key={profile.id} profile={profile} />
~~~

When **profile.id** changes, React unmounts the old editor and mounts a new one. Local form state, refs, and Effects in that subtree are reset or cleaned up with the old instance.

This is often clearer than rendering one profile’s data, then using an Effect to clear every local field after the profile changes. Use it only when a full reset is intended.

A key change does not cache state for the old item. If switching away and back should restore separate drafts, keep the drafts in a parent keyed by profile ID, or keep the relevant component instances mounted. A removed component’s local state is destroyed.

## Keys are scoped to one parent

Keys distinguish siblings under the same parent. A key does not move a component across unrelated parts of the tree, and the same key under a different parent does not preserve the old component instance.

For example, changing a component’s parent wrapper can remove and recreate its subtree even when the child’s key stays the same. Structure and component type still matter.

## Keys do not create DOM IDs

A key is an internal reconciliation hint. It is not rendered as an HTML attribute and is not available to the child component. Use a separate data prop for item identity and an accessibility ID for a label relationship. These solve different problems.

## Reconciliation checklist

When a row shows another item’s state or a form unexpectedly resets, check:

1. Is the component type stable between renders?
2. Is the component still at the same parent position?
3. Does each changing list item have a stable key from its data?
4. Is the key unique among its current siblings?
5. Is the key accidentally an array index or newly generated value?
6. Did a parent, wrapper, or conditional branch change the tree structure?
7. Was a component function defined inside another component?
8. Does the UI need a full reset, or should state be lifted so it survives removal?

## Summary

- Reconciliation compares the new UI tree with the previous tree and decides what to preserve, update, remove, or create.
- State is associated with component identity in the tree, including its type, parent position, and key.
- Same type at the same position usually preserves state; removing it or changing its identity resets its state.
- Stable keys from data identify list siblings when their order or membership changes.
- Index keys can attach state to the wrong item; random keys force repeated remounts.
- Keys are unique among siblings, are consumed by React, and are not passed as props or rendered as DOM IDs.
- Changing a key intentionally resets a subtree, but does not cache its state for later.
- Keep component function definitions at module scope so their types remain stable.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="061-rendering-and-committing.md">061 - Rendering, Committing, and Updating the UI</a></td><td><a href="../qnas/062-reconciliation-and-component-identity.md">Q&amp;A</a></td><td><a href="../examples/062-reconciliation-and-component-identity.md">Example</a></td><td><a href="063-concurrent-rendering-and-priorities.md">063 - Concurrent Rendering and Update Priorities</a></td></tr>
  </tbody>
</table>
