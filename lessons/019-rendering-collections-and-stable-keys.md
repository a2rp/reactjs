# 019. Rendering collections and choosing stable keys

## Learning goals

By the end of this lesson, you should be able to:

- Render repeated UI from an array with `map()`.
- Filter data before rendering it with `filter()`.
- Give each mapped sibling a stable, unique `key`.
- Choose keys from the data instead of generating them during rendering.
- Explain why array indexes can be unsafe keys for changing lists.
- Use a keyed Fragment when one data item produces several sibling elements.
- Pass an item ID as a regular prop when the component needs it, because `key` is reserved for React.

## Keep repeated data in an array

When an interface displays several similar items, store the changing values in data and describe the repeated structure once. For example, each lesson can have its own ID and title:

~~~jsx
const lessons = [
  { id: 'jsx', title: 'Components and JSX' },
  { id: 'props', title: 'Passing data with props' },
  { id: 'lists', title: 'Rendering collections' },
];
~~~

An array makes it possible to add, remove, reorder, or filter lessons without copying a separate block of JSX for every item.

## Use `map()` to turn data into JSX

JavaScript's `map()` calls a function for each array item and returns a new array containing the results. In React, return one JSX element for each item:

~~~jsx
function LessonList() {
  const items = lessons.map((lesson) => (
    <li key={lesson.id}>{lesson.title}</li>
  ));

  return <ul>{items}</ul>;
}
~~~

The callback can use an implicit return when its body is one expression. If you use a block body with braces, write `return` explicitly:

~~~jsx
const items = lessons.map((lesson) => {
  return <li key={lesson.id}>{lesson.title}</li>;
});
~~~

Forgetting that `return` produces `undefined` for each callback, so the expected JSX does not appear. Use the form that keeps the callback easiest to read.

## Filter data before mapping it

`filter()` creates a new array with only the items that pass a boolean test. You can filter first and then map the remaining data to JSX:

~~~jsx
function LessonList({ lessons, topic }) {
  const matchingLessons = lessons.filter(
    (lesson) => lesson.topic === topic,
  );

  return (
    <ul>
      {matchingLessons.map((lesson) => (
        <li key={lesson.id}>{lesson.title}</li>
      ))}
    </ul>
  );
}
~~~

`filter()` and `map()` do not mutate the original array. They return new arrays, which makes them useful for deriving a view from source data. If no items match, the resulting list is empty; the component can show a separate empty-state message when that is useful.

## Give mapped items stable keys

Every JSX element directly returned from a `map()` call needs a `key`. A key is a string or number that identifies that item among its siblings:

~~~jsx
{lessons.map((lesson) => (
  <li key={lesson.id}>{lesson.title}</li>
))}
~~~

Keys help React match an item in one render with the same item in a later render. This matters when list items are inserted, removed, or reordered. With an appropriate key, React can preserve the relationship between an item and its UI while updating the list.

A good key is:

- **Stable:** the same data item keeps the same key between renders.
- **Unique among siblings:** two items in the same rendered collection do not share a key.
- **Derived from the item:** use an ID or another value that identifies that record.

Keys only need to be unique among siblings in one collection. Separate lists can reuse the same key values because they are separate groups.

## Choose an ID from the data

For data from a server or database, use the record's ID:

~~~jsx
<li key={lesson.id}>{lesson.title}</li>
~~~

For locally created records, create an ID when the record is created and store it with that record. Do not create a new key while rendering:

~~~jsx
// Avoid: a new value is created each time this list renders.
<li key={Math.random()}>{lesson.title}</li>
~~~

A random key changes on every render. React then treats each item as a different item, which can recreate its UI and discard local input or component state. Create an ID once when adding the item, then reuse the stored ID for every render.

## Why an array index is usually a poor key

An array index describes a position, not the identity of a record:

~~~jsx
{lessons.map((lesson, index) => (
  <li key={index}>{lesson.title}</li>
))}
~~~

If an item is inserted, removed, or moved, the index at a position can now refer to a different item. React may associate existing UI state with the wrong record. Prefer a stable ID from the data when a list can change.

An index can be acceptable for a list that is truly static: its items are never reordered, inserted, or removed, and have no stateful child UI that could be confused by a positional match. In ordinary data-driven lists, a stable record ID is a safer default.

## `key` is not an ordinary prop

React consumes `key` to identify the element in its sibling list. The child component cannot read it as `props.key`:

~~~jsx
function LessonRow({ lessonId, title }) {
  return <li data-lesson-id={lessonId}>{title}</li>;
}

function LessonList({ lessons }) {
  return (
    <ul>
      {lessons.map((lesson) => (
        <LessonRow
          key={lesson.id}
          lessonId={lesson.id}
          title={lesson.title}
        />
      ))}
    </ul>
  );
}
~~~

Here, `key` is for React's list matching, while `lessonId` is a normal prop that the component can use. Pass the ID separately if the child needs it.

## Key the outer element returned by `map()`

When a component is the mapped result, put the key on that component:

~~~jsx
{lessons.map((lesson) => (
  <LessonRow key={lesson.id} lesson={lesson} />
))}
~~~

A key on an element inside `LessonRow` does not identify `LessonRow` in its parent's list. The keyed element must be the one returned directly for each array item.

## Use a keyed Fragment for multiple siblings

Sometimes one data item needs to produce multiple sibling elements without an extra wrapper in the document. The short Fragment syntax `<>...</>` cannot receive a key. Import `Fragment` and put the key on it:

~~~jsx
import { Fragment } from 'react';

function ContactList({ contacts }) {
  return (
    <dl>
      {contacts.map((contact) => (
        <Fragment key={contact.id}>
          <dt>{contact.name}</dt>
          <dd>{contact.role}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
~~~

The Fragment groups the sibling elements for React but does not add a wrapper element to the rendered document.

## Empty collections and empty states

Mapping an empty array produces an empty array of JSX, so the list has no item elements. A user may still need an explanation. Check the collection and render a clear message:

~~~jsx
function LessonList({ lessons }) {
  if (lessons.length === 0) {
    return <p>No lessons are available yet.</p>;
  }

  return (
    <ul>
      {lessons.map((lesson) => (
        <li key={lesson.id}>{lesson.title}</li>
      ))}
    </ul>
  );
}
~~~

An empty list and a missing list are different cases. If a prop may be `null` or `undefined`, handle that possibility separately or give the prop a safe default before reading `.length`.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Omitting `key` from a mapped JSX element | React cannot reliably match siblings as a list changes. | Add a stable key to the element returned for each item. |
| Giving every item the same key | Keys are ambiguous among siblings. | Use a distinct identifier from each record. |
| Using `Math.random()` as a key while rendering | The key changes on every render and React recreates the item UI. | Store an ID on the record when it is created. |
| Using the array index for a list that can change order | A position may refer to a different record after insertion or sorting. | Use a stable data ID. |
| Putting the key inside the child component | It does not identify that component in the parent's mapped list. | Put `key` on the JSX element returned from `map()`. |
| Expecting `props.key` to be available | React reserves `key` and does not pass it as a normal prop. | Pass the value separately as `id`, `lessonId`, or another prop. |
| Returning multiple sibling elements without grouping them | A `map()` callback must return one JSX value for that item. | Wrap the elements in a keyed Fragment or a meaningful container. |
| Forgetting `return` in a block-bodied map callback | Each callback returns `undefined`. | Add `return` or use an implicit-return expression body. |
| Rendering an empty list without explaining it | The blank area may look like a loading or rendering failure. | Show a helpful empty state when it improves the interface. |

## Key takeaways

- Store repeated content as data and use `map()` to turn each record into JSX.
- Use `filter()` to derive a subset before mapping it.
- Give each mapped sibling a stable key that identifies the record.
- Prefer data IDs; avoid random keys and usually avoid indexes for changing lists.
- Put the key on the outer element returned by the map callback.
- `key` belongs to React; pass a separate prop if the component needs the ID.
- Use an explicitly keyed `Fragment` when one record renders multiple siblings.
- Handle empty collections deliberately when users need an explanation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [018 - Conditional rendering patterns](018-conditional-rendering-patterns.md) | [Open Q&A](../qnas/019-rendering-collections-and-stable-keys.md) | [Open Example](../examples/019-rendering-collections-and-stable-keys.md) | [020 - Component boundaries and responsibility](020-component-boundaries-and-responsibility.md) |