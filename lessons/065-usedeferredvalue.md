# useDeferredValue for Interruptible Updates

Lesson 063 introduced deferred rendering. This lesson focuses on **useDeferredValue** itself: how to pass a changing value to a slower consumer, preserve existing content while React catches up, and avoid common identity and memoization mistakes.

## Learning goals

After this lesson, you should be able to:

- call **useDeferredValue** at the top level of a component;
- keep urgent state, such as a controlled input value, separate from a deferred consumer value;
- explain the initial value, background render, and interruption behavior;
- memoize a slow child so it can skip the urgent parent render while its deferred prop is unchanged;
- indicate when displayed content is stale;
- avoid unstable object values and unnecessary background renders;
- understand how the Hook interacts with Suspense, Effects, Transitions, and network requests.

## What the Hook returns

Call **useDeferredValue** with a value and, optionally, an initial value:

~~~jsx
import { useDeferredValue, useState } from "react";

function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);

  // Use query for immediate UI and deferredQuery for slower UI.
}
~~~

During the initial render, the returned value is the same as **query** unless an **initialValue** argument is provided. During updates, React can first render urgently with the previous deferred value, then try a background render with the new value.

~~~jsx
const deferredQuery = useDeferredValue(query, "popular");
~~~

An explicit initial value can be useful when a slower region should start from a deliberate initial display. Choose it so the first render makes sense to the user. It is not a loading timer or a general-purpose default-state API.

## Keep the source value urgent

For a controlled field, update its value directly from the event. Pass the deferred version only to the slower results region:

~~~jsx
import { memo, useDeferredValue, useState } from "react";

const Results = memo(function Results({ query }) {
  const items = searchItems(query);
  return <ResultList items={items} />;
});

function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <>
      <label>
        Search
        <input
          value={query}
          onChange={(event) => setQuery(event.target.value)}
        />
      </label>

      <section aria-busy={isStale}>
        <p role="status" aria-live="polite">
          {isStale ? "Updating results…" : "Results are up to date."}
        </p>
        <Results query={deferredQuery} />
      </section>
    </>
  );
}
~~~

The input stays current. The results may temporarily reflect the previous query. When the background render completes, React commits the latest results and the deferred value catches up.

## Why memoization matters

During an urgent parent render, **deferredQuery** may still be the same value it was before. A slow child wrapped in **memo** can skip rendering when its props are unchanged:

~~~jsx
const SlowResults = memo(function SlowResults({ query }) {
  return <LargeResultList query={query} />;
});
~~~

Without **memo** or another way for the child to skip that render, it may redo the slow work with the unchanged old prop during the urgent render. That defeats the common responsiveness benefit.

Keep the deferred child’s other props stable too. Passing a new callback, array, or object on each parent render can cause a memoized child to render even when its deferred query has not changed.

**useDeferredValue** changes when React attempts a render; it does not make the child’s work faster. If the child still takes a long time to render, profile it and consider optimizing the component, reducing the work, or using a suitable large-list strategy.

## Show that content is stale

For a primitive query string, compare the immediate and deferred values:

~~~jsx
const isStale = query !== deferredQuery;
~~~

When they differ, the displayed results are behind the input. Give a small, accessible indication near those results. You can dim the region or show a status, while keeping its current content visible:

~~~jsx
<div className={isStale ? "results results--stale" : "results"}>
  <Results query={deferredQuery} />
</div>
~~~

Do not describe this as a network request or exact progress percentage. It only says the rendered consumer is using an older value. Use actual request state for network feedback.

## No fixed delay and no request policy

**useDeferredValue** has no built-in number of milliseconds. React starts its background render when it can; more urgent work can interrupt that render, which React can later retry with the latest value. A fast device may catch up almost immediately. There is no debounce interval to configure.

The Hook defers rendering a consumer. It does not by itself:

- limit how often an event handler runs;
- debounce, throttle, or cancel requests;
- guarantee that old network responses cannot replace newer data;
- speed up a synchronous calculation;
- guarantee a visible stale period.

If the goal is to send fewer requests, use a request strategy such as debouncing or caching. A component that starts work from an Effect may still start a request for a committed deferred value; keep request behavior and rendered-value priority as separate decisions.

## Suspense can keep stale content visible

If rendering with the new deferred value suspends, React can keep showing the previous deferred content instead of replacing already revealed UI with a fallback. Once the data becomes available, React can commit the current result.

This works with Suspense-enabled data sources. Merely wrapping a normal request in **useDeferredValue** does not make that request suspend or provide caching.

For a nested Suspense boundary, its own fallback may still appear according to where the boundary is placed and what has already been revealed. Choose boundaries based on which parts should remain visible together.

## Object identity and unnecessary renders

The Hook compares the supplied value with **Object.is**. Primitive values such as strings and numbers are usually straightforward. Be careful with a new object created during every render:

~~~jsx
// Avoid: this object is new on every render.
const deferredOptions = useDeferredValue({ query, sortOrder });
~~~

Prefer a stable object created outside rendering, or memoize an object that needs to be derived from the current state:

~~~jsx
import { useDeferredValue, useMemo } from "react";

const options = useMemo(
  () => ({ query, sortOrder }),
  [query, sortOrder],
);
const deferredOptions = useDeferredValue(options);
~~~

If only **query** needs to be deferred, passing the string directly is simpler. Do not add object memoization when a primitive value already expresses the intended dependency.

## Interactions with Transitions and Effects

If the value update already happens inside a Transition, **useDeferredValue** returns that new value and does not add another deferred render. Use the mechanism that matches where you control the update:

- Choose **useTransition** when you own a state setter and want to mark that update as non-urgent.
- Choose **useDeferredValue** when a slower consumer should lag behind a value it receives, especially when you do not control the source setter.

Effects caused by a background render run only if that render commits. If React interrupts or abandons a background attempt, its Effects do not run for that discarded attempt.

## Common mistakes

- Passing the deferred value to the input instead of keeping its controlled value urgent.
- Expecting a specific delay before the deferred value changes.
- Assuming the Hook reduces network requests or cancels stale requests.
- Expecting a memoized child to skip rendering while passing it a new object or callback every time.
- Forgetting to memoize a genuinely slow child and then seeing it render during the urgent parent update anyway.
- Expecting a synchronous loop inside a child to be interrupted halfway through.
- Using a stale indicator that says “Loading request” when only rendering is behind.
- Using **useDeferredValue** on a value already updated inside a Transition without a reason.
- Creating a fresh object during render and deferring that new object on every render.
- Assuming ordinary data-fetching code suspends just because it receives a deferred value.

## Summary

- **useDeferredValue(value, initialValue?)** returns the value a slower consumer should render with.
- The initial result matches the input value unless an explicit initial value is supplied.
- During updates, React may keep the old value for urgent rendering and try the new value in an interruptible background render.
- A slow child usually needs **memo** so it can skip the urgent parent render while its deferred props remain unchanged.
- Compare primitive source and deferred values to indicate stale content.
- There is no fixed delay, and the Hook does not optimize calculations, control request frequency, or cancel network work.
- Suspense can preserve revealed stale content while new deferred content is unavailable.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="064-transitions-and-usetransition.md">064 - Transitions with useTransition and startTransition</a></td><td><a href="../qnas/065-usedeferredvalue.md">Q&amp;A</a></td><td><a href="../examples/065-usedeferredvalue.md">Example</a></td><td><a href="066-suspense-boundaries-and-fallbacks.md">066 - Suspense Boundaries and Fallbacks</a></td></tr>
  </tbody>
</table>
