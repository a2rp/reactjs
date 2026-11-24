# Concurrent Rendering and Update Priorities

## Keep urgent interactions responsive

Some updates should appear immediately: typing into a controlled text field, pressing a button, or moving a focus indicator. Other updates may require rendering a large result list, chart, or page. If that rendering is expensive, React can mark it as non-urgent so it does not block the next interaction.

React’s concurrent rendering lets React prepare an update in the background, pause or abandon that render work, and continue with a newer update when needed. It does not run JavaScript on another thread, and it cannot interrupt an arbitrary long-running JavaScript function in the middle of its synchronous work. Keeping render pure is what makes it safe for React to retry or discard render work.

React provides two related tools:

- **useTransition** marks state updates as non-urgent and gives the component an **isPending** flag.
- **useDeferredValue** lets a value used by a slower part of the UI temporarily lag behind its latest value.

Both are for prioritizing rendering. Neither is a general-purpose timer or a way to make expensive calculations intrinsically faster.

## Learning goals

After this lesson, you should be able to:

- explain what interruptible concurrent rendering means and what it does not mean;
- distinguish urgent updates from non-urgent Transition updates;
- use **useTransition** and **startTransition** for non-urgent state changes;
- keep controlled input state urgent while a result view catches up;
- use **useDeferredValue** when the slower consumer receives a changing value;
- show an appropriate pending or stale visual state;
- distinguish rendering priority from debouncing, throttling, and network request control;
- recognize Transition limitations around controlled inputs and async work.

## Concurrent rendering is interruptible work

In ordinary JavaScript, a running function blocks the browser’s main thread until it returns. React cannot interrupt a single long, synchronous loop. During rendering, however, React can yield between units of component work. A low-priority render can be paused or abandoned so React can handle a more urgent update, such as a new keystroke, then restart with the latest state.

This behavior is not the same as multiple JavaScript threads running components at the same time. Nor does it mean every render is automatically non-blocking. Transitions and deferred values let an app tell React which rendering work may wait.

A render may be restarted or discarded before commit. React commits a completed result, so render code must be pure and safe to run more than once.

## Urgent and non-urgent state

A controlled text input must reflect each keystroke promptly. Keep the state that drives its **value** urgent:

~~~jsx
const [query, setQuery] = useState("");

<input value={query} onChange={(event) => setQuery(event.target.value)} />
~~~

A large result view that uses the query can be allowed to lag behind. Use either a Transition for a separate results state or **useDeferredValue** for a slower consumer. Do not put the input’s own value update inside a Transition.

## useTransition: mark a state update as non-urgent

Call **useTransition** at the top level of a component:

~~~jsx
import { useState, useTransition } from "react";

function Tabs() {
  const [selectedTab, setSelectedTab] = useState("overview");
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setSelectedTab(nextTab);
    });
  }

  return (
    <>
      <button onClick={() => selectTab("overview")}>Overview</button>
      <button onClick={() => selectTab("reports")}>Reports</button>
      <p aria-live="polite">{isPending ? "Updating…" : "Ready"}</p>
      <TabContent tab={selectedTab} />
    </>
  );
}
~~~

The Hook returns **isPending** and **startTransition**. While React is preparing the Transition update, **isPending** can be used to show a subtle pending status or disable an action when that is appropriate.

The callback passed to **startTransition** runs immediately. React marks state updates scheduled synchronously inside that callback as non-urgent; it does not delay the callback like **setTimeout**. The resulting render can run in the background and be interrupted by more urgent updates.

### Which update belongs in a Transition?

Use a Transition for a state update that changes a slower, non-urgent part of the UI, such as selecting a tab with a large panel, updating a chart, or showing a substantial set of search results.

Do not use it for the state that controls a text input:

~~~jsx
function handleChange(event) {
  // Keep the controlled input update urgent.
  setQuery(event.target.value);
}
~~~

If typing also updates a large list, defer the query used by the list or store a separate results query in a Transition. The input itself must continue to reflect the latest typed value.

## startTransition: mark an update without a pending flag

The standalone **startTransition** function is useful when code has access to a state setter but cannot call a Hook at that location, such as in a utility or a component-independent action:

~~~jsx
import { startTransition } from "react";

function navigate(setPage, nextPage) {
  startTransition(() => {
    setPage(nextPage);
  });
}
~~~

It marks synchronous state updates inside the callback as Transitions, but it does not return **isPending**. Use **useTransition** when the calling component needs pending status.

Both APIs require access to the setter for the state being updated. If a component only receives a changing value as a prop and cannot mark the update where it is set, consider **useDeferredValue** in the slower consumer.

## useDeferredValue: let a consumer catch up

**useDeferredValue** returns a deferred version of a changing value. During an update, React can first render urgent UI with the previous deferred value, then attempt a background render using the latest value:

~~~jsx
import { memo, useDeferredValue, useState } from "react";

const SlowResults = memo(function SlowResults({ query }) {
  const matches = searchCatalog(query);
  return <Results items={matches} />;
});

function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <>
      <input
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />
      <SlowResults query={deferredQuery} aria-busy={isStale} />
    </>
  );
}
~~~

The input uses the latest **query**. The slower result component receives **deferredQuery**, which may temporarily be older while React completes urgent work. A subtle stale indicator can communicate that the displayed results are catching up.

For this to avoid rendering the slow child during the urgent parent update, the child usually needs **memo** or another way to skip rendering when its deferred prop has not changed. Otherwise, the child may render with the same old value anyway.

On the initial render, the deferred value is normally the same as the supplied value. There is no fixed delay; React starts background work as soon as it can and can interrupt it when more urgent updates arrive.

## Choosing between useTransition and useDeferredValue

| Situation | Consider |
| --- | --- |
| You control the state setter for the non-urgent update | **useTransition** or **startTransition** |
| The component needs to show whether a Transition is pending | **useTransition** |
| You receive a changing value but do not control where it is set | **useDeferredValue** |
| You need the input to update immediately and results to catch up | Urgent input state plus **useDeferredValue**, or separate urgent and Transition state |
| You need to limit how often a network request is sent | Debouncing, throttling, caching, or a request strategy; a Transition alone does not do this |

Do not use both for the same state update without a reason. A Transition already defers its update; **useDeferredValue** does not spawn another background render for a value that is already updating inside a Transition.

## Pending and stale UI

Pending feedback should help users understand the interface without making the whole app feel blocked. Common choices include:

- a small “Updating…” status near a tab panel;
- a subtle opacity change on stale results;
- retaining the current content while the next view is prepared;
- disabling a repeated action only when repeating it would be confusing or unsafe.

Avoid replacing a usable interface with a full-screen spinner for every Transition. Transitions are designed to keep already revealed UI available while React prepares the next result.

For a deferred value, compare the current value with the deferred one when both are comparable primitives:

~~~jsx
const isStale = query !== deferredQuery;
~~~

This indicates the consumer is behind. It is not an exact progress percentage and does not mean a network request is pending.

## Transitions do not debounce or cancel requests

A Transition controls the priority of React state rendering. **useDeferredValue** controls when a value is used by a background render. Neither automatically waits for a chosen number of milliseconds, limits keystrokes, or reduces the number of network requests.

If an input fires one request per keystroke, deferring the result display does not change that. Use a separate request policy such as debouncing, throttling, cancellation, or caching when controlling network work is the goal.

## Async Actions and updates after await

Current **useTransition** supports passing an async Action. The pending flag can remain true while the Action is pending. However, state updates after an **await** currently need another **startTransition** call to mark those later updates as non-urgent:

~~~jsx
const [isPending, startTransition] = useTransition();

function loadReport() {
  startTransition(async () => {
    const report = await fetchReport();

    startTransition(() => {
      setReport(report);
    });
  });
}
~~~

This is a current limitation of async context propagation. Also handle request ordering when users can start multiple async operations quickly; a Transition does not automatically guarantee that older requests cannot overwrite newer results.

A state update scheduled later in a **setTimeout** is outside the synchronous Transition callback. Start a Transition inside that timer if that later update should be non-urgent.

## Common mistakes

- Marking a controlled input’s value update as a Transition.
- Expecting **startTransition** to delay its callback or move JavaScript work onto another thread.
- Assuming a single long synchronous calculation can be interrupted in the middle.
- Expecting a Transition or deferred value to make the expensive calculation itself faster.
- Using **useDeferredValue** without a memoized or otherwise skippable slow child, so it still renders on the urgent parent update.
- Expecting a fixed delay or fewer network requests from **useDeferredValue**.
- Showing no pending or stale feedback when old content could be mistaken for current content.
- Using **startTransition** but expecting an **isPending** value from it.
- Forgetting to re-wrap state updates after an **await** when they should be Transition updates.
- Assuming Transition updates automatically solve async request ordering.

## Summary

- Concurrent rendering lets React pause or abandon render work and prioritize a newer update; it is not multi-threaded JavaScript.
- Keep direct feedback, especially controlled input state, urgent.
- **useTransition** marks state updates as non-urgent and exposes **isPending**; standalone **startTransition** marks updates without a pending flag.
- **useDeferredValue** lets a slower consumer lag behind a value it receives.
- A Transition’s callback runs immediately; React changes the priority of state updates scheduled inside it.
- Transitions and deferred values do not use a fixed delay, speed up calculations, or reduce network requests by themselves.
- Async state updates after **await** currently need another **startTransition** when they should remain non-urgent.
