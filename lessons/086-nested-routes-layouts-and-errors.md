# Nested routes, layouts, and route errors

## Learning goals

After this lesson, you should be able to:

- Model a route tree with parent and child routes.
- Render child route UI inside a parent layout.
- Explain index routes and pathless layout routes.
- Choose a sensible boundary for shared UI and route-specific errors.
- Distinguish an unmatched URL, a missing resource, and an unexpected failure.
- Use route error boundaries without exposing internal error details.
- Recognize which errors route boundaries do and do not catch.

## Why nest routes?

Many application pages share parts of their interface. A course section may have the same heading, navigation, and sidebar on its list page and detail pages. A dashboard may keep its shell while the selected page changes.

Nested routes let the route tree describe that UI hierarchy. The parent route renders shared UI; the matching child route renders inside a designated outlet or slot.

```text
Application
├── Home
└── Courses layout
    ├── Course index
    └── Course details
```

The URL path and UI hierarchy often correspond, but they are not required to be identical. A pathless layout can add shared UI without adding a URL segment.

## Parent routes and outlets

In React Router, a parent route with children can render an `<Outlet />`. The outlet is where the router displays the currently matched child route.

```jsx
import { Link, Outlet } from "react-router";

function CoursesLayout() {
  return (
    <section>
      <h1>Courses</h1>
      <nav aria-label="Course navigation">
        <Link to="/courses">All courses</Link>
      </nav>
      <Outlet />
    </section>
  );
}
```

If the child route matches `/courses`, the index page appears in the outlet. If `/courses/:courseId` matches, the detail page appears in the same outlet. The shared heading and navigation stay owned by the parent.

A parent route does not automatically display its children just because they are nested in the route configuration. The layout needs an outlet at the place where child content should appear.

## Index routes

An index route is the default child at its parent's URL. It has no additional path segment.

```jsx
<Route path="courses" element={<CoursesLayout />}>
  <Route index element={<CourseIndexPage />} />
  <Route path=":courseId" element={<CourseDetailsPage />} />
</Route>
```

The index route matches `/courses`; the dynamic route matches paths such as `/courses/react-basics`. An index route is useful when a parent layout has a sensible default page.

Index routes are not the same as a route with an empty string path. Use the router's index-route convention.

## Pathless layout routes

Some routers allow a layout route without a path. It adds a UI boundary but does not add a URL segment.

```jsx
<Route element={<AccountLayout />}>
  <Route path="account" element={<AccountOverview />} />
  <Route path="account/security" element={<AccountSecurity />} />
</Route>
```

This is helpful when routes should share a shell without making that shell's name part of the URL. The exact syntax depends on the router.

## Route configuration versus file-system layouts

Routers describe nested UI in different ways:

- **Route configuration:** a parent route declares child routes, and its component renders an outlet.
- **File-system routing:** folders and special page/layout files define URL segments and shared layouts.
- **Framework route modules:** files or configuration may define a route's component, loader, action, metadata, and error boundary together.

For example, the same conceptual course structure could be represented with nested route objects in React Router or folders such as `app/courses/layout.jsx` and `app/courses/[courseId]/page.jsx` in a file-system router. These conventions are not interchangeable; follow the selected framework's documentation.

## Layout lifetime and state

A shared layout can keep navigation, page framing, or local state in one place. Whether it remains mounted during a navigation, which parts re-render, and whether route state is preserved depend on the router and rendering mode.

Do not rely on a layout to store global data merely because it wraps several routes. Choose an explicit state or data-sharing strategy, and confirm the router's lifetime behavior. Keep state close to the UI that owns it when possible.

## Route errors

A route can fail for different reasons:

1. **No route matches the URL:** show a not-found route or framework not-found UI.
2. **The route matches, but the requested record does not exist:** show a resource-specific not-found state, often with a 404.
3. **The route or its data operation fails unexpectedly:** show an error fallback and report diagnostic details through a protected logging system.

These cases should not all look like the same blank screen. A useful fallback tells the visitor what happened and what they can do next, while keeping internal details private.

## Nearest error boundary

In a data router or a framework with route-level error boundaries, an error can be handled by the closest ancestor route that defines a boundary.

```text
Root layout + root error boundary
└── Courses layout
    ├── Course list
    └── Course details + course error boundary
        └── Course section
```

If the course detail route fails, its boundary can replace that part of the UI while the root and courses layouts remain. If the course list fails and the courses layout has no boundary, the error can bubble to the root boundary.

A root boundary is a useful safety net. More local boundaries can preserve unrelated parts of the interface and provide recovery actions that make sense for that section.

Not every routing API has route-level error boundaries. Basic declarative route components and data-router/framework routes may have different error handling APIs. Use the mechanism supported by the chosen router mode.

## Error boundaries and expected errors

Error boundaries are primarily for unexpected errors during rendering or route work supported by the router. Expected states should usually be represented directly:

- A missing optional description can render a normal empty state.
- An invalid filter can show validation feedback.
- A missing resource can use the router's not-found mechanism.
- A failed save initiated by a button can be caught in the event/action flow and shown near the form.

Do not intentionally throw arbitrary exceptions just to implement routine branching. A router may support throwing a not-found response from a loader as an explicit route-control mechanism; follow that router's conventions.

## What error boundaries do not catch

A React error boundary generally handles descendant rendering failures and, in a router, failures specifically integrated with that route boundary. It does not automatically handle every error in the application.

- Errors thrown later from ordinary event handlers need local handling.
- Rejected promises in unrelated asynchronous work need explicit handling.
- Errors in a boundary itself are not caught by that same boundary.
- A route boundary does not validate user input or authorize access.
- A browser-network failure may need a loading/data error state, depending on how the router loads data.

Handle each failure at the layer that owns it. For example, a form can preserve its input and show a retry message for an expected save failure instead of replacing the entire route.

## Safe fallback UI

A production fallback should be useful without exposing private diagnostics:

- Provide a short, plain message.
- Offer a reasonable action such as retry, return to the parent route, or go home.
- Keep stack traces, file paths, SQL messages, and secrets out of the visitor-facing UI.
- Log useful diagnostic context on the trusted server, with sensitive values removed.
- Avoid logging credentials or complete request objects that may contain secrets.

A framework may sanitize server errors before they reach production UI, but application code should still avoid intentionally rendering raw error details.

## Route errors are not authorization

A route boundary controls what UI is displayed after an error. A layout guard may improve navigation flow, but neither replaces permission checks in the server or API layer.

A visitor can request a URL directly. Check identity and authorization whenever protected data is loaded or changed. Return a not-found response when revealing whether a protected resource exists would disclose sensitive information.

## Common mistakes

- **Forgetting `<Outlet />`:** matched child routes have nowhere to render inside the parent layout.
- **Treating an index route as another path segment:** it renders at the parent's URL.
- **Adding a path to a layout that should not affect the URL:** use the router's pathless-layout convention when appropriate.
- **Putting all route errors in one global fallback:** local boundaries can keep shared UI and unrelated sections available.
- **Using exceptions for ordinary validation:** represent expected states explicitly.
- **Showing raw errors to visitors:** expose a safe message and keep diagnostics in protected logs.
- **Assuming error boundaries catch event-handler errors:** handle those failures in the event or action flow.
- **Treating a layout as an access-control layer:** enforce authorization where protected data is accessed.
- **Assuming every router uses the same error API:** route error handling differs by mode and framework.

## Summary

Nested routes let a parent layout provide shared UI and render the matched child through an outlet. Index routes represent the parent's default child, while pathless layout routes add structure without changing the URL. Route error boundaries can catch supported failures and let the nearest route boundary replace only the affected section. Use explicit UI for expected states, keep diagnostics private, and enforce authorization in the trusted data layer.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="085-routing-concepts-and-router-choice.md">085 - Routing concepts and choosing a router</a></td><td><a href="../qnas/086-nested-routes-layouts-and-errors.md">Q&amp;A</a></td><td><a href="../examples/086-nested-routes-layouts-and-errors.md">Example</a></td><td><a href="087-url-state-and-navigation.md">087 - URL State, Search Parameters, and Navigation</a></td></tr>
  </tbody>
</table>
