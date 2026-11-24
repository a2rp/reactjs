# Designing Context Providers and Avoiding Overuse — Q&A

## Choosing context

### 1. When should data stay in props?

Keep a value in props when it is used by a nearby child, when explicit inputs make a component reusable, or when forwarding through the few intermediate components remains understandable.

### 2. What is prop drilling?

It is repeatedly passing data through intermediate components that do not use it, only so a distant descendant can receive it. Context can reduce this repetition when many descendants need the value.

### 3. Does prop drilling mean props are poorly designed?

No. Explicit prop flow is often useful and easy to trace. A few layers of props are not automatically a problem.

### 4. How can composition reduce prop forwarding?

A layout component can accept already-created UI through children. This lets the component that owns the data render its consumer inside the layout without the layout receiving or forwarding that data.

### 5. When is context a good fit?

Use it when multiple descendants in a subtree need the same value, intermediate components would only forward it, and making the dependency available to that subtree is clearer than threading props.

### 6. Should every value needed by more than one component be put in context?

Not necessarily. Nearby components can share data through props and a common owner. Context is most useful when the value needs to reach distant consumers or many branches.

## Provider scope and ownership

### 7. Where should a provider usually be placed?

Around the smallest subtree that needs the value, near the component that owns and updates it. A truly app-wide value, such as a site theme, may justify a provider near the root.

### 8. Why is provider placement part of the API design?

The provider's location determines which descendants can read the value and which parts of the UI share its updates. It also communicates the intended scope of that data.

### 9. What is wrong with putting every provider at the application root?

It can expose values to unrelated branches, make ownership unclear, and turn root composition into a broad collection of global-looking dependencies. Put providers higher only when their scope requires it.

### 10. Does context itself own state?

No. A provider component, reducer, or another state owner owns changing values. Context makes the current value and actions available to descendants.

### 11. Is a module-level mutable object a good context replacement?

No. Mutating a shared object outside React's state model does not reliably schedule React updates. Keep changing values in state or a reducer and pass them through a provider.

## Designing a context API

### 12. Why should a context object be created outside a component?

The provider and consumer must refer to the same context object. Creating it during render creates a new object identity and breaks that shared channel.

### 13. What does a feature provider component do?

It owns or receives feature state, provides the values to a subtree, and defines the scope where those values are available. It can hide the context wiring from the rest of the app.

### 14. What does a custom Hook such as useTasks add?

It hides direct context access behind a feature-specific API. It can also detect a missing provider and throw an error with a useful message.

### 15. When is a sentinel default useful?

Use a sentinel such as undefined or null when a consumer is only valid under a required provider. A custom Hook can check for that sentinel and explain which provider is missing.

### 16. When is a meaningful default value appropriate?

Use one when the component can behave correctly without an explicit provider, such as a theme that should fall back to light. Remember that a default is static; it is not live provider state.

### 17. Is an AppProviders wrapper always necessary?

No. It can make an app root easier to read when several genuinely app-wide providers need composition. Keep it small and avoid using it as a catch-all for unrelated state.

## State and dispatch contexts

### 18. Why might state and dispatch be provided through separate contexts?

A component that only dispatches actions need not subscribe to the changing state value. Splitting the contexts can make dependencies more precise and avoid context-driven updates for dispatch-only consumers.

### 19. Is the useReducer dispatch function stable?

React documents the dispatch function returned by useReducer as having stable identity. That makes it suitable for a dispatch context without creating a new function on each reducer update.

### 20. Do separate contexts prevent all re-renders?

No. They prevent a component from receiving updates from a context it does not consume. A component may still render because its parent renders with changed props or children, or for another reason.

### 21. Should every reducer-backed context split state and dispatch?

No. Split them when it clarifies read/write dependencies or improves meaningful update behavior. A small feature may be simpler with one context or ordinary props.

### 22. Does context define the state transitions?

No. A reducer or state setter defines how updates happen. Context only makes state or actions available to descendants.

### 23. Can a provider expose dispatch without exposing state?

Yes. If descendants only need to request actions and do not need to render current state, provide the dispatch function alone.

## Updates and performance

### 24. How are context updates detected?

React compares the previous and next provider values with Object.is. A different value causes consumers of that context below the provider to receive the update.

### 25. Why can an inline object literal cause consumer updates?

Each evaluation creates a new object identity. Even if its fields hold the same data, Object.is treats two different object references as different values.

### 26. Should every provider value be wrapped in useMemo?

No. Start with the simplest clear provider. Memoize object or function values when a provider re-renders often and avoiding unchanged value identity has meaningful benefit.

### 27. Does memo block a consumer from receiving a new context value?

No. A component that reads a changed context still receives the update when wrapped in memo. Memo may skip updates based on equal props, but it does not block its context subscription.

### 28. How can a consumer that needs only part of a large context be structured?

Split the context by ownership or update needs, or have a small outer component read context and pass the needed value as a prop to a child that can be memoized. Choose based on actual component boundaries and measured cost.

### 29. Why might separate contexts reduce unrelated updates?

A consumer subscribes to specific context objects. If unrelated values are split, changing one context does not change the value of another context.

### 30. Does memoization make a broad context design good?

No. Memoization can stabilize references, but it does not fix unclear ownership, excessive scope, or unrelated values bundled into one API.

## Safety and architecture

### 31. Is context a global store?

No. It is scoped to a provider's descendant branch. Other branches can have different providers or no provider at all.

### 32. Is client context an authorization boundary?

No. It may control which UI is displayed, but trusted server code or services must enforce access permissions.

### 33. Should remotely fetched data always live in context?

No. Context can distribute data to descendants, but it is not a request cache or server-state manager. Choose a data-fetching approach based on the application's cache, loading, and synchronization needs.

### 34. How should local input state be treated?

Keep a field's temporary editing state in the input or form that owns it unless multiple distant components need it. Avoid lifting every local value into shared context.

### 35. What is the difference between context scope and nested providers?

A provider gives a value to its descendants. A nested provider for the same context overrides that value only in its own subtree; once outside that subtree, the outer value applies again.

### 36. What is a sensible process for introducing context?

First identify the value's owner and consumers. Try direct props or composition. If many distant descendants need the value, add a provider around the smallest shared subtree, expose a focused custom Hook, and split values only when ownership or update behavior warrants it.

## Short recap

- Use context for shared subtree values, not automatically for every prop.
- Provider placement defines ownership scope and which components can read the value.
- A feature provider and custom Hook can make the context API clear and catch missing providers.
- State and dispatch contexts can be separated when that makes subscriptions more focused.
- Context distributes values; state and reducers own transitions.
- Optimize value identity when it matters, and keep unrelated state out of broad contexts.
