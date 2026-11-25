# React 19.2: Activity and Performance Tooling

React 19.2 introduced `<Activity>`, Effect Events with `useEffectEvent`, and React-specific Performance Tracks in the browser Performance panel. These features help preserve useful interface state, manage work that belongs to Effects, and investigate where rendering time is spent.

This chapter focuses on when these features help and the behavior to check before adopting them. They are tools for specific problems, not automatic performance switches.

## Learning goals

After this lesson, you should be able to:

- explain the visible and hidden modes of `<Activity>`;
- preserve the state of a likely-to-return screen without confusing hidden Effects with mounted Effects;
- use `useEffectEvent` only for event logic fired from an Effect;
- record a browser performance trace and read React's Scheduler and Components tracks;
- distinguish a React render cost from unrelated JavaScript, network, or browser work; and
- evaluate React 19.2's related release changes and production tradeoffs.

## 1. Start from the user-visible problem

Performance work should begin with a concrete observation:

- Does switching back to a screen lose a draft or filter?
- Does a particular interaction take too long?
- Is the delay caused by React rendering, an Effect, JavaScript outside React, a request, or browser layout and paint?
- Does preparing likely next content improve the user's wait enough to justify the extra rendering and memory?

Write down the interaction and expected result before changing code. Capture a baseline trace when investigating performance. Without a baseline, it is easy to add complexity without improving what users notice.

## 2. What `<Activity>` does

React 19.2 provides the `Activity` component with two modes:

```jsx
import { Activity } from "react";

<Activity mode={isSelected ? "visible" : "hidden"}>
  <SettingsPanel />
</Activity>
```

- **visible:** the children are shown and their Effects are mounted.
- **hidden:** React hides the children using `display: none`, cleans up their Effects, and defers hidden updates at a lower priority. Their React state is preserved. When the Activity becomes visible again, React restores the UI state and runs the Effects again.

This can be useful for tabs, panels, editors, or screens that a person is likely to revisit and whose transient state is valuable. For example, switching away from an editor need not discard the draft held in the editor's component state.

A hidden Activity can also render likely next content at lower priority. If rendering that content starts work through a compatible lazy-loading or data integration, the work may be ready sooner when the content is shown. Activity does not automatically fetch every resource or make a slow screen fast.

## 3. Activity compared with conditional rendering

Conditional rendering removes a component from the tree:

```jsx
{isOpen && <SearchPanel />}
```

When this becomes false, the component is removed and its local state is lost. Its Effects clean up because it unmounts.

Activity keeps a boundary for the content and changes its visibility:

```jsx
<Activity mode={isOpen ? "visible" : "hidden"}>
  <SearchPanel />
</Activity>
```

The component's React state is retained while hidden. Effects are cleaned up when it becomes hidden and set up again when it becomes visible. This distinction matters:

- A draft in React state can remain available when the panel returns.
- A subscription or timer must still have correct cleanup and setup logic.
- Hidden content can still receive new props and be rendered at lower priority.
- Preserving many large hidden trees consumes resources and retained state.

Use conditional rendering when the content should be discarded or does not need to remain warm. Use Activity when preserving the state or preparing likely next content provides a measured user benefit.

## 4. Effects must tolerate disconnect and reconnect

An Activity can disconnect Effects without discarding the component's React state. An Effect should already describe how to start and stop an external synchronization:

```jsx
useEffect(() => {
  const connection = connectToRoom(roomId);

  connection.on("message", handleMessage);
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

When the Activity hides, React runs the cleanup. When it becomes visible, React runs the setup again. If cleanup forgets to remove an event listener, stop a timer, or disconnect a subscription, hiding and restoring the Activity can expose a leak or duplicate connection.

Activity does not stop arbitrary browser work by magic. The component's Effects must clean up the resources they start. Strict Mode in development can help reveal incorrect setup/cleanup assumptions.

## 5. Accessibility and rendering considerations

Hidden Activity children are visually hidden with `display: none`, so they are not visible or focusable while hidden. Still, the surrounding controls must communicate which screen is selected and keep keyboard behavior coherent.

If rendering a static text value inside a hidden Activity without an element, React has no DOM element on which to apply `display: none`. Prefer normal semantic structure for visible content and test the actual rendered output.

For server-rendered applications, hidden Activity content is not included in the server HTML and can be rendered on the client later. Visible Activity content can also affect hydration priority. Follow the framework's server-rendering guidance and test the real hydration flow before using Activity around server-rendered content.

## 6. Use `useEffectEvent` for Effect-triggered event logic

React 19.2 also introduced `useEffectEvent`. It separates an event-like callback fired from an Effect from the Effect's synchronization dependencies.

For example, a room connection depends on `roomId`, while the notification should use the latest `theme` without reconnecting just because the theme changed:

```jsx
import { useEffect, useEffectEvent } from "react";

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification("Connected!", theme);
  });

  useEffect(() => {
    const connection = createConnection(roomId);

    connection.on("connected", () => {
      onConnected();
    });

    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return null;
}
```

Effect Events read the latest committed props and state when called. They are not added to the Effect dependency array.

Important restrictions:

- Call `useEffectEvent` at the top level of a component or custom Hook.
- Call the returned function only from an Effect or another Effect Event in the same component or Hook.
- Do not call it from rendering or a regular click handler.
- Do not pass it to child components or other Hooks.
- Do not use it to hide a real Effect dependency or silence the dependency linter.

If a prop changes the external connection itself, it remains a real dependency and the Effect should resynchronize.

## 7. Read React Performance Tracks

React 19.2 added React-specific custom tracks to browser Performance panel recordings. These tracks place React work on the same timeline as JavaScript execution, network requests, and browser activity.

The main client-side tracks are:

### Scheduler track

The Scheduler track shows the priority and sequence of React work, including blocking updates, transitions, Suspense work, and idle work. It can show the update that scheduled a render, the render and commit phases, and when React yields or waits.

Use this track to ask:

- Did the interaction schedule urgent or background work?
- Did a transition get interrupted by more urgent input?
- Did React perform repeated or cascading work?
- Is a long delay inside React, or is React waiting for something else?

### Components track

The Components track visualizes component render and Effect durations. It uses a flamegraph-like view to show where time is spent in component trees. Development traces can also help identify changed props associated with a render.

Use this track to ask:

- Which subtree accounts for a notable part of the render?
- Did an Effect take time or trigger another update?
- Did an Activity reconnect or disconnect an Effect?
- Which components rendered during the interaction?

Tracks show timing and relationships; they do not decide whether a render was unnecessary. Inspect the component logic and the user interaction that scheduled it.

## 8. Capture a useful trace

A practical profiling loop:

1. Use a development or profiling build supported by the project.
2. Open the browser's Performance panel.
3. Record a short trace before the interaction.
4. Perform one representative interaction, such as switching between panels or typing in a large list.
5. Stop the recording and find the interaction on the timeline.
6. Inspect React Scheduler and Components tracks beside JavaScript, network, and browser work.
7. Identify the longest meaningful work and its cause.
8. Change one thing and record the same interaction again.
9. Compare duration and user-visible responsiveness with the baseline.

Keep recordings focused. A long trace with unrelated activity makes it harder to see which action caused the work.

## 9. Development, profiling, and production builds

React Performance Tracks are available in development and profiling builds. Development builds enable them by default. Profiling builds have additional configuration and can have different overhead characteristics. The Components track in a profiling build may require `<Profiler>` wrappers unless the React Developer Tools extension is enabled, depending on the setup.

Instrumentation adds overhead, so the tracks are disabled in production by default. Use them to understand the component work, then verify important results with the production configuration and the browser's normal performance tooling. Do not treat development timings as exact production timings.

The React Developer Tools Profiler and the browser Performance panel answer related but different questions. The React DevTools Profiler is useful for inspecting React commits and component renders. The browser Performance panel places React's tracks beside browser, JavaScript, network, and event-loop work on a shared timeline.

## 10. Understand transitions without making all work non-urgent

A transition marks an update as non-blocking work. It can help React keep urgent interactions responsive while preparing a more expensive result:

```jsx
import { useState, useTransition } from "react";

function ReportPicker({ reports }) {
  const [selectedReport, setSelectedReport] = useState(null);
  const [visibleReport, setVisibleReport] = useState(null);
  const [isPending, startTransition] = useTransition();

  function selectReport(report) {
    setSelectedReport(report); // Update the control immediately.

    startTransition(() => {
      setVisibleReport(report); // Prepare the heavier view in the background.
    });
  }

  return (
    <>
      <ReportList reports={reports} onSelect={selectReport} />
      {isPending && <p role="status">Preparing report…</p>}
      {visibleReport && <Report report={visibleReport} />}
    </>
  );
}
```

This is only useful when the resulting work is actually expensive and can be deferred without making the interface confusing. Do not mark every update as a transition. Use the Scheduler track to verify the priority behavior and inspect the result on a realistic dataset.

## 11. Other React 19.2 changes to know

The 19.2 release also included features and behavior relevant to some applications:

- **Partial pre-rendering and resume APIs:** advanced server-rendering capabilities for producing a shell and resuming work later. Frameworks may manage these APIs; follow their integration guidance rather than adding server calls directly to a client app.
- **`cacheSignal`:** a React Server Components API that exposes the lifetime of a server cache request. It is for server components, not a client-side fetching shortcut.
- **Server rendering streams:** React 19.2 expanded Web Streams support in Node. The React release notes recommend Node stream APIs for many Node deployments because of performance and compression considerations.
- **Suspense reveal behavior:** server-rendered Suspense boundaries are batched briefly to align reveals more closely with client rendering, subject to React's heuristics.
- **`useId` prefix:** the default generated prefix changed. Applications should not depend on the old internal prefix string.

These are version-specific release changes. Confirm framework support and current documentation before adopting advanced server-rendering features.

## 12. Activity decision checklist

Before using Activity for a screen, ask:

- Is the visitor likely to return to it soon?
- Would preserving its draft, filter, or view state help?
- Are its Effects safe to clean up and reconnect?
- Is the hidden tree small enough to keep around?
- Could conditional rendering be simpler and sufficient?
- Does the behavior remain correct with server rendering and hydration?
- Did a realistic measurement show a user benefit?

Use Activity where the answer supports it, not as a default wrapper for every route or component.

## Common mistakes

- **Treating Activity like CSS visibility:** Activity also controls Effect lifecycles and scheduling.
- **Expecting hidden Effects to keep running:** they are cleaned up while the Activity is hidden and set up again when visible.
- **Assuming hidden means no rendering:** hidden children can still receive props and render at lower priority.
- **Keeping every screen hidden forever:** retained state and trees consume memory and can do background work.
- **Assuming Activity always improves speed:** it can help likely next content but may add work and resource cost.
- **Using `useEffectEvent` as a dependency escape hatch:** declare genuine synchronization dependencies and use Effect Events only for event logic fired from Effects.
- **Assuming profiling tracks are production instrumentation:** React disables them in production by default because of their overhead.
- **Optimizing a component before reading the whole trace:** the delay may be network, third-party JavaScript, layout, or another browser task.
- **Comparing traces from different interactions:** use the same steps and environment for a useful before/after comparison.
- **Confusing Activity with routing:** a router still owns URL, navigation, history, and route semantics.

## Key takeaways

- `<Activity>` can hide content while retaining React state; its Effects disconnect and reconnect with visibility.
- `useEffectEvent` is for event-like logic called from an Effect, not a way to skip dependencies.
- React Performance Tracks correlate Scheduler and component work with browser activity in a shared timeline.
- Profile a repeatable user interaction, identify the cause, change one thing, and compare.
- Use development or profiling instrumentation to diagnose, then verify behavior and performance in the actual production setup.
- Adopt React 19.2 features when they solve a measured product problem and fit the framework.
