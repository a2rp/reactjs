# 019. Rendering collections and choosing stable keys - Q&A

## Rendering collections

### 1. How do I render several similar elements from an array?

Use `map()` to return one JSX element for each array item, then include the returned array in your component's JSX:

~~~jsx
const rows = lessons.map((lesson) => (
  <li key={lesson.id}>{lesson.title}</li>
));

return <ul>{rows}</ul>;
~~~

Each row uses the data for its corresponding item.

### 2. What does `map()` return?

`map()` returns a new array containing the result of its callback for every item in the original array. When the callback returns JSX, the result is an array of JSX elements that React can render.

### 3. What is the difference between an implicit return and a block-bodied callback?

An expression-bodied arrow function returns its expression automatically:

~~~jsx
items.map((item) => <li key={item.id}>{item.name}</li>);
~~~

A block-bodied arrow function uses braces and needs an explicit `return`:

~~~jsx
items.map((item) => {
  return <li key={item.id}>{item.name}</li>;
});
~~~

Without that `return`, the callback returns `undefined`.

### 4. How do I render only some items?

Use `filter()` to create a subset, then map that subset:

~~~jsx
const published = articles.filter((article) => article.isPublished);

const rows = published.map((article) => (
  <li key={article.id}>{article.title}</li>
));
~~~

`filter()` keeps an item when its callback returns a truthy result. It does not mutate the original array.

### 5. Does filtering change which keys the items need?

No. Keep using the same stable key from each record. Filtering changes which siblings are shown, but it does not change the identity of the remaining records.

## Keys and identity

### 6. What is a React `key`?

A `key` is a string or number that identifies an element among its siblings. React uses it to match rendered list items across updates, especially when items are inserted, removed, or reordered.

### 7. Why does a mapped JSX element need a key?

A key helps React determine which rendered element corresponds to which data item after the collection changes. Without a key, React may rely on positions and cannot reliably match changing items. React also warns when a mapped list lacks keys.

### 8. Where should the key go?

Put it on the outer JSX element returned for each item by `map()`:

~~~jsx
{lessons.map((lesson) => (
  <LessonRow key={lesson.id} lesson={lesson} />
))}
~~~

A key on an element inside `LessonRow` does not identify `LessonRow` in its parent's list.

### 9. What makes a key stable?

A stable key stays associated with the same data record between renders. A saved database ID is a common choice. For locally created records, create an ID when the record is created and store it on that record.

### 10. How unique must keys be?

Keys must be unique among the siblings in one rendered collection. They do not need to be globally unique across the application. Two different lists can reuse the same key values because they have different sibling sets.

### 11. Can two items with the same name use that name as their key?

Only if the name is guaranteed to be unique and does not change for those sibling records. Names are often duplicated or editable, so a dedicated stable ID is usually safer.

### 12. Can I use an array index as a key?

It can be acceptable for a truly static list that never changes order, gains items, or loses items, especially when children do not hold item-specific state. For a list that can change, an index refers to a position rather than a record, so it can cause React to associate UI with the wrong item. Prefer a stable ID.

### 13. Why is `key={Math.random()}` a bad idea?

It generates a different key on every render. React then sees each item as new, recreates its UI, and may lose local input or component state. Create and store the ID when adding the data instead.

### 14. Is a key passed to the child as a regular prop?

No. `key` is reserved for React and is not available as `props.key`. If the component needs the same ID, pass it under another prop name:

~~~jsx
<LessonRow key={lesson.id} lessonId={lesson.id} lesson={lesson} />
~~~

### 15. Can the same key appear in separate lists?

Yes. Keys only need to be unique among siblings. For example, each of two separate lists can contain an item with `key="intro"` without a conflict.

### 16. Can a key be the number zero?

Yes. A key can be a string or a number, including `0`. What matters is that the value is stable and unique among that item's siblings.

## Multiple elements and empty lists

### 17. What if one item needs to render two sibling elements?

Group them in one parent element, or use a keyed `Fragment` when you do not want an extra DOM wrapper:

~~~jsx
import { Fragment } from 'react';

{contacts.map((contact) => (
  <Fragment key={contact.id}>
    <dt>{contact.name}</dt>
    <dd>{contact.role}</dd>
  </Fragment>
))}
~~~

The short syntax `<>...</>` cannot take a key.

### 18. What happens when `map()` receives an empty array?

It returns an empty array, so no list items render. If that blank result could confuse users, check the array and show an empty-state message.

### 19. Does mapping create a new data array?

Yes. `map()` returns a new array with the callback's results. It leaves the source array unchanged. `filter()` also returns a new array, so a `filter().map()` chain can derive a display list without modifying the original collection.

### 20. Do keys appear in the HTML or DOM?

No. React uses keys for matching elements in its render tree; a key is not rendered as an HTML attribute. If the child needs a record ID for its own behavior or markup, pass that ID separately as a regular prop.

### 21. What if the array is `null` or `undefined`?

Those values do not have array methods such as `map()`. Make sure the value is an array before mapping, give an optional prop a safe default such as `items = []`, or render an appropriate loading or missing-data state.

### 22. Does a key replace the `id` attribute?

No. A `key` is metadata for React's reconciliation and is not rendered. An HTML `id` is a DOM attribute used for document relationships, labels, anchors, or styling. Use each for its own purpose.

### 23. Can I put a key on the parent list instead of each item?

No. The key belongs on each sibling produced by the map. Giving the `<ul>` a key does not identify the individual `<li>` elements inside it.

### 24. What is the safest default for keys?

Use a stable ID that is part of each data record. Keep it unchanged while the record represents the same item, make it unique among siblings, and pass it separately as a prop if the component needs to read it.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="018-conditional-rendering-patterns.md">018 - Conditional rendering patterns - Q&amp;A</a></td><td><a href="../lessons/019-rendering-collections-and-stable-keys.md">Lesson</a></td><td><a href="../examples/019-rendering-collections-and-stable-keys.md">Example</a></td><td><a href="020-component-boundaries-and-responsibility.md">020 - Component boundaries and responsibility - Q&amp;A</a></td></tr>
  </tbody>
</table>
