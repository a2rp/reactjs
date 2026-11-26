# URL State, Search Parameters, and Navigation - Q&A

## Core ideas

### 1. What does it mean to store state in the URL?

It means encoding a page or view choice in the address, usually in the path or query string. The app reads that address and renders the corresponding view. For example, `/products?sort=price` can represent the products page sorted by price.

### 2. Why put a filter or search term in the URL?

A URL-backed filter can survive reloads and can be bookmarked, shared, and revisited with browser Back and Forward. Use this for state that defines a useful view. A temporary open menu usually does not need to be shareable.

### 3. What is the difference between path parameters, search parameters, and a fragment?

The path identifies a route or resource, such as `/products/42`. Search parameters follow `?` and describe options, such as `?sort=price`. A fragment follows `#` and commonly identifies a section in the document, such as `#reviews`.

### 4. Should all React state be put in the URL?

No. Put shareable, reproducible navigation state in the URL. Keep temporary UI state-such as focus, a closed/open menu, or an unsubmitted draft-in component state. Remote data and its loading/cache lifecycle belong to a data layer, even if the URL identifies what data to request.

### 5. Does a query string automatically make server data?

No. It is only part of the address. The application must parse it and use it to choose, fetch, filter, or sort data. The server must independently validate any values it uses.

## Reading and changing search parameters

### 6. Are search parameter values numbers or booleans?

No. `URLSearchParams` returns strings. Convert values explicitly, then validate them. For example, turn `"3"` into a number only after checking that it is an integer in the allowed range.

### 7. What does `URLSearchParams.get()` return when a key is absent?

It returns `null`. Use a default deliberately, for example:

```js
const sort = params.get("sort") ?? "relevance";
```

A present but empty value (`?sort=`) is an empty string, not `null`; decide whether to treat that as invalid or as the default.

### 8. What happens if a query key appears more than once?

`get(name)` returns the first value. `getAll(name)` returns an array of every value. For example, `?tag=css&tag=react` can be read as two selected tags.

### 9. What is the difference between `set()` and `append()`?

`set(name, value)` replaces the key's existing values with one value. `append(name, value)` adds another occurrence of that key. Use the same convention when reading and writing repeated values.

### 10. Should I build a query string by concatenating text?

Usually no. Create `URLSearchParams`, set or append values, and serialize it. The API encodes special characters so spaces, ampersands, and other characters do not accidentally change the structure of the URL.

### 11. Can I mutate the search-parameter object returned by a router hook?

Treat the current object as read-only. Clone it first:

```js
const next = new URLSearchParams(searchParams);
next.set("sort", "price");
setSearchParams(next);
```

This makes the change explicit and avoids relying on mutation that the router may not observe as a navigation.

### 12. How can I update one key without losing other filters?

Clone the current parameters, change only the intended key, then pass the new parameters to the router setter. Starting from a blank object removes all other query keys, which may be wrong if they represent active filters or state owned by another part of the page.

### 13. How should I handle invalid page numbers?

Parse the value and require a positive integer. Choose a fallback such as page 1, or show an invalid-page state. If the page is greater than the number of available pages, clamp it or navigate to a valid URL. Do not trust the value just because the UI normally creates it.

### 14. How should I handle an unknown sort option?

Use an allowlist of supported values. If the URL contains an unknown value, fall back to a documented default, show an error, or normalize the URL. Never pass arbitrary URL input directly into a database field or other privileged operation.

### 15. Should default values appear in the URL?

It depends on the application. Omitting a default keeps URLs shorter, while including it makes the selected option explicit. Whichever approach you choose, parsing must produce the same view when the parameter is omitted. Normalize URLs only when canonical URLs matter and do so without creating a history entry for every correction.

## Navigation and browser history

### 16. What is the difference between push and replace navigation?

Push adds a new history entry, so Back returns to the previous view. Replace modifies the current entry, so Back skips that intermediate URL. Use push for meaningful user choices and consider replace for transient updates.

### 17. Should every keystroke in a search field create a history entry?

Usually not. It can make Back move through every typed character. Common alternatives are submitting the search, debouncing updates, or replacing the current history entry while typing. The right choice depends on whether each intermediate search is useful to revisit.

### 18. How does browser Back work with URL-backed state?

Back moves to the previous history entry. A router observes the location change and renders the route for that URL. The controls should derive their selected values from the current URL so that Back and Forward restore the visible state.

### 19. When should I use a router link?

Use a router link for navigation a user can follow. It preserves normal link behavior such as keyboard activation, copying the destination, and opening a new tab. Use imperative navigation when an action's result should navigate, such as after completing a form.

### 20. Why avoid assigning `window.location` for ordinary in-app navigation?

Assigning it performs a document navigation and usually reloads the app. A router link or navigation API can change the route within the running application while keeping the URL, history, and rendered route coordinated.

### 21. Can I use a normal GET form for search?

Yes. A GET form places named form values in the query string and supports keyboard submission and browser behavior. In a client-rendered app, a router-aware form can enhance the experience without losing the basic form semantics.

### 22. Does setting search parameters behave exactly like React's state setter?

No. Router navigation APIs have their own semantics. In React Router, the setter accepts a value or a callback, but multiple calls in the same tick do not necessarily queue like multiple React state updates. Prefer building one complete next parameter object and navigating once.

## Framework and rendering details

### 23. Is the same URL API available in every React router?

No. APIs and import paths vary across routers and versions. For example, React Router provides search-parameter and location APIs, but another router may use different hooks or route props. Follow the installed router's API rather than copying code from a different version.

### 24. Is a browser search-parameter hook the same as server route parameters?

No. A browser hook reads the current client-side location. A server-rendered route may receive a plain object, an asynchronous value, or another framework-specific representation. Convert only when needed and follow the framework's current conventions.

### 25. Can the server use query parameters to fetch data?

Yes. A server route can parse the requested view and use validated parameters to select or fetch data before rendering. Validate on the server, even if the client already constrains values. Client validation is for the interface and is not a security boundary.

### 26. Will a URL-backed view work after a direct reload?

It should, if the app can resolve the route and reconstruct the view from the URL. The web server must also be configured to serve the client application for its routes, or the route must be handled by server rendering. A client-only implementation cannot help if the server returns a not-found response before the app loads.

## Privacy, security, and practical concerns

### 27. Is it safe to put a password, token, or private value in a query string?

No. URLs may appear in browser history, logs, copied messages, screenshots, analytics, or navigation metadata. Use an appropriate secure transport and storage mechanism for secrets; do not put them in a shareable URL.

### 28. Is a search term always safe to put in the URL?

No. A query can reveal sensitive interests or personal information. Decide whether that search should be shareable and consider the privacy implications of history, logs, analytics, caching, and copied links.

### 29. Can URL values cause cross-site scripting in React?

A URL value is untrusted input, but rendering it as ordinary React text is escaped by React. Do not pass user-controlled query values to raw HTML rendering. Validate values before using them in queries, file paths, or business rules as well.

### 30. Does changing the order of query parameters change their meaning?

Usually, application logic should not depend on key order. `?sort=name&page=2` and `?page=2&sort=name` generally describe the same options. Repeated values may have an order, so preserve or deliberately normalize that order if the application depends on it.

### 31. What should happen to unrelated query parameters when a filter changes?

Usually preserve parameters that are still meaningful. Clone the current parameter set and update the relevant key. If a filter makes another value invalid-for example, changing the result set resets pagination-remove or update that dependent value intentionally.

### 32. Should a filter change reset pagination?

Often yes. If a user is on page 8 and applies a filter with only two pages of results, page 8 is no longer valid. Update the filter and reset the page in the same navigation so the URL represents a valid view.

### 33. How can I tell whether a value belongs in the path or the query string?

Use the path for the identity of the resource or route, such as `/users/42`. Use search parameters for optional view choices on that resource, such as `?tab=activity` or `?sort=recent`. Treat this as a design convention and keep it consistent across the app.

### 34. What should I test for URL state?

Check direct loading and reload, missing and malformed values, repeated keys if supported, preservation of unrelated filters, browser Back and Forward, link copying, page reset when filters change, and server-side validation where relevant.

## Quick review

- URL values are strings and must be validated.
- `get()` returns the first value or `null`; `getAll()` reads repeated values.
- Clone search parameters before updating them.
- Use router-aware links and navigation so history and rendered state stay aligned.
- Choose push or replace based on how Back should behave.
- Keep secrets out of URLs and validate server inputs independently.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="086-nested-routes-layouts-and-errors.md">086 - Nested routes, layouts, and route errors - Q&amp;A</a></td><td><a href="../lessons/087-url-state-and-navigation.md">Lesson</a></td><td><a href="../examples/087-url-state-and-navigation.md">Example</a></td><td><a href="088-organizing-react-applications.md">088 - Organizing Components, Features, and Application Code - Q&amp;A</a></td></tr>
  </tbody>
</table>
