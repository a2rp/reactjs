# useEffectEvent and Non-Reactive Event Logic - Q&A

## Core concepts

### 1. What does **useEffectEvent** do?

It creates an Effect Event: logic that an Effect or another Effect Event can call to handle an event from an external synchronization while reading the latest committed props and state.

### 2. What problem is it designed to solve?

An Effect may need to re-synchronize when one value changes, while a callback fired by that synchronization needs to read other current values without making the Effect restart for those changes.

### 3. What is the difference between reactive Effect logic and an Effect Event?

Effect logic re-runs when its reactive dependencies change. An Effect Event is called by the Effect or its external callback and can read the latest committed values without making those values trigger that Effect.

### 4. How is an Effect Event different from a user event handler?

A user event handler runs because of a particular user interaction, such as a click or submit. An Effect Event runs only when Effect logic calls it, often in response to a callback from a connected external system.

### 5. Is an Effect Event a browser event listener?

No. It is a React function created by **useEffectEvent**. It can be called from Effect logic that handles a browser or service event, but it does not register that listener itself.

### 6. In which React version is **useEffectEvent** available?

It is available in React 19.2 and later. The project's React version and Hooks lint tooling must support the API.

## Dependencies and synchronization

### 7. Which values should stay in an Effect dependency list?

Reactive values that determine the external synchronization. For a chat connection, that can include the selected room ID and server address.

### 8. Which values might move into an Effect Event?

Values needed only to handle an event emitted by the synchronization, when changing those values should not restart or replace the external resource. A notification theme is a common example.

### 9. Why can changing a chat notification theme cause an unnecessary reconnect?

If the Effect reads the theme directly, the theme must be a dependency. React then restarts the connection when the theme changes. If the theme only affects the notification shown after connecting, an Effect Event can read its latest value instead.

### 10. Does **useEffectEvent** let me omit a true dependency?

No. If changing a value should change the external resource or synchronization, that value remains an Effect dependency. Moving it into an Effect Event would hide a real dependency and cause stale synchronization.

### 11. Can I use **useEffectEvent** to silence the dependency linter?

No. It is not a dependency-linter escape hatch. Use it only for logic that is genuinely an event emitted from Effect logic and should not itself cause that Effect to re-synchronize.

### 12. Should an Effect Event itself be included in the dependency array?

No. Effect Event functions are intentionally not stable and are excluded from Effect dependencies. Use current lint tooling that recognizes the Hook.

### 13. If a theme changes while a connection is open, does the connection update?

Not automatically. If the theme only affects later callback behavior, the Effect Event reads the new theme when called. If the external connection itself must receive theme changes, then the theme is part of the synchronization and the Effect should react to it.

### 14. Can I move the room ID into an Effect Event so the connection will not restart?

Not if changing the room ID means the component must connect to a different room. Keep that ID in the Effect's dependencies so the old connection is cleaned up and the new one is established.

## Latest values and event arguments

### 15. What does “reads the latest values” mean?

When the Effect Event is called, its callback can read the latest committed props and state, rather than only the values from the render that first set up the external callback.

### 16. Why pass the room ID into an Effect Event?

The argument can represent the room associated with the particular connection event. The Effect Event can use that event-specific value while reading other values, such as the current theme, from the latest render.

### 17. Why not read every value from inside the Effect Event?

Doing so can hide which changes are supposed to cause an event. Values that define when the Effect should run should remain explicit dependencies and may be passed as arguments to make the event's cause clear.

### 18. How can an Effect log each page visit with the latest cart size?

Make the URL the Effect dependency because each URL change represents a visit. Pass the visited URL to the Effect Event, and read the current cart size inside the Effect Event if cart changes should not count as new visits.

### 19. What happens if an asynchronous callback runs after its Effect inputs change?

An Effect Event reads the latest committed values when called. If the callback must preserve the original input that started its work, pass that original value as an argument from the Effect's closure.

### 20. Can an Effect Event accept parameters?

Yes. Its callback can accept arguments and return a value. Arguments are useful for carrying event-specific information from the Effect that invoked it.

## Calling and ownership rules

### 21. Where must I call **useEffectEvent**?

At the top level of a React component or custom Hook. Do not call it conditionally, inside a loop, or inside a nested callback.

### 22. Where can I call the function returned by **useEffectEvent**?

From an Effect or another Effect Event in the same component or custom Hook. Do not call it during rendering or directly from a user event handler.

### 23. Can a button's click handler call an Effect Event?

No. The button handler should directly handle the user's action. Effect Events are restricted to Effect logic.

### 24. Can I pass an Effect Event to a child component?

No. Keep it local to the component and Effect that use it. Do not pass the returned function to other components.

### 25. Can I pass an Effect Event as the callback parameter to another Hook?

No. Do not pass the returned Effect Event to another Hook. A custom Hook may call **useEffectEvent** internally and use it within an Effect that the custom Hook itself owns.

### 26. Is an Effect Event function identity stable?

No. Its identity intentionally changes on each render. It should not be used as a stable callback or included in a dependency list.

### 27. Why does React make its identity unstable?

The API is intended for local calls from Effect logic, not for passing around as a stable callback. A stable identity could hide incorrect dependencies and make Effect Events look like general-purpose callbacks.

## Choosing the right pattern

### 28. Should a message sent from a Send button use an Effect Event?

No. Sending the message is caused by a user action, so send it directly from the click or form handler.

### 29. Should a connection lifecycle be inside an Effect Event?

No. The connection setup and cleanup are the reactive synchronization. Put them in an Effect with dependencies that determine the connection. Use an Effect Event only for event-like callback logic that should not restart it.

### 30. What if I only need to calculate a value for the UI?

Calculate it during rendering. **useEffectEvent** does not replace derived render logic.

### 31. What if a changed value should restart a timer or subscription?

Keep that value as an Effect dependency. The Effect Event should not hide a value that defines the external resource's lifetime or configuration.

### 32. What if the callback is shared by several components or hooks?

Do not pass an Effect Event around. Extract ordinary reusable logic or write a custom Hook that owns its own Effect and Effect Event.

### 33. Do all Effects need an Effect Event?

No. Most Effects do not. Use an Effect Event only when the Effect has event-like callback logic that needs current values but should not make the synchronization reactive to those values.

## Troubleshooting

### 34. The Effect reconnects when only a notification preference changes. What should I check?

Check whether the preference is used only while handling a connection event. If so, move that event-specific notification behavior into an Effect Event. Keep actual connection inputs in the Effect dependencies.

### 35. The Effect fails to reconnect when the room changes. What should I check?

Make sure the room ID is read by the Effect setup and is included in its dependency list. Do not move it into an Effect Event if it selects the external room.

### 36. The linter asks for an Effect Event in the dependency list. What should I do?

Update **eslint-plugin-react-hooks** to a version that recognizes **useEffectEvent**. Do not add the Effect Event to the dependency list.

### 37. Can an Effect Event prevent stale closures?

It lets its callback read the latest committed props and state at call time. But it does not excuse omitting real synchronization dependencies or passing an old event-specific value when the original value must be preserved.

### 38. How can I check whether my use is legitimate?

Confirm that the function is called only by Effect logic, that it handles an event from that Effect's synchronization, that the Effect still lists every value that should restart the synchronization, and that the Effect Event is not passed elsewhere.

## Quick recap

- **useEffectEvent** extracts event-like callback logic from a reactive Effect.
- The Effect Event reads the latest committed props and state when invoked.
- Keep true synchronization inputs in the Effect dependencies.
- Pass the event's original triggering value as an argument when it must be preserved.
- Effect Events are neither user event handlers nor stable callbacks.
- Call them only from Effect logic where they are declared; do not pass them to components or other Hooks.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="055-when-an-effect-is-unnecessary.md">055 - When an Effect Is Unnecessary - Q&amp;A</a></td><td><a href="../lessons/056-useeffectevent.md">Lesson</a></td><td><a href="../examples/056-useeffectevent.md">Example</a></td><td><a href="057-custom-hooks-and-usedebugvalue.md">057 - Custom Hooks and useDebugValue - Q&amp;A</a></td></tr>
  </tbody>
</table>
