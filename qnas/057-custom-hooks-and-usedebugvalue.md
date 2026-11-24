# Custom Hooks and useDebugValue — Q&A

## Custom Hook basics

### 1. What is a custom Hook?

A custom Hook is a JavaScript function that uses one or more built-in or custom React Hooks to package reusable stateful behavior.

### 2. How is a custom Hook different from a component?

A component returns UI for React to render. A custom Hook returns values or actions to the component that calls it; it does not define a separate component or render its own JSX tree.

### 3. How should a custom Hook be named?

Its name should begin with **use** followed by a capital letter, such as **useOnlineStatus** or **useChatRoom**.

### 4. Why does the Hook name need the **use** prefix?

It signals that the function may call Hooks, so it must be called at the top level of a component or another custom Hook. React's Hooks lint rules use this naming convention too.

### 5. Should a regular helper function that does not call Hooks start with **use**?

Usually no. Give **use** names to functions intended to follow Hook rules. A normal formatter or calculation helper can use names such as **getSortedItems** or **formatStatus** and can be called wherever ordinary functions are allowed.

### 6. Can a custom Hook return any type of value?

Yes. It can return a value, object, array, callback, or nothing, depending on what makes its calling interface clear.

### 7. Can a custom Hook call another custom Hook?

Yes. Custom Hooks can compose built-in and other custom Hooks, while following the Rules of Hooks.

### 8. Can custom Hooks contain Effects?

Yes. A Hook may own an Effect when it encapsulates a specific external synchronization. Its inputs and Effect dependencies must still accurately describe that synchronization.

## State and reuse

### 9. Does calling the same custom Hook in two components share one state variable?

No. Each call has independent Hook state associated with its calling component instance.

### 10. What does a custom Hook share?

It shares reusable logic and behavior. It does not automatically share the state created by each invocation.

### 11. How can multiple components share one React-owned state value?

Give the state one owner, such as a common parent, and pass the value and actions down. Context can distribute shared values to descendants when appropriate.

### 12. Should I call the same custom Hook in each child to share data?

No. That creates independent state for each call. If the components need the same value, lift the state or use an appropriate shared store.

### 13. When should an external-store Hook be used?

When the source of truth lives outside React and changes independently, such as a browser API or a third-party store. For ordinary React-owned state, prefer **useState**, **useReducer**, or context.

### 14. Do custom Hooks make external work happen during rendering?

No. A custom Hook runs as part of rendering, so its render-time code must be pure. Start external synchronization from an Effect or a purpose-built Hook.

### 15. Does extracting an Effect into a custom Hook change when it runs?

No. Its synchronization and dependency behavior should remain the same. The abstraction hides implementation details behind a clearer capability name; it should not change which values cause re-synchronization.

## Rules of Hooks and reactivity

### 16. Where can I call a custom Hook?

At the top level of a React component or another custom Hook.

### 17. Can I call a Hook conditionally?

No. Call it unconditionally in a consistent order on every render. Pass a condition as an argument and handle it inside the Hook when needed.

### 18. Can a custom Hook call another Hook conditionally?

No. The same Rules of Hooks apply inside custom Hooks.

### 19. How should a Hook handle an optional behavior?

Call the Hook unconditionally and pass the option into it. The Hook can conditionally start or stop work inside an Effect while preserving the order of Hook calls.

### 20. Are custom Hooks pure?

Their render-time logic should be pure like component render logic. An Effect inside the Hook may synchronize with an external system after rendering, with correct cleanup.

### 21. What happens when a custom Hook receives a new prop or state value?

It receives the current value on the caller's next render. If the Hook uses that value in an Effect, the Effect must include it as a dependency when it determines the synchronization.

### 22. Does wrapping an Effect in a custom Hook hide its dependencies from React?

It can hide them from the caller, but it does not remove the requirement to declare them inside the Hook. Keep dependency lists accurate and do not suppress the linter to force a lifecycle.

### 23. Should a Hook accept a large set of unrelated options?

Usually not. A focused interface makes the Hook's purpose and synchronization inputs understandable. Many unrelated options can signal that the Hook combines multiple responsibilities.

## Choosing good abstractions

### 24. Should every duplicated line become a custom Hook?

No. Small amounts of duplication can be clearer than an abstraction that has no useful name or behavior boundary.

### 25. What is a good reason to extract a custom Hook?

A coherent stateful capability appears in multiple places, its implementation is complex enough to hide, or a descriptive name makes the caller's intent clearer.

### 26. What makes a custom Hook name useful?

It describes a concrete capability, such as **useChatRoom**, **useOnlineStatus**, or **useIntersectionObserver**, rather than an implementation timing detail.

### 27. Why are generic lifecycle Hooks like **useMount** discouraged?

They wrap an Effect without expressing what is being synchronized. They can hide reactive inputs and prevent the normal dependency linter from identifying missing dependencies.

### 28. How should I start if I think an Effect can be extracted?

Write the direct Effect first with its real dependencies and cleanup. Once its purpose is clear, extract a focused Hook named after the capability if that improves reuse or readability.

### 29. Should a custom Hook wrap one **useState** call just to avoid a repeated declaration?

Not automatically. Extract it when the Hook represents a meaningful reusable behavior or a useful abstraction, not merely to move one line.

### 30. How do I share an event handler with an Effect-owning custom Hook?

Pass it as an option if the Hook needs it. If changing the caller's function identity should not restart the synchronization and the callback is an event fired by the Effect, the Hook can use **useEffectEvent** internally, following that API's restrictions.

### 31. Can a custom Hook return an Effect Event so callers can use it?

No. An Effect Event must stay local to the component or custom Hook that owns its Effect. Do not pass it to another component or Hook. For its full restrictions, follow the useEffectEvent lesson.

## useDebugValue

### 32. What does **useDebugValue** do?

It adds a readable debug label for a custom Hook in React DevTools.

### 33. Does **useDebugValue** render a label in the application UI?

No. It is for inspection in React DevTools. Use normal rendered markup for user-facing status and messages.

### 34. What does **useDebugValue** return?

Nothing. The custom Hook should separately return whatever values its callers need.

### 35. Where should I call **useDebugValue**?

At the top level of a custom Hook, following the Rules of Hooks.

### 36. Should every custom Hook call **useDebugValue**?

No. Use it when a label helps inspect a complex Hook or a shared library Hook. Simple Hooks often need no extra label.

### 37. What is the optional formatter argument for?

It transforms the debug value into a readable label when React DevTools inspects the Hook. This can defer expensive formatting until inspection instead of doing it on every render.

### 38. What arguments does the formatter receive?

It receives the value passed as the first argument to **useDebugValue** and returns the value DevTools should display.

### 39. Can the formatter call Hooks or cause side effects?

No. Keep it a pure formatter. Do not use it to change state, synchronize with a service, or call Hooks.

### 40. Should a debug formatter be created inline?

It can be, but a named pure formatter can make the code easier to understand. Formatting is deferred for inspection; do not rely on the formatter for application behavior.

## Review and troubleshooting

### 41. Two components call **useStopwatch**, but only one timer should exist. What is wrong?

Each Hook call owns independent state and timer logic. Move the Hook call to a common owner and pass the same values and actions to both components, or choose a shared store if that better fits the requirement.

### 42. The UI only needs to show a value derived from props. Is a custom Hook necessary?

Not necessarily. A regular calculation in the component may be clearer. A Hook is most useful when it encapsulates a reusable stateful behavior or other Hook calls.

### 43. A custom Hook behaves differently when an input changes. What should I inspect?

Check how the input flows through the Hook and whether its Effects declare every reactive value that should update the external synchronization. Also check that a generic wrapper has not hidden the dependency list.

### 44. The Hook contains several unrelated Effects and state concerns. What should I consider?

Split it into focused Hooks or keep some logic in the component. Each Hook should have a clear capability and understandable inputs and outputs.

### 45. What is the short rule for deciding whether to create a custom Hook?

Extract a Hook when a meaningful, reusable stateful capability gets a clearer name and interface. Avoid abstraction that merely relocates code or hides dependencies.

## Quick recap

- Custom Hooks package logic; each call owns independent state.
- Lift state or choose a shared store when components need one common value.
- Hook naming and top-level call rules apply inside custom Hooks.
- Reactive inputs and Effect dependencies stay meaningful after extraction.
- Prefer focused capability Hooks over generic lifecycle wrappers.
- **useDebugValue** adds optional DevTools diagnostics, not user-facing UI.
