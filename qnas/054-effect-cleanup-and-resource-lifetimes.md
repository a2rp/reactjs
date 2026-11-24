# Effect Cleanup and Resource Lifetimes — Q&A

## Cleanup and Effect lifetimes

### 1. What is an Effect cleanup function?

It is the function returned by an Effect's setup. React calls it to stop, release, or reverse the external work started by that setup.

### 2. When does React call cleanup?

React calls the previous cleanup before running setup again after a dependency changes. React also calls the last cleanup when the component is removed. In development Strict Mode, React adds a setup-cleanup check before the real setup.

### 3. Is cleanup only for component unmount?

No. Cleanup marks the end of one synchronization lifetime. It can run while the component stays mounted when dependencies change, so React can replace the old external resource with one using current values.

### 4. Which values does cleanup see?

Cleanup closes over the values from the setup that returned it. That lets it release the exact resource created for those values, even if a later render has different props or state.

### 5. Does every Effect need cleanup?

No. Cleanup is needed when setup starts or acquires work that should stop, be released, or be undone. A log or a one-way operation that creates no continuing resource may not need cleanup.

### 6. What does it mean for cleanup to mirror setup?

Each setup action should have a corresponding release or reverse action: connect/disconnect, subscribe/unsubscribe, start/clear, observe/unobserve, or open/close. Cleanup should focus on releasing the work its setup owns.

### 7. Should cleanup close the current resource from a shared variable?

Usually no. A later setup may have overwritten that variable, causing an older cleanup to release the wrong resource. Keep the resource in the setup's local scope and let its cleanup close over that exact instance.

### 8. Does cleanup reset all of the component's state?

No. Cleanup releases external work. It should not be a general mechanism for resetting unrelated React state. State transitions belong in React state logic and rendering.

## Common resources

### 9. How do I clean up an event listener?

Remove the same event type and function reference that setup registered. Keep the handler in setup's scope and use matching capture behavior when removing it.

### 10. Why does creating a new handler function in cleanup fail?

The new function is a different reference from the function registered by setup. The browser cannot use it to remove the original listener.

### 11. How do I clean up a timer?

Save the identifier returned by setInterval or setTimeout in setup, then pass that same identifier to clearInterval or clearTimeout in cleanup.

### 12. Should a timer callback use the previous render's state?

If it calculates the next state from the current state, use a functional state updater. This lets the callback access the latest queued state without closing over a stale state snapshot.

### 13. How do I clean up a subscription?

Call the unsubscribe function returned by the subscription API, or call the API's documented unsubscribe method with the same subscription or handler used by setup.

### 14. Should cleanup disconnect a ResizeObserver or only unobserve one target?

If the observer belongs only to the Effect, disconnecting it is usually simplest. If the observer remains active for other targets, unobserve the target owned by this Effect. Follow the observer's ownership and API contract.

### 15. Why does a dialog opened with showModal need cleanup?

A modal dialog should be closed before another setup opens it. Calling showModal, then close, then showModal is repeatable and behaves like one open dialog to the user.

### 16. How should an animation Effect clean up?

Cancel the animation or restore its initial visual state using the animation library's documented API. Setup-cleanup-setup should not leave duplicate animations or a partially changed element.

### 17. How do I clean up a third-party widget?

Use that widget's documented destroy, dispose, stop, or remove method. Removing a React component does not automatically dispose objects created outside React.

## Asynchronous work

### 18. How should cleanup handle a fetch tied to a changing identifier?

Abort the request when possible and prevent a result from an obsolete setup from updating current UI. A local active flag can ignore a result when cancellation is unavailable or incomplete.

### 19. Does aborting a request guarantee it never reached the server?

No. The request may already have reached or been processed by the remote system. Abort stops or limits local work; it is not a rollback of a remote operation.

### 20. Is ignoring a response the same as cancelling the request?

No. An ignore flag prevents a result from affecting the current UI, but the underlying request may continue. Use abort when supported, and still ensure obsolete results cannot overwrite current state.

### 21. Can cleanup undo a completed POST request?

No. Cleanup cannot reverse a server-side action that has already completed. Trigger user actions such as submitting an order in the event handler, and use server-side idempotency or duplicate-request protection where needed.

### 22. Should a cleanup function be async?

Do not make cleanup async and expect React to wait for it. React calls the synchronous cleanup function but does not await a Promise from it.

### 23. What if a resource shuts down asynchronously?

Start its shutdown from synchronous cleanup and handle completion or failure according to its API. If the next setup must wait until shutdown finishes, implement the serialization required by that external system; React's Effect cleanup does not provide it.

### 24. What if a callback arrives after unsubscribe or disconnect?

First check the external API's documented guarantee. If it can deliver queued callbacks after cleanup, guard those callbacks with a local active flag or other cancellation mechanism. Avoid adding guards when the resource already guarantees callbacks stop.

## Strict Mode and repeated setup

### 25. Why does development show setup, cleanup, then setup?

Strict Mode intentionally performs an extra setup-and-cleanup cycle to expose missing cleanup. This behavior is development-only and helps check that one active resource remains after setup is repeated.

### 26. Should I prevent the second setup with a ref?

No. A ref guard hides the diagnostic and leaves resources unreleased when dependencies change or the component is removed and mounted again. Make each setup pair clean up correctly.

### 27. Does the extra Strict Mode cycle mean there are duplicate production resources?

No. It is a development check. With correct cleanup, the first resource is released before the next setup creates its replacement.

### 28. How can the user-facing behavior remain the same after setup runs twice?

Make cleanup fully release the first setup's resource before the next setup starts. The user should not be able to tell the difference between a single setup and setup-cleanup-setup.

### 29. Does an empty dependency list mean no cleanup is needed?

No. If setup acquires a resource, cleanup is still needed. With no reactive dependencies, the cleanup runs when the component is removed, as well as during Strict Mode's development check.

## Troubleshooting

### 30. A listener fires twice. What should I inspect?

Check whether every setup registration has a matching removal, whether cleanup removes the same callback reference and options, and whether an old listener remains active after dependency changes. Strict Mode can expose missing removal during development.

### 31. My timer keeps running after I navigate away. What is likely missing?

The Effect probably did not clear the interval or timeout it started. Return cleanup that clears the exact timer identifier created by that setup.

### 32. Changing a dependency creates multiple connections. What should I inspect?

Make sure cleanup disconnects the old connection before the new setup creates another. Confirm cleanup closes over the local connection created by that setup rather than reading a shared variable overwritten by the new setup.

### 33. A fetch response updates the wrong screen after navigation. What should I inspect?

The old request may still be able to update state after its Effect was cleaned up. Abort it where possible and guard the result so only the still-active setup can update the UI.

### 34. Can I update state in cleanup to say that a resource disconnected?

Usually that state update is unnecessary and can run while the component is being removed. Model visible state from the current React inputs or from events emitted while the component is active; let cleanup release the external resource.

### 35. What is the best way to review an Effect's cleanup?

Identify the exact resource setup creates, find its documented release operation, verify cleanup uses that same instance, and check behavior when dependencies change, on unmount, and through Strict Mode's extra development cycle.

## Quick resource map

| Setup | Matching cleanup |
| --- | --- |
| Start an interval | Clear its interval identifier |
| Register an event handler | Remove the same handler registration |
| Subscribe to a store or service | Call the matching unsubscribe function |
| Create and connect a resource | Disconnect or close that resource |
| Start observing a target | Unobserve the target or disconnect its observer |
| Open a modal dialog | Close that dialog |
| Start an animation | Cancel it or restore its initial state |
| Start an async request | Abort it where possible and ignore obsolete results |

## Key points

- Cleanup runs before re-synchronization and on unmount.
- Cleanup closes over the exact resource and values from its setup.
- Each continuing resource needs a matching release operation.
- Strict Mode's extra development setup-cleanup cycle checks that cleanup mirrors setup.
- Abort and ignore protect local UI from obsolete work; they do not roll back a completed server action.
- React does not await an asynchronous cleanup function.
