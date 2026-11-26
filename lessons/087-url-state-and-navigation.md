# URL State, Search Parameters, and Navigation

## Learning goals

After this lesson, you should be able to:

- Tell path parameters, query parameters, and URL fragments apart.
- Decide which state belongs in the URL and which belongs in React state.
- Read, validate, and update search parameters safely.
- Use URL state to support sharing, bookmarks, reloads, and browser Back and Forward.
- Choose between adding a history entry and replacing the current one.
- Recognize privacy and framework-specific limits of URL state.

## 1. Why URLs are part of application state

A URL is more than an address for a page. It can describe which resource is open and which view of that resource the user selected.

For example:

```text
/products/keyboard?color=black&sort=price#reviews
```

This URL contains three different pieces:

- **Path**: `/products/keyboard` identifies a resource or route.
- **Search (query) string**: `?color=black&sort=price` describes view options.
- **Fragment (hash)**: `#reviews` points to a location within the document or application.

The browser exposes the current URL through `window.location`. A router builds on the browser's navigation mechanisms and coordinates URL changes with the rendered route.

When view state is encoded in the URL, a user can copy it, bookmark it, reload the page, or return to it with Back and Forward. This works only when the application can reconstruct that view from the URL.

## 2. Choose the right place for state

Keep state in the URL when it should identify a view that a person may want to revisit or share:

- Search terms that are appropriate to expose.
- Filters and sort order.
- A selected page of results.
- A selected tab when that tab is part of the page's navigable view.
- A resource identifier, usually in the path.

Keep state in React component state when it is temporary UI detail:

- Whether a menu is open.
- A text field draft that has not been submitted.
- A hover or focus state.
- An unsaved, short-lived interaction.

Server or remote data is a separate concern. A URL can describe *which* data to request, while a data-fetching layer manages loading, caching, errors, and freshness.

Avoid storing every piece of component state in the URL. A URL should describe a useful, reproducible view, not every moment-to-moment interaction.

## 3. Search parameters are strings

Use the built-in `URLSearchParams` interface to read and construct a query string:

```js
const params = new URLSearchParams("?q=blue%20mug&page=2");

params.get("q");       // "blue mug"
params.get("page");    // "2"
params.get("missing"); // null
```

Important details:

- Values are strings. Convert and validate numbers, booleans, and enums yourself.
- `get(name)` returns the first value for that name, or `null` when it is absent.
- `getAll(name)` returns every value for a repeated key.
- `set(name, value)` replaces that key's values with one value.
- `append(name, value)` adds another value for the same key.
- `delete(name)` removes that key.
- `toString()` serializes the parameters without the leading question mark.
- The API handles encoding and decoding. Avoid manually concatenating untrusted values into a query string.

Repeated parameters are valid:

```js
const params = new URLSearchParams("tag=beginner&tag=react");

params.get("tag");    // "beginner"
params.getAll("tag"); // ["beginner", "react"]
```

Choose a repeated-key format only when the application needs multiple values for one key. Keep parsing and updating consistent with that format.

## 4. Parse and validate URL input

Users can edit a URL, old bookmarks can outlive an interface, and links can contain malformed values. Treat every URL value as input, then apply explicit rules.

For a positive page number:

```js
function readPage(value) {
  const page = Number(value);

  return Number.isInteger(page) && page > 0 ? page : 1;
}
```

For a choice from a fixed set:

```js
const allowedSorts = new Set(["relevance", "price", "name"]);

function readSort(value) {
  return allowedSorts.has(value) ? value : "relevance";
}
```

Decide what to do with invalid or missing values. Common choices include using a default, showing a not-found state, or redirecting to a canonical URL. Do not assume that a value is valid just because your own controls normally generate it.

Also consider duplicate values. If the application expects one `page` parameter, decide whether to use the first value, reject duplicates, or normalize them.

## 5. Use a router to connect URL changes to React

A React router provides route-aware links and navigation. Use its APIs instead of assigning `window.location` for ordinary in-app navigation. Assigning `window.location` causes a document navigation and may reload the application.

In React Router, `useSearchParams` reads the current query parameters and returns a setter. Updating the parameters navigates to the resulting URL and causes the matching React route to render with the new state.

```jsx
import { useSearchParams } from "react-router";

function ProductSearch() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get("q") ?? "";

  function updateQuery(nextQuery) {
    const next = new URLSearchParams(searchParams);

    if (nextQuery.trim()) {
      next.set("q", nextQuery.trim());
    } else {
      next.delete("q");
    }

    setSearchParams(next);
  }

  return (
    <button onClick={() => updateQuery("blue mug")}>
      Search for blue mugs (current query: {query || "none"})
    </button>
  );
}
```

The returned parameters represent the current location. Treat them as read-only: create a new `URLSearchParams` before changing values, then pass the new object to the setter.

When updating one key, clone the current parameters if other keys should remain. Building a URL from scratch can accidentally remove another active filter or a parameter managed by a different part of the page.

## 6. Prefer links for navigation

For navigation that a user can follow, use a router link. Links support opening in a new tab, copying the destination, keyboard use, and browser conventions.

```jsx
import { Link } from "react-router";

function ProductLink({ product }) {
  return <Link to={`/products/${product.id}`}>{product.name}</Link>;
}
```

Use an imperative navigation API when navigation is a consequence of an action or result, such as redirecting after a successful submission. Do not use imperative navigation merely to render something that should be an ordinary link.

A link can include a query string. Construct parameter values with `URLSearchParams` when building the URL in code:

```js
const params = new URLSearchParams();
params.set("q", "blue mug");
params.set("sort", "price");

const destination = `/products?${params.toString()}`;
```

## 7. Browser history: push or replace

A navigation can either add a new history entry or replace the current entry:

- **Push** adds an entry. Back returns to the previous view.
- **Replace** changes the current entry. Back skips the replaced intermediate value.

A committed search, filter, or page selection often deserves a new history entry. A rapidly changing value-such as a query synchronized on every keystroke-may use replacement or debouncing so Back does not need to step through dozens of keystrokes.

Choose intentionally. If each meaningful selection is pushed, Back can undo those selections. If each transient update replaces the current entry, Back leaves the page instead of revisiting every intermediate value.

Router APIs provide options for replacement. The exact API and rendering behavior depend on the router and its version. Let the router manage its history rather than calling `history.pushState` behind the router's back.

## 8. Forms and progressive enhancement

A search can be represented by a normal GET form. Submitting it places the form values in the URL, which provides useful browser behavior even before adding client-side enhancements.

```html
<form action="/products" method="get">
  <label for="product-query">Search products</label>
  <input id="product-query" name="q" type="search">
  <button type="submit">Search</button>
</form>
```

In a client-rendered application, a router-aware form or submit handler can update the route without a full document reload. Preserve the benefits of a real form: a label, a submit button, keyboard submission, and a useful destination.

## 9. URL state and server rendering

The URL can be read on the server to render or fetch the selected view. The shape of the API is framework-specific:

- A client-side router may expose a hook that reads the current query string.
- A server-rendered route may receive search parameters as a route prop.
- Some frameworks make server search parameters asynchronous or treat reading them as opting into request-time rendering.
- A client hook may require a client component boundary or a suspense boundary under static rendering.

Check the conventions of the framework and version used by the application. Do not assume a server route prop has the same type or lifecycle as the browser's `URLSearchParams`.

If the server uses a query parameter to load data, validate it on the server as well as in the UI. Client-side validation improves the interface; it does not replace server-side validation.

## 10. Privacy and security

URLs are visible in the address bar and may be stored in browser history, copied into messages, recorded in logs, or sent as part of navigation metadata. Do not put passwords, access tokens, private personal data, or other secrets in a query string.

Treat query values as untrusted input. Validate them before using them for data access or business rules. Render text through normal React text interpolation so React escapes it; do not turn user-controlled query text into HTML.

A search term can itself be sensitive. Consider whether it belongs in a shareable URL at all, and choose caching and logging behavior with that exposure in mind.

## 11. Common mistakes

- Keeping filters only in component state when users expect links and Back to preserve them.
- Putting temporary details such as an open dropdown in the URL without a reason.
- Treating query values as numbers or trusted enum values without parsing them.
- Updating one parameter by rebuilding the whole query string and dropping unrelated parameters.
- Mutating the router's current search-parameter object directly.
- Calling `window.location` for normal client-side route changes.
- Creating a history entry on every keystroke without considering Back behavior.
- Putting sensitive information in a URL.
- Depending only on client-side checks for values used by a server.
- Assuming URL APIs work identically across routers and rendering frameworks.

## Key takeaways

- The path identifies a route or resource; search parameters commonly describe a view; the fragment identifies a document location.
- Put shareable, reproducible view state in the URL and short-lived interaction state in React.
- Search parameter values are strings and must be parsed, validated, and given sensible defaults.
- Use router links and navigation APIs so the URL, history, and rendered route stay coordinated.
- Push meaningful navigation steps; replace transient updates when that matches the desired Back behavior.
- URLs are public-facing input. Keep secrets out and validate values on the server when needed.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="086-nested-routes-layouts-and-errors.md">086 - Nested routes, layouts, and route errors</a></td><td><a href="../qnas/087-url-state-and-navigation.md">Q&amp;A</a></td><td><a href="../examples/087-url-state-and-navigation.md">Example</a></td><td><a href="088-organizing-react-applications.md">088 - Organizing Components, Features, and Application Code</a></td></tr>
  </tbody>
</table>
