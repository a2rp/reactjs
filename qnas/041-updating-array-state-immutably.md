# 041. Updating array state immutably - Q&A

## Array state and replacement

### 1. Can an array be stored in React state?

Yes. An array is a JavaScript value and can be stored in state. Treat it as read-only and replace it with a new array when its contents need to change.

### 2. Why should I avoid mutating an array in state?

Mutation changes the existing array from a previous render. React may not detect a change if the same array reference is passed back, and other code sharing that array can observe unexpected changes.

### 3. Does calling the setter with a new array append it to the old array?

No. The setter replaces the whole array state value. To keep existing items and add another, create a new array that includes both.

### 4. What is wrong with this update?

~~~jsx
items.push(newItem);
setItems(items);
~~~

push mutates the current state array, and setItems receives the same reference. Create a new array instead, such as setItems((currentItems) => [...currentItems, newItem]).

### 5. Is an array spread a deep copy?

No. It creates a new outer array but retains references to object items inside it.

## Adding items

### 6. How can I add an item to the end?

~~~jsx
setItems((currentItems) => [...currentItems, newItem]);
~~~

The spread copies the existing item references into a new array, and newItem is added at the end.

### 7. How can I add an item to the beginning?

Put it before the spread:

~~~jsx
setItems((currentItems) => [newItem, ...currentItems]);
~~~

### 8. Can I use concat instead of spread?

Yes. concat returns a new array:

~~~jsx
setItems((currentItems) => currentItems.concat(newItem));
~~~

### 9. Is mutating a newly created array allowed?

Yes. Mutating a fresh local copy is safe if it is not shared with state, props, or other code. For example, sorting a copied array is safe.

### 10. When should I generate an item's ID?

Generate it when creating the item, such as in the event handler, then store it on the item. Do not generate a new key during each render.

## Removing items

### 11. What is a common way to remove an item by ID?

Use filter to return a new array without the matching item:

~~~jsx
setItems((currentItems) =>
  currentItems.filter((item) => item.id !== itemId)
);
~~~

### 12. Does filter mutate the original array?

No. filter returns a new array containing the items for which its callback returns true.

### 13. What happens if multiple items have the same ID?

The filter expression removes every item with that ID. IDs used to identify individual items should be unique if an update is meant to affect exactly one item.

### 14. Can I use pop, shift, or splice on a state array?

Do not call them directly on the state array because they mutate it. Use filter or construct a new array with slice and spread.

## Replacing and updating items

### 15. How can I replace an item at a matching ID?

Use map to return a new array and return the replacement for the matching item:

~~~jsx
setItems((currentItems) =>
  currentItems.map((item) =>
    item.id === itemId ? replacementItem : item
  )
);
~~~

### 16. How can I change one property of an object item?

Create a new object for the matching item:

~~~jsx
setItems((currentItems) =>
  currentItems.map((item) =>
    item.id === itemId
      ? { ...item, done: true }
      : item
  )
);
~~~

### 17. Does map mutate the original array?

No. map returns a new array. The callback determines which item references go into that array.

### 18. Should I create new objects for every item when changing one?

Usually no. Return a new object for the changed item and reuse references for unchanged items. This clearly communicates which item changed and avoids unnecessary copying.

### 19. What is wrong with copying the array and then changing one object item?

~~~jsx
const nextItems = [...items];
nextItems[0].done = true;
setItems(nextItems);
~~~

The new array's first element is still the same object as items[0]. Assigning to done mutates the object shared with the old state. Use map and object spread to create a new item object.

### 20. How do I update a nested property on an item?

Copy the array, the changed item, and each nested object along the path:

~~~jsx
setItems((currentItems) =>
  currentItems.map((item) =>
    item.id === itemId
      ? {
          ...item,
          location: {
            ...item.location,
            city: nextCity,
          },
        }
      : item
  )
);
~~~

### 21. Is an array an object?

Yes. Arrays are JavaScript objects with ordered elements and special array methods. The same rule applies: don't mutate an array already stored in state.

## Inserting, sorting, and reversing

### 22. What is the difference between slice and splice?

slice returns a copy or portion without changing the original array. splice changes the array it is called on.

### 23. How can I insert an item at an index without splice?

Combine two slices with the item:

~~~jsx
const nextItems = [
  ...items.slice(0, index),
  newItem,
  ...items.slice(index),
];
~~~

Decide what your application should do when index is outside its supported range.

### 24. Why shouldn't I call sort directly on the state array?

sort changes the array in place. Sort a copy instead:

~~~jsx
const sortedItems = [...items].sort(compareItems);
~~~

### 25. How do I reverse the order without mutating state?

Copy first, then reverse the copy:

~~~jsx
const reversedItems = [...items].reverse();
~~~

### 26. Can I sort a copy and store it as new state?

Yes:

~~~jsx
setItems((currentItems) =>
  [...currentItems].sort(compareItems)
);
~~~

The copy is a new array, so sorting it does not mutate the previous state array.

## State updaters and identity

### 27. When should I use a functional updater for an array?

Use one when the next array should be based on the pending array or when several updates to that state could be queued before a render:

~~~jsx
setItems((currentItems) => [...currentItems, newItem]);
~~~

### 28. Is a functional updater always required?

No. A direct expression such as setItems([...items, newItem]) can be fine for a single update based on the current render. Choose an updater when it makes the relationship to pending state clearer or is needed for correctness.

### 29. Can I call a setter with the same array reference after mutation?

Avoid doing this. React may skip the update because the reference is identical, and the old state was already mutated.

### 30. Does making a new array guarantee that every item is new?

No. Array spread and methods like filter or map make a new outer array. The values in it may still be the same object references as before.

### 31. Why should previous arrays and items remain unchanged?

Earlier renders and other references can continue to represent the data they were given. This makes rendering, debugging, and shared data safer.

## Rendering lists

### 32. Why does each rendered item need a key?

A key helps React match each rendered list item with the same item across renders when items are inserted, removed, or reordered.

### 33. What makes a good key?

A stable, unique identifier stored on the item, such as item.id. It should remain the same when that item is updated.

### 34. Why can an array index be a poor key?

If items are inserted, removed, or reordered, an item's index may change. React can then associate a rendered component's local state with the wrong item.

### 35. Why shouldn't I generate a random key during render?

It changes on every render, making React treat the item as new each time. Generate and store the ID when creating the item.

## Check your understanding

### 36. What does this add operation do?

~~~jsx
setItems((currentItems) => [newItem, ...currentItems]);
~~~

It creates a new array with newItem first, followed by the current items in their existing order.

### 37. What does this remove operation do?

~~~jsx
setItems((currentItems) =>
  currentItems.filter((item) => item.id !== 'b')
);
~~~

It returns a new array containing all items whose IDs are not 'b'. If IDs are unique, that removes the item with ID 'b'.

### 38. What does this update change?

~~~jsx
setItems((currentItems) =>
  currentItems.map((item) =>
    item.id === 'b' ? { ...item, done: true } : item
  )
);
~~~

It returns a new array, creates a new object for item 'b' with done set to true, and preserves the other item references.

### 39. Why is this not safe even though it uses a new array?

~~~jsx
const nextItems = [...items];
nextItems[0].name = 'New name';
~~~

The new array contains the same object reference at index 0. Changing its name mutates the item that belongs to the old state too.

### 40. What should I do if I need to change an object item and a nested property?

Use map to create a new array, object spread for the changed item, and additional spreads for every nested object along the changed path.

### 41. Can the unchanged items in a new array keep their old object references?

Yes. If an item has not changed, returning the same item reference is appropriate. Create a new object for an item whose properties changed.

### 42. What should I ask before choosing an array update method?

Ask what operation is intended: add, remove, replace, insert, sort, or reverse. Then choose a method that returns a new array and, if an object item changes, return a new item object too.

## Summary

- Treat arrays in state as read-only and replace them with new arrays.
- Use spread or concat to add, filter to remove, and map to replace or update items.
- Array copies are shallow; object items remain shared until copied explicitly.
- Copy each nested object along the path to a changed property.
- Sort or reverse a copy, not the state array.
- Use stable, unique IDs for item updates and rendered list keys.
