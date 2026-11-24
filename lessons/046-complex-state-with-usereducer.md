# Complex State with useReducer

## Why reducers are useful

A component can start with a few independent state values and simple event handlers. As it grows, several handlers may update the same object or array in related ways. Update rules can become scattered, duplicated, and difficult to inspect.

The useReducer Hook moves those state transition rules into one reducer function. Event handlers describe what happened by dispatching actions. The reducer receives the current state and an action, then returns the next state.

A reducer is not required for every component. It is a way to make related state transitions easier to organize.

## Learning goals

After this lesson, you should be able to:

- explain the roles of state, actions, reducers, and dispatch;
- decide when useReducer is useful and when useState is simpler;
- write a pure reducer that returns the complete next state;
- dispatch descriptive actions from event handlers;
- update objects and arrays immutably;
- handle unknown actions and avoid missing state fields;
- initialize reducer state directly or with an initializer function;
- keep side effects out of reducers.

## The useReducer contract

Call useReducer at the top level of a component:

~~~jsx
const [state, dispatch] = useReducer(reducer, initialState);
~~~

It returns two values:

- state: the current state for this render;
- dispatch: a function used to send an action to the reducer.

An action is usually an object that describes an interaction:

~~~jsx
dispatch({ type: "counter/incremented" });
~~~

React passes the current state and dispatched action to the reducer. The reducer returns the next state. React then renders the component with that state.

~~~text
user interaction
    └── event handler dispatches an action
reducer(current state, action)
    └── returns the next state
React stores next state and renders again
~~~

The event handler says what happened. The reducer decides how that event changes state.

## A small counter

~~~jsx
import { useReducer } from "react";

function counterReducer(state, action) {
  switch (action.type) {
    case "counter/incremented":
      return { count: state.count + 1 };

    case "counter/decremented":
      return { count: state.count - 1 };

    case "counter/reset":
      return { count: 0 };

    default:
      throw Error("Unknown action: " + action.type);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <section>
      <p>Count: {state.count}</p>
      <button
        type="button"
        onClick={() => dispatch({ type: "counter/decremented" })}
      >
        Decrement
      </button>
      <button
        type="button"
        onClick={() => dispatch({ type: "counter/incremented" })}
      >
        Increment
      </button>
      <button
        type="button"
        onClick={() => dispatch({ type: "counter/reset" })}
      >
        Reset
      </button>
    </section>
  );
}
~~~

The reducer is a normal function outside the component. It has no access to component variables unless they are passed in through state or the action. This makes its inputs and outputs clear.

## State, action, reducer, and dispatch

### State

State is the data the component needs to remember between renders. A reducer can manage an object, array, primitive value, or another suitable data structure.

Choose a state shape that reflects the information and relationships the component needs. Do not combine unrelated data just because useReducer accepts one state value.

### Action

An action describes an event or request that should change state. It usually contains:

- a type that identifies the event;
- the minimal payload required to apply it.

For example:

~~~jsx
dispatch({
  type: "task/renamed",
  taskId: task.id,
  nextName: inputValue,
});
~~~

The action should describe the interaction. It should not contain an entire next state object unless replacing the whole state is genuinely the action.

### Dispatch

dispatch sends an action to React. It does not return the next state and does not mutate the state variable in the current render.

~~~jsx
function handleIncrement() {
  dispatch({ type: "counter/incremented" });
  // state.count is still the snapshot from this render here.
}
~~~

The next state becomes available when React renders the component again. If an event handler needs information for an action, include it in the action payload or calculate it from the current render before dispatching.

### Reducer

A reducer has the shape reducer(state, action) and returns the complete next state:

~~~jsx
function reducer(state, action) {
  // Return the complete next state.
}
~~~

A reducer should be pure: the same state and action should produce the same next state without changing anything outside the function.

## Refactor from useState to useReducer

Suppose a task list has handlers for adding, editing, and deleting tasks. The state logic can be spread across the component:

~~~jsx
const [tasks, setTasks] = useState(initialTasks);

function handleDelete(taskId) {
  setTasks(tasks.filter((task) => task.id !== taskId));
}
~~~

When more transitions are added, it can help to consolidate them:

1. Replace setter calls in event handlers with dispatch calls.
2. Define actions for the interactions.
3. Move transition logic into a reducer.
4. Replace useState with useReducer.
5. Render from the state returned by the reducer.

For example:

~~~jsx
function handleDelete(taskId) {
  dispatch({ type: "task/deleted", taskId });
}
~~~

The handler now names the event. The reducer contains the state transition rule.

## A reducer for an object state

If state is an object, each branch must return the complete object:

~~~jsx
function profileReducer(state, action) {
  switch (action.type) {
    case "profile/nameChanged":
      return {
        ...state,
        name: action.name,
      };

    case "profile/ageIncremented":
      return {
        ...state,
        age: state.age + 1,
      };

    default:
      throw Error("Unknown action: " + action.type);
  }
}
~~~

Reducers do not merge returned objects automatically. If a branch returns only { name: action.name }, every other property disappears. Copy the fields that should remain, commonly with the spread operator, and then replace the field being changed.

Spread is shallow. For nested data, copy every object along the path to the changed value:

~~~jsx
return {
  ...state,
  settings: {
    ...state.settings,
    theme: action.theme,
  },
};
~~~

Do not mutate an existing object and then return it.

## Updating arrays in a reducer

Return new arrays for collection changes:

~~~jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case "task/added":
      return [
        ...tasks,
        {
          id: action.taskId,
          title: action.title,
          done: false,
        },
      ];

    case "task/deleted":
      return tasks.filter((task) => task.id !== action.taskId);

    case "task/toggled":
      return tasks.map((task) =>
        task.id === action.taskId
          ? { ...task, done: !task.done }
          : task
      );

    default:
      throw Error("Unknown action: " + action.type);
  }
}
~~~

Common immutable operations include:

- map to replace an item;
- filter to remove an item;
- spread to add or copy items;
- creating a new object for the item being changed.

Do not call push, splice, sort, or directly assign to state-owned arrays or objects unless using a library that explicitly provides safe draft semantics.

## Reducers must be pure

A reducer should only calculate and return state. Do not perform side effects inside it, such as:

- sending a network request;
- writing to local storage;
- generating random values;
- reading the current time;
- changing a module-level counter;
- modifying an existing object or array.

Create values such as IDs or timestamps in the event handler and include them in the action:

~~~jsx
function handleAddTask(title) {
  const taskId = crypto.randomUUID();

  dispatch({
    type: "task/added",
    taskId,
    title,
  });
}
~~~

The ID is generated once for the user interaction. If the reducer generated it, calling the reducer twice with the same state and action could produce different results.

Side effects belong in event handlers or Effects, depending on whether they are caused by a specific user event or by synchronization with an external system.

## Handle action types clearly

A switch statement is a common way to organize reducer cases. Each case should return a state value. A default branch that throws for an unknown action helps reveal spelling mistakes and incomplete transitions:

~~~jsx
default:
  throw Error("Unknown action: " + action.type);
~~~

For a deliberately extensible reducer, a no-op return can be an intentional policy, but silently ignoring misspelled actions can hide bugs. Choose and document the behavior.

Avoid accidental fall-through between cases. Return or break deliberately. Braces around cases help scope variables when a case declares them.

## Keep actions meaningful

Prefer actions that describe the event:

~~~jsx
dispatch({ type: "cart/itemAdded", productId });
dispatch({ type: "cart/quantityChanged", productId, quantity });
dispatch({ type: "cart/itemRemoved", productId });
~~~

This makes update logs easier to interpret and keeps the transition rule inside the reducer.

If a single user interaction changes several related fields, one action can describe that interaction. For example, form/submitted may update status, clear a draft, and store a confirmation in one transition.

Avoid dispatching several tiny setter-style actions for one event when they create intermediate states that should never appear.

## Initial state

The second argument to useReducer is the initial state:

~~~jsx
const [state, dispatch] = useReducer(reducer, { count: 0 });
~~~

React uses the initial state when the component initializes. A new initial argument on a later render does not reset the existing reducer state.

For state that needs to be created from an input or expensive setup, use the optional initializer function:

~~~jsx
function createInitialState(username) {
  return {
    draft: "",
    tasks: [],
    owner: username,
  };
}

function TaskEditor({ username }) {
  const [state, dispatch] = useReducer(
    tasksReducer,
    username,
    createInitialState
  );

  // ...
}
~~~

React calls createInitialState with the second argument to produce the initial state. Pass the function itself, not createInitialState(username), when using this form.

The initializer should also be pure. In development Strict Mode, React may call the reducer and initializer more than once to expose accidental impurity. They must not mutate external data or trigger side effects.

If a prop changes later and the reducer state should reset or adapt, define that behavior explicitly. Options include dispatching a reset action in response to an event, or intentionally remounting a component with a different key when a new identity should start fresh. Do not assume the initial argument resets state on every render.

## When to use useReducer

useReducer is often helpful when:

- many event handlers update the same state object or array;
- transitions involve several related fields;
- update logic is duplicated across handlers;
- action names would make state changes easier to understand;
- isolating transition logic would make the component easier to reason about.

useState is often clearer when:

- state is simple and independent;
- each value has only a few straightforward updates;
- a reducer would add more ceremony than structure.

You can use both Hooks in one component. A reducer is not automatically more performant and does not replace a server-state library. Choose it for the organization and clarity of transitions.

## Common mistakes

### Mutating state

Changing an existing object or array and returning it can cause React to miss the update and violates the reducer's pure-function contract. Return new objects and arrays.

### Forgetting existing object fields

A reducer returns the whole next state. It does not merge the new object with the old one. Copy fields that should remain.

### Returning no value

Every handled action must return the next state. Falling through without a return can make state undefined.

### Silently accepting unknown actions

A misspelled type can look like an interaction that did nothing. Throwing in a default branch often makes this failure easier to find.

### Doing side effects in the reducer

Reducers may run more than once in development and should be deterministic. Generate IDs and timestamps before dispatch; perform requests outside the reducer.

### Reading the old state immediately after dispatch

dispatch requests an update; it does not change the current render's state snapshot. Read the updated state on the next render.

### Choosing a reducer for every value

A simple boolean with two direct updates may be easier with useState. Use useReducer when its central transition logic makes the code clearer.

### Treating useReducer as global state

useReducer manages state in the component that calls it. To share it, pass state and callbacks or dispatch down, or use Context where appropriate.

## Practice

1. What are the two values returned by useReducer?
2. What does dispatch do?
3. What must a reducer return?
4. Why should a reducer not generate a random ID?
5. How can a reducer update one item in an array without mutation?
6. If a reducer returns { name: action.name } for an object state, what happens to other fields?
7. Why might an unknown action throw an error?
8. Does a new initialState prop automatically reset useReducer state?
9. When might useState be a better choice?
10. Can one user action update several related fields in a reducer?
11. Why can logging state immediately after dispatch show the old value?
12. Where should a network request started by a user action happen?

Suggested answers:

1. The current state and a dispatch function.
2. It sends an action to React; React passes it with the current state to the reducer and schedules the returned state for a render.
3. The complete next state value.
4. The reducer must return the same result for the same inputs. Generate the ID in the event handler and include it in the action.
5. Use map to create a new array and a new object for the changed item.
6. The other fields are absent from the returned next state unless they are copied.
7. It exposes misspelled or unhandled action types instead of silently ignoring them.
8. No. The initial state is used when the component initializes. A later prop change needs explicit reset behavior.
9. When the state and update logic are simple enough that a reducer would add unnecessary structure.
10. Yes. One action can represent one interaction that changes multiple related fields together.
11. The current event handler sees the state snapshot from its render. The new state is available on a later render.
12. In the event handler or an Effect, depending on what starts and governs the request, not in the reducer.

## Summary

- useReducer returns current state and dispatch.
- Event handlers dispatch descriptive actions; a reducer returns the complete next state.
- Reducers must be pure and must not mutate objects, arrays, or external data.
- Copy existing fields and nested paths when creating changed state.
- Keep side effects, IDs, and timestamps outside the reducer.
- Use an initializer function for computed initial state when useful.
- Choose useReducer when it clarifies related transitions; useState remains appropriate for simple state.
