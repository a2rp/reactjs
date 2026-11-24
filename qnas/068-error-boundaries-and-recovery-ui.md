# Error Boundaries and Recovery UI — Q&A

## Fundamentals

### 1. What is a React Error Boundary?

It is a component that catches eligible errors thrown by descendant components while React renders them and displays fallback UI for the failed region.

### 2. What problem does an Error Boundary solve?

It can keep an error in one region from removing useful surrounding interface. The boundary replaces its failed descendants with a recovery view.

### 3. Is an Error Boundary a special HTML element?

No. It is a React component implemented with specific class component methods.

### 4. Can I create an Error Boundary with a function component and Hooks alone?

React does not currently provide a built-in Hook that turns a function component into an Error Boundary. Use a class with the boundary methods or a library that provides a function-style API.

### 5. Can a class Error Boundary wrap function components?

Yes. A boundary can wrap function components and their descendant tree.

### 6. Which boundary handles a descendant error?

The nearest Error Boundary above the component that throws handles it.

### 7. What happens if there is no Error Boundary above a render error?

React may remove the affected application UI rather than leave a potentially inconsistent tree on screen.

### 8. Do I need a boundary around every component?

No. Choose boundaries around meaningful regions that can fail and have a useful local fallback.

### 9. Can one boundary cover the whole application?

Yes, and a top-level boundary can serve as a last-resort screen. Local boundaries are also useful so one panel failure does not replace the entire application.

### 10. Can I nest Error Boundaries?

Yes. An inner boundary handles its descendant failures first. If the inner boundary fails while rendering its own fallback, an outer boundary can catch that error.

## Implementing the boundary

### 11. Which method changes the boundary state after an error?

`static getDerivedStateFromError` returns state that tells the boundary to render its fallback.

### 12. Should `getDerivedStateFromError` perform logging or other side effects?

No. It should be pure and only calculate the state needed for the fallback.

### 13. What is `componentDidCatch` used for?

It can record diagnostic information, such as the thrown value and the React component stack, in an error reporting system.

### 14. Do I need both boundary methods?

Use `getDerivedStateFromError` to display fallback UI. `componentDidCatch` is optional and useful for side effects such as reporting.

### 15. Should `componentDidCatch` call `setState` to show the fallback?

No. Prefer `getDerivedStateFromError` to update state for the fallback.

### 16. What arguments does `componentDidCatch` receive?

It receives the thrown value and an info object that includes the component stack for the descendant tree.

### 17. Is the thrown value always an Error object?

No. JavaScript allows throwing strings, null, and other values. Do not assume every thrown value has a safe `message` property.

### 18. Can I show `error.message` directly in the user-facing fallback?

Usually avoid it. Raw messages can expose implementation or sensitive details. Show a plain user-facing explanation and keep technical diagnostics for developers or approved reporting.

### 19. Can an Error Boundary catch an error from its own `render` method?

No. A boundary cannot catch its own failure. An outer boundary may catch it.

### 20. Can an Error Boundary catch an error in its fallback component?

Not with that same boundary. The fallback is rendered by the boundary itself, so use a dependable fallback or place an outer boundary above it.

## What boundaries catch

### 21. Do boundaries catch errors thrown while descendants render?

Yes. This is their main purpose.

### 22. Do boundaries catch errors from event handlers?

No. Handle those with normal event-handler logic, such as `try`/`catch` or state that renders an error message.

### 23. Do boundaries catch a failed form submission request?

Not just because the request failed. Handle the Promise rejection or response status and show the expected form error state.

### 24. Do boundaries catch a rejection from `setTimeout` or `requestAnimationFrame`?

No. Ordinary asynchronous callbacks are outside the render error handling path. Handle their errors where the callback runs.

### 25. How should asynchronous errors be handled?

Handle Promise rejections and callback errors in their async path. If the result should affect the UI, store an appropriate state value and render a normal error state.

### 26. Can an async error ever reach an Error Boundary?

An async error itself is not caught by the boundary. If the app records the error and later deliberately throws during rendering, that render error can be caught; this should be an intentional pattern, not an assumption about all Promises.

### 27. Do boundaries catch server-rendering errors?

No. Handle errors in the server rendering or framework layer that performs the server work.

### 28. Do boundaries catch errors outside React?

No. A boundary only covers errors from its React descendant tree in the supported render-error cases.

### 29. Is there an exception for errors in a React Transition?

React documents that errors thrown inside the function passed to its `startTransition` API can propagate to an Error Boundary. Do not extend this exception to all asynchronous callbacks.

### 30. Do boundaries validate props or prevent bugs?

No. They provide a fallback after an eligible error. Validate inputs and handle expected cases in the components that own those concerns.

## Error Boundaries and Suspense

### 31. What is the difference between an Error Boundary and Suspense?

Suspense shows fallback UI while supported content is pending. An Error Boundary shows fallback UI after an eligible failure is thrown.

### 32. Does Suspense show the error UI for a rejected lazy import?

No. Suspense handles the pending state. A rejected lazy import is handled by an Error Boundary.

### 33. How should I wrap a lazy component for loading and error states?

Place a Suspense boundary to show loading UI and an Error Boundary around it to handle import or render failures.

### 34. Does Suspense catch ordinary fetch calls from an Effect?

No. Suspense does not detect data fetched inside an Effect or event handler by itself.

### 35. Is every fetch rejection an Error Boundary case?

No. A rejected request handled by application code is normally an expected request or form state. A boundary is for unexpected errors thrown through React's supported render path.

## Placement and fallback design

### 36. Where should I put an Error Boundary?

Around a feature or region whose failure can be isolated, such as a dashboard panel, route, editor, or message list.

### 37. Why keep navigation outside a feature boundary?

So users can still navigate away or continue using unaffected parts of the application if that feature fails.

### 38. What should a recovery fallback say?

State plainly which region is unavailable and provide a useful next step without exposing internal diagnostics.

### 39. Should a fallback include a stack trace?

No. Keep stack traces and component stacks in developer tools or a protected reporting system.

### 40. Should there be an Error Boundary around every avatar or small text node?

Usually not. A boundary should match a meaningful failure and recovery unit.

### 41. Can a fallback contain buttons?

Yes. A fallback can provide actions such as retry, refresh, or navigation, provided the action has a real and understandable behavior.

### 42. Should an error fallback use `role="alert"`?

For an important failure message that should be announced promptly, an alert role can help assistive technology users. Choose semantics based on the actual interaction and avoid unnecessary repeated announcements.

## Recovery and reset behavior

### 43. Does a boundary automatically reset after showing fallback UI?

No. It stays in its failed state until the application resets it, remounts it, or otherwise changes the boundary state.

### 44. How can I reset a boundary in application code?

Add a retry action that clears the boundary's failed state and, when needed, changes the child input or application state that caused the error.

### 45. Can changing a boundary's key reset it?

Yes. A new key makes React unmount the old boundary and mount a fresh one, which resets its state and remounts its descendants.

### 46. Does resetting the boundary guarantee the error is fixed?

No. If the same child immediately throws again, the fallback will return. A reset is useful only when the underlying condition can recover or the user can change the relevant state.

### 47. Is a retry button always appropriate?

No. Use it when the retry can do something meaningful, such as re-render after correcting input or retrying an operation designed to be retried. Otherwise offer refresh, navigation, or a clear next step.

### 48. Does resetting an Error Boundary retry a rejected lazy import?

Not necessarily. React caches the Promise returned by a lazy component loader. Re-rendering the same lazy component type may reuse the rejected Promise; boundary reset is not a guaranteed import retry.

## Diagnostics, privacy, and testing the concept

### 49. What should I send to an error reporting service?

Send diagnostics that help identify and fix the issue, such as the error and component stack, while respecting the product's privacy rules.

### 50. Should I send form contents or credentials with the error report?

No. Avoid sending passwords, tokens, private form values, and other sensitive data unless a carefully reviewed product requirement explicitly permits it.

### 51. Can the reporting call itself fail?

Yes. The user-facing fallback should still render even if the reporting service is unavailable. Keep monitoring code from breaking the recovery UI.

### 52. How can I trigger a render error in a learning demo?

Pass a prop that makes a child throw during its render, then toggle that prop from a parent control outside the boundary. This tests a render error without throwing directly from the button handler.

### 53. Why should the demo's trigger control stay outside the boundary?

If the control is inside the region that gets replaced, it disappears when the error occurs and the user may have no way to correct the condition.

### 54. What is the difference between an expected empty state and an error fallback?

An empty state is a normal result such as zero search matches. An error fallback represents an unexpected failure. Render expected outcomes directly instead of throwing to reach an Error Boundary.

### 55. What is the practical rule for Error Boundary placement?

Place boundaries where a failure can be isolated and where a user can recover or continue using the rest of the interface.
