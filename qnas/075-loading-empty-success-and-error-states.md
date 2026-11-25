# Loading, Empty, Success, and Error States — Q&A

## State model

### 1. What are the four data-view states in this lesson?

Loading while work is pending, success when usable data arrives, empty when success contains no items, and error when the request or its data cannot be used.

### 2. Is an empty result the same as an error?

No. Empty means the operation succeeded and returned no matching data. Error means the operation failed. These should have different UI and different recovery actions.

### 3. Is “empty” always an API status?

No. It is usually a UI interpretation of successful data, such as an empty array. The API contract determines whether an HTTP response means no records, a missing resource, or a failure.

### 4. Should an initial request have an idle state too?

It can. A request that has not started may be idle, for example before a user submits a search. A request that starts immediately on screen load can begin in loading. Choose states that describe actual transitions in the component.

### 5. Why not represent request state with several booleans?

Independent booleans allow contradictory combinations such as loading and error at the same time. A single status field makes the active state explicit. TypeScript discriminated unions can also require each status to carry only its relevant data.

### 6. How can a discriminated union help?

A TypeScript union such as `loading | success | error` lets the compiler narrow the available properties by status. Loading does not have to pretend it already has an error, and error does not have to carry a successful data array.

### 7. Should the empty case be a separate request status?

Usually it is a successful response whose data is empty. The UI can branch on the successful data length. A separate `empty` status may be appropriate in a chosen state machine, but do not confuse it with a failed request.

### 8. What are examples of useful explicit states beyond the basic four?

An idle state before a request, submitting for a form mutation, refreshing while keeping prior data visible, refresh-error while retaining usable prior data, or pagination loading. Add a state when it changes the UI meaningfully.

## Loading and success UI

### 9. What should a loading state communicate?

That the requested content is being loaded and the interface is still working. Use a concise message, progress indicator, or skeleton that fits the layout; avoid leaving a large blank region that looks broken.

### 10. Is a spinner always the right loading indicator?

No. A small inline update may need only text, while a larger content area may benefit from a skeleton. The indicator should match the size and expected duration of the work.

### 11. How should a successful non-empty result be rendered?

Show the returned data in a useful semantic structure, provide stable keys for repeated React elements, and ensure the content remains usable with keyboard navigation and assistive technology.

### 12. Why should repeated list items have stable keys?

Keys let React match items between renders. Stable data identifiers help React update the correct item when results change; array indexes can be misleading when items are inserted, removed, or reordered.

### 13. Should a successful result always replace the whole page?

Not necessarily. A request may update one section while the rest of the page remains usable. Keep the loading and error UI scoped to the data region that is changing.

### 14. Should a short request still show a loading indicator?

That is a design choice. A very brief spinner can flicker, but hiding all feedback can make a slow request seem unresponsive. Frameworks and data libraries may provide a delay or transition strategy.

## Empty and error states

### 15. How should an empty-state message differ for search and a new account?

For search, explain that nothing matched and suggest changing or clearing the query. For a new account, explain that no records exist yet and offer the action that creates the first one.

### 16. Should an empty state include an action?

When there is a useful next step, yes. Examples include clearing filters, changing a query, inviting a teammate, or creating the first item. Do not add an irrelevant action just to fill space.

### 17. Does `fetch` reject on a 404 response?

Usually no. It resolves with a `Response` for HTTP error statuses. Check `response.ok` or the status and map the response according to the API contract.

### 18. Which failures might an application need to represent?

Network failure, a non-success HTTP status, invalid or unreadable JSON, authentication or authorization failure, and application-level validation errors. The user-facing response depends on which kind of operation failed.

### 19. What should a data-fetch error message say?

Use plain language about what failed and offer a relevant recovery action such as retry. Do not expose stack traces, secret values, or raw server internals to ordinary users.

### 20. When should an error use `role="alert"`?

For important information that should be announced promptly, such as a failed operation that blocks the user. Do not use assertive alerts for every small background update; they can become noisy.

### 21. When should loading text use `role="status"`?

When a concise status change should be announced politely without moving focus, such as a search result loading message. Use live-region semantics only when they improve access to the update.

### 22. What does `aria-busy="true"` communicate?

That a region is being updated. It can be useful for a region that is temporarily changing, and should be cleared when that update is complete. It does not replace visible feedback or correct state modeling.

## Retry and refresh

### 23. What should a Retry button do?

Start a new request attempt. Hiding the error message without re-running the operation is not a retry.

### 24. How can an Effect retry the same query?

Include a request version or retry key in the Effect dependencies. Increment it when the user activates Retry. The query remains the same, but the key change starts a new attempt.

### 25. Should an app automatically retry every failed request forever?

No. Persistent errors can create load and hide a real problem. If automatic retries are used, they should have a deliberate limit and backoff policy; keep a user-controlled recovery path where appropriate.

### 26. Should old data disappear during a refresh?

That depends on the interaction. A simple implementation may replace it with loading UI. Another may keep the old data visible and show a refreshing status. Clearly indicate that the visible results may be stale.

### 27. What should happen if a refresh fails while old data is still useful?

The app may keep the previous data, label it as not refreshed, show a refresh failure, and offer Retry. Do not make stale data appear newly current.

### 28. How should an app avoid a duplicate request when changing a query and retrying?

Choose one request trigger for that interaction. If the Effect already runs when the query changes, do not also increment a retry key unless a second attempt is actually desired.

## Query identity and asynchronous behavior

### 29. Why store the query with the result?

It identifies which request produced that data. The render can avoid presenting an old query's result as though it belongs to the current input.

### 30. Can old results briefly appear before the new Effect starts?

They can if the component renders after the input changes but before the Effect resets state. Compare the result's query/key with the current one during render, or use a data layer that manages the key.

### 31. How does this lesson use the previous cancellation lesson?

When the query changes, the earlier request may become irrelevant. Cleanup should abort supported work or ignore its result so stale success and error responses cannot update the current state. See lesson 074.

### 32. Should errors and success results use the same stale-result guard?

Yes. An obsolete error can be just as misleading as an obsolete success. Only the current request should publish its status to the current query's UI.

### 33. If an old result remains visible while refreshing, is it necessarily a bug?

No. Stale-while-revalidate interfaces may intentionally keep it visible. Make its stale/refreshing status clear and do not associate it with a different query.

## Effects, Suspense, and forms

### 34. Does a request started in an Effect show a Suspense fallback?

No. An Effect-based request updates state and renders explicit loading UI. Suspense handles render-time suspension from a Suspense-enabled data source.

### 35. Does an Error Boundary replace request error state?

Not always. A Promise rejection read through `use` can reach an Error Boundary. An ordinary Effect catches or records its request error in state. These are different error paths.

### 36. Should form submission status share the same state as a page's data load?

Usually keep them separate. A page read may be loading while a form is idle, or the page may be loaded while a save is submitting. Separate states should describe separate operations.

### 37. Where should a user-triggered write request start?

In the event handler for the submit or button action that caused it. A write should not be triggered by an Effect merely because some state changed.

### 38. Should failed form submission clear the user's input?

Usually no. Preserve their entered data so they can correct the problem or retry, unless the product has a clear reason to discard it.

## Debugging and design choices

### 39. What should I check when the UI stays on loading forever?

Check that the Promise settles, error paths update state, the Effect dependencies and cleanup are correct, and no obsolete-result guard accidentally blocks the current request. Inspect the Network panel and the request key being rendered.

### 40. What should I check when an empty state appears after a server error?

Verify that the code checks `response.ok` and handles status codes before parsing the body as successful data. A failed response should not be converted into an empty array unless the API contract explicitly defines that behavior.

### 41. Is a blank page an acceptable error state?

Usually not. It gives no explanation or recovery path. Render a scoped error message or an application-level fallback appropriate to the impact of the failure.

### 42. How do I choose whether to keep stale data on refresh?

Ask whether old data remains useful and safe, whether users can distinguish it from fresh data, and how disruptive replacing it would be. Use an explicit refreshing or refresh-error state if it remains visible.

### 43. What is the simplest state model for a basic search request?

A status with loading, success, and error branches, plus a check for an empty result within success. Include the query key so stale data cannot be mistaken for current results.

### 44. What is the key principle for reliable request UI?

Describe the operation honestly. Loading means pending, empty means success with nothing to show, success means usable data arrived, and error means the operation failed. Tie each result to the request key and make recovery a real action.