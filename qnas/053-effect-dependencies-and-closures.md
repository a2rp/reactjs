# Effect Dependencies, Closures, and Synchronization - Q&A

## Closures and render snapshots

### 1. What is a closure?

A closure is a function together with access to the variables from the scope where that function was created. A callback can keep using those values after the outer function has returned.

### 2. What does it mean that a render is a snapshot?

Each call to a function component receives the props and state for that render and creates its own local values and functions. Event handlers and Effect callbacks created during that render see those values. A later render creates a new snapshot; it does not rewrite old functions.

### 3. Does calling a state setter change the state variable inside the current function?

No. A setter requests another render. The current function continues to see the state snapshot it started with. The next render receives the updated value.

### 4. Why can an interval callback read old state?

The interval retains the callback registered by the Effect setup. That callback closes over the values from the render that created it. If the Effect does not re-synchronize when a relevant value changes, the interval may keep using the earlier values.

### 5. What is a stale closure bug?

It occurs when a long-lived callback keeps values from an older render even though the UI or application now has newer values. A missing Effect dependency is a common cause, but stale closures can also arise from timers or callbacks registered outside React.

### 6. Does every callback from a past render become a bug?

No. It is expected for a callback to retain the values from its render. That behavior is useful for event handlers that should act on the interaction's snapshot. It becomes a bug when a long-lived external synchronization is expected to follow a value but is not updated when that value changes.

## Reactive values and dependencies

### 7. Which values are reactive to an Effect?

Props, state, values calculated in the component body, and functions declared in that body are reactive. They may have different values or identities on a later render.

### 8. Is a local variable reactive if it is calculated from props?

Yes. If the component calculates a variable during rendering from props or state, it may change when those inputs change. If the Effect reads that variable, include it or restructure the code so the Effect reads the actual values it needs.

### 9. Are values declared at module scope reactive?

A stable value or helper declared outside the component does not change because the component rendered again, so it does not need to be an Effect dependency. Avoid mutating such shared values; React does not automatically observe mutations.

### 10. Do state setter functions need to be dependencies?

React guarantees that a setter returned by useState has stable identity. It is safe to omit it. Including it is also safe because it does not change between renders.

### 11. Does the ref object returned by useRef need to be a dependency?

Its identity is stable, so it may be omitted. Including the ref object is harmless, but it does not make changes to its current property reactive.

### 12. Why is ref.current not a reactive dependency?

Changing ref.current does not request a render. React therefore has no new render in which it could compare the dependency and restart the Effect. Use state for values that drive rendered UI, or subscribe to an external store through an appropriate API.

### 13. Can I list a global mutable value as a dependency?

Listing it does not make it reactive. If the value changes outside React's render flow, React will not know to render and compare it. Use the router or a supported subscription mechanism for externally changing data.

### 14. What does the dependency list mean?

It declares the reactive values read by setup and cleanup. React compares the corresponding entries between committed renders using Object.is. If an entry differs, React cleans up the old synchronization and starts it again with the new values.

### 15. Can I choose dependencies to make an Effect run on a preferred schedule?

No. The dependency list must match the code. To change when synchronization should restart, change which reactive values the Effect reads and how the work is structured.

### 16. Are dependency arrays compared as objects?

No. React compares their entries one by one. The array is written inline with a constant number and order of entries; its own identity is not the trigger.

### 17. What is the difference between no dependency argument and an empty list?

With no dependency argument, the Effect runs after every commit. An empty list says that the Effect reads no reactive values that should cause it to re-synchronize. It does not silence dependency checking.

### 18. What happens if a reactive dependency is missing?

The Effect may keep using a value from an earlier render. For a connection, that can mean remaining in the wrong room. For a timer, it can mean repeatedly using the initial count or increment.

### 19. What does the exhaustive-deps lint rule do?

It checks that reactive values used by an Effect are listed as dependencies. The React Hooks ESLint plugin can report this and other React rule violations. A warning often reveals either a missing dependency or code that should be reorganized.

### 20. Should I disable exhaustive-deps when the list is inconvenient?

No. A disabled warning or false dependency list can leave stale callbacks active. Include the reactive value, then restructure the code if synchronizing on that value is not the intended behavior.

## Object and function dependencies

### 21. Why does an object dependency often cause repeated synchronization?

An object literal created during render has a new identity on each render. Even if its fields contain the same values, React sees a different reference under Object.is and re-runs the Effect.

### 22. How should I handle an object used only inside an Effect?

Create it inside the Effect and list the reactive primitive inputs used to construct it. This avoids making the Effect depend on a fresh wrapper object.

### 23. What if a parent passes an options object?

Read the primitive fields the Effect actually needs and list those fields as dependencies. A newly created wrapper object from the parent then does not restart the Effect unless the actual inputs changed.

### 24. Should I mutate an object to keep its identity stable?

No. Mutation can hide changes from React and makes the data flow harder to reason about. Use immutable values and dependencies that describe the actual inputs.

### 25. Why does a function declared in a component cause re-synchronization?

A new function object is created on each render. If an Effect depends on that function, React sees a changed dependency even when the function's behavior appears the same.

### 26. How can I avoid a function dependency that is only used by one Effect?

Move the helper inside the Effect setup. If the helper reads no component values, move it outside the component. In both cases, keep dependencies for the reactive values that determine the synchronization.

### 27. Should I always wrap a function in useCallback to fix this?

No. useCallback can preserve identity when a consumer needs a stable callback, but it can add complexity without improving the design. First consider moving an Effect-only helper inside the Effect or a non-reactive helper outside the component.

### 28. Does useMemo make a dependency correct?

Memoization can stabilize an object or function when stable identity is genuinely part of an API. It does not justify omitting reactive values that the Effect reads. The dependency list still has to reflect the code.

## State updaters and callback state

### 29. How can a functional state updater remove a state dependency?

If the Effect only reads state to calculate the next state, pass an updater function to the setter. React provides the latest pending state to that updater, so the Effect no longer closes over that state variable.

~~~jsx
useEffect(() => {
  const timerId = window.setInterval(() => {
    setCount((currentCount) => currentCount + 1);
  }, 1000);

  return () => window.clearInterval(timerId);
}, []);
~~~

This Effect does not read count. Its callback asks React to increment whatever the current count is when the update is processed.

### 30. Why not put the current count in the dependency list instead?

That may be correct if each count change should restart the external process. For an interval that should keep running continuously, it would clear and recreate the interval after every tick. The updater expresses the intended state transition without coupling the interval lifetime to the count.

### 31. Does a state updater let me omit any dependency?

No. It only removes a dependency when the Effect no longer reads that value. If the Effect also reads a changing step, room, URL, or other reactive input, that input remains a dependency.

### 32. Why can reading a messages array reconnect a chat?

If the Effect reads messages and lists it as a dependency, each new message changes that array and restarts the Effect. Use a state updater to append the new message without reading the previous array from the Effect closure.

### 33. If I add all missing dependencies, will the code always behave as intended?

The closure will be current, but the Effect may restart more often than desired. That is a signal to inspect the synchronization and restructure it, not a reason to falsify the dependency list.

## Debugging and code changes

### 34. How can I find which dependency changes unexpectedly?

Log the dependency values from consecutive renders and compare the corresponding items with Object.is. A newly created object or function is a common cause. Remove temporary logs after finding the source.

### 35. What should I do when adding a dependency causes repeated setup?

First verify that the Effect represents one external synchronization. Then check whether an object or function is recreated, whether an updater can avoid reading previous state, or whether a value can be moved into setup or to module scope. Keep any dependency that truly changes the synchronization target.

### 36. Can I compare objects by their fields instead of identity in a dependency list?

React compares dependency items by identity using Object.is; it does not deep-compare fields. Extract the fields that determine the external process and list those values individually.

### 37. What if an Effect needs the latest value but should not restart when that value changes?

Do not omit the dependency by hand. That requires an intentional design for separating reactive synchronization from logic that merely reads the latest value. The later useEffectEvent chapter covers React's dedicated pattern for this case.

### 38. What if the work should happen only after a specific click or submit?

That may be interaction-specific work rather than synchronization. Handle it in the corresponding event handler. The later chapter on unnecessary Effects explores this decision in more detail.

### 39. How are Effect dependencies different from values captured by an event handler?

Both functions close over the render that created them. An event handler normally runs in response to that interaction and uses that render's snapshot. An Effect controls an ongoing synchronization, so its reactive dependencies tell React when to replace that synchronization with one based on newer values.

### 40. What is the safest way to change a dependency list?

Change the surrounding code first so it reads the intended reactive values. Then make the list match that code, follow the linter, and verify that the external process starts, updates, and stops under the intended conditions.

## Key points

- Closures keep the values from the render that created their functions.
- Reactive values include props, state, derived render values, and component-body functions.
- Dependencies describe the Effect's code; they are not a manual scheduling control.
- React compares dependency entries with Object.is.
- Missing dependencies can create stale closures.
- Fresh objects and functions can trigger unnecessary re-synchronization.
- Use state updaters to calculate from previous state without capturing it.
- Refs and mutable globals do not trigger renders and cannot make an Effect reactive.
- Change code instead of suppressing the dependency linter.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="052-effect-hooks-and-synchronization.md">052 - Effect Hooks and Synchronization - Q&amp;A</a></td><td><a href="../lessons/053-effect-dependencies-and-closures.md">Lesson</a></td><td><a href="../examples/053-effect-dependencies-and-closures.md">Example</a></td><td><a href="054-effect-cleanup-and-resource-lifetimes.md">054 - Effect Cleanup and Resource Lifetimes - Q&amp;A</a></td></tr>
  </tbody>
</table>
