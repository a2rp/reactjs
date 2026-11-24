# Reducer Design, Transitions, and State Machines — Q&A

## State-machine foundations

### 1. What does a reducer do?

It receives the current state and an action and returns the next state. The action describes an event; the reducer applies the transition rule.

### 2. What is a state machine?

It is a model with a defined set of states and rules for which events can move the system from one state to another.

### 3. Is every reducer a state machine?

No. A reducer can update ordinary state without modeling mutually exclusive modes or an explicit transition graph. A reducer can implement a state machine when the workflow benefits from explicit states and transitions.

### 4. What is a state in a state-machine model?

A state is the current mode of the workflow, such as contact, review, loading, or success. It is different from the whole React state object, which may also hold data used across modes.

### 5. What is context in state-machine terminology?

Context is the data carried along with the current mode, such as form values, a selected item, or a result list.

### 6. What is an event or action?

It describes something that happened or was requested, such as fieldChanged, nextRequested, or searchSucceeded.

### 7. What is a transition?

It is the rule that maps the current state and an event to the next state, possibly updating context.

### 8. What is a guard?

A guard is a condition that must be satisfied before a transition is allowed, such as valid input before moving to the next form step.

### 9. Why write a transition table before coding?

It makes states, events, guards, next states, and missing cases visible. It can reveal unsupported paths before they become scattered conditionals in event handlers.

### 10. Does every action have to change the state-machine mode?

No. An action may update context while leaving the current mode unchanged. Invalid input may also leave the mode unchanged while adding validation errors.

## State shape and invariants

### 11. Why use one step value instead of several booleans for a wizard?

A single step represents the fact that exactly one stage is active. Several booleans could accidentally make multiple exclusive stages active at once.

### 12. What is an invariant?

It is a condition that should always be true of valid state, such as a loading state having a request ID or a workflow having only one active step.

### 13. How can state shape help preserve an invariant?

Use values that encode the allowed combinations directly. One status string is safer than unrelated booleans when modes are mutually exclusive.

### 14. What is a discriminated state shape?

It is a representation where a status or mode field identifies which other fields are relevant. For example, success may contain results while error contains an error message.

### 15. Why might each status have its own object shape?

It can avoid irrelevant or contradictory data, such as an error message remaining on a success state. This can make valid states easier to reason about.

### 16. Should unrelated state be forced into one reducer?

No. A reducer can manage related transitions. Independent state may remain in useState or in another reducer when that creates clearer ownership.

### 17. Can one action update several state fields?

Yes. If several fields change as part of one meaningful interaction, one transition can update them together and preserve consistency.

## Guards and invalid transitions

### 18. What should happen when a user requests Next but the form is invalid?

Keep the current step and return state containing the validation errors. Do not advance.

### 19. Is invalid user input an exceptional programming error?

Usually not. It is an expected application condition and should be represented in state for the interface to display.

### 20. How is invalid input different from an unknown action type?

Invalid input is a normal user outcome, such as a missing email. An unknown action usually indicates a programming mistake, such as a misspelled action type.

### 21. Should validation errors be thrown from the reducer?

No. Return a state that records the errors and stays in the current mode. Throwing is better reserved for unexpected programmer errors, depending on the application's policy.

### 22. What can a reducer do when an action is invalid for the current mode?

It can return the existing state to ignore the invalid transition, or throw if the action indicates a programming error. Choose a consistent policy and document important behavior.

### 23. Why might an event handler still validate before dispatching?

It can avoid dispatching an obviously invalid event or provide immediate interaction logic. However, the reducer should own transition rules when those rules must remain consistent across callers.

### 24. If a reducer ignores invalid transitions, how can those cases be found?

Use explicit UI controls and reducer conditions, and consider development diagnostics for impossible events. Do not silently ignore unknown action types, since that can hide misspellings.

## Async transitions and side effects

### 25. Why should a reducer not perform a network request?

A reducer should be a pure state calculation. Network requests are side effects whose timing and results are external to the state transition function.

### 26. Where should a request started by a button happen?

The event handler can start the request, dispatch a started action, then dispatch a success or failure action when the request settles. Other architectures may use an Effect or a data-fetching library.

### 27. What information should a started action carry?

The minimum information the reducer needs to represent the request, often a request ID and perhaps the query or parameters.

### 28. Why can two overlapping requests cause a stale result?

An earlier request can finish after a later request. If both results are accepted, the older response can overwrite the newer one.

### 29. How can a request ID help?

Store the active request ID in the loading state and include the ID in result actions. The reducer accepts a result only if its ID matches the active request.

### 30. Should every asynchronous action use a request ID?

No. Add request IDs when requests can overlap or stale results could cause incorrect UI. Avoid extra machinery when it solves no real case.

### 31. What should the reducer do with a result from an old request?

Ignore it or otherwise prevent it from replacing the current request's state. A comparison between the action's request ID and the active state's ID is a common guard.

### 32. Where should a random ID or timestamp be generated?

Outside the reducer, usually in the event handler, and then passed in the action. The reducer should produce deterministic output for a given state and action.

### 33. Can a reducer set a timeout or write to local storage?

No. Those are side effects. Perform them in an event handler, Effect, or another explicit effect layer.

## Action and reducer design

### 34. What makes an action useful?

It describes a meaningful event and contains the minimal payload required for the reducer to apply the transition.

### 35. Why is nextRequested more informative than setStep?

nextRequested describes what the user did. The reducer can validate and decide the appropriate next step instead of letting callers choose arbitrary state.

### 36. Should event handlers contain all transition rules?

No. When a reducer is the chosen state owner, handlers should dispatch events and keep the transition rules in the reducer or pure transition helpers.

### 37. Can helper functions make a reducer clearer?

Yes. Pure helpers for validation or a specific transition can reduce nested switch logic. Keep a helper close to the reducer unless moving it makes the module boundary clearer.

### 38. When should a reducer be split into smaller reducers?

When state domains have clear boundaries and splitting makes each transition easier to understand. Splitting solely to reduce line count can obscure how related state changes together.

### 39. When might a state-machine library be useful?

When the transition graph, nested or parallel modes, orchestration, or repeated workflow behavior becomes difficult to express with a plain reducer. A small finite flow often needs only a mode field and a reducer.

### 40. Does useReducer itself provide state-machine validation?

No. useReducer calls the reducer. The reducer's code defines which states and transitions are valid.

## Errors and debugging

### 41. Should a default reducer branch throw for unknown actions?

That is often a useful policy because it reveals misspellings or missing cases. A deliberately extensible reducer can choose another documented behavior.

### 42. Why should expected validation failure not share the unknown-action error path?

The UI needs to respond to invalid input by displaying errors and preserving the current step. An unknown action indicates a different, usually programming-related, problem.

### 43. What should every reducer case return?

The complete next state value. React does not automatically merge partial objects with the previous state.

### 44. How can action logs make a stateful workflow easier to debug?

If actions describe user events clearly and carry the needed payload, the sequence shows how the workflow reached its current state.

### 45. What happens if the reducer returns a partially updated object?

Any fields omitted from that object are absent from the next state. Copy fields that should remain or use a state shape tailored to the current mode.

### 46. Can invalid transitions return the same state object?

Yes, if ignoring that event is the intended policy and the state should not change. Do not mutate that object.

## Apply the concepts

### 47. A registration wizard must not reach review before name and email are valid. What models this rule?

A guard in the transition from contact to the next step. If validation fails, the reducer returns the contact step with errors.

### 48. A search interface has idle, loading, success, and error modes. How should those modes be represented?

Use one status or a discriminated state shape rather than separate booleans that can contradict each other.

### 49. A user clicks Reset during a multi-step workflow. What kind of action is appropriate?

One reset action that returns the entire workflow to its initial state in a single transition.

### 50. A success response includes result data. Where should that data enter state?

The request handler dispatches a success action with the result payload. The reducer stores it in the success state.

### 51. The reducer receives searchSucceeded after the user has started a new search. What should it check?

It should compare the result action's request ID with the ID in the current loading state. If they do not match, the result is stale and should not replace current state.

### 52. A simple disclosure has open or closed state and one toggle handler. Does it need a formal state-machine library?

Probably not. A boolean or small reducer is enough unless more states and transition rules make the workflow complex.

## Key takeaway

Define valid states, events, guards, and transitions explicitly when a workflow needs them. Keep the reducer pure, represent exclusive modes clearly, and keep side effects outside the transition logic.
