# 037. State with useState - Q&A

## State basics

### 1. What is React state?

State is information that a component remembers between renders and can update in response to interactions or other work.

### 2. Why does a component need state?

A component needs state when a changing value must survive another render and affect the interface it returns.

### 3. Why is a regular local variable not enough for a counter?

The variable is recreated when the component function runs, and changing it does not tell React to render again. React-managed state persists and its setter requests a render.

### 4. What does useState return?

It returns a two-item array: the current value for this render and a setter function for that state.

### 5. What is the usual naming convention for the pair?

Name the value and setter together, such as count and setCount, isOpen and setIsOpen, or query and setQuery. These names are conventions, not required API names.

### 6. Can a component have more than one state variable?

Yes. A component can call useState multiple times for distinct pieces of state, such as a current count and whether details are visible.

### 7. Can useState store different kinds of values?

Yes. State can contain values such as numbers, strings, booleans, null, objects, arrays, and functions. Functions have special initializer and updater behavior.

### 8. What should the initial state represent?

It should be a valid value for the interface before interaction. For example, text often starts as an empty string, a count as zero, a disclosure flag as false, and a not-yet-selected record as null.

### 9. Does React use the initial value again on every render?

No. The initial argument initializes that state when the component instance is created. Later renders receive the current stored value.

### 10. Does changing a prop to a new value reset a useState value automatically?

No. A new initial argument does not reset already initialized state. State reset and component identity are covered in a later chapter.

### 11. What does it mean that state is local?

State belongs to the particular rendered component instance that declared it. Another component cannot directly read or change that private state.

### 12. If I render the same stateful component twice, do both copies share state?

No. Each rendered instance has its own state. Clicking a control in one instance does not change the other instance unless the state is shared through a common owner.

## Declaring and updating state

### 13. Where should useState be called?

Call it at the top level of a function component or custom Hook, before conditional returns. Do not put it inside conditions, loops, event handlers, or nested functions.

### 14. Why must Hook calls remain in a stable order?

React associates Hook state with the order of Hook calls during rendering. Conditional or looped calls could change that order between renders.

### 15. What does calling a state setter do?

It asks React to store a next value and render the component again so that its output can reflect that value.

### 16. Should I assign directly to the state variable?

No. Use the setter. A direct assignment changes only a local binding and does not update React's stored state.

### 17. Does the setter return the new value?

No. The setter's return value is undefined. The updated state is provided when React renders the component again.

### 18. Does calling the setter change the state variable in the current handler immediately?

No. The current handler continues to use the value from its render. The next chapter explains this render-snapshot behavior in detail.

### 19. How can I set a state variable to a known value?

Pass the next value to its setter, such as setCount(0) or setIsOpen(true).

### 20. How do I update a value based on its previous value?

Pass an updater function to the setter, such as setCount((previousCount) => previousCount + 1). The queued-update chapter explains why this form matters when updates build on one another.

### 21. What should an updater function do?

It should calculate and return the next state from its argument. Keep it pure: do not mutate external values, perform requests, or cause other side effects inside it.

### 22. Can Strict Mode call an updater function more than once in development?

Yes. Strict Mode may invoke updater functions an extra time to expose impure logic. React ignores the extra result; a pure updater returns the same result for the same input.

### 23. Can I call a setter during rendering?

Normally, state updates belong in event handlers or other deliberate update flows, not in render. React permits a narrow same-component render-phase update pattern, but it is uncommon and should not be used as a default technique.

### 24. What happens if the next state is equal to the current state?

React may skip rendering the component's children when the next value is identical according to Object.is. Code should not rely on a particular render happening as an optimization detail.

### 25. Does a setter have a new identity on every render?

The setter function returned by useState has a stable identity. It is safe to pass to event handlers or other components.

## Initializer functions and value types

### 26. What is a lazy initializer?

If calculating the initial value is expensive, pass an initializer function to useState. React calls it to calculate the initial state rather than evaluating that calculation on every render.

### 27. What is the difference between passing createInitialData and createInitialData()?

Passing createInitialData gives React the initializer function. Calling createInitialData() runs it during every render before useState receives its result, even though later results are ignored.

### 28. Must an initializer function be pure?

Yes. It should calculate and return an initial value without changing external state or causing side effects. Strict Mode may call it extra times in development.

### 29. What if I pass a function as the state value?

React treats a function passed as the useState argument as an initializer to call. To store a function itself, return it from an initializer function.

### 30. How do I replace a function stored in state?

Pass a wrapper updater that returns the function, so React does not interpret the function itself as an updater. Storing functions in state is uncommon.

### 31. Does useState merge object properties automatically?

No. A useState setter replaces the state value; it does not shallow-merge object properties. Create the next object with the properties you want to keep.

### 32. Can I mutate an object or array stored in state?

Do not mutate it. Create a new object or array for the next value. Mutating the existing reference can prevent React from recognizing a meaningful change and can make state harder to reason about.

### 33. What initial value should a controlled text input use?

Usually an empty string, so the input has a string value from its first render. Keep the type stable as the person edits it.

### 34. Should a boolean checkbox value start as an empty string?

No. Model whether it is checked as a boolean, typically starting with false, and render its checked property from that state.

## Choosing and debugging state

### 35. Should every value in a component be state?

No. Use state for values that change and affect rendered output. Constants, temporary calculations, and values that do not affect the interface can remain ordinary variables.

### 36. Should a derived value also be stored in state?

Usually no. Calculate it from existing props or state while rendering. Duplicating a value creates a risk that the copies will disagree.

### 37. Should I use state to store a DOM node?

Usually no. A ref is the usual tool for retaining a DOM node or a mutable value that does not itself need to render. State is for data that affects the UI.

### 38. Should I use an Effect just to update one state value from another?

Usually no. If a value can be calculated from current props or state during rendering, derive it directly instead of adding another state variable and Effect.

### 39. Why does my button click change a variable but not the page?

The value may be an ordinary local variable. Use a state setter for data that should persist and update rendered output.

### 40. Why did the state reset after I navigated to another view?

The component instance may have been removed and later created again, or its identity may have changed. State is retained for a mounted component instance; a later chapter explains how position and keys affect that identity.

### 41. How can two components share one piece of state?

Move the state to their nearest common parent and pass the value and event handlers as props. This gives both children a single shared source of truth.

### 42. What is the core rule for useState?

Call it unconditionally at the top level, choose an initial value that fits the UI, update it through its setter, and render from the current value.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="036-accessible-interactive-form-project.md">036 - Build an accessible interactive form feature - Q&amp;A</a></td><td><a href="../lessons/037-state-with-usestate.md">Lesson</a></td><td><a href="../examples/037-state-with-usestate.md">Example</a></td><td><a href="038-state-as-a-render-snapshot.md">038 - State as a render snapshot - Q&amp;A</a></td></tr>
  </tbody>
</table>
