# When an Effect Is Unnecessary - Q&A

## Core idea

### 1. What is the main purpose of an Effect?

An Effect synchronizes a committed React component with an external system that React does not own, such as a browser API, subscription, timer, network service, or non-React widget.

### 2. What question should I ask before adding an Effect?

Ask what external system the component must synchronize with. If there is no external system, first consider rendering from current props and state, changing state ownership, or handling the work in an event handler.

### 3. Why are unnecessary Effects a problem?

They can duplicate information already available from props or state, show stale intermediate UI, cause extra renders, create ordering problems, and make data flow harder to trace.

### 4. Are Effects general-purpose callbacks that run after every update?

No. An Effect is not a general place for arbitrary follow-up logic. It synchronizes with an external system according to its dependencies. Interaction-specific work belongs to the event that caused that interaction.

### 5. What are the three common places for component work?

Calculate values from current inputs during rendering, handle a particular user action in its event handler, and use an Effect or a dedicated Hook to synchronize with an external system.

## Derived values and state

### 6. What is a derived value?

It is a value that can be calculated from current props or other state, such as a filtered list, total, count, label, or validation result.

### 7. Should a derived value usually be stored in state?

No. Calculate it during rendering. Keeping both the source values and a derived copy creates redundant state that can become stale.

### 8. Why is an Effect that filters a list usually unnecessary?

React can filter the current list during rendering. An Effect version can first commit a UI with the previous filtered result and then trigger another render with the new result.

### 9. How should I calculate a value from two state variables?

Use an ordinary expression during rendering. For example, derive a full name from the current first and last names instead of maintaining a third state variable.

### 10. Should I use an Effect to calculate a total or item count?

Usually no. Calculate the total or count from the current items during rendering.

### 11. What if the calculation is expensive?

Start with the direct calculation and profile the application. If repeated work is a measured performance problem, consider caching it with **useMemo** or choosing a more suitable algorithm. Correctness should not depend on the cache.

### 12. Is **useMemo** required to derive values correctly?

No. It is a performance optimization. A component must produce the correct result even if React recalculates the value.

### 13. Is every repeated calculation worth memoizing?

No. Memoization adds dependencies and complexity. Use it when there is a real, measured cost rather than as a default pattern.

### 14. Can I use an Effect to calculate an expensive value and save it in state?

That is generally the wrong design. It adds redundant state and another render. Calculate during rendering, and consider memoization if profiling shows the calculation needs caching.

### 15. Should I store both a selected item's ID and the whole selected object?

Usually not if the object can be found from the ID. Storing two representations of the same fact can let them disagree.

## Props and component state

### 16. What does it mean to mirror a prop into state?

It means initializing local state from a prop, such as setting state to the initial value of a name prop. The local state does not automatically follow later prop changes.

### 17. What is wrong with initializing state from a normal prop?

The initializer is used for the initial render only. If the parent passes a different prop later, the state still holds its old value.

### 18. How should a read-only prop be displayed?

Use the prop directly instead of copying it into local state.

### 19. What if the parent should own the editable value?

Make the component controlled: pass the current value as a prop and pass a callback for changes. Keep one clear owner for the value.

### 20. When can local state intentionally use an initial prop?

When the child owns a draft or initial setting and should intentionally ignore later changes to that prop. Names such as **initialName** or **defaultName** can make this choice clear.

### 21. How can I reset an editor when switching to a different record?

Give the editor a stable key based on the record's identity. When the key changes, React treats it as a new component instance and initializes its local state again.

### 22. What else does changing a component key reset?

It resets the state of the keyed component and its descendants. This may also discard unsaved input, focus, and other local UI state, so the reset should be intentional.

### 23. Should I change a key for every prop update?

No. Use a key when a new identity should start a new component state lifetime, such as switching from editing one profile to another. A key is too broad for adjusting one small value.

### 24. How can I reset only one piece of state when a prop changes?

First see whether the value can be derived, whether the state should live in the parent, or whether the state model can be redesigned. React also supports a conditional update to the currently rendering component's own state, but this is a specialized and harder-to-read pattern.

### 25. What rules apply to a state update during rendering?

The condition must become false after the update so rendering makes progress. Only update the currently rendering component's state. Do not update a parent, sibling, or other component during render. Prefer derivation or a keyed reset when they fit.

### 26. Is a render-time state adjustment a general replacement for Effects?

No. It is a narrow option for adjusting local React state when a prop changes and clearer alternatives do not fit. It is not a place for external work such as changing the DOM, starting a timer, or making a request.

## User actions and state ownership

### 27. Where should a purchase request caused by a Buy button go?

In the Buy button's event handler. That handler knows that the user chose to buy and can perform the associated request and notification.

### 28. Why should a form submission usually be handled by its submit handler?

The handler runs because the form was submitted and has access to the submitted values. An Effect that watches a flag or state value can lose the direct connection to that specific action and may repeat it later.

### 29. Are all network requests inappropriate in Effects?

No. An Effect can synchronize results with current rendered inputs, such as search results for the current query. For user commands such as purchasing or saving, handle the command in the relevant event handler. For app data loading, consider the framework's data-loading facilities as well.

### 30. Why can an Effect-based command accidentally repeat?

It is tied to rendering and its dependencies, not to a unique click. State restoration, remounting, or a dependency change can cause the Effect to run again. Remote commands may need server-side duplicate protection too.

### 31. Should a child use an Effect to notify its parent that its state changed?

Usually no. Call the parent's callback from the event handler that changes the value, or let the parent own and pass down the value.

### 32. What does lifting state up accomplish?

It gives related components one owner for a shared fact. This avoids maintaining separate copies that need to be synchronized later.

### 33. When should related state updates happen together?

When they are consequences of one user action, update them in that event handler. Use a reducer when one action needs a coordinated transition across several related state fields.

### 34. Why are chains of Effects for internal calculations hard to maintain?

They turn intermediate calculated values into state and make the order of updates matter. Derive display values directly or update source state together at the action that caused the change.

## External systems and Hooks

### 35. Give examples of work that can justify an Effect.

Subscribing to a browser event, controlling a non-React widget, synchronizing a document title, opening or closing an external connection, or synchronizing with a service based on current rendered inputs.

### 36. How do I tell whether an Effect really synchronizes an external system?

Identify the system outside React and describe what the setup synchronizes. If the Effect only copies one React value into another React state variable, it is likely not external synchronization.

### 37. What Hook should I consider for a mutable external store?

Consider **useSyncExternalStore**. It takes a subscribe function and a snapshot reader so React can subscribe to external changes and read the current snapshot.

### 38. Should I use **useSyncExternalStore** for ordinary local component state?

No. Prefer **useState** or **useReducer** for state that belongs to React. **useSyncExternalStore** is for integrating with data that lives outside React and changes independently.

### 39. Is reading a mutable browser value during rendering always enough?

No. If the browser value can change independently of React, the component needs a way to subscribe and be rendered again. A suitable external-store Hook can provide that integration.

### 40. What happens to an Effect when its dependencies change?

React runs the previous cleanup, if any, then runs the new setup to synchronize with the latest values. This is why dependencies should describe the reactive values the synchronization uses.

### 41. Does removing unnecessary Effects mean I should remove all Effects?

No. Keep Effects for external synchronization. The goal is to use the right mechanism for each cause of work.

## Troubleshooting and decisions

### 42. What should I try if an Effect calculates a display value?

Delete the state that only stores that calculated value and calculate it from current props and state during rendering.

### 43. What should I try if an Effect resets state whenever a record prop changes?

Choose whether the value should be controlled by the parent, derived from current props, intentionally initialized only once, or reset with a key when the whole editor changes identity.

### 44. What should I try if an Effect runs after a button click?

Move the action into the button or form handler if the work is caused by that particular interaction.

### 45. What if several components each keep a synchronized copy of one value?

Choose a single owner. Often this means lifting the state to their nearest common parent and passing the current value and callbacks down.

### 46. Does removing an Effect always reduce the number of component renders?

Not necessarily in every situation, but it often removes a follow-up state update and avoids committing an intermediate stale result. The primary benefit is simpler, more direct data flow.

### 47. Can render calculations perform side effects?

No. Rendering must remain pure: calculate the UI from inputs. Do not mutate external systems, start timers, or make requests during rendering.

### 48. What is a useful final checklist before keeping an Effect?

Name the external system, identify the values it synchronizes, make the dependencies accurate, and provide cleanup for resources that must stop. If no external system exists, solve the problem with rendering, state ownership, or an event handler.

## Quick recap

- Derive values from current props and state during rendering.
- Keep source state, not a synchronized copy of its calculated result.
- Use **useMemo** only for measured performance needs.
- Put event-specific commands in their event handlers.
- Use one owner for shared state.
- Use a stable key when a new component identity should reset its state.
- Treat render-time state adjustment as a narrow advanced pattern.
- Use Effects and external-store Hooks for their intended external synchronization jobs.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="054-effect-cleanup-and-resource-lifetimes.md">054 - Effect Cleanup and Resource Lifetimes - Q&amp;A</a></td><td><a href="../lessons/055-when-an-effect-is-unnecessary.md">Lesson</a></td><td><a href="../examples/055-when-an-effect-is-unnecessary.md">Example</a></td><td><a href="056-useeffectevent.md">056 - useEffectEvent and Non-Reactive Event Logic - Q&amp;A</a></td></tr>
  </tbody>
</table>
