# Routing concepts and choosing a router

## Learning goals

After this lesson, you should be able to:

- Explain how a router maps a URL to the UI.
- Identify the path, dynamic segments, query string, and fragment in a URL.
- Explain client-side navigation and browser history at a high level.
- Distinguish a router library from a full application framework.
- Compare common React routing approaches by their capabilities.
- Choose a routing approach based on the application's needs and constraints.
- Recognize that route matching is not an authorization boundary.

## What routing does

Routing connects a location, usually a browser URL, to the interface and data for that location. A route can define which page renders, what parameters it receives, and how it relates to shared layouts or data-loading behavior.

A router typically helps with:

- Matching the current URL to a route.
- Rendering the route's component or route module.
- Creating links and navigating between routes.
- Reading dynamic path parameters.
- Representing not-found and error states.
- Coordinating route loading, pending UI, and code splitting when supported.

Routing is part of the application architecture. React itself does not prescribe one router API for every web app.

## Anatomy of a URL

Consider this example:

```text
/courses/react-basics?tab=overview#requirements
```

| Part | Example | Typical purpose |
| --- | --- | --- |
| Path | `/courses/react-basics` | Identifies a route or resource |
| Path segment | `courses`, `react-basics` | Organizes the route hierarchy |
| Dynamic segment | `react-basics` in `/courses/:courseId` | Supplies a value such as an ID or slug |
| Query string | `?tab=overview` | Optional, shareable view state or filters |
| Fragment | `#requirements` | Points to a location within the current document |

The URL is user-controlled input. A path parameter or query value may be absent, malformed, or deliberately changed. Validate it before using it to load data. A route match does not prove that the visitor is allowed to see or change a resource.

## Route definitions

A route definition connects a pattern to UI. In a declarative router, routes may be written as components or configuration:

```jsx
<Routes>
  <Route path="/" element={<HomePage />} />
  <Route path="/courses" element={<CourseList />} />
  <Route path="/courses/:courseId" element={<CourseDetails />} />
</Routes>
```

The `:courseId` segment is dynamic. A router can match a URL such as `/courses/react-basics` and provide `react-basics` to the route component.

A route may also define nested UI, loading, data, error handling, and metadata depending on the router and mode. The next lessons cover nested layouts and URL state in more detail.

## Client-side navigation and browser history

In a single-page application, a router can respond to a link click, update the browser's history, and render the new route without requesting a completely new document for every navigation. The browser's Back and Forward buttons still work because the router coordinates with the History API.

Use the router's link component for in-app navigation:

```jsx
<Link to="/courses">Browse courses</Link>
```

The link renders a real anchor with a destination. This preserves useful browser behavior such as opening a link in a new tab and copying its address.

Use a regular HTML anchor when navigating to another website, downloading a file, or intentionally requesting a full document navigation:

```jsx
<a href="/reports/export.csv">Download the report</a>
```

For in-app navigation, avoid replacing links with clickable `div` elements or calling a navigation hook for every ordinary link. Native link semantics improve keyboard support, accessibility, and browser behavior.

A client-side router does not automatically configure the web server. When a visitor opens or refreshes a nested URL directly, the hosting server must route that request to the application's document or server handler. The exact fallback depends on whether the app is a client-only SPA, a server-rendered framework, or another deployment model.

## Common route patterns

### Static route

A fixed path such as `/about` maps to the same route definition.

### Dynamic route

A pattern such as `/courses/:courseId` matches multiple course pages. The route receives a value from the URL and should validate it before using it.

### Index route

An index route represents the default page at a parent route's path, such as the course list at `/courses`.

### Nested route

A route may render inside a shared parent layout. For example, course pages can share the same navigation and sidebar while the selected child page changes.

### Not-found route

A catch-all route can render a useful not-found page when no configured route matches. In a client-only SPA this may be a rendered screen while the hosting server still returns the application shell; HTTP status behavior is framework- and hosting-specific.

## Router library or framework?

A **router library** focuses on mapping URLs to UI and handling navigation. The application usually chooses and assembles other pieces such as its build tool, data fetching, server rendering, deployment, and code-splitting strategy.

A **framework** provides routing as part of a larger application model. It may add conventions for page files, data loading, layouts, server rendering, static generation, streaming, server functions, and deployment.

The boundary is not always strict. Some routing tools have several modes, ranging from basic navigation to a full application framework.

## Common React routing approaches

| Approach | Useful when | Trade-offs to consider |
| --- | --- | --- |
| React Router Declarative mode | You need route matching, links, active states, and navigation in an existing client app | The app remains responsible for broader data, rendering, and deployment choices |
| React Router Data mode | You want route loaders, actions, pending states, and error handling while keeping more control over the app setup | You assemble more of the build and application architecture yourself |
| React Router Framework mode | You want route modules, code splitting, and a framework-managed setup with SPA, server-rendered, or static rendering options | You adopt its conventions and framework configuration |
| TanStack Router | You value a route tree with strong TypeScript support, especially for navigation and validated URL search state | Its APIs and route model are a deliberate choice to learn and adopt |
| Next.js App Router | You want file-system routes, nested layouts, and a framework integrated with React Server Components and server features | You adopt Next.js conventions and its application/deployment model |

This is a guide to trade-offs, not a ranking. Features change over time, and each project's requirements matter more than popularity.

## How to choose

Before choosing, answer these questions:

1. **What are you building?** A small client-only tool, a content site, a large dashboard, a mobile app, or a full-stack product may need different architecture.
2. **Does the app need server rendering or static generation?** If yes, prefer a framework/router whose supported rendering and deployment model fits the app.
3. **Where should route data load?** Decide whether route loaders, server components, a client data library, an existing API, or a combination fits the team's model.
4. **How complex is URL state?** Search filters, tabs, and pagination may benefit from validation and typed route APIs.
5. **How much convention do you want?** File-system routes can make conventions visible; code-defined routes can make the route tree explicit in one place.
6. **What does the project already use?** Migration cost, deployment support, team experience, accessibility, and maintenance are practical constraints.
7. **What can the team operate?** A framework can provide more integrated features, but the team still needs to understand its routing, rendering, caching, and hosting model.

For a new production React app, consider a framework first because routing is connected to rendering, data loading, code splitting, and deployment. Build a smaller app from React and a router when that better fits the constraints, when you already have those surrounding systems, or when the goal is to learn the pieces directly.

## URL state or local state?

Use the URL when a view state should survive refreshes or be bookmarkable and shareable. Examples include a selected course, a search query, a page number, or a meaningful filter.

Use local React state when the state is temporary and does not need its own address, such as whether a tooltip is open or text currently being typed into an unsent form.

A URL is public and can be edited. Do not put secrets, access tokens, or authorization decisions in a path or query parameter. The next lesson on URL state explores this distinction further.

## Routing is not security

A route guard can control which interface the router displays, but hiding a route or link does not secure its data. A visitor can enter a URL directly or call a backend endpoint independently.

For protected routes:

- Verify identity and permissions in the trusted server or API layer.
- Validate path and query values before using them.
- Check ownership or tenant access for the specific resource.
- Do not rely on the client router to enforce access control.

## Common mistakes

- **Treating React as if it includes one required router:** React lets applications choose an appropriate routing model.
- **Choosing by a feature checklist alone:** Consider deployment, team familiarity, migration cost, and the whole data/rendering model.
- **Using an anchor for every internal route in a client SPA:** Use the router's link component for normal in-app navigation.
- **Using a clickable element instead of a link:** A link communicates navigation and retains browser features.
- **Assuming client navigation handles direct URL requests:** The server or static host must also be configured for the app's routes.
- **Putting temporary UI state in the URL:** Only make state addressable when refresh, sharing, or browser history should preserve it.
- **Treating route matching as authorization:** Enforce permissions where protected data is read or changed.
- **Confusing a router with a full-stack framework:** A router may be only one part of the build, data, server, and deployment setup.

## Summary

A router maps URLs to UI and coordinates navigation. Route paths may include static segments, dynamic values, query state, and fragments. Router libraries and frameworks offer different levels of help with data loading, rendering, code splitting, and deployment. Choose based on the application's needs and the team's ability to operate the whole stack. Use semantic links, configure the host for direct route requests, validate URL input, and enforce security outside the client router.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="084-client-server-boundaries-and-serialization.md">084 - Client and server boundaries, serialization, and security</a></td><td><a href="../qnas/085-routing-concepts-and-router-choice.md">Q&amp;A</a></td><td><a href="../examples/085-routing-concepts-and-router-choice.md">Example</a></td><td><a href="086-nested-routes-layouts-and-errors.md">086 - Nested routes, layouts, and route errors</a></td></tr>
  </tbody>
</table>
