# Reducer Design, Transitions, and State Machines

## Why reducer design matters

useReducer gives a component one place to describe how actions change state. A reducer can still become difficult to understand if its state shape is unclear, actions are vague, or transitions allow impossible combinations.

Good reducer design starts before the switch statement. Define what the state means, which events can change it, which transitions are valid, and what each state must contain.

A **state machine** makes these choices explicit. It represents an interface as a set of known states and a set of allowed transitions between them. A reducer can implement a state machine, but not every reducer is automatically a state machine.

## Learning goals

After this lesson, you should be able to:

- model an interface with explicit states and transitions;
- describe transitions in a table before implementing them;
- use guards to allow or reject transitions based on current data;
- choose state shapes that prevent contradictory combinations;
- distinguish normal validation errors from programming errors;
- handle stale asynchronous results with request identifiers;
- keep reducers pure while placing effects outside them;
- split complex reducer logic without overengineering it.

## Start with the state model

Before writing a reducer, list the distinct modes the interface can be in.

A registration flow might have these steps:

~~~text
contact → session → review → complete
    ↑          ↑
    └── Back ──┘
~~~

Each step has a different interaction:

- contact collects a name and email;
- session collects a choice and accessibility preference;
- review shows the collected values;
- complete confirms that the local flow finished.

The flow should not show the contact and completion screens as simultaneous states. Represent the current step with one value:

~~~jsx
{
  step: "contact",
  values: {
    name: "",
    email: "",
    session: "",
    needsAccessSupport: false,
  },
  errors: {},
}
~~~

This is clearer than multiple flags such as isContactOpen, isReviewOpen, and isComplete. Separate booleans can accidentally allow several mutually exclusive screens to be active at once.

## Distinguish state, context, and transitions

A state machine often has:

- **states**: the current mode, such as contact or review;
- **context**: data carried between states, such as form values;
- **events/actions**: things that happen, such as a field changing or the user choosing Next;
- **transitions**: rules that say which next state follows an event;
- **guards**: conditions that must be true for a transition to happen.

In a React reducer, all of these can be represented with ordinary JavaScript values. A state object might contain a step and form values. An action identifies an event. The reducer applies the transition rule and returns the next state.

## Write a transition table first

A transition table makes missing cases visible before code is written:

| Current step | Action | Condition | Next step |
| --- | --- | --- | --- |
| contact | next | name and email valid | session |
| contact | next | values invalid | contact with errors |
| session | back | none | contact |
| session | next | session selected | review |
| session | next | no session selected | session with errors |
| review | back | none | session |
| review | complete | none | complete |
| complete | reset | none | contact |

The table also clarifies that Next is not always a transition. When validation fails, the step stays the same and the reducer records errors.

## Model exclusive modes with one status

For a request lifecycle, avoid unrelated booleans such as isLoading, hasError, and isSuccess. They allow contradictory combinations.

Use one status value:

~~~jsx
{ status: "idle" }
{ status: "loading", requestId: "r1" }
{ status: "success", data: results }
{ status: "error", message: "Could not load results." }
~~~

Only the data relevant to a state needs to be present. This is sometimes called a discriminated state shape: status identifies which fields make sense.

The same idea applies to tabs, dialogs, checkout steps, upload phases, and multi-step forms.

## Use guards for conditional transitions

A guard checks whether a transition is allowed. For a form, moving from contact to session might require a valid name and email.

~~~jsx
function validateContact(values) {
  const errors = {};

  if (!values.name.trim()) {
    errors.name = "Enter your name.";
  }

  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email)) {
    errors.email = "Enter a valid email address.";
  }

  return errors;
}
~~~

The reducer can run this pure validation rule when it receives a next action. If errors exist, it returns the same step with an errors object. If there are no errors, it advances.

Validation failure is a normal application outcome, not an exceptional programming error. It should be represented in state, not thrown as an unknown-action error.

## Keep invariants visible

An invariant is a condition that should always be true for valid state. Examples:

- only one workflow step is active;
- a success state has result data;
- a loading state has a request ID;
- a completed task is not simultaneously being edited;
- a selected item ID refers to an available item.

Design state and transitions so invalid combinations are hard to create. For example, this representation permits inconsistent flags:

~~~jsx
{
  isLoading: true,
  hasError: true,
  isSuccess: true,
}
~~~

One status property communicates the exclusive mode better:

~~~jsx
{
  status: "loading",
}
~~~

When related values change together, one action and one reducer transition can preserve the invariant atomically.

## Separate validation from unknown actions

There are two different failure cases:

1. **Expected invalid user data**: return state with validation errors and stay in the current step.
2. **Unexpected action type**: throw an error or apply a deliberately documented policy.

Do not throw for routine input mistakes. Do not silently ignore programmer mistakes unless ignoring them is an explicit design decision.

## Keep actions specific to events

An action should describe what happened, with the information needed to apply it:

~~~jsx
dispatch({ type: "registration/nextRequested" });
~~~

~~~jsx
dispatch({
  type: "registration/fieldChanged",
  field: "email",
  value: nextEmail,
});
~~~

Prefer one action for one meaningful interaction. If clicking Next validates three fields and advances or records errors, one nextRequested action can describe that interaction. The reducer can update all related state at once.

Action names are internal to the feature. A consistent naming scheme helps scan a reducer, but complicated naming conventions are not a substitute for a clear state model.

## Example reducer structure

A reducer can keep event handling and transition decisions explicit:

~~~jsx
function registrationReducer(state, action) {
  switch (action.type) {
    case "registration/fieldChanged":
      return updateRegistrationField(state, action);

    case "registration/nextRequested":
      return advanceRegistration(state);

    case "registration/backRequested":
      return goBack(state);

    case "registration/completed":
      if (state.step !== "review") {
        return state;
      }

      return {
        ...state,
        step: "complete",
        confirmationCode: action.confirmationCode,
        errors: {},
      };

    case "registration/reset":
      return initialState;

    default:
      throw Error("Unknown action: " + action.type);
  }
}
~~~

Small pure helpers can make transitions easier to read. Keep them close to the reducer when they are feature-specific; move them to another module when that makes the codebase clearer.

Returning the shared initialState constant for reset is safe when the reducer and components treat it as read-only and every later update creates new objects and arrays. Do not mutate the initial object.

## Handle asynchronous transitions outside the reducer

Reducers are synchronous pure calculations. They do not fetch data, set timers, or perform other effects.

A component can dispatch a start action, perform the request outside the reducer, then dispatch a result action:

~~~jsx
async function handleSearch() {
  const requestId = crypto.randomUUID();

  dispatch({
    type: "search/started",
    requestId,
    query,
  });

  try {
    const results = await searchItems(query);

    dispatch({
      type: "search/succeeded",
      requestId,
      results,
    });
  } catch (error) {
    dispatch({
      type: "search/failed",
      requestId,
      message: "Search could not be completed.",
    });
  }
}
~~~

The reducer handles search/started, search/succeeded, and search/failed. The request itself stays outside.

If a second request can start before the first finishes, include a request ID in the loading state and result actions. The reducer can ignore a result whose ID is not the current request:

~~~jsx
case "search/succeeded":
  if (
    state.status !== "loading" ||
    state.requestId !== action.requestId
  ) {
    return state;
  }

  return { status: "success", results: action.results };
~~~

This prevents an older response from overwriting newer results. This guard is useful only when overlapping requests are possible; do not add request IDs without a reason.

## Keep reducer output complete and immutable

Every transition returns the complete next state. For object state, copy fields that should remain:

~~~jsx
return {
  ...state,
  step: "session",
  errors: {},
};
~~~

For nested data, copy each object along the modified path. For arrays, use operations that return new arrays, such as map, filter, and spread. Do not mutate state and return the same reference.

## Keep the reducer understandable

A reducer with many states may need structure:

- group cases by feature or transition;
- move repeated validation into pure helper functions;
- use a state shape that keeps irrelevant fields out of modes where they do not apply;
- split independent domains into separate reducers or useState values;
- use subreducers when they make boundaries clearer.

Avoid building a generic state-machine framework for a small component. Plain objects, actions, a switch, and a transition table are often enough.

## Do not confuse a reducer with a formal state-machine library

A reducer is a function that maps current state and action to next state. A finite-state model adds explicit states and transition rules. Reducers can implement such models, but complex workflows may benefit from a state-machine library if the transition graph, parallel states, or orchestration becomes difficult to maintain.

Choose the simplest representation that makes valid states and transitions clear.

## Common mistakes

### Using several booleans for exclusive phases

Multiple flags can represent impossible combinations. Prefer one status or step field when only one mode may be active.

### Advancing before validating

The reducer should preserve the current step and record errors when a guard fails. The UI should not skip required transitions.

### Mixing errors with unknown actions

Invalid user data is expected and should usually become state. Unknown action types indicate a programming issue and may warrant an error.

### Performing effects in a reducer

Requests, timers, random IDs, and external writes do not belong in a pure reducer. Perform them outside and dispatch the resulting event data.

### Accepting stale async results

If requests may overlap, a previous request can finish after a newer one. Track a request ID and ignore results that do not belong to the active request.

### Returning partial state

React does not merge reducer results automatically. Return every field required by the next state.

### Making transitions too generic

Actions like setStatus or setField can hide the meaning of an interaction when used indiscriminately. Use actions that communicate the event and define the transition in the reducer.

### Overengineering a small state update

A simple boolean or independent input may not need a state machine. Use explicit transitions when the workflow or invariants justify them.

## Practice

1. A workflow can be in exactly one of idle, loading, success, or error. What state shape is clearer than four booleans?
2. What is a guard in a state transition?
3. Should invalid form input throw an unknown-action error?
4. What should a reducer do when Next is requested but validation fails?
5. Why should a network request not run inside a reducer?
6. When are request IDs useful?
7. What does a transition table help you discover?
8. Does every useReducer reducer implement a finite-state machine?
9. When can several state fields be updated in one action?
10. Why might you keep saved data and an unsaved draft as separate context values?

Suggested answers:

1. A single status field with one of the four values; include data relevant to that status.
2. A condition that must be true before the transition is allowed.
3. No. Validation failure is expected application state; record errors and remain in the current state.
4. Return the current step with validation errors rather than advancing.
5. A reducer must be pure and deterministic. Perform the request elsewhere and dispatch success or failure.
6. When requests can overlap and an older result must not replace the state for a newer request.
7. Missing transitions, invalid actions, guard conditions, and where validation or reset behavior belongs.
8. No. A reducer can update ordinary state without modeling a finite set of modes.
9. When they are consequences of one meaningful user interaction and should remain consistent together.
10. They represent different stages of data: persisted/current data and changes not yet committed.

## Summary

- Design the state model and transition rules before writing reducer cases.
- Represent mutually exclusive modes with one status or step value.
- Use guards to keep invalid transitions from occurring.
- Treat validation errors as normal state and unknown actions as programming errors.
- Keep reducers pure; perform asynchronous work and other effects outside them.
- Use request identifiers when overlapping responses could arrive out of order.
- Use a formal state-machine library only when a plain reducer no longer expresses the workflow clearly.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="046-complex-state-with-usereducer.md">046 - Complex State with useReducer</a></td><td><a href="../qnas/047-reducer-design-and-state-machines.md">Q&amp;A</a></td><td><a href="../examples/047-reducer-design-and-state-machines.md">Example</a></td><td><a href="048-preserving-and-resetting-state.md">048 - Preserving and Resetting State with Tree Position and Keys</a></td></tr>
  </tbody>
</table>
