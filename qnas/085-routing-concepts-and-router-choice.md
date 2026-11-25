# Routing concepts and choosing a router — Q&A

## Routing basics

### 1. What is routing in a React application?

Routing maps a browser location to the UI and behavior for that location. A router matches the URL, renders the route, and can coordinate navigation, data loading, and route-specific states.

### 2. Does React include one required router?

No. React provides the component model, while an application chooses a router or a framework with routing built in.

### 3. What is a route?

A route is a rule that associates a URL pattern with UI and, depending on the tool, data loading, actions, layouts, and error handling.

### 4. What are the parts of a URL?

A URL can contain a path, path segments, query parameters, and a fragment. For example, in `/courses/react?tab=overview#details`, the path is `/courses/react`, the query string is `tab=overview`, and the fragment is `details`.

### 5. What is a dynamic route segment?

A segment that stands for a variable value, such as `:courseId` in `/courses/:courseId`. The router supplies the matched value to the route. Validate it before using it to load data.

### 6. What is the difference between a path parameter and a query parameter?

A path parameter identifies a resource or route segment, such as a course ID. A query parameter usually represents optional view state, filters, sorting, or pagination. Both are caller-controlled input.

### 7. What does a fragment do?

A fragment identifies a location within the current document, often an element with a matching ID. It is handled by browser navigation and does not normally identify a different application route.

## Navigation behavior

### 8. What is client-side navigation?

A client router can handle an in-app link, update the browser's address and history, and render the next route without loading a full new document for each navigation.

### 9. Do Back and Forward still work after client-side navigation?

Yes. A browser router coordinates with the History API so the browser's Back and Forward buttons can restore prior locations.

### 10. Should I use a router link or an HTML anchor for an internal route?

In a client-routed app, use the router's link component for ordinary in-app navigation. It renders link semantics and lets the router handle navigation.

### 11. When should I use a plain HTML anchor?

Use an anchor for another site, a file download, or an intentional full-document navigation. Some routes may also deliberately use document navigation based on the application architecture.

### 12. Why should I avoid making a div clickable for navigation?

A link communicates that its destination is navigable and provides keyboard, context-menu, open-in-new-tab, and copy-link behavior. A clickable div must recreate these behaviors and is usually less accessible.

### 13. Why can a nested URL fail on refresh even if navigation works inside the app?

Client-side navigation may be handled after the app has loaded, but refreshing asks the hosting server for that path directly. The server or static host must be configured for the application's routing model.

### 14. Does every not-found route return an HTTP 404 status?

No. A client-only SPA may render a not-found screen after the application shell loads while the server response itself is still a successful shell response. Status behavior depends on the framework and hosting setup.

## Route models

### 15. What is the difference between a router library and a framework?

A router library focuses on URL matching and navigation. A framework can provide routing as part of a larger system for data loading, layouts, rendering, code splitting, and deployment. Some routers offer several levels of framework support.

### 16. What are React Router's Declarative, Data, and Framework modes?

They are different levels of React Router integration. Declarative mode provides route matching and navigation. Data mode adds route loaders, actions, pending states, and related APIs. Framework mode adds a framework-managed application setup and route modules.

### 17. Should I always use the most feature-rich router mode?

No. More integrated features can mean more conventions and less control over application setup. Choose the amount of framework support that fits the project and team.

### 18. What is file-based routing?

The filesystem layout defines route paths through folders and special files. It can make route organization visible and consistent, but it ties the project to the framework's file conventions.

### 19. What is code-based routing?

Routes are declared in code or configuration. It keeps the route map explicit and may fit an existing app, but the route tree is maintained as code rather than derived from a filesystem convention.

### 20. When might React Router be a good choice?

It can fit an existing React app that needs client-side navigation, a data router, or a framework-oriented setup. Choose the mode based on whether you want routing only, route data features, or a broader application framework.

### 21. When might TanStack Router be a good choice?

It may fit an app that values an explicit route tree and strong TypeScript support for navigation, parameters, and search state. Consider its conventions and learning cost along with those features.

### 22. When might Next.js App Router be a good choice?

It may fit a full-stack React app that wants file-system routes, nested layouts, and integration with React Server Components and server-side features. The team should be comfortable with Next.js conventions and deployment.

### 23. Is there one best router for every React app?

No. The right choice depends on rendering needs, data loading, URL complexity, type safety, deployment, existing code, team knowledge, and how much application structure you want the tool to provide.

### 24. What does React recommend for a new application?

React's current guidance recommends considering a framework first for a new app. A from-scratch setup with a router can still be appropriate when existing constraints, architecture, or the learning goal calls for it.

## URL state and security

### 25. Which state belongs in the URL?

State belongs in the URL when it should survive refreshes or be bookmarkable and shareable, such as a selected resource, search query, filter, sort order, or page number.

### 26. Which state should remain local to a component?

Temporary UI state that does not need a shareable address can stay local, such as whether a tooltip is open or the current text of an unsent input.

### 27. Can the user change a route parameter?

Yes. A user can edit the address bar, construct a request, or follow a malformed link. Validate route parameters and query values before using them.

### 28. Does a route guard secure a protected page?

Not by itself. Hiding a route or redirecting in the client is a UI behavior. Enforce authentication and authorization in the trusted server or API layer that reads or changes the protected data.

### 29. Should I put a secret or access token in a path or query parameter?

No. URLs can be copied, logged, stored in browser history, and shared. Keep secrets out of URLs and rendered client state.

## Architecture and practical choices

### 30. Should route data always be fetched in a component effect?

No. A router loader or framework server mechanism may coordinate route data and avoid request waterfalls. The right approach depends on the chosen router, rendering mode, and data architecture.

### 31. Do nested routes matter when choosing a router?

Yes. Shared layouts, child routes, route-specific errors, and loading UI affect how the application is organized. The following lesson covers nested routes and layouts in detail.

### 32. What should I evaluate before adopting a router?

Check supported rendering and deployment modes, route and data conventions, code splitting, URL/search handling, TypeScript support, accessibility, error states, ecosystem maintenance, migration cost, and what the team can operate.

### 33. How should a small client-only learning project choose?

A simple client router is enough to learn URL matching, links, parameters, and browser history. Avoid adopting a full framework unless the project needs its additional features or you specifically want to learn them.

### 34. What is the safest mental model for routing?

A router decides which UI responds to a URL and how users move between routes. It does not make user-provided URL values trustworthy and does not replace server-side permission checks.
