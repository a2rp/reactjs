# Q&A: React 19.2 Activity and Performance Tooling

## 1. What is the main focus of React 19.2 in this chapter?

This chapter focuses on `<Activity>`, `useEffectEvent`, and React Performance Tracks. They address different needs: retaining hidden UI state, separating Effect-triggered event logic, and investigating React work in a browser performance trace.

## 2. What does `<Activity>` do?

It can hide a child UI while preserving its React state, then restore that UI state when shown again. It also controls scheduling and the lifecycle of Effects inside the boundary.

## 3. What are the two Activity modes in React 19.2?

`visible` displays the children and mounts their Effects. `hidden` hides the children, cleans up their Effects, and defers hidden updates at lower priority.

## 4. Does hidden Activity destroy a component's local React state?

No. Preserving UI state for later restoration is one of Activity's main uses. Its Effects are cleaned up while hidden and recreated when it becomes visible.

## 5. What does Activity use to hide its children?

A hidden Activity visually hides its children with `display: none`. Hidden content is not visible or focusable while hidden.

## 6. How is Activity different from conditional rendering?

Conditional rendering such as `{open && <Panel />}` removes the panel from the tree when false, which discards its local state. Activity keeps the boundary and state while hiding the children and disconnecting their Effects.

## 7. Should Activity replace conditional rendering everywhere?

No. Use conditional rendering when the content should be discarded or when retaining it offers no user benefit. Activity is useful when restoring a screen's local state or preparing likely next content justifies keeping it around.

## 8. What kinds of UI are good Activity candidates?

A tab panel, editor, search view, or workspace that a user is likely to revisit and where restoring a draft, filter, or scroll-related UI state would help.

## 9. Does Activity guarantee a faster screen?

No. It may let likely next content render at lower priority and start compatible render-driven work earlier, but it can also add background rendering and retain memory. Measure with the actual interaction and content.

## 10. Does Activity automatically fetch data and preload every asset?

No. Activity is not a general prefetch command. It can allow work initiated by the rendered content or a compatible data/loading integration to begin, but it does not automatically fetch arbitrary resources.

## 11. Do Effects inside a hidden Activity continue to run?

No. React runs their cleanup when the boundary becomes hidden. When it becomes visible again, React sets those Effects up again.

## 12. What should an Effect do to work correctly with Activity?

It should clean up every external resource it starts, such as subscriptions, timers, or event listeners. Setup should be able to run again without creating duplicate connections or leaking old work.

## 13. Does Activity automatically pause every browser API used by a component?

No. The component's Effect cleanup must stop timers, media subscriptions, listeners, or other external work. Activity handles React's Effect lifecycle; it does not infer how arbitrary browser resources should be managed.

## 14. Can hidden Activity children receive new props?

Yes. Hidden children can still rerender in response to new props, but their work is lower priority than visible content.

## 15. Does a hidden Activity always produce hidden DOM for text-only children?

A hidden Activity relies on a DOM element to apply its visual hiding. A component that returns only text has no element on which to apply `display: none`; React documents that it will not produce output in the DOM in that case.

## 16. What should server-rendered applications consider before using Activity?

Hidden Activity content is not included in the server HTML and may render on the client later. Visible Activity content can affect hydration priority. Test the application's framework integration and actual hydration behavior.

## 17. What is `useEffectEvent`?

It is a React 19.2 Hook for separating event-like callback logic that is fired by an Effect from the Effect's synchronization logic.

## 18. What value does an Effect Event read?

It sees the latest committed props and state when called, without forcing the surrounding Effect to resynchronize because those values changed.

## 19. When should `useEffectEvent` be used?

When an external event emitted by an Effect-triggered connection needs to call logic using the latest values, but changing those values should not restart the connection itself.

## 20. When should `useEffectEvent` not be used?

Do not use it for regular user click handlers, to pass callbacks to child components, to call during rendering, or to hide a real Effect dependency. It should not be used simply to quiet the dependency linter.

## 21. Should an Effect Event appear in its Effect's dependency array?

No. The returned Effect Event function is deliberately excluded. Keep the actual synchronization dependencies, such as a connection's room ID, in the array.

## 22. Can every dependency be removed by moving callback logic into an Effect Event?

No. If a value changes which external system the Effect must connect to or synchronize with, it remains a real dependency. An Effect Event is only for event-like logic triggered by that Effect.

## 23. What are React Performance Tracks?

They are React-specific custom entries in a browser's Performance panel timeline. They show React work alongside JavaScript, network requests, and other browser activity.

## 24. What does the Scheduler track show?

It shows React's work and priorities, including blocking work, transitions, Suspense work, and idle work. It can help trace what scheduled an update and how React rendered and committed it.

## 25. What does the Components track show?

It visualizes component render and Effect durations in a tree-like timeline. It can help identify which subtree consumed time and when Effects mounted, disconnected, reconnected, or ran.

## 26. Do Performance Tracks explain the cause of every slow interaction automatically?

No. They show timing and relationships. You still need to inspect the interaction, code, props, Effects, network activity, browser work, and any relevant third-party code to determine the cause.

## 27. Where do React Performance Tracks appear?

They appear in performance recordings in browser developer tools that support the required Performance panel extensibility, including Chrome DevTools. React DevTools is also useful, but its Profiler and the browser's Performance panel provide different views.

## 28. Are Performance Tracks enabled in a regular production build?

No. They are available in development and profiling builds; production instrumentation is disabled by default because profiling adds overhead.

## 29. Why can a development trace differ from production?

Development builds include extra checks and instrumentation, and profiling builds also change instrumentation. Use those builds to locate React work, then verify the user experience and browser performance with the actual production configuration.

## 30. What is the difference between the React DevTools Profiler and browser Performance panel?

The React DevTools Profiler focuses on React commits and component rendering. The browser Performance panel can place React's custom tracks beside JavaScript execution, network activity, event loop, and other browser timing on one timeline.

## 31. How should a useful trace be recorded?

Record a short, repeatable interaction. Find that action on the timeline, inspect React tracks and surrounding browser work, identify a likely cause, make one targeted change, and record the same steps again for comparison.

## 32. What does `startTransition` do in this context?

It marks an update as non-blocking background work so more urgent interactions can stay responsive. It is useful when preparing a genuinely expensive view, not as a wrapper for every state update.

## 33. Should every tab switch be placed inside `startTransition`?

No. A tab selection is often expected to respond immediately. Mark only the expensive result as a transition when doing so improves the interaction and the pending state remains understandable.

## 34. What is `cacheSignal` in React 19.2?

It is an API for React Server Components that exposes the lifetime of a server cache operation. It is not a client-side fetch shortcut.

## 35. What is partial pre-rendering?

It is an advanced server-rendering capability for producing a static shell and resuming dynamic rendering later. Use it through a compatible framework or server integration and follow that stack's guidance.

## 36. Did React 19.2 change the default `useId` prefix?

Yes. React 19.2 changed the default internal prefix. Application code should not depend on the exact string generated by `useId`; use the returned ID for its intended relationships.

## 37. How can Activity be tested for accessibility?

Verify the visible navigation or selection control communicates the active view. Confirm hidden content is not keyboard-focusable, focus remains sensible when switching views, and state restoration does not confuse screen-reader users.

## 38. What is the main principle for using React 19.2's performance features?

Choose the feature that matches a measured problem. Use Activity when state retention or background preparation helps, Effect Events only for Effect-fired events, and Performance Tracks to locate work before optimizing.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="117-react-upgrades-and-version-changes.md">117 - Q&amp;A: Upgrading React and Handling Version Changes</a></td><td><a href="../lessons/118-react-19-2-features.md">Lesson</a></td><td><a href="../examples/118-react-19-2-features.md">Example</a></td><td><a href="119-react-19-3-features.md">119 - Q&amp;A: React 19.3 View Transitions, Fragment refs, and browser APIs</a></td></tr>
  </tbody>
</table>
