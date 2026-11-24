# 040. Updating object state immutably - Q&A

## Object state and mutation

### 1. Can React state contain an object?

Yes. A state variable can hold any JavaScript value, including an object.

### 2. What does it mean to mutate an object?

Mutation means changing the existing object's contents, such as assigning a new value to one of its properties.

### 3. Why should I avoid mutating an object in state?

It changes the object from a previous render without creating a new state value. React may not render the update when the same reference is passed back, and previous snapshots or other references can unexpectedly observe the mutation.

### 4. Does declaring an object with const make its properties immutable?

No. const prevents reassigning the variable, but the object's properties can still be changed. Avoid changing an object held in state even if its variable uses const.

### 5. Does React freeze objects stored in state?

No. JavaScript objects are technically mutable. Immutability here is a rule to follow: treat state objects as read-only and return replacement objects when changing state.

### 6. Is mutation okay if I never call a state setter?

No. Changing a state object directly still mutates the previous state. React also has not been asked to render the changed value.

### 7. What if I mutate an object and then pass the same object to its setter?

The old state has already been changed, and the setter receives the same reference. React may skip the update because the next value is identical to the current value according to Object.is.

## Creating a replacement object

### 8. How should I update one property of a flat object?

Return a new object that copies the old properties and overrides the changed property:

~~~jsx
setPerson({
  ...person,
  firstName: 'Mira',
});
~~~

### 9. Does object spread change the original object?

No. Object spread creates a new outer object and copies the source object's enumerable own properties into it.

### 10. What is the meaning of the ...person part?

It copies person’s top-level properties into the new object. The new object has a different identity from person.

### 11. What happens if I update a property before spreading?

Later properties override earlier ones. If the spread comes after the new value, the old property copied from person can replace the update.

### 12. Does a state setter merge an object with the old state automatically?

No. The setter replaces the state value with the value you pass. If the new object omits existing fields, those fields are absent in the next state.

### 13. Why can this update cause fields to disappear?

~~~jsx
setPerson({
  firstName: 'Mira',
});
~~~

It replaces the complete person object with one containing only firstName. To preserve other fields, spread the old object first.

### 14. Can I use an updater function to create the replacement object?

Yes. For an update based on pending state, return a new object from the updater:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  firstName: 'Mira',
}));
~~~

### 15. What does the updater receive?

It receives the pending state value for person. Use that value as the source for the new object.

## Shallow copying and nested properties

### 16. What does it mean that object spread is shallow?

It copies only the outer object's properties. If a property points to a nested object, the copied property still points to that same nested object.

### 17. Does spreading an object deep-copy all its nested objects?

No. The outer object is new, but its nested object values remain shared references unless you explicitly copy them.

### 18. How do I update artwork.city without mutating the old artwork object?

Create a new artwork object and a new outer person object:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  artwork: {
    ...currentPerson.artwork,
    city: 'Oslo',
  },
}));
~~~

### 19. Why must each object on the path be copied?

The outer object points to the nested object. If you copy only the outer object, both old and new outer objects still point to the same nested object. Copying each level gives the new state a new path without changing the old one.

### 20. Do I need to copy every object in the whole state tree?

No. Copy the objects along the path to the property being updated. Unchanged branches can retain their existing references.

### 21. If I spread person but then mutate nextPerson.artwork.city, is that safe?

No. The spread made only a shallow copy, so nextPerson.artwork is still the same object as person.artwork. Copy artwork too before changing city.

### 22. What is an object reference alias?

It is another variable or property that points to the same object. Mutating the object through one alias changes what every alias sees.

### 23. Why is it useful for old state objects to remain unchanged?

Previous renders, saved snapshots, and other references can continue to represent their original values. This also makes comparisons, debugging, and undo history more reliable.

## Form updates

### 24. How can one handler update several flat fields?

Give each input a name matching the state property and use a computed property name:

~~~jsx
function handleChange(event) {
  const { name, value } = event.target;

  setPerson((currentPerson) => ({
    ...currentPerson,
    [name]: value,
  }));
}
~~~

### 25. What do the square brackets around name do?

They tell the object literal to use the value of name as the property key. If name is "email", the returned object updates the email field.

### 26. Is the generic flat-field handler suitable for nested state?

No. It sets a top-level property named by the input. For nested state, write an update that copies each object along the nested path.

### 27. Why capture event.target.value before an updater?

It makes the event value explicit and keeps the updater as a pure calculation based on that value and the pending state. For example:

~~~jsx
const city = event.target.value;

setPerson((currentPerson) => ({
  ...currentPerson,
  artwork: {
    ...currentPerson.artwork,
    city,
  },
}));
~~~

### 28. How can I add a new property?

Spread the current object and add the new property:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  nickname: 'Mimi',
}));
~~~

### 29. How can I remove a property?

Destructure it out and return the remaining properties:

~~~jsx
setPerson((currentPerson) => {
  const { nickname, ...personWithoutNickname } = currentPerson;
  return personWithoutNickname;
});
~~~

This creates a new outer object without nickname.

## Safe local construction and related topics

### 30. Is it ever safe to mutate an object?

Yes. Mutating a new local object that is not in state and is not shared with existing data is safe. A common alternative is to construct it directly with an object literal.

### 31. Is assigning to a brand-new object before setting it state mutation?

Technically yes, but it is a local mutation of a fresh object. It does not change previous state, so it is safe.

### 32. Is object spread a deep cloning tool?

No. It makes a shallow copy. Use explicit copies along the changed path rather than assuming one spread recursively copies everything.

### 33. Should I use JSON.stringify and JSON.parse to copy every state object?

Usually no. That approach can change or discard values such as undefined, functions, and dates, and it copies more data than necessary. Shallow copies along the changed path are usually enough.

### 34. What should I do if nested object updates are becoming hard to follow?

Consider whether the state shape can be simplified or flattened. An immutable update library such as Immer is another option, but it adds a dependency and is not required for object state.

### 35. Is an updater function allowed to mutate its argument?

No. The updater should return a new object and leave the previous object unchanged.

### 36. Can I perform a request or analytics call inside an updater?

No. Updaters should be pure. React can call them more than once in development Strict Mode, so side effects belong in the appropriate event handler or Effect.

### 37. Does an updater for an object make all other captured values current?

No. The updater receives pending state for that object variable only. Any other props or local values it closes over still come from the render that created it.

### 38. Does this chapter cover arrays too?

No. Arrays are also treated as read-only in state, but their update patterns use array operations. The next chapter covers array state.

## Check your understanding

### 39. What properties remain after this update?

~~~jsx
setPerson({
  ...person,
  firstName: 'Mira',
});
~~~

All existing top-level properties remain, and firstName is replaced with "Mira".

### 40. What is wrong with this nested update?

~~~jsx
setPerson({
  ...person,
  artwork: {
    city: 'Oslo',
  },
});
~~~

It creates a new outer person object but replaces artwork with an object containing only city. Other artwork properties such as title are dropped. Spread the previous artwork object too.

### 41. Why is this not an immutable update?

~~~jsx
setPerson((currentPerson) => {
  currentPerson.score += 1;
  return currentPerson;
});
~~~

It mutates the previous object and returns the same reference. Return a new object with the incremented score instead.

### 42. What does this update produce, and does it mutate the old person?

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  artwork: {
    ...currentPerson.artwork,
    city: 'Oslo',
  },
}));
~~~

It produces a new person object and a new artwork object with city set to "Oslo". The old person and its artwork remain unchanged.

## Summary

- Treat state objects as read-only.
- A setter replaces the state object; it does not merge automatically.
- Object spread makes a shallow copy.
- Copy every object along the path to a nested property.
- Use functional updaters when the new object should be based on pending state.
- Keep previous objects unchanged so old render snapshots stay reliable.
