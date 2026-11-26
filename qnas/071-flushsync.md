# Synchronous DOM Updates with flushSync - Q&A

## Core behavior

### 1. What is `flushSync`?

`flushSync` is a React DOM escape hatch that forces React to synchronously flush relevant updates so the DOM is current before the call returns.

### 2. Where is `flushSync` exported from?

Import it from `react-dom`:

~~~jsx
import { flushSync } from "react-dom";
~~~

It is not imported from `react` or `react-dom/client`.

### 3. What does `flushSync` accept?

It accepts a callback. React invokes that callback and synchronously flushes the updates it contains.

~~~jsx
flushSync(() => {
  setStatus("Ready");
});
~~~

### 4. What does `flushSync` return?

It returns `undefined`. It does not return the new state or a DOM node.

### 5. What is available after the call returns?

React has committed the relevant updates to the DOM. Code after the call can read that committed DOM if a synchronous integration needs it.

### 6. Does `flushSync` change a state variable captured by the current render?

No. It does not rewrite JavaScript variables or update a closure. Use state or props for application values; `flushSync` only changes when React commits updates to the DOM.

### 7. Does every state setter need `flushSync`?

No. React's normal scheduling and batching are preferred for ordinary application updates. Use the escape hatch only for a concrete synchronous DOM requirement.

### 8. Why can a DOM read immediately after a state setter be stale?

React may batch updates until the current event handler completes. A DOM read in the same handler can happen before React commits the update.

### 9. What problem does `flushSync` solve in that case?

It forces React to commit the relevant update before execution continues, allowing a synchronous browser API or third-party integration to read the updated DOM.

### 10. Is `flushSync` a way to make the whole app faster?

No. It can hurt performance by preventing React from batching and scheduling work normally. It is a targeted integration escape hatch.

## Appropriate use

### 11. When is `flushSync` most useful?

When code outside React synchronously inspects or uses the DOM before the current callback returns, and that code must see a React update made in that callback.

### 12. Can I use it from a user event handler?

Yes, calling it from an event handler is safe. It is only needed there if code later in the same handler truly requires the updated DOM immediately.

### 13. Can it help with a browser callback such as `beforeprint`?

Yes. If the browser will use the document immediately after the callback, a synchronous React update can ensure the needed markup is present before the browser proceeds.

### 14. Should I use it for ordinary print styling?

Usually not. A CSS `@media print` rule is simpler when it can express the print-specific appearance. Use state and `flushSync` only when the printed interface must change based on application state or markup.

### 15. Should it be used to update a React screen before taking a screenshot?

Only if the screenshot integration synchronously captures the DOM before React's normal commit can happen, and a supported callback or ref-based integration cannot solve the timing. Otherwise, use the normal update flow.

### 16. Can a third-party UI library be a reason to use it?

Possibly. First use the library's documented React integration, refs, or callbacks. If it synchronously reads the DOM and requires the new markup before returning, a small `flushSync` around the required state update may be appropriate.

### 17. Should the whole event handler go inside the callback?

No. Keep the callback narrow and place only the update that must be committed synchronously inside it. Keep external reads or calls after it returns.

## Effects and side effects

### 18. Can I call `flushSync` during component rendering?

No. React cannot flush synchronously in the middle of rendering. It may warn and do nothing.

### 19. Can I call it inside `useEffect`?

No. An Effect is already running as part of React's work. Calling `flushSync` there can warn and do nothing.

### 20. Can I call it inside `useLayoutEffect`?

No. It should not be called from Effect setup, including `useLayoutEffect`. That hook runs as part of the commit lifecycle and is not a general place to force arbitrary updates.

### 21. Can a class component lifecycle method call it?

No. Lifecycle methods are part of React's rendering and commit process. Move an actual synchronous integration call to an appropriate event or external callback.

### 22. What if the update is triggered by an Effect and seems to need a synchronous commit?

First check whether a normal Effect or layout Effect is the right mechanism. If a specialized external integration truly requires deferred synchronous work, it may be possible to schedule it after the current React work, but this is an exceptional escape hatch and can worsen performance.

### 23. Does calling `flushSync` from an Effect make an update more reliable?

No. React cannot flush while it is already running the Effect and may warn. It can fail to achieve the intended timing.

## What may be flushed

### 24. Does it flush only updates created inside its callback?

Not necessarily. React may also flush pending work outside the callback when needed to apply the callback's update.

### 25. Can pending Effects run during a `flushSync` call?

Yes. React may run pending Effects and synchronously apply updates scheduled by those Effects before returning.

### 26. Can it flush updates that were already pending before the call?

Yes. Pending work can be flushed if React needs to process it to complete the synchronous commit.

### 27. Can `flushSync` make a Suspense fallback appear again?

Yes. If an update suspends, a pending Suspense boundary may show its fallback. Do not assume a forced update preserves already revealed content.

### 28. Does it guarantee only the DOM touched by one state setter changes?

No. The synchronous commit may include related or pending work. Do not rely on it to isolate exactly one update.

## Alternatives and common misunderstandings

### 29. What should I do if the next line only needs the new application value?

Compute or keep that value in JavaScript state, props, or a local variable. There is no need to read it back from the DOM.

### 30. What if I need to measure layout after React renders?

Use a ref and a layout measurement pattern such as `useLayoutEffect` when appropriate. It runs after a commit and before paint, but it is not a generic way to force an arbitrary state update to flush in the middle of the current callback.

### 31. Is `flushSync` the same as `useLayoutEffect`?

No. `flushSync` forces an update to commit before the function returns. `useLayoutEffect` runs layout-related code after React commits and before the browser paints.

### 32. Is it the same as calling `setTimeout`?

No. `setTimeout` schedules work later. It does not guarantee that a particular React update has committed before the current callback continues.

### 33. Is it a replacement for React's state model?

No. Continue to use state, props, and context as the source of truth. `flushSync` only controls commit timing for an integration boundary.

### 34. Should I call it repeatedly to force every update to happen immediately?

No. Repeated forced commits reduce batching opportunities, can block the browser, and may expose loading fallbacks. Remove calls that do not serve a specific synchronous integration.

### 35. Can it solve a stale closure?

No. The handler still closes over values from the render that created it. A DOM commit does not mutate those variables.

## Troubleshooting

### 36. Why do I see a warning that `flushSync` was called inside a lifecycle method?

It was called during React rendering or a lifecycle phase, such as a component render, an Effect, or a class lifecycle method. Move it to an event or external callback if there is a genuine synchronous DOM requirement.

### 37. Why did my `flushSync` callback not make a console variable contain the new value?

The callback flushes React's DOM work; it does not change variables from the current render. Read a ref or the DOM after the call, or use the state value through React's normal data flow.

### 38. Why did a loading fallback flash after I added `flushSync`?

A synchronous update may suspend and cause a Suspense boundary to show its fallback. Reconsider the update and the boundary design; do not add more forced commits to hide the symptom.

### 39. Why did performance get worse after adding it?

The app may be committing updates that React previously could batch. Keep it to the smallest required update, remove redundant calls, and profile whether the external integration truly needs immediate DOM.

### 40. What is the safest decision rule?

Use `flushSync` only when code outside React needs the updated DOM before the current callback returns and no simpler integration pattern meets that requirement. Otherwise let React schedule the update normally.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="070-react-dom-roots-and-client-apis.md">070 - React DOM Roots, Hydration, and Client APIs - Q&amp;A</a></td><td><a href="../lessons/071-flushsync.md">Lesson</a></td><td><a href="../examples/071-flushsync.md">Example</a></td><td><a href="072-react-dom-resource-hints-and-browser-apis.md">072 - React DOM Resource Hints and Browser APIs - Q&amp;A</a></td></tr>
  </tbody>
</table>
