# Synchronous DOM Updates with flushSync

React normally schedules and batches updates so it can render efficiently. Most application code should let React choose when to commit an update. `flushSync` is an escape hatch for the rare integration that must read or use the updated DOM before the current JavaScript callback continues.

Import it from `react-dom`. Wrap the state update in its callback. When `flushSync` returns, React has synchronously committed the relevant updates to the DOM. The callback itself does not return a rendered value.

## Learning goals

After this lesson, you should be able to:

- explain the difference between scheduling a state update and synchronously committing it;
- import and call `flushSync` for a real external integration requirement;
- identify why an immediate DOM read may be stale without a synchronous commit;
- avoid calling `flushSync` during rendering, Effects, and lifecycle methods;
- recognize its effects on batching, pending Effects, and Suspense;
- choose ordinary React patterns when synchronous DOM access is not actually required.

## The API

`flushSync` is exported by `react-dom`:

~~~jsx
import { flushSync } from "react-dom";

flushSync(() => {
  setMessage("Ready");
});

// React has committed the relevant update before execution reaches here.
const text = messageElement.textContent;
~~~

It accepts one callback. React calls that callback and synchronously flushes the updates it contains. The function returns `undefined`; use state, refs, or the DOM to access the result you need.

`flushSync` does not mean “return the next state value.” It ensures React has committed the relevant update to the DOM before the next line runs.

## Why would code need it?

In normal React code, a state setter schedules an update and React commits it at an appropriate point. Usually that is exactly what the application needs.

Some external APIs or libraries synchronously inspect the DOM during a callback. If React has not committed a state update before that inspection, the external code sees the previous DOM. A narrowly placed `flushSync` can make the update visible to that integration before the callback continues.

~~~jsx
function handleExternalRequest() {
  flushSync(() => {
    setStatus("Ready for the external system");
  });

  // An external system that reads the DOM here now sees the committed status.
  thirdPartyWidget.readCurrentMarkup();
}
~~~

Use this only when the external code truly requires the updated DOM synchronously. If the code after the setter does not inspect the DOM or invoke a synchronous integration that depends on it, `flushSync` is usually unnecessary.

## Comparing a normal update with a forced commit

Inside an event handler, React commonly batches state updates until the handler finishes. An immediate DOM read after a setter can therefore see the previous render:

~~~jsx
function handleNormalUpdate() {
  setMessage("Updated");
  console.log(messageRef.current.textContent); // May still be the previous text.
}
~~~

When a synchronous integration requires the new DOM before the handler proceeds, flush the state update first:

~~~jsx
function handleSynchronousIntegration() {
  flushSync(() => {
    setMessage("Updated");
  });

  console.log(messageRef.current.textContent); // The DOM now reflects "Updated".
}
~~~

The second version has an explicit cost: React must complete rendering and DOM work immediately instead of batching it with other work. Do not add it just to make an example appear more immediate.

## Browser callbacks: printing

A browser callback such as `beforeprint` may need the DOM updated before the browser opens its print dialog. A component can set print-specific UI synchronously in that callback and restore the state when printing ends:

~~~jsx
import { useEffect, useState } from "react";
import { flushSync } from "react-dom";

function PrintStatus() {
  const [isPrinting, setIsPrinting] = useState(false);

  useEffect(() => {
    function handleBeforePrint() {
      flushSync(() => {
        setIsPrinting(true);
      });
    }

    function handleAfterPrint() {
      setIsPrinting(false);
    }

    window.addEventListener("beforeprint", handleBeforePrint);
    window.addEventListener("afterprint", handleAfterPrint);

    return () => {
      window.removeEventListener("beforeprint", handleBeforePrint);
      window.removeEventListener("afterprint", handleAfterPrint);
    };
  }, []);

  return <p>Print mode: {isPrinting ? "on" : "off"}</p>;
}
~~~

The `beforeprint` handler is an external browser callback where the browser may use the document as soon as the callback completes. The synchronous commit makes the print-mode content available by then. Register and remove listeners in an Effect, but call `flushSync` from the callback-not while the Effect itself is running.

For basic print styling, prefer CSS `@media print` when it is sufficient. Use a React state change only when the printed interface genuinely depends on application state or markup.

## What may flush

The callback's update is the reason to call `flushSync`, but React may also flush other pending work when needed:

- pending updates that were already queued before the call;
- Effects that are due to run, and updates scheduled inside those Effects;
- updates outside the callback that must be processed to commit the callback's update.

This means the visible effect of a `flushSync` call can be broader than one setter. Keep the callback small and avoid relying on unrelated work being postponed.

## Suspense and performance costs

A forced synchronous commit can hurt performance because React loses the opportunity to batch or schedule work at a more suitable time. Repeated calls can block the browser from processing other work smoothly.

If an update suspends, a pending `Suspense` boundary may show its fallback again. This can produce a sudden loading state even when some content had already appeared.

`flushSync` is not a general priority control, a way to make every interaction faster, or a substitute for choosing a good loading and Suspense design. Use it sparingly and check whether the integration can work with React's normal commit timing.

## Where not to call it

Do not call `flushSync` while React is already rendering or running a lifecycle phase. That includes:

- a component's render function;
- `useEffect` or `useLayoutEffect` setup;
- class component lifecycle methods.

React cannot synchronously flush while it is already in the middle of rendering. Such calls can warn and do nothing.

~~~jsx
useEffect(() => {
  // Avoid: this Effect is already running as part of React's work.
  flushSync(() => {
    setReady(true);
  });
}, []);
~~~

If the update is caused by a user action, move the call to that event handler when a synchronous external read is truly required. If an exceptional integration requires deferring it until the current render finishes, scheduling work later may be possible, but deferral is not a routine replacement for sound data flow and can make performance worse.

A normal state update inside an event handler is safe. Add `flushSync` there only when the code after it must observe the committed DOM immediately.

## Prefer simpler approaches first

Before reaching for `flushSync`, ask what the next line actually needs:

- If it needs the next application value, compute or keep that value in JavaScript; do not read it back from the DOM.
- If it needs to display new UI, set state and let React commit normally.
- If it needs to measure layout after React commits, a ref and an Effect or `useLayoutEffect` may fit better. `useLayoutEffect` is for layout work after a commit, not a general request to synchronously flush arbitrary state setters.
- If it only needs print-specific styling, use a print stylesheet when possible.
- If a third-party library supports callbacks, refs, or scheduled updates, use its integration contract before forcing a synchronous commit.

The need to synchronously inspect React-managed DOM is relatively uncommon. Keep application logic in state and props instead of using the DOM as the source of truth.

## Common mistakes

- **Importing from the wrong package:** `flushSync` comes from `react-dom`, not `react-dom/client`.
- **Using it for every setter:** normal React scheduling and event batching are desirable; do not force a commit without a specific integration requirement.
- **Expecting a value back:** `flushSync` returns `undefined`. Read the needed state from your own variables or the committed DOM.
- **Calling it in an Effect or render:** React is already working; the call can warn and do nothing.
- **Putting too much inside the callback:** a small, explicit update is easier to reason about and less likely to flush unrelated work.
- **Assuming it affects only the callback's update:** pending updates and Effects outside the callback may also run.
- **Ignoring Suspense behavior:** a fallback can appear again if the synchronous update suspends.
- **Using it to solve a stale closure:** it does not change the JavaScript value captured by the current render. It commits DOM updates; it does not rewrite existing variables.
- **Reading the DOM when application state is enough:** calculate with state or props instead of using a synchronous DOM read as a data channel.

## Practical checklist

1. Does code outside React need to read or use the updated DOM before the current callback returns?
2. Can a ref, callback, CSS rule, or normal state update meet that need?
3. If not, import `flushSync` from `react-dom` and wrap only the necessary state update.
4. Call it from an event or external callback, not during render, an Effect, or a lifecycle method.
5. Account for pending work, Effects, Suspense fallbacks, and the performance cost.
6. Remove it if the synchronous requirement goes away.

## Summary

`flushSync(callback)` forces React to synchronously flush relevant updates so the DOM is current before the next line runs. It is an uncommon escape hatch for browser or third-party integrations that require immediate DOM changes. It can flush unrelated work, reveal Suspense fallbacks, and hurt performance, so prefer normal React scheduling and use it only when a concrete synchronous DOM requirement remains.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="070-react-dom-roots-and-client-apis.md">070 - React DOM Roots, Hydration, and Client APIs</a></td><td><a href="../qnas/071-flushsync.md">Q&amp;A</a></td><td><a href="../examples/071-flushsync.md">Example</a></td><td><a href="072-react-dom-resource-hints-and-browser-apis.md">072 - React DOM Resource Hints and Browser APIs</a></td></tr>
  </tbody>
</table>
