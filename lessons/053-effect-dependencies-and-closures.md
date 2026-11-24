# Effect Dependencies, Closures, and Synchronization

## The Effect sees the render that created it

A React component is called again to produce a new UI snapshot whenever its props or state change. Each call creates new local variables and new functions. A function created during one render closes over that render's values; it does not automatically start reading values from later renders.

This matters when an Effect sets up a timer, subscription, or connection whose callback runs later. The callback keeps the values from the render in which that Effect setup ran. React keeps the synchronization current by comparing the Effect's dependencies and re-running setup when a dependency changes.

This chapter develops the dependency model in detail. It explains stale closures, reactive values, identity changes, and the ways to make the dependency list accurately describe the synchronization.

## Learning goals

After this lesson, you should be able to:

- explain why a callback can keep an earlier render's state or props;
- recognize which values are reactive and must be dependencies;
- understand how dependency equality and reference identity affect synchronization;
- diagnose stale closures and Effects that reconnect too often;
- change the surrounding code safely when dependencies should change;
- use a state updater to avoid reading previous state inside an Effect;
- understand why refs and mutable globals do not become reactive dependencies;
- use the dependency linter as feedback rather than suppressing it.

## Every render is a snapshot

Consider a component with a state value:

~~~jsx
function Greeting({ name }) {
  const message = "Hello, " + name;

  function showMessage() {
    console.log(message);
  }

  return (
    <button type="button" onClick={showMessage}>
      Show greeting
    </button>
  );
}
~~~

On each render, React calls the Greeting component and creates a new message value and a new showMessage function. The event handler attached to the committed button belongs to that render and closes over that render's message.

State works the same way. A state variable is a snapshot for the current render. Calling its setter requests another render; it does not change the variable inside the already-running function or rewrite functions created by that render.

~~~jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count); // The snapshot from this render
  }

  return <button onClick={handleClick}>Count: {count}</button>;
}
~~~

The log reports the value from the render that created handleClick. After React renders again, the new handler closes over the new count.

A callback registered by an Effect also closes over its setup render. If the Effect keeps running without re-synchronizing after a value changes, the external system may keep calling a callback that only knows the old value.

## What makes a value reactive?

For an Effect, a reactive value is one that can change as a result of rendering. This includes:

- props, including values destructured from props;
- state values;
- values calculated from props or state in the component body;
- functions declared in the component body, because a new function is created on each render.

~~~jsx
function ChatRoom({ roomId, serverUrl }) {
  const connectionLabel = serverUrl + " / " + roomId;

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => connection.disconnect();
  }, [serverUrl, roomId]);

  return <p>Connected to: {connectionLabel}</p>;
}
~~~

serverUrl, roomId, and connectionLabel are reactive. The Effect reads the first two, so both are dependencies. It does not read connectionLabel, so that value does not belong in this Effect's dependency list.

Values declared outside a component do not change because that component rendered again. A stable, module-level constant or helper therefore does not need to be an Effect dependency:

~~~jsx
const defaultServer = "primary";

function createConnectionOptions(server, room) {
  return { server, room };
}

function ChatRoom({ roomId }) {
  useEffect(() => {
    const options = createConnectionOptions(defaultServer, roomId);
    const connection = createConnection(options);
    connection.connect();

    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Room: {roomId}</h1>;
}
~~~

The important boundary is that defaultServer and createConnectionOptions cannot change because this component rendered. If a value is mutable and changes outside React's render flow, that does not make it reactive; React still needs a subscription to be notified of its changes.

React guarantees stable identity for state setter functions and the ref object returned by useRef. They may be omitted from dependencies. Including them is also safe because their identities do not change.

## The dependency list describes the code

You do not choose dependencies to control a preferred schedule. The list describes which reactive values the Effect reads. React compares each listed value with its previous value using Object.is. If one differs, React runs the previous cleanup and starts the Effect again with the current render's values.

~~~jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();

  return () => connection.disconnect();
}, [serverUrl, roomId]);
~~~

The list is written inline and must keep the same number and order of items on every render. React compares the items individually; it does not deep-compare objects or compare the dependency-array container by identity.

When a dependency is unchanged, React leaves that synchronization alone, even if the component re-rendered for some other reason. This is how the Effect avoids reconnecting after an unrelated state update.

An Effect with no dependency argument runs after every commit. An empty list means that the Effect reads no reactive values. Neither form is a way to silence the dependency checker.

## A missing dependency creates a stale closure

This interval contains a bug:

~~~jsx
function Counter({ step }) {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timerId = window.setInterval(() => {
      setCount(count + step);
    }, 1000);

    return () => window.clearInterval(timerId);
  }, []); // Incorrect: the Effect reads count and step.

  return <p>Count: {count}</p>;
}
~~~

The setup runs with the initial render's count and step. The interval callback keeps those values. If they are initially 0 and 1, every tick requests setCount(1); it does not keep incrementing. A later change to step is also invisible to the callback.

Adding every value that the code reads fixes the stale closure, but a naïve fix makes the interval restart on every count change:

~~~jsx
useEffect(() => {
  const timerId = window.setInterval(() => {
    setCount(count + step);
  }, 1000);

  return () => window.clearInterval(timerId);
}, [count, step]);
~~~

Here count changes after every tick. Since it is a dependency, React clears and recreates the timer every second. That may be correct for a particular synchronization, but it is usually not what a stopwatch interval needs.

If the next state is calculated from the previous state, use the functional updater. This makes the callback independent of the current count snapshot while preserving the reactive step dependency:

~~~jsx
useEffect(() => {
  const timerId = window.setInterval(() => {
    setCount((currentCount) => currentCount + step);
  }, 1000);

  return () => window.clearInterval(timerId);
}, [step]);
~~~

The timer now continues across count updates. If step changes, React replaces the interval so its callback uses the new step. The dependency list still matches what setup reads.

## Why state updater functions can remove dependencies

Suppose an Effect receives messages from a connection and adds each message to state. This version reads the messages snapshot from setup:

~~~jsx
useEffect(() => {
  const connection = createConnection(roomId);

  connection.onMessage((message) => {
    setMessages([...messages, message]);
  });

  connection.connect();
  return () => connection.disconnect();
}, [roomId, messages]);
~~~

Because messages is read in the Effect, it is a dependency. Each incoming message changes the array, which makes React clean up and recreate the connection. The dependency is accurate, but the code's relationship to messages can be expressed better.

~~~jsx
useEffect(() => {
  const connection = createConnection(roomId);

  connection.onMessage((message) => {
    setMessages((currentMessages) => [...currentMessages, message]);
  });

  connection.connect();
  return () => connection.disconnect();
}, [roomId]);
~~~

The updater receives the latest state from React when it processes the update. The Effect no longer reads messages, so that value is no longer a dependency. It still depends on roomId, which determines the connection.

An updater is not a trick for ignoring a value. Use it when the next state is genuinely a calculation from the previous state.

## Object and array identity

React compares dependencies with Object.is, not by looking inside an object. This Effect reconnects after every render:

~~~jsx
function ChatRoom({ roomId, serverUrl }) {
  const options = { roomId, serverUrl };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return <h1>{roomId}</h1>;
}
~~~

Each render creates a new options object. Even if the two fields are unchanged, the new object is not Object.is-equal to the old one.

Create the object inside the Effect and list the primitive values that define the connection:

~~~jsx
function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const options = { roomId, serverUrl };
    const connection = createConnection(options);
    connection.connect();

    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>{roomId}</h1>;
}
~~~

If a parent passes an options object, read the fields the Effect actually needs and depend on those fields:

~~~jsx
function ChatRoom({ options }) {
  const { roomId, serverUrl } = options;

  useEffect(() => {
    const connection = createConnection({ roomId, serverUrl });
    connection.connect();

    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return <h1>{roomId}</h1>;
}
~~~

This avoids reconnecting just because a parent created a fresh wrapper object. If either actual connection field changes, the Effect still re-synchronizes.

Do not mutate an object to keep its identity stable. Mutation hides changes from React and can make rendering unpredictable. Prefer immutable data and explicit primitive dependencies.

## Function identity and Effect dependencies

A function declared in a component is also a new value on each render:

~~~jsx
function ChatRoom({ roomId }) {
  function connectToRoom() {
    return createConnection(roomId);
  }

  useEffect(() => {
    const connection = connectToRoom();
    connection.connect();
    return () => connection.disconnect();
  }, [connectToRoom]);

  return <h1>{roomId}</h1>;
}
~~~

Since connectToRoom is new on every render, this Effect re-synchronizes after every render. If the helper is only used by this Effect, define it inside setup:

~~~jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    function connectToRoom() {
      return createConnection(roomId);
    }

    const connection = connectToRoom();
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>{roomId}</h1>;
}
~~~

If a helper does not use component values, move it outside the component. useCallback can preserve a function identity when a consumer specifically needs that stable identity, but adding memoization solely to silence a dependency warning can obscure the real synchronization. First consider whether the helper belongs inside the Effect or outside the component.

## Refs and mutable values are not reactive

A ref object is stable, but its current property can be changed without a render. Adding ref.current to a dependency list does not make React notice those changes:

~~~jsx
const latestValueRef = useRef(value);

useEffect(() => {
  latestValueRef.current = value;
}, [value]);
~~~

Here the Effect correctly updates the ref when the reactive value changes. By contrast, this does not subscribe an Effect to future ref mutations:

~~~jsx
useEffect(() => {
  doSomething(latestValueRef.current);
}, [latestValueRef.current]);
~~~

If some code changes current, React receives no notification and may not render again to compare dependencies. Use state for values that participate in rendering, or use an appropriate subscription mechanism when reading an external mutable store. Do not use a ref as a hidden dependency channel.

Global mutable values have the same issue. For example, adding a browser location property to dependencies will not cause React to re-render when the browser location changes outside React. Use the router or a supported external-store subscription to observe that state.

## The dependency linter

The exhaustive-deps rule checks whether reactive values read by an Effect are listed. Treat a warning as a useful clue about the code's synchronization model:

1. Find the value that the Effect reads.
2. Decide whether the Effect should re-synchronize when that value changes.
3. If yes, declare it as a dependency.
4. If that causes unwanted synchronization, restructure the code so the Effect does not read the unnecessary value.
5. Re-check that setup and cleanup still describe the intended external process.

Common restructurings include moving static helpers outside the component, constructing Effect-only objects inside setup, reading primitive fields from a prop object, and using an updater for state transitions.

Avoid disabling the rule or writing a false empty dependency list. A false list can keep an old closure alive and create bugs that depend on the order and timing of user actions.

## Debugging unexpected re-synchronization

If an Effect runs more often than expected:

1. Log the dependency array from two renders.
2. Compare each corresponding item with Object.is.
3. Find the item whose identity or value changed.
4. Ask whether that change should restart the synchronization.
5. If not, restructure the code to depend on the actual primitive inputs instead of a newly created object or function.

A temporary console comparison can help:

~~~js
console.log(Object.is(previousDependencies[0], nextDependencies[0]));
console.log(Object.is(previousDependencies[1], nextDependencies[1]));
~~~

Remove debugging logs after finding the changing value. Do not fix the symptom by omitting that dependency.

## A dependency checklist

Before finalizing an Effect, check:

1. Does setup or cleanup read a prop, state value, or component-body value?
2. Is each such reactive value included?
3. Does any new object or function get recreated on each render?
4. Can an object be constructed inside setup or can its primitive fields be dependencies?
5. Is previous state being read only to calculate the next state? If so, can an updater express that?
6. Is a value stored in a ref or global being mistaken for reactive state?
7. If a dependency changes, should the external synchronization really restart?
8. Does the linter agree with the code's actual data flow?

## Key takeaways

- Every render creates its own values and functions; closures keep that render's snapshot.
- Effects re-synchronize when a listed dependency differs by Object.is.
- Props, state, derived render values, and component-body functions are reactive.
- The dependency list describes what the Effect reads; it is not a schedule you choose.
- Missing dependencies can leave callbacks using stale values.
- New object and function identities can trigger unnecessary synchronization.
- Move Effect-only values into setup, move stable helpers outside the component, and use primitive dependencies where appropriate.
- Use functional state updaters when an Effect needs to calculate the next state from the previous state.
- Ref mutations and global mutations do not trigger React renders and cannot make an Effect reactive.
- Use the linter to find stale closures, and change code rather than suppressing a real dependency.
