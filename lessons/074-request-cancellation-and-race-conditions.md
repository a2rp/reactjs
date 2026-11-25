# Request Cancellation, Stale Responses, and Race Conditions

When a search term, selected record, or route changes quickly, several requests can be in flight at once. Their responses may arrive in a different order from the requests. If every response updates the same state, an older response can overwrite the data for the user's latest choice.

An Effect cleanup can mark work obsolete, and an `AbortController` can cancel supported operations such as `fetch`. These solve related but different problems: ignoring a result protects UI correctness; aborting can also stop client-side work that is no longer needed.

## Learning goals

After this lesson, you should be able to:

- explain how response ordering creates a race condition;
- ignore the result of an Effect after its inputs change or it unmounts;
- abort a fetch and response-body read with an `AbortSignal`;
- distinguish cancellation from stale-result protection;
- prevent an older result from being shown for a newer query;
- choose between cancellation, ignore flags, and request identifiers;
- understand what cancellation does not guarantee.

## What is a stale response?

Suppose the user types `c`, then `cs`, then `css`. The app starts one request for each value:

~~~text
Request order:   "c"  →  "cs"  →  "css"
Response order:  "css" →  "cs" →  "c"
~~~

If every completion calls `setResults`, the last response to arrive wins. In this example the final screen shows results for `c` even though the input contains `css`.

This is a race condition: the program's result depends on which asynchronous operation finishes first. The UI usually wants to keep the result associated with the latest relevant input, regardless of completion order.

Races can occur with searches, route parameters, filters, selected users, autocomplete, and any asynchronous operation whose earlier result can become irrelevant.

## Mark obsolete Effect work as ignored

React runs an Effect's cleanup before it runs that Effect again with changed dependencies, and when the component unmounts. A boolean local to each Effect setup can mark that setup obsolete:

~~~jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    try {
      const items = await fetchItems(query);

      if (!ignore) {
        setItems(items);
      }
    } catch (error) {
      if (!ignore) {
        setError(error);
      }
    }
  }

  load();

  return () => {
    ignore = true;
  };
}, [query]);
~~~

Each setup has its own `ignore` variable. When the query changes, cleanup flips the old setup's variable. If that request finishes later, its continuation sees `ignore === true` and does not update state.

This prevents an obsolete result from affecting the component. It does **not** cancel the network request. The browser and server may continue doing work until that request finishes.

Use the same relevance check for every state update associated with the asynchronous operation, including loading, success, and error state. Do not let an obsolete failure replace the current query's UI with an error either.

## Abort an operation with `AbortController`

The browser's Fetch API accepts an `AbortSignal`. Aborting its controller signals the request to stop:

~~~jsx
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    try {
      const response = await fetch(
        "/api/items?q=" + encodeURIComponent(query),
        { signal: controller.signal },
      );

      if (!response.ok) {
        throw new Error("Request failed: " + response.status);
      }

      const items = await response.json();
      setItems(items);
    } catch (error) {
      if (controller.signal.aborted) {
        return;
      }

      setError(error);
    }
  }

  load();

  return () => controller.abort();
}, [query]);
~~~

Create a new controller for each Effect setup. An aborted signal stays aborted; it cannot be reset and reused for the next query.

For Fetch, aborting can reject the request, and it can also abort response-body consumption if the response has arrived but the body has not been read yet. Treat abort as expected control flow for an obsolete request, not as a user-facing failure.

## Use cancellation and an ignore guard together

Cancellation is useful when the operation supports it. A relevance guard is useful for state correctness even if cancellation is delayed, unsupported, or another async step does not observe the signal.

~~~jsx
useEffect(() => {
  const controller = new AbortController();
  let ignore = false;

  async function load() {
    setResult({ status: "loading", query });

    try {
      const response = await fetch(
        "/api/items?q=" + encodeURIComponent(query),
        { signal: controller.signal },
      );

      if (!response.ok) {
        throw new Error("Request failed: " + response.status);
      }

      const items = await response.json();

      if (!ignore) {
        setResult({ status: "success", query, items });
      }
    } catch (error) {
      if (ignore || controller.signal.aborted) {
        return;
      }

      setResult({ status: "error", query, error });
    }
  }

  load();

  return () => {
    ignore = true;
    controller.abort();
  };
}, [query]);
~~~

Here the controller asks supported work to stop, while `ignore` prevents this Effect setup from publishing stale success or error state. The pattern is particularly clear when an operation includes work that may not support cancellation.

Do not add guards mechanically to every async function. Understand who owns the request and which results are still relevant. A framework or query library may already implement cancellation, request deduplication, or stale-result handling.

## Prevent old state from appearing under a new input

Cleanup runs when React processes the new Effect. A render for the changed query can happen before that Effect sets loading state. Store the query (or another resource key) alongside the result and only display it for the matching input:

~~~jsx
function Results({ query, result }) {
  if (result.query !== query || result.status === "loading") {
    return <p role="status">Loading results…</p>;
  }

  if (result.status === "error") {
    return <p role="alert">{result.error.message}</p>;
  }

  return <ResultsList items={result.items} />;
}
~~~

This is a separate concern from cancelling a request. Cancellation prevents or reduces obsolete work; associating state with its key prevents results for one key from being presented as if they belong to another.

Sometimes an app intentionally keeps old results visible while refreshing. That can be a good design, but label or style them as previous results so users are not misled into thinking they match the new query.

## Use a request identifier when needed

An incrementing request ID is useful when requests are started from several paths or when a single component can have overlapping requests outside one Effect instance:

~~~jsx
const latestRequest = useRef(0);

async function search(query) {
  const requestId = ++latestRequest.current;

  setResult({ status: "loading", query });

  try {
    const items = await fetchItems(query);

    if (requestId === latestRequest.current) {
      setResult({ status: "success", query, items });
    }
  } catch (error) {
    if (requestId === latestRequest.current) {
      setResult({ status: "error", query, error });
    }
  }
}
~~~

Only the latest-started request may publish a result. This is a "latest request wins" policy. It is not always the correct policy: pagination may need multiple requests to contribute results, and independent panels may each have their own current request. Keep identifiers scoped to the state they protect.

An ignore flag scoped to an Effect is often simpler for Effect-driven fetching. Choose the smallest rule that matches the request lifecycle.

## Cancellation is not rollback

Aborting a browser request does not guarantee that a server never received it or stopped processing it. The request may have reached the server before the abort signal was handled. Cancellation is useful for stopping client-side waiting, transfer, and body reading where supported; it is not a transaction rollback.

This matters especially for writes. Do not use component cleanup as the mechanism for undoing a purchase, saving a record, or sending another user-visible action. Start such work from the user event that caused it, and design server-side write operations with appropriate correctness guarantees.

For shared or cached requests, one component should not blindly abort work that another component still needs. A data library can coordinate multiple subscribers and decide when a shared request is no longer needed.

## Cancellation is not debouncing

Cancellation and debouncing have different goals:

- **Cancellation** stops or ignores work that is no longer relevant.
- **Debouncing** waits until input has been quiet for a period before starting work.

A search can use both: debounce to avoid starting a request for every keystroke, then cancel or ignore a request if the query changes while it is in flight. Cancellation alone does not reduce the number of requests already started.

## Handle aborts without hiding real errors

A common pattern checks the signal in the catch block:

~~~js
try {
  const response = await fetch(url, { signal: controller.signal });
  // Check response.ok and read the body.
} catch (error) {
  if (controller.signal.aborted) {
    return;
  }

  reportError(error);
}
~~~

Checking the specific controller's signal is robust when aborts use custom reasons. Checking `error.name === "AbortError"` is also common for Fetch APIs using the default abort reason, but unrelated errors should still reach the normal error UI.

Do not silently ignore all errors. A network failure, malformed JSON, server error status, or programming bug is not the same as a deliberate cancellation.

## Strict Mode and Effect cleanup

In development, Strict Mode may run an extra setup-and-cleanup cycle for an Effect to expose missing cleanup. A request can appear in the Network panel and then be aborted during this check. Make the Effect safe to set up, clean up, and set up again; do not hide the behavior with a ref that forces it to run only once.

The correctness goal is not "only one request is ever visible in development." The goal is that obsolete work cannot corrupt the current UI and that cleanup stops work the Effect no longer owns.

## Common mistakes

- Assuming responses always arrive in request order.
- Updating state after every response without checking whether it is still relevant.
- Ignoring stale success but allowing stale errors to replace the current UI.
- Assuming an ignore flag cancels network or server work.
- Assuming an abort always rolls back server processing.
- Reusing an already-aborted controller for a later Effect.
- Treating expected cancellation as a visible application error.
- Swallowing every error just because the request can be aborted.
- Assuming cancellation also debounces input or prevents future requests.
- Aborting a shared request while another component still needs it.
- Showing old data as if it belongs to a new query.
- Using Effect cleanup to undo a user action that should have started in an event handler.

## Practical checklist

1. What input makes this request obsolete?
2. Does cleanup run when that input changes and when the component unmounts?
3. Can the operation accept an `AbortSignal`?
4. Do all success and error updates verify that the request is still current?
5. Is the rendered result associated with the query or resource key that produced it?
6. Is abort handled as expected control flow while real errors remain visible?
7. Is cancellation sufficient, or does this input also need debouncing?
8. Is this request shared with other components or managed by a data library?
9. Is the request a write whose server-side effects cannot be undone by aborting?
10. Would a request ID or cache policy express the intended ordering more clearly?

## Summary

Asynchronous responses can arrive out of order. Effect cleanup can ignore results from obsolete setups; `AbortController` can signal supported work to stop; request IDs can enforce a latest-request-wins rule across overlapping operations. Use cancellation to reduce obsolete work and relevance checks to protect state. Keep results tied to their query, handle abort separately from real failures, and remember that aborting a browser request is not a server-side rollback.