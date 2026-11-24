# 035. Working with browser APIs and external event listeners

## Goal

React event props handle interactions on elements that React renders. Some behavior comes from browser APIs outside that element tree, such as the window's resize event, a media query changing, or an observer reporting a change. A component can synchronize with those browser systems in an Effect and release its subscriptions when they are no longer needed.

This chapter explains how to choose between a JSX event handler and an Effect, subscribe to browser events, keep the subscription in sync with dependencies, clean it up correctly, and avoid browser-only globals during server rendering.

## Learning outcomes

After this chapter, you should be able to:

- Decide whether an interaction belongs in a React event prop or an Effect.
- Subscribe to a browser event with addEventListener and remove it with removeEventListener.
- Explain Effect setup, dependency changes, cleanup, and unmounting.
- Read browser APIs without accessing window or document during server rendering.
- Apply the same setup and cleanup pattern to media queries, timers, and observers.
- Avoid duplicate listeners, stale values, and unnecessary global event handlers.

## Start with React event props

For an interaction on an element rendered by React, use the element's React event prop:

~~~jsx
function SaveButton({ onSave }) {
  return <button type="button" onClick={onSave}>Save</button>;
}
~~~

React manages this interaction as part of the rendered interface. A click is caused by a particular user action, so its work belongs in the event handler. Do not add a second native click listener to the same button with an Effect.

Use a browser-level subscription when the event source is outside the React element, or when it must remain active independently of a particular rendered control:

- Listen for the browser window being resized.
- Respond to the Escape key while a dialog is open.
- Track a browser media query changing.
- Observe an element with IntersectionObserver.
- Start a timer while a component is active.

The distinction is based on what causes the work. A form submission caused by a person's submit action belongs in onSubmit. Keeping a browser subscription active while a component is displayed is synchronization and belongs in an Effect.

## An Effect owns the subscription lifetime

An Effect starts a synchronization process after React commits the component. If the setup subscribes to an event, its cleanup should unsubscribe from that event.

~~~jsx
import { useEffect } from 'react';

function ScrollLogger() {
  useEffect(() => {
    function handleScroll() {
      console.log(window.scrollX, window.scrollY);
    }

    window.addEventListener('scroll', handleScroll);

    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);

  return <p>Scroll position is logged while this component is mounted.</p>;
}
~~~

The Effect's lifecycle is:

1. React commits the component and runs the setup function.
2. Setup registers the listener.
3. If a dependency changes, React runs the previous cleanup and then the new setup.
4. When the component unmounts, React runs the final cleanup.

A listener is a resource the component temporarily owns. If setup creates it, cleanup must release it. This prevents an unmounted component from continuing to receive events and prevents duplicate subscriptions after updates.

## Match the listener when removing it

The browser identifies a registered listener using the event type, callback function, and capture setting. The callback used by removeEventListener must be the same function object that was passed to addEventListener.

This works because handleResize is created once for this Effect setup and both calls close over that same function:

~~~jsx
useEffect(() => {
  function handleResize() {
    console.log(window.innerWidth);
  }

  window.addEventListener('resize', handleResize);

  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);
~~~

This does not remove the listener:

~~~jsx
window.addEventListener('resize', () => console.log(window.innerWidth));
window.removeEventListener('resize', () => console.log(window.innerWidth));
~~~

The two arrow expressions create different function objects. Save the callback in a variable so setup and cleanup share its identity. If you use the capture option, use the matching capture setting when removing the listener as well.

## Read the current viewport size

The following component uses one global browser event source. It registers the listener, reads an initial value, and removes the listener during cleanup:

~~~jsx
import { useEffect, useState } from 'react';

function ViewportSize() {
  const [viewport, setViewport] = useState(null);

  useEffect(() => {
    function updateViewport() {
      setViewport({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', updateViewport);
    updateViewport();

    return () => {
      window.removeEventListener('resize', updateViewport);
    };
  }, []);

  if (viewport === null) {
    return <p>Reading the viewport size…</p>;
  }

  return (
    <p>
      Viewport: {viewport.width} by {viewport.height} CSS pixels
    </p>
  );
}
~~~

The initial state is null because the component has not read the browser yet. The Effect reads window.innerWidth and window.innerHeight after the component commits, then keeps the displayed state synchronized with resize events. A state update causes React to render the new text; the listener itself does not directly mutate the DOM.

The resize event is sent for the window. If you need the size of a particular element rather than the viewport, an element observer is usually a better source than listening globally and guessing from the window dimensions.

## Dependencies and values captured by a listener

A callback created inside an Effect closes over the values from that Effect's render. When the Effect uses a prop or state value, include that reactive value in the dependency list so the subscription is updated when the value changes.

~~~jsx
function EscapeDismiss({ onDismiss }) {
  useEffect(() => {
    function handleKeyDown(event) {
      if (event.key === 'Escape') {
        onDismiss();
      }
    }

    window.addEventListener('keydown', handleKeyDown);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, [onDismiss]);

  return null;
}
~~~

When onDismiss changes, React removes the listener that captured the old callback and registers a new listener that captures the current callback. Do not omit a dependency just to keep an Effect from running again. If a dependency changes too frequently, first reconsider the component boundary and callback design; chapters on Effect dependencies and useEffectEvent cover those choices in more depth.

A state setter such as setViewport has a stable identity, so the viewport example does not need to list it as a dependency. Do include props, state values, and component-local functions that the Effect reads, unless you deliberately restructure the code to avoid reading them there.

## Strict Mode is a cleanup check

In development, Strict Mode can run an extra setup, cleanup, and setup cycle when a component first appears. A correctly written subscription ends up with one active listener after that check:

1. Setup adds the listener.
2. Cleanup removes that same listener.
3. Setup adds the listener that will remain active.

Do not use a ref flag to suppress the second setup. That hides missing cleanup rather than fixing it. Make setup and cleanup symmetrical so the Effect is safe to start, stop, and start again.

The extra cycle is a development check. It is not a reason to remove Strict Mode or to make event registration happen during rendering.

## Browser APIs follow the same pattern

An Effect is useful for synchronizing with a browser API that lives outside React. Use the specific API's setup and cleanup operations.

### Media query changes

~~~jsx
useEffect(() => {
  const query = window.matchMedia('(prefers-reduced-motion: reduce)');

  function handleChange(event) {
    setReducedMotion(event.matches);
  }

  setReducedMotion(query.matches);
  query.addEventListener('change', handleChange);

  return () => {
    query.removeEventListener('change', handleChange);
  };
}, []);
~~~

Read query.matches for the current value, then subscribe to change if the interface should respond when the preference changes. A media query is not a substitute for honoring the preference in CSS when the behavior is purely visual.

### Timers

~~~jsx
useEffect(() => {
  const timerId = window.setTimeout(() => {
    setMessage('The delay has finished.');
  }, 1000);

  return () => {
    window.clearTimeout(timerId);
  };
}, []);
~~~

If a timer should run only while the component is present, cancel it during cleanup. An interval should similarly be paired with clearInterval. If the callback reads state that changes over time, account for the Effect's dependencies or use a state updater where appropriate.

### Element observers

~~~jsx
useEffect(() => {
  const node = observedElementRef.current;

  if (!node) {
    return;
  }

  const observer = new IntersectionObserver((entries) => {
    setIsVisible(entries.some((entry) => entry.isIntersecting));
  });

  observer.observe(node);

  return () => {
    observer.disconnect();
  };
}, []);
~~~

Use a ref to get the DOM node, observe it after it exists, and disconnect the observer during cleanup. This is usually more suitable for visibility changes of an element than handling every scroll event yourself.

The same lifecycle applies to subscriptions from other libraries: connect or subscribe in setup, then disconnect or unsubscribe in cleanup.

## Browser globals and server rendering

Effects run only in the browser after React commits. A component can therefore read window or document inside an Effect, as in the viewport example. Avoid reading these browser globals while rendering or in a module-level initializer:

~~~jsx
// Avoid: this can fail when the component is rendered on a server.
const [width, setWidth] = useState(window.innerWidth);
~~~

A server has no browser window. Use a neutral initial state, then read the browser value inside an Effect. Keep the initial server-rendered and first client-rendered output compatible so hydration can attach to the expected markup. If the feature must render a meaningful value on the server, provide that value through the server or framework rather than assuming the browser API exists there.

Do not use an Effect merely to delay ordinary content until after hydration. Use this pattern when the component actually needs to synchronize with a browser-only system.

## Avoid unnecessary global listeners

A global listener sees events beyond one local component and may run often. Prefer the narrowest event source that satisfies the behavior:

- Put ordinary click, input, submit, and keyboard interactions on the relevant JSX element.
- Listen on window only for events that are truly window-wide or must work outside a specific control.
- Listen on document only when the behavior needs document-level observation.
- Observe a particular element with an observer when the API provides that behavior.
- Remove subscriptions as soon as the component no longer needs them.

Do not call setState for every high-frequency event without considering the cost. For scroll or pointer movement, use an observer or schedule work appropriately if the interface requires frequent updates. Keep the work small, and do not install multiple listeners that all calculate the same value.

## Common mistakes

- **Adding a listener during render:** rendering can happen more than once and must stay free of external side effects. Register in an Effect.
- **Forgetting cleanup:** unmounted or replaced subscriptions continue to run. Return a cleanup function.
- **Using a new function in removeEventListener:** it does not match the function that was registered. Reuse the callback variable.
- **Leaving out a reactive dependency:** the callback can keep using an old prop or state value. Include the dependency or restructure the Effect.
- **Using an Effect for a button action:** a click-triggered action belongs in onClick or the appropriate React event prop.
- **Duplicating a React event with a native listener:** one interaction can then run the behavior twice. Choose one owner.
- **Reading window during server rendering:** browser globals are unavailable on the server. Read them after the component commits.
- **Using window resize to measure an element:** viewport size and element size are different. Observe the element when that is what matters.
- **Treating Strict Mode's extra cycle as a production duplicate:** it is a development cleanup check. Make the subscription resilient to setup and cleanup.

## Debugging checklist

When a browser event behaves incorrectly, check:

1. Is this a React-rendered element interaction or an external browser subscription?
2. Does setup run in an Effect rather than during render?
3. Does cleanup remove the same event type and callback?
4. Does the listener use the latest values it needs?
5. Are all reactive dependencies included?
6. Does cleanup run when a dependency changes and when the component unmounts?
7. Is the browser API being accessed only in a browser-safe place?
8. Is a global listener necessary, or can a JSX event or element-specific API handle it?

## Practice

1. Build a component that displays the current viewport dimensions and updates after resizing the browser.
2. Add an Escape key subscription that is active only while a dismissible panel is open. Remove it when the panel closes.
3. Track a reduced-motion media query and render the current preference without changing the page's CSS behavior.
4. Replace a scroll listener that detects element visibility with an IntersectionObserver.
5. Enable Strict Mode and confirm that every setup has a matching cleanup.

## Summary

React event props handle interactions on React-rendered elements. Effects synchronize components with external browser systems such as window events, media queries, timers, and observers. Pair every subscription or scheduled resource with cleanup, list the reactive values used by the Effect, and access browser-only globals after the component commits.