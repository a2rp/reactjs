# useRef for DOM Nodes and Mutable Values

## A persistent value that does not render the UI

A ref is a small mutable object that React preserves between renders. The **useRef** Hook creates it, and its **current** property holds the value:

~~~jsx
const valueRef = useRef(initialValue);
~~~

Changing **valueRef.current** does not ask React to render again. Use a ref for information that must persist but is not used to calculate the visible UI, or for a DOM node that needs an imperative browser operation such as focus or scrolling.

If a value should appear on screen or determine what JSX is rendered, use state instead.

## Learning goals

After this lesson, you should be able to:

- explain how a ref differs from state;
- keep mutable, non-visual values such as timer IDs between event handlers;
- attach a ref to a DOM element and use it after React commits the element;
- focus, scroll, or call a browser API without making the DOM your source of truth;
- avoid reading or writing refs during rendering;
- clean up resources stored in refs;
- recognize callback refs for dynamic nodes and their attachment lifetime.

## Create and retain a ref

Call **useRef** at the top level of a component or custom Hook:

~~~jsx
import { useRef } from "react";

function SearchBox() {
  const inputRef = useRef(null);

  // ...
}
~~~

It returns an object similar to:

~~~js
{
  current: null
}
~~~

React retains the same ref object for that component instance across renders. You may read or update its **current** property from event handlers and Effects.

The initial argument is used only for the first render. On later renders, React returns the same ref object and ignores the new initial value.

## State and refs serve different purposes

| Question | State | Ref |
| --- | --- | --- |
| Does changing it trigger a render? | Yes, using the state setter | No |
| Should it determine visible JSX? | Usually, yes | Usually, no |
| Can its value be changed directly? | No; request an update with the setter | Yes; assign to **current** |
| What does a render observe? | A snapshot for that render | A mutable object not tracked by React |
| Typical uses | Form values, selected items, UI status | DOM nodes, timer IDs, non-visual bookkeeping |

For a number displayed on screen, use state:

~~~jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount((currentCount) => currentCount + 1);
}

return <p>Clicked {count} times</p>;
~~~

Changing a ref would not update that paragraph:

~~~jsx
const countRef = useRef(0);

function handleClick() {
  countRef.current += 1;
}
~~~

This can be useful for a count used only by event handlers, but it is wrong when the count is part of the UI. React does not know that **current** changed, so it does not render the new number.

A ref is not a shortcut for avoiding a needed state update. If rendering depends on a value, make that value state.

## Use refs for DOM operations

The most common ref points to a DOM node. Initialize it with **null**, then pass the ref object to the element's **ref** prop:

~~~jsx
function SearchBox() {
  const inputRef = useRef(null);

  function handleFocusClick() {
    inputRef.current?.focus();
  }

  return (
    <>
      <input ref={inputRef} aria-label="Search" />
      <button type="button" onClick={handleFocusClick}>
        Focus search
      </button>
    </>
  );
}
~~~

React fills **inputRef.current** with the input DOM node during the commit when the node is attached. Before then, its value is **null**. When the node is removed or replaced, React clears the ref.

Read a DOM ref from an event handler or an Effect after the element has committed. Do not expect the DOM node to exist while React is calculating the initial render.

### Imperative browser APIs

A ref is useful when an event needs an operation that React does not express as rendered markup:

- call **focus()** on an input;
- call **scrollIntoView()** on a result or list item;
- read a measurement such as **getBoundingClientRect()** after commit;
- call **play()** or **pause()** on a media element.

Use the browser API for the operation, but keep application data and visible content in React state. For example, represent whether a dialog should exist with state and conditional JSX rather than manually removing a DOM node that React owns.

## Refs can hold non-visual values

A normal local variable is recreated on every render. State persists but triggers rendering when updated. A ref persists without triggering a render, which makes it useful for mutable values that event handlers and cleanup need to share.

### Store a timer ID

~~~jsx
function Timer() {
  const [ticks, setTicks] = useState(0);
  const [running, setRunning] = useState(false);
  const intervalRef = useRef(null);

  function handleStart() {
    if (intervalRef.current !== null) {
      return;
    }

    intervalRef.current = window.setInterval(() => {
      setTicks((currentTicks) => currentTicks + 1);
    }, 1000);

    setRunning(true);
  }

  function handleStop() {
    if (intervalRef.current === null) {
      return;
    }

    window.clearInterval(intervalRef.current);
    intervalRef.current = null;
    setRunning(false);
  }

  useEffect(() => {
    return () => {
      if (intervalRef.current !== null) {
        window.clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <section>
      <p>{ticks} ticks</p>
      <button type="button" onClick={handleStart} disabled={running}>
        Start
      </button>
      <button type="button" onClick={handleStop} disabled={!running}>
        Stop
      </button>
    </section>
  );
}
~~~

The tick count and running status are state because they appear in the UI. The interval ID is a ref because it is needed to stop the interval but is not itself rendered.

The cleanup reads the current timer ID when the component unmounts. Do not start the timer during rendering; start it in the event handler and release it when it is no longer needed.

### Other non-visual values

Refs can retain values such as:

- a timeout or animation-frame ID;
- an imperative object that belongs to one component instance;
- an event count used only for diagnostics;
- a previous value used later by an event handler;
- a mutable cache that does not affect rendered output.

If a change to one of these values must update the UI, keep the rendered part in state instead.

## Read and write refs outside rendering

React expects rendering to be pure: the same props and state should produce the same UI. Since changing **ref.current** does not trigger rendering, reading or changing it during render can make output depend on invisible mutations.

Avoid:

~~~jsx
function Counter() {
  const countRef = useRef(0);

  // Avoid: a ref mutation during rendering.
  countRef.current += 1;

  return <p>{countRef.current}</p>;
}
~~~

Use state when a value affects JSX. Read or write the ref from an event handler or Effect:

~~~jsx
function Counter() {
  const countRef = useRef(0);
  const [lastClick, setLastClick] = useState(0);

  function handleClick() {
    countRef.current += 1;
    setLastClick(countRef.current);
  }

  return <button onClick={handleClick}>Recorded clicks: {lastClick}</button>;
}
~~~

The handler updates a non-visual ref and separately updates state for what the user should see.

### One-time initialization exception

A predictable, side-effect-free initialization may assign a ref while rendering:

~~~jsx
const playerRef = useRef(null);

if (playerRef.current === null) {
  playerRef.current = createPurePlayerConfiguration();
}
~~~

This should only initialize the ref once and return the same kind of value for the same component instance. Do not use this pattern to start timers, attach listeners, access the DOM, or create external resources that need cleanup. Those actions belong in event handlers or Effects.

Also note that **useRef(expensiveExpression())** evaluates the expression during every render even though React only uses the first result. If construction is expensive but pure, the guarded initialization pattern can avoid repeating it. If construction has side effects or needs cleanup, do not construct it during rendering.

## Callback refs for changing collections

A ref object is convenient when there is one known node. When the number of nodes is dynamic, a callback ref can register each node in a map keyed by item identity:

~~~jsx
<li
  key={item.id}
  ref={(node) => {
    if (node !== null) {
      itemNodesRef.current.set(item.id, node);
    }
  }}
>
  {item.label}
</li>
~~~

The callback runs when React attaches the node. In current React, a callback ref may return a cleanup function that removes the node from the map when it detaches. Keep callback identity and cleanup behavior in mind: an inline callback is a new function on each render, so React may detach the old callback and attach the new one. Strict Mode also performs an extra callback-ref setup and cleanup cycle in development.

Do not call **useRef** inside a loop or inside **map()**. Keep one ref to a map, then let callback refs maintain entries for the dynamic nodes. Use stable item IDs rather than array positions when items can move or be removed.

## Ref attachments and timing

React updates refs during the commit, not while calculating JSX:

1. During render, the ref object may still contain **null** or the previous node.
2. During commit, React applies DOM changes and attaches the new node to **current**.
3. When a node detaches, React clears the object ref or runs the callback-ref cleanup.

For an operation tied to a user action, use an event handler once the target exists. For synchronization that should happen after a rendered change, use an Effect or, when the operation must happen before paint, learn the appropriate layout-effect API. Do not read **ref.current** during render to guess whether a node has been committed.

A ref mutation itself is not reactive. Putting **ref.current** in an Effect dependency array does not make React observe changes to it. If a changing DOM node must trigger work, use a callback ref that records the node in state or choose another explicit reactive signal.

## Ref safety and Strict Mode

Refs are local to a component instance. Do not move a component's ref object to module scope: that would share mutable state across instances.

In development Strict Mode, React may call your component function twice and discard one render result to detect impure rendering. Create the ref with **useRef** and keep render-time initialization pure. Effects and callback refs also receive development checks, so pair resource setup with cleanup.

Do not manually add or remove DOM nodes that React is responsible for rendering. Focusing, scrolling, measuring, and calling browser APIs are usually safe. Replacing React-managed children behind React's back can make its next update inconsistent or crash.

Refs are an escape hatch. If most application data flow depends on mutable refs, consider whether state, props, or context would make the behavior easier to follow.

## Key takeaways

- **useRef** returns the same mutable object across renders.
- Changing **current** does not trigger a render; use state for values that affect the UI.
- Use DOM refs for imperative browser operations such as focus and scrolling.
- Store non-visual values, such as timer IDs, in refs when handlers or cleanup need them.
- Read and write refs in event handlers, Effects, callback refs, or the one-time initialization pattern; do not use them as render-time state.
- Let React own the DOM structure and clean up resources that refs help you manage.
