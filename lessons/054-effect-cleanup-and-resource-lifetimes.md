# Effect Cleanup and Resource Lifetimes

## Cleanup releases what setup owns

An Effect may start work outside React: a timer, event subscription, connection, observer, animation, dialog, or request. Its cleanup function releases, stops, or reverses that work when it is no longer current.

Treat each setup and cleanup pair as the lifetime of one external resource:

1. Setup creates or acquires a resource using the values from its render.
2. That resource remains active while those values remain current.
3. If a dependency changes, React runs the matching cleanup using the old values.
4. React then runs a new setup using the new values.
5. When the component is removed, React runs the final cleanup.

Cleanup is not only an unmount callback. It runs whenever React must stop one synchronization before starting another.

## Learning goals

After this lesson, you should be able to:

- pair each resource created by an Effect with its exact cleanup;
- explain cleanup timing during dependency changes and unmounting;
- clean up listeners, timers, connections, observers, dialogs, and animations;
- cancel or ignore obsolete asynchronous results;
- handle development Strict Mode's setup-cleanup-setup check;
- distinguish stopping local work from undoing a remote action;
- avoid cleanup code that releases the wrong resource or changes unrelated React state.

## A resource belongs to one setup

Keep the resource in the setup function's local scope. The cleanup returned by that setup closes over that exact resource:

~~~jsx
useEffect(() => {
  const connection = createConnection(serverKey, roomId);
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [serverKey, roomId]);
~~~

When roomId changes, React first calls the cleanup from the previous setup. That cleanup disconnects the previous connection. React then runs setup with the new roomId and creates a different connection.

This local ownership is important. Avoid storing the current connection in a shared mutable variable and looking it up later during cleanup. A newer setup might overwrite that variable, causing an older cleanup to disconnect the wrong resource.

Write each Effect as one independent synchronization process. If a component owns a connection and a separate browser listener, separate Effects can make the two lifetimes easier to understand and clean up independently.

## Cleanup mirrors setup

Cleanup should reverse or release the work performed by its setup:

| Setup starts or acquires | Cleanup stops or releases |
| --- | --- |
| setInterval or setTimeout | clearInterval or clearTimeout |
| addEventListener | removeEventListener |
| subscribe | unsubscribe |
| connect | disconnect or close |
| observe | unobserve or disconnect |
| showModal | close |
| start an animation | cancel or reset it |
| start a request | abort it or ignore its obsolete result |

A setup that creates no ongoing work may not need cleanup. Do not add cleanup that performs unrelated work merely to have a return function.

For example, logging a render does not create a resource:

~~~jsx
useEffect(() => {
  console.log("The committed value is", value);
}, [value]);
~~~

No cleanup is needed for that log. By contrast, starting an interval without clearing it would leave work running after the Effect's lifetime ends.

## Event listeners: remove the exact registration

The cleanup must remove the listener that setup registered. Keep the handler and relevant options in the setup scope:

~~~jsx
useEffect(() => {
  function handleVisibilityChange() {
    reportVisibility(document.visibilityState);
  }

  const options = { capture: true };

  document.addEventListener(
    "visibilitychange",
    handleVisibilityChange,
    options
  );

  return () => {
    document.removeEventListener(
      "visibilitychange",
      handleVisibilityChange,
      options
    );
  };
}, []);
~~~

A newly created function with the same body is a different function reference, so it will not remove the registered listener. Likewise, keep the event type and capture behavior consistent between registration and removal.

If setup registers several related listeners, cleanup removes each one:

~~~jsx
useEffect(() => {
  function handleOnline() {
    setOnline(true);
  }

  function handleOffline() {
    setOnline(false);
  }

  window.addEventListener("online", handleOnline);
  window.addEventListener("offline", handleOffline);

  return () => {
    window.removeEventListener("online", handleOnline);
    window.removeEventListener("offline", handleOffline);
  };
}, []);
~~~

The state setters are stable. These handlers do not read changing props or state, so an empty dependency list is appropriate for this example.

## Timers: keep and clear the timer identifier

Store the identifier created by this setup and clear that same identifier in cleanup:

~~~jsx
useEffect(() => {
  if (!running) return;

  const timerId = window.setInterval(() => {
    setSeconds((currentSeconds) => currentSeconds + 1);
  }, 1000);

  return () => {
    window.clearInterval(timerId);
  };
}, [running]);
~~~

When running changes from true to false, cleanup clears the active interval. When it changes from false to true, a new setup creates one. The functional state updater lets the callback add to the latest state without capturing seconds.

The same pattern applies to a timeout:

~~~jsx
useEffect(() => {
  const timeoutId = window.setTimeout(showHint, delay);

  return () => window.clearTimeout(timeoutId);
}, [delay, showHint]);
~~~

If delay or showHint changes, React cancels the old timeout and schedules a new one. Include any reactive values that setup reads.

## Subscriptions and connections

An external subscription normally provides an unsubscribe function. Capture and invoke that function during cleanup:

~~~jsx
useEffect(() => {
  const unsubscribe = store.subscribe(() => {
    setSnapshot(store.getSnapshot());
  });

  return unsubscribe;
}, [store]);
~~~

A connection may have explicit connect and disconnect methods:

~~~jsx
useEffect(() => {
  const connection = createConnection(roomId);
  connection.connect();

  return () => connection.disconnect();
}, [roomId]);
~~~

The connection is created for this setup and the cleanup closes over the same instance. Do not disconnect a connection created by a later setup.

Some APIs may deliver a final queued callback after unsubscribe or disconnect begins. If that API documents this behavior, guard the callback with a local active flag and turn it off before releasing the resource. Prefer the API's documented cancellation contract rather than adding guards without a reason.

## Observers and third-party objects

Observers also have explicit lifetimes. Disconnect an observer created by the setup:

~~~jsx
useEffect(() => {
  const node = targetRef.current;
  if (!node) return;

  const observer = new ResizeObserver((entries) => {
    const entry = entries[0];
    if (entry) {
      setWidth(entry.contentRect.width);
    }
  });

  observer.observe(node);

  return () => observer.disconnect();
}, []);
~~~

When observing several targets but keeping the observer alive, cleanup can unobserve the exact targets. If the observer is only used by this Effect, disconnecting it is often the simpler release.

A third-party widget may expose its own setup and disposal methods:

~~~jsx
useEffect(() => {
  const widget = createWidget(hostRef.current, configuration);
  widget.mount();

  return () => widget.destroy();
}, [configuration]);
~~~

Follow the library's contract. Cleanup might need to call destroy, dispose, stop, or remove, and may need to release more than one resource. Avoid assuming that removing the React component automatically disposes objects created outside React.

## Native dialogs and reversible DOM state

Some browser methods are not safely repeatable without a matching inverse. A modal dialog opened with showModal must be closed before another setup opens it:

~~~jsx
useEffect(() => {
  const dialog = dialogRef.current;
  if (!dialog) return;

  dialog.showModal();

  return () => {
    dialog.close();
  };
}, []);
~~~

The setup-cleanup-setup sequence is then valid: open, close, and open again. The visible result is one open dialog, while the earlier dialog state has been properly released.

For a visual style applied directly through a third-party API, cleanup should restore the prior visual state or cancel the animation. If that work must happen before paint to prevent flicker, chapter 052 explains when useLayoutEffect is appropriate.

## Async requests: abort work and guard results

A request associated with a changing identifier can finish after the component has moved on. Cleanup should prevent obsolete work from updating the current UI.

~~~jsx
useEffect(() => {
  const controller = new AbortController();
  let active = true;

  async function loadRecord() {
    try {
      const response = await fetch(
        "/api/records/" + encodeURIComponent(recordId),
        { signal: controller.signal }
      );

      if (!response.ok) {
        throw new Error("The record request failed.");
      }

      const record = await response.json();

      if (active) {
        setRecord(record);
      }
    } catch (error) {
      if (active && !controller.signal.aborted) {
        setError(error);
      }
    }
  }

  loadRecord();

  return () => {
    active = false;
    controller.abort();
  };
}, [recordId]);
~~~

When recordId changes, cleanup marks the old setup inactive and aborts its request. The active check also guards against APIs or asynchronous work that cannot be cancelled, or work that has already progressed far enough that cancellation alone is not sufficient.

Cleanup cannot reverse a request the server has already processed. Aborting a request is not a guarantee that a remote operation never happened. For a user-triggered write such as submitting an order, handle the action in its event handler and use the server's idempotency or duplicate-submission strategy where needed.

A fetching Effect is a low-level tool. Framework data APIs or a query library can also provide caching, request deduplication, preloading, and server rendering.

## Cleanup and asynchronous teardown

React calls the cleanup function; it does not wait for a Promise returned by cleanup. Therefore, do not make cleanup itself async and expect React to await its completion:

~~~jsx
// Avoid relying on React to wait for this Promise.
return async () => {
  await connection.close();
};
~~~

If a resource has asynchronous shutdown, start that shutdown from a synchronous cleanup and handle its completion or failure according to the resource's API:

~~~jsx
return () => {
  connection.close().catch(reportCloseError);
};
~~~

Whether shutdown should block another setup is a property of the external system. If it cannot safely have old and new instances alive at the same time, that system needs an explicit serialized lifecycle strategy. React's Effect cleanup callback does not provide that serialization by itself.

## Strict Mode tests resource lifetimes

With Strict Mode enabled, React runs one extra setup-and-cleanup cycle for Effects in development. A connection can therefore connect, disconnect, and connect again during initial development setup.

This is a stress test for resource ownership and cleanup. The user should not be able to distinguish one production setup from the development sequence of setup, cleanup, and setup. There should never be two live subscriptions or timers when the design intends one.

Do not use a ref merely to make setup appear to run once. That hides missing cleanup and does not fix what happens when dependencies change or the component is removed and later mounted again.

## Common cleanup mistakes

### Returning no cleanup for a continuing resource

An interval, subscription, or connection keeps running after the UI no longer needs it. Return a cleanup that stops that exact resource.

### Cleaning up a different resource

A shared variable was overwritten by a later setup, so an earlier cleanup closes the wrong instance. Keep each resource local to its own setup.

### Removing a different event handler

A new function was created in cleanup instead of removing the exact handler registered in setup. Define the handler inside setup and reuse its reference.

### Treating cleanup as only an unmount callback

Cleanup also runs before setup re-runs for changed dependencies. Make it safe to release the old synchronization whenever its inputs change.

### Preventing Strict Mode from repeating setup

A ref flag hides the extra development check but does not release the resource. Make setup and cleanup repeatable instead.

### Assuming abort reverses a server action

A request may already have reached and been processed by the server. Cancellation limits local work and stale results; it cannot roll back a completed remote operation.

### Updating unrelated React state during cleanup

Cleanup should release the external work it owns. Do not use it as a general place to reset application state. Model UI state with React state transitions and render logic.

## A resource-lifetime review

For every Effect that acquires a resource, ask:

1. What exact resource does this setup create or acquire?
2. Which local value identifies that resource?
3. What exact API releases or reverses it?
4. Does cleanup close over that same resource?
5. Will cleanup work before a replacement setup and on unmount?
6. Can callbacks or asynchronous results still arrive after cleanup?
7. Is cancellation local only, or does the remote system need its own duplicate protection?
8. Does the Effect remain correct through Strict Mode's extra development cycle?

## Key takeaways

- Setup and cleanup define one external resource's lifetime.
- Cleanup runs before re-synchronization and on unmount, using values from the setup it cleans up.
- Release the exact listener, timer, subscription, connection, observer, dialog, or animation created by that setup.
- Make cleanup and setup safe to repeat in Strict Mode.
- Abort requests when possible and ignore results from obsolete work.
- Cancellation cannot undo a remote operation that has already completed.
- React does not await asynchronous cleanup.
- Keep cleanup focused on releasing the external work that its setup owns.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="053-effect-dependencies-and-closures.md">053 - Effect Dependencies, Closures, and Synchronization</a></td><td><a href="../qnas/054-effect-cleanup-and-resource-lifetimes.md">Q&amp;A</a></td><td><a href="../examples/054-effect-cleanup-and-resource-lifetimes.md">Example</a></td><td><a href="055-when-an-effect-is-unnecessary.md">055 - When an Effect Is Unnecessary</a></td></tr>
  </tbody>
</table>
