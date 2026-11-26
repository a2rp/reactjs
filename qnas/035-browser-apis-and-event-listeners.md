# 035. Working with browser APIs and external event listeners - Q&A

## Choosing the right event boundary

### 1. When should I use a React event prop?

Use a React event prop such as `onClick`, `onChange`, or `onSubmit` for an interaction on an element rendered by React. The event handler runs because a person or browser interaction triggered that element.

### 2. When is a native browser event listener appropriate?

Use a native listener when the event source is a browser object outside the React element tree, such as the window's resize event, or when a subscription must remain active while a component is displayed.

### 3. Should I add a native click listener to a React button?

Usually no. Put the behavior in the button's React event prop. Registering both a React handler and a native listener for the same click can make the action happen twice.

### 4. Is an Effect just another name for an event handler?

No. An event handler responds to a particular interaction. An Effect keeps a component synchronized with an external system because the component is present or because a reactive value changed.

### 5. Should a purchase request run in an Effect?

No. If the request is caused by a person's click, perform it in that click handler. An Effect can run again as the component synchronizes or remounts, so it is the wrong place for a one-time action caused by that click.

### 6. Can I add a browser listener during rendering?

No. Rendering should calculate the interface without setting up external resources. An Effect runs after React commits and gives the component a lifecycle for the subscription.

### 7. Does every interaction with the browser need an Effect?

No. A user's click, typing, or form submission normally belongs in a React event handler. Use an Effect for ongoing synchronization with an external API, not as a general place to move event logic.

## Effect setup and cleanup

### 8. What does setup do for an event subscription?

Setup creates the listener callback and registers it with the correct event target using addEventListener.

### 9. What should cleanup do?

Cleanup should undo setup. For an event subscription, call removeEventListener with the same event type, callback function, and capture setting.

### 10. When does React run Effect cleanup?

React runs cleanup before the Effect sets up again with changed dependencies, and when the component unmounts. In development Strict Mode, React also runs an extra setup-and-cleanup cycle to check the Effect.

### 11. Why must an Effect remove its event listener?

Without cleanup, the callback can keep running after the component no longer needs it. It can also leave duplicate subscriptions after a dependency change or remount.

### 12. Why must addEventListener and removeEventListener use the same callback object?

The browser matches the listener by event type, callback identity, and capture setting. Two separately created arrow functions have different identities, even if their source text looks identical.

### 13. Is this cleanup correct?

~~~js
window.addEventListener('resize', () => console.log(window.innerWidth));
window.removeEventListener('resize', () => console.log(window.innerWidth));
~~~

No. Each arrow expression creates a new function. Define one named or stored function in the Effect and pass that same function to both methods.

### 14. Does removeEventListener need the same options object?

It does not need the identical options object, but the capture setting must match. Keeping options consistent makes the code easier to review. Options such as passive and once affect listener behavior; once removes the listener after its first call.

### 15. Can AbortController clean up a listener?

Yes. Where supported by the API, pass an AbortSignal in the listener options and abort the controller during cleanup. For introductory code, removeEventListener is a clear way to show exactly which subscription is removed.

### 16. What does Strict Mode do to an Effect in development?

It may run setup, cleanup, and setup again when the component first mounts. A properly paired subscription ends with one active listener. This extra cycle helps expose missing cleanup.

### 17. Should I prevent the extra Strict Mode setup with a ref?

No. A flag hides the extra setup without releasing the listener. Implement cleanup so the Effect can safely stop and restart.

### 18. Does an empty dependency array mean the Effect will literally run only once?

It means the Effect has no reactive dependencies to watch, so it runs when the component is added and cleans up when it is removed. Development Strict Mode may still check it with an extra setup-and-cleanup cycle.

## Dependencies and callback values

### 19. What belongs in an Effect dependency list?

Include reactive values read by the Effect's setup or cleanup, such as props, state, and component-local variables or functions. React compares dependencies between renders and resynchronizes when one changes.

### 20. What happens if a listener reads a prop but the Effect omits it from dependencies?

The registered callback can keep the value from the render that created it. It may respond with an outdated prop or state value. Include the dependency or restructure the code so the Effect no longer reads that value.

### 21. If a callback prop changes, what should happen to the listener?

With that callback prop in the dependency list, React cleans up the listener that captured the previous callback and registers a new one with the current callback.

### 22. Do state setter functions need to be dependencies?

React state setters have stable identities, so they do not need to be added solely because the Effect calls them. Other reactive values used by the Effect still belong in its dependency list.

### 23. How can a timer update state without capturing an old count?

Use a functional state update when the next value depends on the previous value, for example `setCount((count) => count + 1)`. This avoids reading count from the closure and can keep it out of that Effect's dependencies.

### 24. Should I omit a dependency to stop an Effect from re-running?

No. Omitting a value can make the callback stale. If the Effect re-runs more often than intended, reconsider which values it reads or how the component and callback are structured.

## Browser APIs and server rendering

### 25. How do I show the current window dimensions?

Start with a neutral state, read window.innerWidth and window.innerHeight inside an Effect, then update that state from a window resize listener. Remove the listener in cleanup.

### 26. Why is the initial viewport state often null?

The component has not read the browser yet, and the same component may be rendered on a server where there is no window. A neutral value avoids accessing a browser-only global during render.

### 27. Do Effects run during server rendering?

No. Effects run in the browser after React commits. That makes an Effect an appropriate place to read a browser API, but it does not make a direct window read during rendering safe.

### 28. What is a hydration mismatch in this context?

If server-rendered markup and the browser's first render produce different output, hydration may not match the existing markup. Use a compatible initial value and read the browser-only value in an Effect after hydration.

### 29. How do I listen for a media query changing?

Call window.matchMedia with the query, read its matches property for the current value, subscribe to its change event, and remove that listener during cleanup.

### 30. Should prefers-reduced-motion only be handled in JavaScript?

No. If the preference affects visual motion, CSS can honor it directly. JavaScript subscription is useful when the interface also needs to change behavior or display the current preference.

### 31. How do I clean up a timer?

Save the identifier returned by setTimeout or setInterval. In cleanup, pass it to clearTimeout or clearInterval respectively.

### 32. How do I clean up an IntersectionObserver?

Call disconnect on the observer during cleanup. If the Effect observes a particular DOM node, make sure setup observes the intended node and cleanup releases the observer.

### 33. Should I use window resize to detect an element's size?

Usually not. The viewport and an individual element are different targets. Use an element sizing observer when the behavior depends on that element's dimensions.

## Scope, performance, and debugging

### 34. Should every component register its own window listener?

Only if each component independently needs that subscription. If many components need the same browser value, consider a shared subscription abstraction or a custom Hook rather than duplicating unrelated global listeners.

### 35. Is a global keyboard listener always better than an onKeyDown prop?

No. Use a local React event prop when the focused element should handle the key. A window or document listener is appropriate only when the behavior must be observed beyond one rendered element and its active lifetime is clearly defined.

### 36. What should I consider for scroll and pointer events?

These events may fire frequently. Keep callbacks small, update state only when the displayed value needs to change, and consider an observer or scheduled work when appropriate.

### 37. Why can a resize listener appear to run multiple times?

The listener might have been registered more than once because setup has no matching cleanup, the Effect re-runs after dependencies change, or both React and native handlers are attached. Check each setup against its cleanup and verify that there is a single event owner.

### 38. How can I tell whether cleanup is working?

Unmount the component or change an Effect dependency and confirm the previous listener no longer responds. In development Strict Mode, verify the setup-cleanup-setup cycle leaves only one active subscription.

### 39. Can an event listener update React state?

Yes. A listener can call a state setter. React then schedules a render with the new state. The listener should update state only for information the interface needs, and the subscription still needs cleanup.

### 40. Does calling setState in cleanup make sense?

Usually cleanup should release or undo the external resource, not update state for a component that is being removed. If a dependency change requires a new displayed value, let the next setup or the new render produce it.

### 41. How do I choose between a scroll event and IntersectionObserver?

Use a scroll listener when you need scroll position or custom scroll calculations. Use IntersectionObserver when the actual question is whether an element intersects a viewport or another root. The observer can report visibility without manually processing every scroll event.

### 42. What is the first debugging checklist for a browser subscription?

Confirm that setup is inside an Effect, cleanup removes the exact callback, all reactive dependencies are included, browser globals are accessed in a browser-safe place, and the listener is attached to the narrowest suitable event source.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="034-file-inputs-and-upload-flows.md">034 - File inputs and upload flows - Q&amp;A</a></td><td><a href="../lessons/035-browser-apis-and-event-listeners.md">Lesson</a></td><td><a href="../examples/035-browser-apis-and-event-listeners.md">Example</a></td><td><a href="036-accessible-interactive-form-project.md">036 - Build an accessible interactive form feature - Q&amp;A</a></td></tr>
  </tbody>
</table>
