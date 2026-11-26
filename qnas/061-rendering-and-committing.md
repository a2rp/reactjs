# Rendering, Committing, and Updating the UI - Q&A

## The update sequence

### 1. What are the main steps in a React screen update?

A render is triggered, React calls components to calculate the next UI, React commits the necessary DOM changes, and the browser paints the result.

### 2. What does “rendering a component” mean in React?

It means React calls the component function to calculate the JSX for the current props, state, and context. It does not mean React has already changed the browser DOM.

### 3. What does the commit phase do?

React applies the required changes to the browser DOM to bring it in line with the calculated UI. On an initial mount, it creates the needed DOM nodes.

### 4. What is browser paint?

It is when the browser draws the updated DOM as pixels on screen. It happens after the relevant DOM commit, although browser scheduling determines exactly when paint occurs.

### 5. Does every component render cause a DOM update?

No. React can call a component, compare its result with the previous tree, and find that no DOM changes are needed.

### 6. Why does React calculate a tree before changing the DOM?

The calculation lets React determine what the interface should be for the current inputs and which host changes are actually necessary. Rendering is a description; commit applies the changes.

### 7. What are common reasons React starts rendering?

The initial root render, a state update, a parent render that supplies new inputs, or a context update for a consumer can cause relevant work. An update is a request to recalculate, not a command to rewrite the whole DOM.

### 8. Does calling setState synchronously change the state variable in the current function?

No. The variable is a snapshot for that render. A setter queues or requests a future render; the current handler continues to see its existing snapshot.

### 9. Why do three calls to setCount(count + 1) often add only one?

All three use the same **count** snapshot and request the same next value. They do not read the result of the previous setter call.

### 10. How do I apply several state updates in sequence?

Use updater functions such as **setCount(current => current + 1)**. React applies each updater to the result of the previous queued update.

### 11. What does state batching mean?

React can group related updates and render after the current event handling work. Code should describe the intended update rather than rely on immediate synchronous changes to state variables.

## Render purity

### 12. What does it mean for a component render to be pure?

For the same props, state, and context, it returns the same JSX and does not change pre-existing values or perform side effects during rendering.

### 13. Can a component mutate props while rendering?

No. Props are inputs and should be treated as read-only. Make a new value if a calculation needs to transform them.

### 14. Can I mutate a module-level variable from inside a component render?

No. It creates shared, order-dependent behavior and can produce different output when React calls components again or in another order.

### 15. Is it ever okay to mutate an array created during the current render?

Yes, local mutation of a newly created value is okay if it cannot affect existing data or another component and the output remains predictable.

### 16. Why must render be safe to repeat?

React may call components more than once, pause or restart work, or discard an uncommitted render. A render with side effects could then repeat or leave changes that do not match the committed UI.

### 17. Does Strict Mode call components twice in development?

It may call component functions and other pure calculations more than once in development to expose accidental impurities. This is a development check, not a guarantee that production commits duplicate DOM changes.

### 18. Should I put a network request or timer directly in a component body?

No. The component body runs during render, which may repeat. Start user-triggered work in an event handler, or synchronize external systems with an Effect and cleanup when appropriate.

### 19. Is console logging during render a good way to count commits?

No. A render log shows that React called the component, not that it committed a DOM change. Strict Mode and restarted work can make the count differ. Use React DevTools or a commit-aware profiler for performance analysis.

## DOM reconciliation and ownership

### 20. Does React rebuild the entire DOM on each render?

No. React compares the next tree with the previous one and applies necessary changes. Unchanged DOM nodes can remain in place.

### 21. What happens if only text in a heading changes?

React can update the heading’s text while leaving adjacent, unchanged nodes such as an input in place.

### 22. What if the render result is equivalent to the previous result?

React may have no DOM updates to commit, even though component code ran.

### 23. What helps React match list items between renders?

Stable keys associated with the data help React identify siblings when items are inserted, removed, or reordered. Use IDs from the data, not random values created during render.

### 24. Can I directly remove a DOM child that React rendered?

Avoid it. React owns the DOM inside its root and may later try to update a node that your code removed. Describe structural changes with state so React can commit them consistently.

### 25. Does React own all DOM on the page?

It manages the DOM inside the root or roots it controls. Other page regions may be managed by separate systems, but each integration should have a clear ownership boundary.

### 26. If a component re-renders, will an uncontrolled input always lose what the user typed?

No. If React reuses the same input node and does not update its value, the user’s DOM value can remain. A controlled input is different: React renders its value from state.

## Roots and mounting

### 27. What does createRoot do?

It creates a React root for a browser DOM container. Call **root.render(...)** to display the React tree inside it.

### 28. How many roots does a fully React-built app usually need?

Usually one root for the application. Put nested components under the root component instead of creating a separate root for each component.

### 29. What does the first root.render call do to existing container contents?

It clears existing HTML inside the root container and lets React manage that content. If that HTML was already rendered by React on the server, use hydration instead.

### 30. What is the difference between createRoot and hydrateRoot?

**createRoot** starts a client-rendered React tree in a container. **hydrateRoot** attaches React behavior to HTML that React already rendered on the server.

### 31. Can I use createRoot for server-rendered HTML?

No. Use **hydrateRoot** so React can attach to the existing markup instead of clearing and recreating it.

### 32. Should I call root.render again for every state update?

Usually no. Create and render the root at app startup; use state setters, props, and context to update the tree. Re-rendering the root manually is mainly for cases that update the root-level React node itself.

### 33. When should root.unmount be called?

When an outside system removes a DOM container that contains a React root, call **root.unmount()** so React can detach handlers and clean up the tree. Routine updates in a normal React app do not require manually unmounting its main root.

### 34. Should I use another root to put a modal elsewhere in the DOM?

Usually no. For a React subtree that needs to render elsewhere in the DOM tree, use a portal so it remains part of the existing React tree.

## Putting phases together

### 35. What happens after clicking a state-updating button?

The event handler runs, a state setter requests an update, React calls components to calculate the next UI, React commits the necessary DOM changes, and the browser paints the updated result.

### 36. Does JSX creation itself insert an element into the page?

No. JSX describes React elements. React uses that description to determine what should be committed to the DOM.

### 37. Do refs update during render?

DOM refs are attached or updated during commit. Do not read a DOM ref during render expecting it to represent the next DOM; it may still be null or point to the previously committed node.

### 38. When do Effects run relative to rendering?

Effects are for synchronizing with external systems after React has rendered and committed. **useLayoutEffect** and **useEffect** have different timing; do not use either as a substitute for pure render calculations.

### 39. Does a render always get committed?

No. React may determine there is no visible host change to apply, or abandon a render that was not committed. This is another reason render must not perform side effects.

### 40. What is the simplest phase mental model?

An update is requested, components calculate the next UI, React applies needed DOM changes, and the browser paints. Keep calculation, user actions, DOM commits, and external synchronization in their proper roles.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="060-memoization-useid-usememo-usecallback.md">060 - Memoization, useId, useMemo, and useCallback - Q&amp;A</a></td><td><a href="../lessons/061-rendering-and-committing.md">Lesson</a></td><td><a href="../examples/061-rendering-and-committing.md">Example</a></td><td><a href="062-reconciliation-and-component-identity.md">062 - Reconciliation, Component Identity, and Keys - Q&amp;A</a></td></tr>
  </tbody>
</table>
