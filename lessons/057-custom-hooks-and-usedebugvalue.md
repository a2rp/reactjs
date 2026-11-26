# Custom Hooks and useDebugValue

## Reuse behavior without hiding the data flow

A custom Hook is a JavaScript function that uses React Hooks to package reusable, stateful behavior. It lets components share the logic for a capability-such as a timer, online status, or chat connection-while keeping their own rendering decisions.

A custom Hook does not create one shared state value for every component that calls it. Each call runs as part of its calling component and owns an independent Hook state position. To share the same state between components, give that state one owner and pass it down, or use an appropriate shared store.

**useDebugValue** is an optional Hook for adding a readable label to a custom Hook in React DevTools. It helps inspect complex Hook state; it does not display UI or change the Hook's behavior.

## Learning goals

After this lesson, you should be able to:

- recognize and name a custom Hook;
- extract reusable stateful behavior without confusing it with shared state;
- compose built-in Hooks and other custom Hooks;
- pass reactive inputs through a custom Hook while keeping dependencies correct;
- decide when a Hook abstraction adds clarity and when it adds indirection;
- avoid generic lifecycle wrappers that hide Effect behavior;
- use **useDebugValue** for useful, optional DevTools diagnostics.

## What makes a Hook custom?

A custom Hook is an ordinary JavaScript function that follows the Rules of Hooks and calls one or more built-in or custom Hooks. Its name begins with **use** followed by a capital letter, such as **useOnlineStatus** or **useChatRoom**.

~~~jsx
import { useSyncExternalStore } from "react";

function subscribeToResize(callback) {
  window.addEventListener("resize", callback);
  return () => window.removeEventListener("resize", callback);
}

function getWindowWidth() {
  return window.innerWidth;
}

function getServerWindowWidth() {
  return 0;
}

function useWindowWidth() {
  return useSyncExternalStore(
    subscribeToResize,
    getWindowWidth,
    getServerWindowWidth
  );
}
~~~

A component can use this behavior without putting the subscription details in its body:

~~~jsx
function WindowSizeLabel() {
  const width = useWindowWidth();

  return <p>Window width: {width}px</p>;
}
~~~

This browser API Hook uses **useSyncExternalStore** so React can subscribe to resize events and read the current width. Its server snapshot is a 0-pixel placeholder because a server has no browser window; after hydration, React reads the actual browser width.

## A custom Hook shares logic, not state

Consider two independent stopwatch components that each call **useStopwatch**:

~~~jsx
function StopwatchPanel({ title }) {
  const stopwatch = useStopwatch();

  return (
    <section>
      <h2>{title}</h2>
      <p>{stopwatch.elapsedSeconds.toFixed(1)} seconds</p>
      <button onClick={stopwatch.start}>Start</button>
      <button onClick={stopwatch.stop}>Stop</button>
      <button onClick={stopwatch.reset}>Reset</button>
    </section>
  );
}

function App() {
  return (
    <>
      <StopwatchPanel title="First timer" />
      <StopwatchPanel title="Second timer" />
    </>
  );
}
~~~

Each component invocation gets its own **elapsedSeconds** and **running** state because the custom Hook's calls belong to two different component instances. Starting one timer does not start the other.

The Hook makes the behavior reusable. It does not turn those independent values into a shared timer.

### If components need to share the same value

Choose one owner for that value:

- Lift the state and Hook call to a common parent, then pass the value and actions to children.
- Use context when many descendants need the same React-owned value.
- Use an external store and a suitable subscription Hook when the source truly lives outside React.

Calling the same custom Hook in several components is not a substitute for choosing shared state ownership.

## A custom Hook is part of rendering

React calls a custom Hook as part of rendering its component. The Hook receives the current arguments, calls its own Hooks in order, and returns values to the component. Its render-time logic must be pure, just like a component's render-time logic.

~~~jsx
function useGreeting(name) {
  const [prefix, setPrefix] = useState("Hello");

  const greeting = prefix + ", " + name;

  return {
    greeting,
    setPrefix,
  };
}
~~~

The custom Hook can contain Effects or other Hooks when needed, but do not start external work directly during render. Put synchronization in an Effect or a purpose-built Hook. Keep event-specific work in event handlers.

### The Rules of Hooks still apply

Call Hooks at the top level of a component or custom Hook:

~~~jsx
function useOptionalTimer(enabled) {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    if (!enabled) {
      return;
    }

    const intervalId = window.setInterval(() => {
      setSeconds((current) => current + 1);
    }, 1000);

    return () => window.clearInterval(intervalId);
  }, [enabled]);

  return seconds;
}
~~~

Do not conditionally call the Hook:

~~~jsx
function Timer({ enabled }) {
  if (enabled) {
    const seconds = useOptionalTimer(true);
  }

  // ...
}
~~~

Instead, call it unconditionally and pass the condition as data, as in **useOptionalTimer**. Keep the order of Hook calls consistent on every render. Follow the same rule when one custom Hook calls another.

A helper that does not call Hooks is just a normal function. It can be called conditionally, passed around, or used outside a component. Do not give it a **use** name unless it is intended to be a Hook; the name tells readers and the linter that Hook rules apply.

## Custom Hook inputs remain reactive

A custom Hook should treat props, state, and arguments as current inputs. If it uses those values in an Effect, the Effect must keep synchronizing when the relevant inputs change.

~~~jsx
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });
    connection.connect();

    return () => connection.disconnect();
  }, [serverUrl, roomId]);
}
~~~

The component passes the current values to the Hook:

~~~jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useChatRoom({ serverUrl, roomId });

  return (
    <label>
      Server address
      <input
        value={serverUrl}
        onChange={(event) => setServerUrl(event.target.value)}
      />
    </label>
  );
}
~~~

When **serverUrl** or **roomId** changes, the Effect inside the Hook re-synchronizes. Extracting an Effect into a custom Hook does not change its dependency rules or make its values non-reactive.

Keep Effects inside a custom Hook focused on the capability the Hook represents. A name such as **useChatRoom** communicates what is synchronized and what inputs control it.

## Custom Hooks that accept event callbacks

Sometimes a Hook owns a subscription but its caller should decide what to do when an event arrives. Pass the callback as an option, and keep the subscription's reactive inputs separate from the event response.

With React 19.2 or later, an Effect Event can let the callback read the caller's latest values without reconnecting just because the callback was recreated during rendering:

~~~jsx
function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const connection = createConnection({ serverUrl, roomId });

    connection.on("message", (message) => {
      onMessage(message);
    });

    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]);
}
~~~

The server address and room determine the connection, so they remain dependencies. The caller's message handler is used only when a message event arrives. **onMessage** is declared and used inside the custom Hook that owns this Effect; do not return or pass an Effect Event to another component or Hook.

Use this pattern only when it matches the intended behavior. If a changed callback should replace the subscription, keep it reactive instead. For the full Effect Event rules and limitations, see the preceding lesson.

## Decide when to extract a custom Hook

A useful custom Hook gives a behavior a clear name and a focused interface:

~~~jsx
useOnlineStatus()
useChatRoom({ serverUrl, roomId })
useIntersectionObserver({ targetRef, rootMargin })
~~~

It lets the caller express intent without repeating subscription or synchronization details. A Hook can also make a complex behavior easier to reuse, reason about, and review independently.

Do not extract every repeated line or wrap a single state variable just to create a file. Some duplication is clearer than an abstraction with no meaningful purpose. Before extracting, ask:

1. Is there a coherent behavior that appears in more than one place?
2. Can I give that behavior a clear name?
3. Are the inputs and returned values understandable at the call site?
4. Does the Hook own a specific synchronization or stateful capability?
5. Does the abstraction preserve the true reactive dependencies?

If the name is difficult to choose or the Hook needs many unrelated options, the behavior may not have a useful boundary yet.

## Avoid generic lifecycle wrappers

Do not create abstractions such as **useMount**, **useEffectOnce**, or **useUpdateEffect** to disguise an Effect's synchronization model:

~~~jsx
function useMount(callback) {
  useEffect(() => {
    callback();
  }, []);
}
~~~

A caller might pass reactive values into **callback**, but the wrapper's empty dependency list hides them from the linter and prevents the Effect from responding to those changes.

Start with the real Effect and its dependencies. When it represents a reusable capability, extract a specifically named Hook such as **useChatRoom** or **useWindowListener**. The Hook name should tell callers what it does, not merely when React happens to run an Effect.

## Add readable custom Hook labels with useDebugValue

A custom Hook can call **useDebugValue** at its top level to show a helpful label for its internal value in React DevTools:

~~~jsx
function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );

  useDebugValue(isOnline ? "Online" : "Offline");

  return isOnline;
}
~~~

When a component using this Hook is inspected, DevTools can show a readable **OnlineStatus** value instead of only the underlying boolean. The Hook still returns **isOnline** to the component; **useDebugValue** itself returns nothing and does not render UI.

### Defer formatting until inspection

The optional second argument is a formatter. React DevTools can call it when the Hook is inspected, which avoids doing potentially expensive formatting on every render:

~~~jsx
function formatStopwatch({ elapsedMs, running }) {
  const state = running ? "running" : "paused";
  return state + " · " + (elapsedMs / 1000).toFixed(1) + "s";
}

function useStopwatch() {
  const [elapsedMs, setElapsedMs] = useState(0);
  const [running, setRunning] = useState(false);

  useDebugValue({ elapsedMs, running }, formatStopwatch);

  // The rest of the Hook implements the timer.
}
~~~

The formatter receives the debug value and returns the label to display. It should be a pure formatting function. Do not call Hooks inside the formatter.

### Do not label every Hook

A debug value is most useful when a custom Hook has complex internal data or is part of a shared library. Simple Hooks often reveal enough information through their returned values and state in DevTools already. Add labels where they improve inspection, not as decoration.

**useDebugValue** is for developers inspecting a component tree. Use accessible UI for user-facing status, validation, and errors.

## Key takeaways

- A custom Hook packages reusable stateful behavior and is named with **use** followed by a capital letter.
- Every call to a custom Hook has independent state; a Hook shares logic, not state.
- Lift state or use a shared store when components need one common value.
- The Rules of Hooks, purity, and dependency rules apply inside custom Hooks.
- Prefer focused hooks with names that describe a capability; avoid generic lifecycle wrappers.
- **useDebugValue** adds optional React DevTools labels and does not render UI.
- Provide a formatter when lazy, readable debug output is useful; do not add debug labels to every Hook.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="056-useeffectevent.md">056 - useEffectEvent and Non-Reactive Event Logic</a></td><td><a href="../qnas/057-custom-hooks-and-usedebugvalue.md">Q&amp;A</a></td><td><a href="../examples/057-custom-hooks-and-usedebugvalue.md">Example</a></td><td><a href="058-useref.md">058 - useRef for DOM Nodes and Mutable Values</a></td></tr>
  </tbody>
</table>
