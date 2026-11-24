# Rendering, Committing, and Updating the UI

## React works out the next interface before changing the DOM

React updates the screen through a sequence of work:

1. **Trigger** — an initial root render or an update requests work.
2. **Render** — React calls components to calculate the next UI description.
3. **Commit** — React applies the necessary changes to the browser DOM.
4. **Paint** — the browser draws the updated DOM on screen.

A component render is a calculation. It does not mean React immediately rewrites the entire DOM. React compares the new result with the previous tree and commits only the changes needed to make the DOM match.

This distinction explains why a component function can run while some DOM nodes remain untouched, why render code must be pure, and why setting state requests a later render instead of changing the state variable immediately.

## Learning goals

After this lesson, you should be able to:

- describe trigger, render, commit, and browser paint in order;
- explain what React means by rendering a component;
- identify common causes of an initial render and an update;
- distinguish calculating JSX from applying DOM changes;
- explain why a re-render does not necessarily mutate the DOM;
- keep component rendering pure and safe to repeat;
- understand that state is a snapshot for one render and setters request future work;
- use one React root correctly and distinguish client rendering from hydration.

## 1. Trigger: request UI work

A React app needs an initial render. In a client-rendered app, the entry point finds a DOM container, creates a root, and renders the app:

~~~jsx
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

const container = document.getElementById("root");
const root = createRoot(container);
root.render(<App />);
~~~

A later update can be triggered by a state setter, a parent rendering with new props, or a context value changing for a consumer. An update tells React that some UI may need to be recalculated; it does not directly tell the browser which node to change.

A typical fully React-built app creates one root. Components that belong to that app are added under its root component, rather than by creating another root for each component.

## 2. Render: calculate the next UI

**Rendering** means React calls components to find out what the UI should look like for the current inputs. A function component receives a snapshot of props, state, and context for that render and returns a JSX description:

~~~jsx
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}
~~~

On the initial render, React starts at the root component and follows the returned component tree. On an update, React evaluates the relevant component tree to determine the next result. React may call a component more than once while preparing work; a render can also be restarted or discarded before commit.

The component’s return value describes UI. JSX creation alone does not directly insert DOM nodes.

## 3. Commit: apply the needed DOM changes

After React has a result to commit, it applies the necessary DOM operations. On the initial commit, it creates the DOM nodes for the UI. On an update, it compares the new tree with the previous one and applies only changes that are needed.

For example, if only a heading’s text changes, React can update that text without removing an adjacent input. If the rendered result is equivalent, React may have no DOM change to commit at all.

Refs attached to DOM nodes are updated during commit. Effects also have their own timing rules; do not use render code to perform work that belongs in an event handler or an Effect.

## 4. Paint: the browser draws

After DOM changes are committed, the browser paints the updated page. React calls this process **rendering**, while browsers also use the word “render” for drawing pixels. In React lessons, “component render” means calculating a UI result; browser paint means drawing that result to the screen.

A commit and a paint are related but separate steps. A render may commit no DOM changes, and browser scheduling determines when pixels are painted.

## A component render must be pure

React expects rendering to be a predictable calculation:

- with the same props, state, and context, return the same JSX;
- do not mutate props, state, existing objects, or global variables;
- do not perform side effects such as network requests, timers, subscriptions, or DOM edits during render.

~~~jsx
function BadCounter() {
  let count = 0;
  count += 1;
  return <p>Count: {count}</p>;
}
~~~

This local variable resets each time the function runs, so it is not persistent component state. Mutating a module-level variable during render is also a problem because repeated or reordered renders can change shared data unexpectedly.

Use state for visible values and event handlers for actions:

~~~jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount((current) => current + 1)}>
      Count: {count}
    </button>
  );
}
~~~

It is fine to create and mutate a new local object or array during a render, as long as it does not mutate data that existed before the render and the output remains predictable.

## Strict Mode can reveal impure rendering

In development, **StrictMode** may call component functions and other pure calculations more than once to help find accidental side effects. This is a development check; it does not mean React commits the same DOM changes twice in production.

Do not make correctness depend on a component being called exactly once. Keep render calculations pure and move user-triggered actions into event handlers. Manage external synchronization with an Effect and cleanup when appropriate.

## State is a snapshot for one render

A component’s state variable is the value for that particular render. Calling its setter requests a future render; it does not replace the variable in the currently running function or event handler:

~~~jsx
function Counter() {
  const [count, setCount] = useState(0);

  function addThree() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return <button onClick={addThree}>Count: {count}</button>;
}
~~~

All three expressions use the same **count** snapshot, so they each request the same next value. To apply several updates based on the result of the previous queued update, use updater functions:

~~~jsx
function addThree() {
  setCount((current) => current + 1);
  setCount((current) => current + 1);
  setCount((current) => current + 1);
}
~~~

React can batch related updates and perform a render after the event handler completes. Write the state transition you mean; do not expect a setter to synchronously alter the state variable you already read.

## Re-rendering is not the same as rebuilding the DOM

When a component renders again, React calculates its next JSX. React then reconciles that result with the previous tree:

- nodes in matching positions with compatible types can often be reused;
- changed text or props are updated as needed;
- nodes that are no longer present are removed;
- newly present nodes are created;
- unchanged DOM can remain in place.

The exact matching rules also depend on component type and keys. A key is how React matches data items among siblings; use stable data IDs for reorderable lists.

Because React owns the DOM within its root, avoid manually adding, removing, or reordering React-managed nodes. Update state so the next render describes the structure you want.

## Client render and server hydration

Use **createRoot** for a client-rendered root. Its first render clears existing contents inside the chosen root container and lets React manage that DOM.

If the HTML already contains markup rendered by React on the server, attach React with **hydrateRoot** instead. Hydration reuses and attaches behavior to existing server HTML; creating a fresh root over that markup is the wrong operation.

If a root container is removed by an outside system, call **root.unmount()** so React can detach handlers and clean up the rendered tree. A normal React app does not need to manually unmount its main root during routine updates.

## A useful mental model

Think of React as calculating a new description before making a change:

~~~text
state, props, context, or initial root render
                    │
                    ▼
             React calls components
                    │
                    ▼
             next JSX tree calculated
                    │
                    ▼
        React commits required DOM changes
                    │
                    ▼
               browser paints
~~~

The component function describes what should be visible. State setters, event handlers, and Effects have different jobs; keep each operation in its appropriate phase.

## Summary

- A UI update follows trigger, render, commit, and browser paint.
- Component rendering is React calling components to calculate a UI description.
- Rendering does not itself change the DOM; React commits only the necessary DOM updates.
- A render can be repeated or discarded, so render logic must be pure and side-effect free.
- State is a snapshot for one render. Setters request future work, and updater functions compose queued updates.
- React manages DOM inside its root. Use state to describe structural changes rather than editing React-owned structure by hand.
- Use **createRoot** for client-rendered markup and **hydrateRoot** for React-rendered server markup.
