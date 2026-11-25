# Profiling Component Behavior During Development

## Learning goals

After this lesson, you should be able to:

- Distinguish a component render from a DOM change and from an Effect.
- Record the same user interaction in React DevTools or browser Performance tools.
- Read the main phases shown in React Performance tracks.
- Understand the values reported by the `<Profiler>` callback.
- Identify repeated and cascading updates in a component subtree.
- Use a small, realistic profile to decide what to inspect next.
- Compare behavior before and after a change without confusing development overhead with production performance.

## 1. What profiling during development is for

Profiling is a way to observe what React did during a specific interaction. It answers questions such as:

- Which component subtree rendered after this input changed?
- Did React render a component repeatedly before the screen settled?
- Did a state update trigger another update from an Effect?
- Which part of the React tree took the most render time?
- Did a targeted change reduce the work for the interaction that felt slow?

A profile is evidence about one run, one build, one data set, and one environment. It does not prove that every user sees the same performance. Begin with a user-visible symptom and reproduce it with representative data.

## 2. Render, commit, and Effect are different work

A **render** is React calling components to calculate the next UI. A render can be started and then discarded; it does not promise that React changed the DOM.

A **commit** is when React applies the completed changes to the host environment, such as updating DOM nodes. If the rendered output is unchanged, React may call a component without needing to change the DOM.

An **Effect** synchronizes committed UI with an external system. A passive `useEffect` generally runs after the commit and often after paint; `useLayoutEffect` runs as part of the commit before the browser paints. An Effect that updates state can schedule another render and commit.

These distinctions matter when reading a trace. A component function running does not alone prove that the user saw a DOM change. Repeated renders may be harmless, but they can also expose expensive work, impure rendering, or an update loop.

## 3. Prepare a useful development profile

Use a development build with source maps and React DevTools available. Before recording:

1. Pick one interaction that exhibits the behavior, such as typing in a search field or opening a menu.
2. Use realistic data and a consistent browser window size.
3. Wait for initial data and unrelated activity to settle.
4. Close unrelated tabs and stop other test or recording activity where practical.
5. Record only the interaction and the small amount of work needed to observe its result.
6. Repeat once to see whether the pattern is stable.

Keep React Strict Mode enabled while debugging. It intentionally repeats some development work to reveal impure rendering and missing Effect cleanup. When the trace contains extra development-only work, identify it as such rather than assuming production behaves identically.

Do not compare raw development timings with a production build as if they were the same measurement. Development tooling and profiling instrumentation add overhead. Use the development profile to understand behavior and a production-like or profiling build when you need representative timing measurements.

## 4. Record React Performance tracks

In a supported browser and React development build, the browser Performance panel can show React-specific tracks alongside JavaScript, network, and browser work. Record the interaction in the Performance panel and inspect the React tracks around the same timeline region.

The React Performance tracks can include:

- **Scheduler** events that show the kind of update and its scheduling priority.
- **Components** entries that show component render durations as a tree or flamegraph.
- **Effects** work associated with the render and commit sequence.
- **Server** tracks in development setups that expose React Server Component work.

A render pass can be represented by an update, render, commit, and remaining Effects. An update scheduled while React is already rendering can produce a **cascading update**: React may need to discard work or begin another pass. In a development trace, inspect the component or method that scheduled it.

Track names and detail depend on the React build, browser, and available developer tools. If React-specific tracks are unavailable, use the React DevTools Profiler and browser Performance panel separately.

## 5. Use the React DevTools Profiler

The Profiler panel is useful when you want to focus on a React interaction without reading the full browser timeline:

1. Start recording.
2. Perform one interaction that can be repeated consistently.
3. Stop recording as soon as the resulting UI settles.
4. Select the commit caused by that interaction.
5. Use the flamegraph or ranked view to find the subtree with the most work.
6. Inspect the component and changed props to determine why it rendered.
7. Compare the same interaction after a targeted change.

Do not optimize a component just because it appears in the recording. It may be fast, and it may be expected to render. Look for work that correlates with the reported delay or causes a measurable cost across realistic usage.

## 6. Measure a subtree with `<Profiler>`

React's `<Profiler>` can report render measurements for a selected part of the component tree:

```jsx
import { Profiler } from 'react';

function reportProfile(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  console.table({
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime,
  });
}

function App() {
  return (
    <Profiler id="SearchResults" onRender={reportProfile}>
      <SearchResults />
    </Profiler>
  );
}
```

The callback receives:

- `id`: the `Profiler` label for the subtree that committed.
- `phase`: whether it mounted, updated, or had a nested update.
- `actualDuration`: time spent rendering the profiled tree for this update.
- `baseDuration`: an estimate of the render cost for the whole subtree without skipping work, based on recent component durations.
- `startTime`: when React began rendering this update.
- `commitTime`: when React committed it; profilers participating in the same commit share this value.

`actualDuration` and `baseDuration` describe React render work, not total user-perceived latency. They do not include all event handling, network, browser layout, painting, or time before React receives the update. Use browser tools to investigate those parts.

Profiling adds overhead, and the regular production build disables profiling instrumentation by default. Use `<Profiler>` around the subtree you need to understand, not around every component without a measurement question.

## 7. Read a flamegraph without over-interpreting it

Each component entry represents work for that component and its descendants in the selected render. A wide or long entry points to an area to investigate, not automatically to a defect.

When a component rendered unexpectedly, ask:

- Did its own state change?
- Did a parent render and pass new props?
- Did a context value change?
- Did the component mount again because its type or key changed?
- Did an Effect schedule another update?
- Is the work expensive for realistic data, or is it merely visible in the trace?

Inspect changed props where the tool provides them. New object or function identities can cause a memoized child to receive changed props even if their contents look alike. Verify the source of the change and whether it matters before adding memoization.

## 8. Diagnose repeated and cascading updates

A repeated render can have several causes. Follow the trace and state transitions rather than counting console messages alone.

### Parent updates

When a parent renders, React normally calls its child components as it calculates the next tree. A child can render even when the DOM it returns stays the same. A measured child can sometimes skip work with `memo` if its props are stable, but only apply that after profiling shows a useful opportunity.

### Derived state in an Effect

If a value can be calculated from current props or state, storing it in state and recomputing it in an Effect often adds a render pass. Calculate it during render; use `useMemo` only when the calculation is measurably expensive and its dependencies are clear.

### Effect-driven updates

An Effect that updates state can cause another render. Check whether it truly synchronizes with an external system. If it only derives one value from another, remove the redundant state. If it subscribes, starts a timer, or connects to a service, make cleanup undo the setup.

### State placed too high

Typing into a field can render a large parent subtree when its state lives at the top of the application. If other regions do not depend on that input, moving the state closer to the field may reduce the work. Confirm the relationship in a trace before changing the tree.

### Unstable values

An inline object or callback has a new identity each time its owner renders. This can invalidate memoized children or Effect dependencies. First check whether the receiving child is actually costly; then consider moving a value, deriving it outside the component, or memoizing the specific value or callback.

## 9. Correlate React work with browser work

Use the browser Performance panel when the page is slow but React's component render time is small. The delay may come from:

- A long event handler before React schedules an update.
- JavaScript parsing or other application code.
- Network or data processing.
- Layout, style calculation, or paint.
- A large image, font, or other resource.

A React flamegraph and a browser timeline answer related but different questions. Correlate timestamps and repeat the interaction instead of assuming that the most visible React component caused the whole delay.

## 10. Make a before-and-after comparison

A useful optimization experiment is narrow and repeatable:

1. State the user-visible problem and the expected improvement.
2. Record the same interaction with the same data and environment.
3. Identify the work that contributes to the symptom.
4. Make one targeted change.
5. Repeat the same recording.
6. Compare both the React work and the user's outcome.

Keep the change only if it improves the relevant behavior without making the code harder to reason about or changing the interface incorrectly. A lower render duration is not useful if it increases total latency or introduces stale UI.

## 11. Common profiling mistakes

- **Profiling the entire session:** record one interaction and keep the trace focused.
- **Using unrealistic tiny data:** repeat with data that resembles actual use.
- **Treating every render as wasted work:** confirm what changed and whether the render is expensive.
- **Counting `console.log` calls as a precise render metric:** Strict Mode and discarded render work can make logs confusing; use profiling tools.
- **Comparing unrelated builds or machines:** keep the environment consistent.
- **Optimizing before identifying a cause:** record first, change one thing, and compare.
- **Ignoring browser work:** React render duration does not include all layout, paint, network, and application code.
- **Removing Strict Mode to get quieter traces:** keep development checks and understand what they expose.
- **Leaving broad profiling instrumentation everywhere:** measure the subtree that answers the current question and remove temporary diagnostics when finished.

## Summary

Development profiling shows which React work happened during a chosen interaction. Distinguish render, commit, and Effects; inspect the subtree and changed inputs; and look for cascades or expensive work that matches the user-visible symptom. Use React-specific tracks for component behavior and browser Performance tools for the full page. Make a narrow change and compare the same interaction before and after, accounting for development instrumentation and Strict Mode.
