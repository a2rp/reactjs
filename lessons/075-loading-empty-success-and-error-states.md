# Loading, Empty, Success, and Error States

A request does not have only two outcomes. A useful data view must tell the difference between waiting for a response, showing results, showing that no results matched, and reporting a failure. These states should reflect what happened and offer a clear next action.

This lesson models a client-side request as a small state machine, renders each state intentionally, and keeps state tied to the query that produced it. It builds on [lesson 073](073-client-side-data-fetching-and-use-api.md) and [lesson 074](074-request-cancellation-and-race-conditions.md).

## Learning goals

After this lesson, you should be able to:

- model loading, success, empty, and error UI explicitly;
- distinguish an empty successful response from a failed request;
- avoid contradictory combinations of status booleans;
- render accessible status and error messages;
- provide a useful retry action;
- decide whether a refresh replaces or preserves previously loaded data;
- prevent data from one query appearing to belong to another.

## Model the request as states

A simple request follows this flow:

~~~text
loading ── success with items ── results
   │
   ├────── success with no items ── empty state
   │
   └────── error ── error message and retry
~~~

Empty is not a network phase. It is a successful response whose data contains no items to display. Keep it distinct from an error:

- **Empty:** the request succeeded; there are no matching records.
- **Error:** the request did not produce usable data.
- **Loading:** the request has not finished.
- **Success with results:** the request succeeded and produced records.

The server's API contract determines what counts as empty. A 200 response with an empty array commonly represents no matching records; a 404 may instead indicate that a specific resource does not exist. Interpret the status and body according to that API rather than treating every empty-looking response the same way.

## Avoid impossible boolean combinations

Several independent flags can describe states that should never happen together:

~~~jsx
const [isLoading, setIsLoading] = useState(false);
const [hasError, setHasError] = useState(false);
const [items, setItems] = useState([]);
~~~

This allows invalid combinations such as loading and error both being true, or an error flag with old items that appear current. Every transition must remember to update every flag.

Use one status field to make the active state explicit:

~~~jsx
const [result, setResult] = useState({ status: "loading" });

// Later:
setResult({ status: "success", query, items });
setResult({ status: "error", query, error });
~~~

For TypeScript, a discriminated union can make invalid states harder to express:

~~~ts
type Result<T> =
  | { status: "loading"; query: string }
  | { status: "success"; query: string; data: T }
  | { status: "error"; query: string; error: Error };
~~~

Then render from the status:

~~~jsx
if (result.status === "loading") {
  return <p role="status">Loading results…</p>;
}

if (result.status === "error") {
  return (
    <div role="alert">
      <p>Could not load results: {result.error.message}</p>
      <button onClick={retry}>Try again</button>
    </div>
  );
}

if (result.data.length === 0) {
  return <p>No results match this search.</p>;
}

return <Results items={result.data} />;
~~~

In JavaScript, follow the same shape even without TypeScript checking it. Use one source of truth for the state.

## Render each state with a useful message

### Loading

Tell users that content is on the way. A small inline message, progress indicator, or skeleton can work depending on the layout and expected wait. Avoid showing a blank region that looks broken.

For screen readers, a changing status can use `role="status"`, which announces a polite update. Keep the message concise. Don't announce every tiny progress step for a simple request.

### Success with results

Render the data and give repeated elements stable keys. Ensure the structure works with keyboard navigation and narrow screens. Keep the result visible until the next request begins or intentionally preserve it during refresh.

### Empty

Explain why there is no content and what the user can do next. The best message depends on the cause:

- **No records exist yet:** offer the action that creates the first record.
- **No search matches:** suggest changing or clearing the search.
- **Filters removed all results:** make it easy to adjust filters.

An empty state is part of the successful experience, not a failure. Do not show a generic error or spinner when the data loaded successfully as an empty list.

### Error

Explain that the request failed in plain language and offer a relevant recovery action, such as retrying or changing a query. Use `role="alert"` for an important error that should be announced immediately. Do not expose stack traces, credentials, or raw server internals to the user.

A Fetch request can resolve even for an HTTP error status, so check `response.ok` before treating the body as successful data:

~~~js
const response = await fetch("/api/products");

if (!response.ok) {
  throw new Error("Request failed with status " + response.status);
}

const products = await response.json();
~~~

Network failures, non-success HTTP responses, invalid JSON, and application-level errors all need a defined path into the error state.

## Keep the result associated with its query

When a request depends on a query, save that query alongside the result. During render, compare the saved key with the current key:

~~~jsx
const visibleResult =
  result.query === query
    ? result
    : { status: "loading", query };
~~~

This prevents results from a previous query flashing under a new search term before the Effect for the new term has started. It also makes the UI's relationship to the data explicit.

When the query changes, reset the request state for the new query. Cleanup from lesson 074 should abort supported old work or ignore its result so it cannot publish stale success or error state.

## Retry through a clear user action

A retry should start a new request, not merely hide the error. Keep the request code in one place and trigger it with a version, key, or action:

~~~jsx
const [requestVersion, setRequestVersion] = useState(0);

function retry() {
  setRequestVersion((version) => version + 1);
}

useEffect(() => {
  // Start the request for the current query and version.
}, [query, requestVersion]);
~~~

The Effect depends on the version so retrying the same query starts work again. If a query change already triggers a request, do not also trigger a duplicate request in the same interaction.

If the user needs to edit a form field or authentication before retrying, guide them to that action. Avoid an endless automatic retry loop for a persistent failure.

## Initial loading and background refresh

For a first load, showing a loading state in place of absent data is usually clear. Refreshing already displayed data is a separate design choice:

- **Replace while loading:** hide the old data and show a loading state. This is simple, but can feel disruptive.
- **Keep old data while refreshing:** leave the existing results visible and show a small “Refreshing…” status. Mark that the data may be stale.
- **Keep old data after a refresh error:** this can be useful if the content remains safe to use. Explain that refresh failed and offer retry.

If the UI preserves data, model that state explicitly, for example `{ status: "refreshing", data, query }`. Do not label old data as newly loaded. Data libraries often provide these distinctions as part of their query state.

## Effects and Suspense have different loading UI

An Effect-based request sets state and renders its own loading, empty, success, or error UI. A Suspense-enabled Promise can show a Suspense fallback while pending and send a rejection to an Error Boundary. An Error Boundary does not replace the empty state, and an Effect request does not activate Suspense on its own. See [lesson 073](073-client-side-data-fetching-and-use-api.md).

For ordinary request state, explicit state is often easier to reason about. Use the loading mechanism your data source actually supports; do not add a Suspense boundary and expect it to catch every fetch.

## Forms and mutations have their own states

A form submission may need states such as idle, submitting, succeeded, and failed, plus field-specific validation messages. Keep those separate from the loading state of a page's read request.

Start a user-triggered write in the submit or button handler. Do not send it from an Effect just because a state variable changed. A failed write may need to preserve the user's input so they can correct or retry it.

## Accessibility details

- Put loading announcements in a concise element with `role="status"`.
- Put important request failures in an element with `role="alert"`.
- Keep the retry action a real button with a descriptive label.
- Do not communicate state using color alone.
- Keep focus in a sensible place after an update; do not move focus for every background refresh.
- If a whole region is temporarily being updated, consider `aria-busy="true"` on that region and remove it when the update completes.

Use these semantics only where they match the interaction. Too many live announcements can become noisy.

## Common mistakes

- Treating a successful empty array as an error.
- Showing the same generic message for empty, loading, and failed states.
- Keeping several booleans that can form contradictory combinations.
- Showing results for the previous query as though they match the new query.
- Ignoring errors from response parsing or non-success HTTP statuses.
- Rendering a Retry button that does not actually start a new request.
- Automatically retrying a persistent failure without a limit or user control.
- Clearing useful data during every background refresh without considering the experience.
- Keeping stale data visible without labeling it as stale.
- Using `role="alert"` for every minor update and overwhelming assistive technology.
- Expecting Suspense to handle an Effect-based request.

## Practical checklist

1. Is the request pending, successful, empty, or failed?
2. Is empty represented as successful data rather than an error?
3. Can impossible combinations of loading, success, and error appear?
4. Does each result carry the query or key that produced it?
5. Does changing the query make old work unable to update current state?
6. Does Retry actually start a new attempt?
7. During refresh, should old data disappear, remain marked as stale, or remain with a refresh error?
8. Are loading and error changes announced accessibly?
9. Are HTTP status, response parsing, and application errors handled?
10. Does the chosen state model match the data library or framework already in use?

## Summary

A reliable data view distinguishes pending work, successful results, successful emptiness, and failure. Model the state explicitly, associate data with its query, and make retry a real action. Decide intentionally whether refresh replaces existing data or keeps it visible as stale, and use accessible status and error messages.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="074-request-cancellation-and-race-conditions.md">074 - Request Cancellation, Stale Responses, and Race Conditions</a></td><td><a href="../qnas/075-loading-empty-success-and-error-states.md">Q&amp;A</a></td><td><a href="../examples/075-loading-empty-success-and-error-states.md">Example</a></td><td><a href="076-caching-and-revalidation.md">076 - Caching, Deduplication, Invalidation, and Revalidation</a></td></tr>
  </tbody>
</table>
