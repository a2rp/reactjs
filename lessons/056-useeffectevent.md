# useEffectEvent and Non-Reactive Event Logic

## The problem: one Effect, two kinds of logic

An Effect often starts a synchronization that should respond to some values, while its callbacks need to read other values without restarting that synchronization.

Imagine a chat room that must stay connected to the selected room. When the connection reports success, the app also displays a notification using the current theme and the user's current notification preference.

Changing the room should reconnect. Changing the theme should only change how a future notification appears. Those values have different jobs, even though the Effect's callback needs both.

React 19.2 added **useEffectEvent** for separating event-like logic fired by an Effect from the Effect's reactive synchronization logic.

## Learning goals

After this lesson, you should be able to:

- distinguish reactive Effect logic from event-like callbacks fired by an Effect;
- explain why ordinary Effect dependencies must remain complete;
- use **useEffectEvent** to read the latest committed props and state without re-synchronizing an Effect;
- pass event-specific values as arguments when the Effect's triggering value must be preserved;
- follow the API's calling, identity, and ownership restrictions;
- recognize when an event handler, a split Effect, or another state design is more appropriate.

## Reactive Effects and interaction events

Props, state, and values calculated in a component body are reactive: a render can give them new values.

An Effect is reactive. If its setup reads a reactive value, a change to that value normally means React must run cleanup for the old synchronization and setup for the new one.

A user event handler behaves differently. It runs because a particular interaction happened. Typing a new value does not repeat an earlier click handler.

An Effect Event is a third kind of logic: it is invoked from an Effect or another Effect Event, often when an external system reports something. It is not a user event handler and it does not itself cause the Effect to re-synchronize.

| Logic | What causes it to run again? | Typical purpose |
| --- | --- | --- |
| Render calculation | A render with current inputs | Calculate UI values |
| User event handler | The user performs that interaction again | Submit, save, select, or send |
| Effect setup | The component needs to synchronize with current reactive inputs | Connect, subscribe, observe |
| Effect Event | An Effect or its external callback invokes it | Handle an event from the synchronization using latest values |

## Separate the synchronization from its event callback

Without an Effect Event, the theme is a reactive value read directly in the Effect. It must be a dependency, so changing the theme reconnects the room:

~~~jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(roomId);

    connection.on("connected", () => {
      showNotification("Connected to " + roomId, theme);
    });

    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);
}
~~~

If the theme affects only the appearance of a notification, restarting the connection for a theme change is not part of the intended synchronization.

Move only the notification logic into an Effect Event:

~~~jsx
import { useEffect, useEffectEvent } from "react";

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent((connectedRoomId) => {
    showNotification("Connected to " + connectedRoomId, theme);
  });

  useEffect(() => {
    const connection = createConnection(roomId);

    connection.on("connected", () => {
      onConnected(roomId);
    });

    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
}
~~~

Now the Effect synchronizes the connection with **roomId**. The Effect Event reads the latest committed **theme** when the connection reports success. A theme update does not reconnect the room. A room update still cleans up the previous connection and starts the new one.

The Effect Event is omitted from the dependency array. It is intentionally not a stable dependency and React's Hooks linter understands this API.

## Preserve the value that caused the Effect event

Notice that the Effect passes **roomId** into **onConnected**:

~~~jsx
onConnected(roomId);
~~~

This argument represents the room associated with that particular connection event. The callback can still read the latest **theme** from the component.

Passing the room explicitly is useful when an asynchronous callback may run after the reactive value has changed. It separates:

- the event-specific value from the Effect instance that produced the event;
- the latest values that the Effect Event should read when it runs.

For example, when logging a page visit, the URL is the event being recorded. Pass it from the Effect that reacts to URL changes. A shopping cart count can be read as its latest value inside the Effect Event without making cart changes count as new visits:

~~~jsx
function Page({ url, itemCount }) {
  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, itemCount);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
}
~~~

If **url** were read only inside the Effect Event and the Effect had an empty dependency array, later page visits could be missed. The Effect must remain reactive to values that define when the event should happen.

## The Effect Event always reads latest committed values

An Effect Event's callback can read the latest committed props and state when it is called. This helps with callbacks registered in an Effect that may fire later:

~~~jsx
function PointerTracker({ enabled }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const onPointerMove = useEffectEvent((event) => {
    if (enabled) {
      setPosition({ x: event.clientX, y: event.clientY });
    }
  });

  useEffect(() => {
    function handlePointerMove(event) {
      onPointerMove(event);
    }

    window.addEventListener("pointermove", handlePointerMove);
    return () => {
      window.removeEventListener("pointermove", handlePointerMove);
    };
  }, []);

  return (
    <p>
      Pointer: {position.x}, {position.y}
    </p>
  );
}
~~~

The listener is installed once for the component's lifetime. Its callback invokes the Effect Event, which reads the current **enabled** value. Toggling **enabled** does not require replacing the listener.

The setup must still declare values that really determine the listener or subscription. If this listener were attached to a particular DOM node passed as a prop, and that node changed, the Effect would need to re-synchronize with the new node.

## Do not use Effect Events to hide dependencies

**useEffectEvent** is not a way to silence or work around the dependency linter.

If changing a value means the external synchronization itself must change, that value remains a dependency. For a connection, room ID and server address may determine which resource to connect to. Keep them in the Effect dependency list.

Use an Effect Event only for logic that represents an event fired from that synchronization and should read current values without causing the synchronization to restart.

A useful review question is:

> If this value changes, should the external resource be replaced or reconfigured?

- If yes, it belongs in the Effect's reactive inputs.
- If no, and it is needed only while handling an event from the Effect, it may belong inside an Effect Event.
- If the work is caused by a user action, use an event handler instead.

Do not suppress the dependency linter to force a lifecycle you want. Restructure the code so the dependencies express the actual synchronization.

## API rules and limitations

### Call the Hook at the top level

Call **useEffectEvent** at the top level of a component or of a custom Hook that owns the Effect. Do not call it conditionally, in a loop, or in a nested callback.

### Call the returned function only from Effect logic

Invoke an Effect Event from an Effect or another Effect Event in the same component. Do not call it during rendering or from a click handler.

A click handler should directly handle the user's click. It should not call an Effect Event to get access to the latest values.

### Keep the Effect Event local to its Effect

Do not pass an Effect Event to another component or pass it as a callback argument to another Hook. It belongs next to the Effect that uses it. A custom Hook can define its own Effect Event internally when it also owns the Effect.

### Do not add it to the dependency array

Effect Event functions intentionally do not have stable identity. Do not add one to an Effect's dependency list. It is not an ordinary callback whose identity should control synchronization.

### Use a supported React version and current lint tooling

The Hook is available in **React 19.2 and later**. Use a current **eslint-plugin-react-hooks** version so the dependency linter recognizes Effect Events and enforces their restrictions.

## When not to use useEffectEvent

### The logic is caused by a user's action

Sending a message because the user clicked Send belongs in that click handler. The same applies to submitting a form or purchasing an item. These are interactions, not events emitted by an Effect's external synchronization.

### The Effect does not have an external synchronization to keep

If an Effect only derives one piece of React state from another, remove the Effect and derive the value during rendering or restructure the state. An Effect Event does not make that internal state pipeline a good design.

### The supposedly non-reactive value changes the resource itself

If changing a value means connecting to another server, subscribing to a different topic, or observing a different target, keep it reactive in the Effect. Do not move it into an Effect Event just to avoid a reconnect.

### The callback should be reusable outside the Effect

Effect Events are intentionally restricted. If the behavior is shared by UI controls or other components, extract ordinary reusable logic or create a custom Hook with its own Effect rather than passing an Effect Event around.

## A small design checklist

Before extracting an Effect Event:

1. Identify the external synchronization and list the values that determine it.
2. Keep those values as Effect dependencies.
3. Identify a specific event emitted by that synchronization.
4. Move only the event's handling logic into **useEffectEvent**.
5. Pass event-specific values from the Effect as arguments when their original value matters.
6. Read latest props or state inside the Effect Event only when changes to those values should not restart the synchronization.
7. Confirm the returned function is called only from the owning Effect logic and is not passed elsewhere.

## Key takeaways

- Effects react to changes in their synchronization inputs; event handlers react to user interactions.
- Effect Events handle event-like logic fired by an Effect while reading the latest committed props and state.
- Keep true synchronization triggers, such as a room ID, in the Effect dependencies.
- Pass the triggering event's specific values as arguments when they must retain their original identity.
- Effect Events are not user event handlers, stable callbacks, or a way to bypass dependency rules.
- Call them only from Effect logic in the component or custom Hook that owns them.
