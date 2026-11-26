# Complex State with useReducer - Q&A

## Basics

### 1. What does useReducer return?

It returns an array containing the current state and a dispatch function.

### 2. What is a reducer?

A reducer is a function that receives the current state and an action and returns the complete next state.

### 3. What does dispatch do?

It sends an action to React. React passes the action and current state to the reducer, stores the returned state, and renders the component with it.

### 4. Does dispatch update the state variable immediately?

No. The event handler continues to see the state snapshot from its current render. The updated state is available after React renders again.

### 5. Does dispatch return the next state?

No. It is an instruction to React to process the action; it does not return the reducer's result.

### 6. What is an action?

An action is a value describing an event or request that should update state. It is commonly an object with a type property and a minimal payload.

### 7. What should an action type describe?

It should identify the event or interaction, such as task/added or form/submitted. Descriptive names make the transition history easier to understand.

### 8. What belongs in an action payload?

The information the reducer needs to calculate the next state, such as a task ID, edited text, or a generated timestamp.

### 9. Should an action usually contain the complete next state?

Usually not. It should describe what happened and provide the necessary data. The reducer should define how state changes in response.

### 10. Who decides the update rule: the event handler or reducer?

The event handler reports what happened by dispatching an action. The reducer contains the rule that calculates the next state.

## Choosing a reducer

### 11. When can useReducer be clearer than useState?

When many event handlers update the same related object or array, when transitions involve several fields, or when repeated update logic is difficult to inspect.

### 12. When is useState simpler?

When state is small and updates are direct, such as an independent boolean or a value with only a few simple transitions.

### 13. Does every component need a reducer?

No. useReducer is optional. Components can use useState and useReducer together, each where it provides the clearest model.

### 14. Does useReducer automatically make a component faster?

No. It primarily organizes state transition logic. Choose it for clarity; measure performance separately.

### 15. Is useReducer global state?

No. State belongs to the component that calls useReducer. To share it, pass state and callbacks or dispatch through props, or use Context when appropriate.

## Reducer behavior and purity

### 16. What does a reducer have to return?

The complete next state value. React does not automatically merge a returned object with the old one.

### 17. What happens if an object-state reducer returns only one field?

All fields not included in the returned object are lost from the next state. Copy fields that should remain, then override the changed field.

### 18. Why must a reducer be pure?

React may call it during rendering and may call it more than once in development Strict Mode. Pure functions produce the same result for the same inputs without outside effects or mutation.

### 19. What is an example of an impure reducer?

One that sends a network request, changes a module variable, reads the current time, generates random data, writes to local storage, or mutates state.

### 20. Where should a network request initiated by a button happen?

In the event handler or another appropriate effectful layer, not in the reducer. After the request, dispatch an action to represent a result if the UI state should change.

### 21. Where should a new task ID or timestamp be generated?

Generate it outside the reducer, typically in the event handler, and include it in the action payload. The reducer then remains deterministic.

### 22. Why can Strict Mode call a reducer more than once in development?

This helps reveal impure logic. If the reducer only calculates a result and does not mutate or cause side effects, the extra call should not change behavior.

### 23. What happens if the reducer mutates an object and returns the same reference?

React may ignore the update because the returned state is identical by Object.is. Mutation also violates the reducer's purity requirement.

### 24. Should a reducer's default case silently return the existing state?

It can be an intentional policy in a designed extensible system, but it can hide misspelled or unhandled actions. Throwing an error often makes mistakes easier to detect.

### 25. What can make the entire state become undefined?

A reducer branch may forget to return a value, or an action type may not match any case. Ensure every supported action returns state and handle unknown actions deliberately.

## Immutable updates

### 26. How do you update one field in an object state?

Return a new object, copy the existing fields, and set the new field value.

~~~jsx
return {
  ...state,
  name: action.name,
};
~~~

### 27. Does object spread make a deep copy?

No. Spread is shallow. For nested objects, copy every object along the path to the changed value.

### 28. How do you add an item to an array state immutably?

Return a new array, commonly by spreading the old array and adding the new item.

### 29. How do you remove an item from an array state immutably?

Use filter to return a new array without the item.

### 30. How do you update one item in an array immutably?

Use map to return a new array and create a new object for the matching item, leaving other items unchanged.

### 31. Can a reducer use push or splice on a state array?

Not directly. Those methods mutate the existing array. Return a new array using immutable operations. A library such as Immer provides a specific draft model if adopted intentionally.

### 32. Why is copying every object on a nested path important?

It ensures no existing state object is mutated. New references let React and other tools observe which parts changed.

## Actions and event handlers

### 33. Should the reducer contain click handlers?

No. Event handlers belong in components. They dispatch actions; the reducer handles the resulting state transition.

### 34. Can a single action update multiple related fields?

Yes. An action should represent a user interaction, even when that interaction changes multiple related fields. For example, one form/submitted action can clear a draft and update submission status together.

### 35. Why is an action like task/renamed more descriptive than setText?

It describes the domain event and its purpose. The reducer can apply the correct update rule without exposing the state implementation to every handler.

### 36. Can an action include a computed ID?

Yes. Compute the ID outside the reducer and include it in the action. The reducer should not generate it as a side effect.

### 37. Can several dispatch calls happen from one event?

They can, but if one user interaction represents one coherent transition, a single action that updates related state is often clearer and avoids describing artificial intermediate steps.

### 38. Is the dispatch function stable between renders?

React guarantees that the dispatch function identity is stable. Including it in dependencies is generally not required, although including it does not cause a change.

## Initial state

### 39. When is the second argument to useReducer used?

It supplies the initial state when the component initializes. Later changes to that argument do not automatically reset existing reducer state.

### 40. What is the third argument to useReducer for?

It supplies an initializer function. React calls that function with the second argument to calculate the initial state.

### 41. Why use an initializer function?

It avoids performing an expensive initial-state calculation on every render when the calculation only needs to happen during initialization.

### 42. Should you pass createInitialState(input) or createInitialState as the initializer?

Pass the function itself as the third argument and pass input as the second argument. React calls the initializer with that input during initialization.

### 43. Does changing a prop used to initialize state automatically reinitialize the reducer?

No. Initial state is used when the component initializes. If later prop changes should affect state, define an explicit transition or intentionally remount the component with a new key when it represents a new identity.

### 44. Must the initializer be pure?

Yes. React may call it more than once in development Strict Mode to help detect impurities.

## Troubleshooting and review

### 45. Why can console.log immediately after dispatch show the previous state?

The handler reads the snapshot from its current render. Dispatch requests a subsequent render; it does not change the already-running handler's state variable.

### 46. Why might the screen not update after an action?

The reducer may have mutated state and returned the same object or array reference, returned an unchanged value, or failed to return the expected next state.

### 47. What usually causes “Too many re-renders” with dispatch?

An action may be dispatched unconditionally during rendering. Dispatch in an event handler or an appropriate effect rather than calling it while computing JSX.

### 48. How can you make a list of reducer transitions easier to debug?

Use descriptive action types, small explicit payloads, immutable transitions, and a reducer with clear cases. Each action should help explain what happened.

### 49. Is a reducer required when state is an object?

No. useState can manage object state. A reducer is useful when centralizing related transition logic makes the component easier to understand.

### 50. What is the key difference between an action and the next state?

An action describes what happened. The reducer transforms the current state and that action into the next state.

## Key takeaway

Dispatch describes an interaction, and the reducer returns the complete next state. Keep reducers pure and immutable, then choose useReducer only when it makes related state transitions clearer.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="045-controlled-and-uncontrolled-component-apis.md">045 - Controlled and Uncontrolled Component APIs - Q&amp;A</a></td><td><a href="../lessons/046-complex-state-with-usereducer.md">Lesson</a></td><td><a href="../examples/046-complex-state-with-usereducer.md">Example</a></td><td><a href="047-reducer-design-and-state-machines.md">047 - Reducer Design, Transitions, and State Machines - Q&amp;A</a></td></tr>
  </tbody>
</table>
