# 040. Updating object state immutably

## Goal

Objects are valid React state, but treat each object from a render as read-only. To update one property, create a new object with the changed value and pass that object to the setter. For nested data, create a new object at every level from the changed property up to the state object.

This chapter covers object identity, object spread, controlled forms, nested object updates, and common mutation mistakes. The next chapter focuses on arrays in state.

## Learning outcomes

After this chapter, you should be able to:

- Explain why mutating an object in state is unsafe.
- Create an updated object with object spread.
- Preserve unchanged properties when updating one field.
- Explain why object spread makes only a shallow copy.
- Update a nested property by copying each object along its path.
- Use a functional updater when the new object depends on pending state.
- Avoid replacing an object with an incomplete object by accident.
- Keep older state snapshots unchanged.

## Objects are references

A number or string is a primitive value. An object is a reference to an object in memory. A variable can hold that reference, and more than one variable or object property can point to the same object.

~~~js
const first = { city: 'Lima' };
const second = first;

second.city = 'Oslo';

console.log(first.city); // 'Oslo'
~~~

first and second refer to the same object. Changing it through either reference changes what the other reference observes.

This matters for React state because a render receives the current state value. If that value is an object and you change its properties directly, you are changing the object from that render and any other reference to it.

## Treat state objects as read-only

Do not directly assign to a property of an object held in state:

~~~jsx
function handleChange(event) {
  person.firstName = event.target.value;
}
~~~

This changes the existing object, but it does not tell React that a new state value should be rendered. It also changes the object used by previous renders, so earlier snapshots and saved references can no longer reliably represent the earlier value.

Instead, create a replacement object and pass it to the setter:

~~~jsx
function handleChange(event) {
  setPerson({
    firstName: event.target.value,
    lastName: person.lastName,
    email: person.email,
  });
}
~~~

This works, but manually copying every unchanged property becomes repetitive. Object spread makes a shallow copy of the existing object's own enumerable properties:

~~~jsx
function handleChange(event) {
  setPerson({
    ...person,
    firstName: event.target.value,
  });
}
~~~

The new firstName overrides the copied firstName. The other properties remain on the new object.

## A setter replaces the state value

A setter for an object does not automatically merge the object you pass with the existing state object. This replaces the whole state object:

~~~jsx
setPerson({
  firstName: 'Mira',
});
~~~

If person also had lastName and email, they are absent from the next object. The inputs using those properties may show empty or undefined values.

To change one property and keep the rest, copy the existing object first:

~~~jsx
setPerson({
  ...person,
  firstName: 'Mira',
});
~~~

The spread must come before the property being changed. If you place an old property after the new value, the old value overwrites the update:

~~~jsx
const nextPerson = {
  firstName: 'Mira',
  ...person,
};
~~~

If person.firstName was 'Asha', nextPerson.firstName will also be 'Asha'. Later properties in an object literal win when keys repeat.

## Use a functional updater when appropriate

If an update should use the pending object state, use an updater function:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  score: currentPerson.score + 1,
}));
~~~

The updater receives the pending object for person and returns a new object. The object spread is inside parentheses because an arrow function returning an object literal needs the parentheses to distinguish the object from a function body.

For one ordinary input change, using the person value from the current render can be fine. Use a functional updater when multiple updates may build on pending state or when that is the clearest way to base the new object on its current value.

An updater should remain a pure calculation. Do not mutate currentPerson or perform side effects inside the updater.

## Shallow copies and nested objects

Object spread copies only one level. If a property refers to another object, the copied property still points to the same nested object.

~~~js
const nextPerson = {
  ...person,
};

console.log(nextPerson !== person); // true
console.log(nextPerson.artwork === person.artwork); // true
~~~

The outer object is new, but artwork is still shared. This would still mutate the old state:

~~~js
const nextPerson = {
  ...person,
};

nextPerson.artwork.city = 'Oslo'; // Mutates the shared artwork object.
~~~

To update a nested property, create a new object for the nested level and a new object for the outer level:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  artwork: {
    ...currentPerson.artwork,
    city: 'Oslo',
  },
}));
~~~

Now the new person points to a new artwork object, and the old person still points to its unchanged artwork object.

Copy each level along the path to the property being changed. For a path like settings.notifications.email, create a new notifications object, a new settings object that points to it, and a new root state object that points to the new settings.

Properties on other paths can continue to reference their old objects when they have not changed. You do not need to deep-copy the entire state tree.

## Updating a nested property from an input

Read the input value from the event, then return a new state object at every changed level:

~~~jsx
function handleCityChange(event) {
  const city = event.target.value;

  setPerson((currentPerson) => ({
    ...currentPerson,
    artwork: {
      ...currentPerson.artwork,
      city,
    },
  }));
}
~~~

The city local variable is the value selected by this event. The updater uses it while copying the latest pending person and artwork objects. Keeping the updater focused on returning state also keeps it pure.

## One handler for several flat fields

For several inputs that update flat string properties on the same object, give each input a name matching the state property:

~~~jsx
function handleChange(event) {
  const { name, value } = event.target;

  setPerson((currentPerson) => ({
    ...currentPerson,
    [name]: value,
  }));
}
~~~

Square brackets around name create a computed property name. If name is 'email', the returned object updates the email property. Use this pattern only when the input names are controlled by your component and correspond to valid state fields.

Do not use the flat-field handler to update nested fields. For nested data, explicitly copy each object along the path.

## Adding or removing a property

Add or update a property by copying the object and adding the property:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  nickname: 'Mimi',
}));
~~~

Remove a property by destructuring it out and keeping the remaining properties:

~~~jsx
setPerson((currentPerson) => {
  const { nickname, ...personWithoutNickname } = currentPerson;
  return personWithoutNickname;
});
~~~

Both approaches create a new outer object and leave the previous object unchanged.

## Local mutation of a newly created object

Mutation is unsafe when it changes an object that already belongs to state or is shared with another part of the program. It is fine to build a brand-new local object and mutate it before giving it to React, as long as no existing state object is changed:

~~~jsx
function handleMove(event) {
  const nextPosition = {};
  nextPosition.x = event.clientX;
  nextPosition.y = event.clientY;
  setPosition(nextPosition);
}
~~~

This is safe because nextPosition was just created and is not the object from a previous render. In most cases, an object literal is shorter and clearer:

~~~jsx
setPosition({
  x: event.clientX,
  y: event.clientY,
});
~~~

## Why preserving old objects is useful

When an update creates new objects instead of changing old ones:

- React can compare old and new references to detect a state change.
- A previous render or saved reference keeps its previous values.
- Unchanged branches can keep their existing references.
- Debugging, undo history, and comparing before-and-after state are easier.

If you mutate the existing object and pass the same reference back to a setter, React may skip the update because the new value is identical to the current value according to Object.is. Even where a mutation seems to update the screen due to another render, it makes state behavior unreliable. Always create a replacement object for a state update.

## Do not deep-clone everything

A shallow copy plus copies along the changed path is usually enough. Avoid using JSON.stringify and JSON.parse as a general copying technique: it changes or drops values such as undefined, functions, and dates, and it needlessly copies data that did not change.

If deeply nested updates become difficult to follow, consider simplifying the state shape. Libraries such as Immer can provide draft-based update syntax, but they are an additional tool and are not required for object state.

## Common mistakes

### Mutating and then setting the same object

~~~jsx
person.firstName = 'Mira';
setPerson(person);
~~~

The old state object was mutated and the same reference is passed back. Create a new object instead.

### Copying only the changed field

~~~jsx
setPerson({
  firstName: 'Mira',
});
~~~

This replaces the whole object and drops the other properties. Spread the existing object before overriding the changed field.

### Copying only the outer object

~~~jsx
setPerson((currentPerson) => {
  const nextPerson = { ...currentPerson };
  nextPerson.artwork.city = 'Oslo';
  return nextPerson;
});
~~~

The artwork object is still shared with currentPerson, so this changes the old state. Copy artwork too.

### Mutating the updater's argument

~~~jsx
setPerson((currentPerson) => {
  currentPerson.score += 1;
  return currentPerson;
});
~~~

This mutates the previous object and returns the same reference. Return a new object:

~~~jsx
setPerson((currentPerson) => ({
  ...currentPerson,
  score: currentPerson.score + 1,
}));
~~~

### Expecting spread to merge recursively

~~~jsx
setPerson({
  ...person,
  artwork: {
    city: 'Oslo',
  },
});
~~~

This replaces artwork and drops its other properties such as title. Copy the nested object too:

~~~jsx
setPerson({
  ...person,
  artwork: {
    ...person.artwork,
    city: 'Oslo',
  },
});
~~~

## Practice

Given this state:

~~~jsx
const [person, setPerson] = useState({
  firstName: 'Asha',
  lastName: 'Rao',
  artwork: {
    title: 'Blue Window',
    city: 'Lima',
  },
});
~~~

Write an update that changes firstName to 'Mira' while keeping lastName and artwork.

Then write an update that changes artwork.city to 'Oslo' while keeping artwork.title and both name fields.

Finally, explain what is lost and what is mutated in this update:

~~~jsx
setPerson({
  artwork: {
    city: 'Oslo',
  },
});
~~~

## Recap

- Treat objects from React state as read-only.
- Create and set a new object instead of mutating the existing state object.
- A state setter replaces the whole object; it does not merge the new object automatically.
- Object spread makes a shallow copy, not a deep copy.
- Copy every object along the path to a nested property before changing it.
- Use a functional updater when the next object should be based on pending state.
- Preserve previous objects so earlier render snapshots stay unchanged.
