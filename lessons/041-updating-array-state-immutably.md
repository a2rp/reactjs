# 041. Updating array state immutably

## Goal

Arrays are mutable JavaScript objects, but an array stored in React state should be treated as read-only. To add, remove, reorder, or replace an item, create a new array and pass it to the setter.

This chapter covers common non-mutating array update patterns, updating objects stored in arrays, stable item identity, and the difference between copying an array and copying its items.

## Learning outcomes

After this chapter, you should be able to:

- Explain why mutating an array in state is unsafe.
- Add items with spread syntax or concat.
- Remove items with filter.
- Replace or update items with map.
- Insert an item at a specific position with slice.
- Sort or reverse a copy without changing state.
- Update an object item without mutating that object.
- Use stable IDs for list items and targeted updates.
- Choose an updater when the next array depends on pending state.

## Treat state arrays as read-only

Do not change an array already stored in state with mutating methods or index assignment:

~~~jsx
items.push(newItem);
items[0] = replacement;
items.splice(1, 1);
items.sort(compareItems);
items.reverse();
~~~

These operations change the existing array. React state should instead be replaced with a new array:

~~~jsx
setItems([...items, newItem]);
~~~

The setter replaces the array state value. It does not merge the contents of the new array with the old one automatically.

## Common operations

| Goal | Avoid mutating the state array with | Create a replacement using |
| --- | --- | --- |
| Add at the end | push | spread syntax or concat |
| Add at the beginning | unshift | spread syntax |
| Remove item or items | pop, shift, splice | filter |
| Replace an item | index assignment, splice | map |
| Insert at a position | splice | slice and spread |
| Sort or reverse | sort, reverse | copy first, then sort or reverse |

JavaScript method names can be easy to confuse: slice returns a copy or portion, while splice changes the original array.

## Add an item

Spread the existing items into a new array and add the new item:

~~~jsx
function handleAdd(item) {
  setItems((currentItems) => [...currentItems, item]);
}
~~~

Place the new item before the spread to add it at the beginning:

~~~jsx
setItems((currentItems) => [item, ...currentItems]);
~~~

concat also returns a new array:

~~~jsx
setItems((currentItems) => currentItems.concat(item));
~~~

Do not call push on the state array. If an item needs a generated ID, create that ID in the event handler before the updater, then return the new array. Keep side effects and ID generation out of updater functions.

## Remove an item

Use filter to create an array containing the items that should remain:

~~~jsx
function handleRemove(itemId) {
  setItems((currentItems) =>
    currentItems.filter((item) => item.id !== itemId)
  );
}
~~~

For unique IDs, the item whose ID matches itemId is removed. filter does not modify the original array.

Avoid using splice directly on the state array. If you need to remove an item by index, filter is often clearer. slice can also create the portions before and after an index, but take care with index bounds.

## Replace or update an item

Use map to create a new array. Return a new object for the item that changes, and return the original item references for items that do not change:

~~~jsx
function handleRename(itemId, nextName) {
  setItems((currentItems) =>
    currentItems.map((item) =>
      item.id === itemId
        ? { ...item, name: nextName }
        : item
    )
  );
}
~~~

map returns a new array. The matching item is also a new object because its name changes. Unchanged items can keep their existing object references.

This matters because copying only the array does not copy its object items. The next section explains that distinction.

## Insert at a specific position

slice can copy portions of an array without changing the original. Combine those portions with the new item:

~~~jsx
function insertAt(items, index, newItem) {
  return [
    ...items.slice(0, index),
    newItem,
    ...items.slice(index),
  ];
}
~~~

This inserts newItem before the item currently at index. If the index is outside the range you support, decide whether to clamp it, reject it, or treat it as an append. Be explicit about that behavior in application code.

For simple lists, adding at the beginning or end is usually easier to read than a general insert helper.

## Sort or reverse without changing state

sort and reverse mutate the array on which they are called. Copy the array first:

~~~jsx
const sortedItems = [...items].sort((first, second) =>
  first.name.localeCompare(second.name)
);
~~~

This sorts the new array and leaves items unchanged.

To reverse order:

~~~jsx
const reversedItems = [...items].reverse();
~~~

If the reordered array itself is the next state, pass the copy to the setter:

~~~jsx
setItems((currentItems) =>
  [...currentItems].sort((first, second) =>
    first.name.localeCompare(second.name)
  )
);
~~~

The comparison function should produce a consistent ordering for the values you compare. Do not call sort or reverse directly on the state array.

## Copying an array does not copy its items

Array spread is shallow. It creates a new array, but each item is still the same value or object reference:

~~~js
const nextItems = [...items];

console.log(nextItems !== items); // true
console.log(nextItems[0] === items[0]); // true for an object item
~~~

This still changes the old state:

~~~jsx
const nextItems = [...items];
nextItems[0].name = 'Updated'; // Mutates the shared item object.
setItems(nextItems);
~~~

The array is new, but nextItems[0] and items[0] point to the same object. Create a new object for the item that changes, usually with map and object spread.

## Update an object stored in an array

Use map to create a new array and return a new object for the matching item:

~~~jsx
function handleToggle(itemId, nextDone) {
  setItems((currentItems) =>
    currentItems.map((item) =>
      item.id === itemId
        ? { ...item, done: nextDone }
        : item
    )
  );
}
~~~

This creates:

- A new array for the state.
- A new object for the item whose done property changes.
- The existing object references for items that did not change.

If the item itself contains a nested object, copy that nested object too:

~~~jsx
function handleChangeCity(itemId, city) {
  setItems((currentItems) =>
    currentItems.map((item) =>
      item.id === itemId
        ? {
            ...item,
            location: {
              ...item.location,
              city,
            },
          }
        : item
    )
  );
}
~~~

Copy every object along the path to the changed property, just as with nested object state.

## Keep different lists independent

Two arrays can contain references to the same item object. If one list mutates that shared object, another list may appear to change too.

Immutable updates avoid changing that shared object. When mapping a list, create a replacement object for the item being updated and return other items unchanged. This preserves the old values and avoids accidentally changing another state's data through a shared reference.

## Stable IDs and React keys

When rendering a list, use a stable ID from each item as its key:

~~~jsx
<ul>
  {items.map((item) => (
    <li key={item.id}>{item.name}</li>
  ))}
</ul>
~~~

The key helps React match items across renders. Keep IDs stable when an item changes. Avoid using the array index as a key for lists that can be inserted, removed, or reordered, because an item's position can change while its identity stays the same.

Generate an ID when the item is created, not each time the list renders. For example, create the ID in the click handler, then add the item with an immutable array update.

## Choose direct values or functional updaters

For a single update from the current rendered list, this can be fine:

~~~jsx
setItems([...items, newItem]);
~~~

Use a functional updater when the new array should be based on the pending array or when multiple updates might be queued before the next render:

~~~jsx
setItems((currentItems) => [...currentItems, newItem]);
~~~

The updater receives the pending array for that state variable. Return a new array and do not mutate currentItems.

## Local mutation of a fresh array

Mutation is unsafe when it changes the array already stored in state. It is fine to mutate a new local copy:

~~~jsx
const nextItems = [...items];
nextItems.sort(compareItems);
setItems(nextItems);
~~~

nextItems is a fresh array. Sorting it does not change the state array. The same distinction applies to local arrays you create from scratch: do not mutate an array that belongs to state, props, or another shared reference.

## Common mistakes

### Calling push on the state array

~~~jsx
items.push(newItem);
setItems(items);
~~~

This mutates the current array and passes the same reference back. Create a new array with spread or concat.

### Calling sort directly on state

~~~jsx
items.sort(compareItems);
setItems(items);
~~~

sort changes items in place. Sort a copy and pass the copy to the setter.

### Copying the array but mutating an item

~~~jsx
const nextItems = [...items];
nextItems[index].done = true;
setItems(nextItems);
~~~

The item object is still shared with the previous array. Use map and copy the changed item object.

### Using splice instead of slice

splice changes the original array. slice returns a new array portion. For insertion, combine slice results with spread or use a purpose-built immutable helper.

### Using unstable keys

Generating a new key during every render tells React that each item is a different item each time. Use IDs that are created once and remain stable.

## Practice

Given this state:

~~~jsx
const [tasks, setTasks] = useState([
  { id: 'a', title: 'Read', done: false },
  { id: 'b', title: 'Practice', done: false },
]);
~~~

Write an update that adds a new task at the end.

Then write an update that removes the task with ID 'a'.

Then write an update that marks task 'b' as done without mutating either the array or its task object.

Finally, explain why this is still a mutation even though nextTasks is a new array:

~~~jsx
const nextTasks = [...tasks];
nextTasks[0].done = true;
~~~

## Recap

- Treat arrays in React state as read-only.
- Create a new array for each state update.
- Use spread or concat to add, filter to remove, and map to replace or update items.
- Copy an object item when changing it; array spread does not deep-copy items.
- Copy nested objects along the path to a changed property.
- Sort or reverse a copy instead of the state array.
- Use stable IDs for rendered list keys and item updates.
