# Nested routes, layouts, and route errors - Q&A

## Nested routes and layouts

### 1. What is a nested route?

A nested route is a child in a route hierarchy. It can add a URL segment, render inside a parent layout, or do both, depending on the router configuration.

### 2. Why use nested routes?

They let related pages share a layout and keep the route tree organized. A course list and course detail page can share a course navigation shell while displaying different child content.

### 3. What is an outlet?

An outlet is the location inside a parent route component where the router renders the currently matched child route. In React Router, this is the `<Outlet />` component.

### 4. What happens if the parent layout does not render an outlet?

The parent can match and render, but its child route has no outlet location in which to appear. Add an outlet where the child UI belongs.

### 5. What is an index route?

An index route is the default child route at the parent's URL. For example, a courses index route renders at `/courses`, while a sibling child with a course ID can render at `/courses/:courseId`.

### 6. Does an index route add a path segment?

No. It renders at the parent route's path and represents the parent's default child content.

### 7. Can an index route have child routes?

Usually not. An index route is already the terminal default child. If that page needs its own route hierarchy, define a regular parent route with child routes.

### 8. What is a pathless layout route?

It is a route that adds shared UI and nesting without adding a URL segment. Its children can use the layout while keeping their existing paths.

### 9. Are nested routes always nested URL paths?

No. Pathless layouts and route groups can affect UI composition without adding URL segments. Router conventions differ.

### 10. What is the difference between a parent route and a layout route?

A parent route is any route that contains children. A layout route is a parent whose main role is to wrap shared UI around those children. Some routers distinguish layout routes explicitly; others use ordinary nested routes.

### 11. Do all routers use `<Outlet />`?

No. React Router uses `<Outlet />` for nested route content. File-system frameworks may pass nested page UI through a `children` prop or use other conventions.

### 12. Do layouts always stay mounted during navigation?

That depends on the router and rendering mode. Some frameworks preserve shared layouts across sibling route navigations, but do not assume state lifetime without checking the chosen tool's behavior.

### 13. Should a layout store all application state?

No. Keep state close to the UI that owns it or use an explicit shared state strategy. A layout is a composition boundary, not automatically a global state manager.

### 14. How are nested routes represented in file-system routing?

Folders usually represent route segments and special files define pages and layouts. For example, a folder for courses may contain a shared layout and a dynamic folder for each course ID. Exact filenames and conventions depend on the framework.

## Route errors

### 15. What is a route error boundary?

A route error boundary is a router- or framework-provided fallback for supported errors associated with a route and its descendants. It replaces the failed route content with an error UI.

### 16. What does “nearest error boundary” mean?

When a route fails, the closest ancestor route with an error boundary handles the failure. If the failing route has no boundary, the error can bubble to a parent boundary.

### 17. Why add a boundary near a route instead of only at the root?

A local boundary can replace only the failed section while preserving surrounding layouts and navigation. A root boundary remains useful as a final fallback.

### 18. Should an application have a root error boundary?

Yes, when the router supports route boundaries. It provides a fallback for errors that are not handled by a more specific child boundary.

### 19. Do all React routers have route-level error boundaries?

No. Error APIs differ between declarative routers, data routers, and full frameworks. Use the error handling feature supported by the selected mode.

### 20. What is the difference between a not-found route and a missing resource?

A not-found route handles a URL that matches no configured route. A missing resource happens when a valid route pattern matches but its requested record does not exist.

### 21. Should a missing record throw an error?

Use the router's supported not-found behavior. In a data router, a loader may be able to signal a 404 to the nearest route boundary. A simple client component may instead render an explicit not-found state. Follow the tool's convention.

### 22. Should expected validation failures go to an error boundary?

Usually no. Validation errors, empty states, and expected save failures are normal application states. Render them near the relevant form or control instead of replacing the whole route.

### 23. Are error boundaries a good way to implement normal branching?

No. They are intended for failures, not routine control flow. Use explicit conditionals for expected states; use the router's documented not-found mechanism for missing resources.

### 24. Do route boundaries catch event-handler errors?

Not generally. Errors thrown later by ordinary event handlers need local handling. Catch them in the event or action flow and render a useful state.

### 25. Do route boundaries catch every asynchronous failure?

No. Only failures integrated with the router's route error mechanism are caught there. Handle unrelated promises, network work, and event-triggered operations according to their owning flow.

### 26. Can a boundary catch errors thrown inside itself?

No. An error boundary does not catch errors in its own fallback rendering. A parent boundary may catch them.

### 27. Is a loading boundary the same as an error boundary?

No. A loading boundary handles pending work or fallback content. An error boundary handles a failure. Frameworks may provide separate mechanisms for each.

### 28. Should the visitor see the raw exception message?

Usually not. Show a short, safe message and a recovery option. Keep stack traces, SQL errors, file paths, and secrets in protected diagnostics.

### 29. How should route errors be logged?

Log diagnostic details through the trusted error-reporting path, removing credentials and unnecessary personal data. Do not expose internal logs or stack traces in the page.

### 30. Does a route error boundary enforce access control?

No. It controls fallback UI, not permissions. Authenticate and authorize in the server or API layer that reads or mutates protected data.

### 31. Can I hide a protected child route by omitting its link?

No. A visitor can enter the route URL directly. The protected data operation must verify permissions regardless of what navigation the page displays.

### 32. How should I choose the error boundary hierarchy?

Add a root fallback first. Add local boundaries where a section can recover independently and where the fallback can offer a useful action. Avoid so many boundaries that errors become difficult to trace.

### 33. Should every child route define its own boundary?

No. Use local boundaries where they preserve useful surrounding UI or provide better recovery. Otherwise, allow the error to bubble to an appropriate parent.

### 34. What happens to shared UI when a child route fails?

If the nearest boundary is inside the shared layout, that layout can remain visible while the child outlet is replaced. If the boundary is above the layout, more of the page may be replaced.

### 35. What is the key mental model for nested route errors?

Layouts compose shared UI, outlets place child routes, and error boundaries define where supported failures are recovered. Expected states should be rendered directly, while permissions and sensitive diagnostics belong to trusted layers.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="085-routing-concepts-and-router-choice.md">085 - Routing concepts and choosing a router - Q&amp;A</a></td><td><a href="../lessons/086-nested-routes-layouts-and-errors.md">Lesson</a></td><td><a href="../examples/086-nested-routes-layouts-and-errors.md">Example</a></td><td><a href="087-url-state-and-navigation.md">087 - URL State, Search Parameters, and Navigation - Q&amp;A</a></td></tr>
  </tbody>
</table>
