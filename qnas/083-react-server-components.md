# React Server Components and server rendering - Q&A

## Core concepts

### 1. What is a React Server Component?

A React component rendered in a server environment by a framework or build tool that supports the React Server Components architecture. Its implementation is not included in the browser's client component bundle.

### 2. Is every React component a Server Component?

No. Server Components exist only in an RSC-enabled application. In a supported framework, components in a particular server-rendered tree may be Server Components by default. Other React setups do not gain RSC support automatically.

### 3. Where can a Server Component run?

It can run while a framework builds or pre-renders an application, or while the server handles a request. The framework determines the rendering mode and data-loading lifecycle.

### 4. Can a Server Component be asynchronous?

Yes. In a supported environment, an async Server Component can await data before returning its UI. The framework controls how pending work, errors, and streaming are handled.

### 5. Does a Server Component need an API endpoint to read server data?

Not necessarily. It can access a server-side data layer directly when the framework allows it. The browser should not receive database credentials or private implementation details.

### 6. Does adding a React component make it a Server Component?

No. The application must use an RSC-capable framework or build tool. A component file or an async function alone does not create the server execution environment.

### 7. Do Server Components have to be in a separate server application?

No. A framework can manage server and client component rendering within one application. Its deployment architecture decides which processes or services are involved.

## Server and Client Components

### 8. What is a Client Component?

A component whose module is included in the client-side component graph so that its browser behavior can run. It is the right place for state, effects, event handlers, and browser APIs.

### 9. How is a Server Component different from a Client Component?

A Server Component runs in the server component environment and is useful for server data access and rendering. A Client Component ships the code needed for browser interaction. They can be composed in the same UI tree.

### 10. What does the "use client" directive do?

In frameworks that use this convention, it marks a module as a Client Component entry point. It should appear before imports and other statements. The framework then includes that module and its client-side dependencies in the client module graph.

### 11. Does "use client" mean the component only renders in the browser?

No. A framework can produce initial HTML for a Client Component on the server and then hydrate it in the browser. The component is still a Client Component because its client-side code provides its interactive behavior.

### 12. Can a Server Component use useState or useEffect?

No. State, effects, and event handlers require a Client Component. Keep the interactive behavior in a client boundary and let surrounding Server Components handle data and composition.

### 13. Can a Server Component read window or localStorage?

No. Browser APIs are not available in the server rendering environment. Use a Client Component and access browser APIs at an appropriate time, such as in an effect or event handler.

### 14. Can a Server Component pass props to a Client Component?

Yes. It can pass values supported by the framework's server-to-client transport. Keep the values small and safe, and do not pass server-only objects or secrets as ordinary props.

### 15. Can a Client Component import a Server Component?

A Client Component should not import a Server Component into its client module graph. Instead, let a Server Component import both components and pass the server-rendered UI through a supported prop such as children.

### 16. Can a Client Component render Server Component children?

Yes, when the framework supports that composition. A Server Component can create the children and pass them to the Client Component. The Client Component can arrange or conditionally display those children without importing their server-only implementation.

### 17. Does a Client Component make every component in the whole application a Client Component?

No. The directive affects its module and the modules it imports. Server Components outside that client module graph can remain on the server. Place client boundaries deliberately to avoid pulling unrelated code into the browser bundle.

### 18. Can Server Components use React context?

They cannot read client React context in the normal Client Component sense. Put the provider in a Client Component and compose the server-rendered subtree through it where supported by the framework.

### 19. Should every button be a Client Component?

Only if it needs browser behavior such as an event handler, local state, or client interaction. A button-like visual element with no client behavior may not need its own client boundary, depending on the framework and markup.

## Rendering and hydration

### 20. What is server-side rendering (SSR)?

SSR is producing HTML on a server for a page response. It describes a rendering technique, while Server Components describe where components run and how their output participates in the React tree.

### 21. Are Server Components and SSR the same thing?

No. They are related but distinct. An application can use Server Components and also produce initial HTML on the server. A Client Component may be included in that HTML and later hydrated.

### 22. What is hydration?

Hydration is the browser attaching React's client behavior to server-rendered HTML. It enables Client Components to respond to events and use browser-side state after the initial page is delivered.

### 23. Is the Server Component response only HTML?

Not necessarily. RSC-capable frameworks use a representation that coordinates server-rendered output with Client Components. Frameworks may use it together with HTML for the initial load and updates for navigation.

### 24. Is the Server Component's source code sent to the browser?

The Server Component implementation is not sent as client-side component code. However, its rendered output and any values passed to client UI may be visible to the user.

### 25. Can a framework stream a page containing Server Components?

Yes, a framework may stream parts of a render as they become ready. The exact behavior, Suspense integration, loading UI, and error handling depend on that framework.

### 26. Does rendering on the server mean the data is fresh on every request?

No. Frameworks may pre-render or cache results. Configure caching and revalidation using the chosen framework's rules and the freshness requirements of the data.

## Data, security, and framework behavior

### 27. Should authorization be checked in a Server Component?

Yes, before reading or returning protected user-specific data. Server execution is not an authorization policy; verify the current user's identity and permissions in the appropriate trusted layer.

### 28. Is data safe just because it was fetched on the server?

No. Data rendered into HTML or sent in the component payload can reach the user. Return only the fields required by the UI and keep secrets out of rendered output and client props.

### 29. Can a Server Component read a database directly?

It can call a server-side data layer when the framework and application architecture allow it. Keep credentials in server-only configuration, validate access, and avoid leaking full database records to the client.

### 30. Do Server Components replace Server Functions or Server Actions?

No. A Server Component renders UI. A Server Function is an asynchronous server operation that supported client code can invoke; when used for a mutation or form Action, it is commonly called a Server Action. These are separate concepts.

### 31. Does React decide all caching and revalidation behavior for Server Components?

No. Frameworks provide important data-loading, cache, revalidation, routing, and deployment behavior. Check the conventions for the specific framework and version in use.

### 32. Can the same Server Component be used in any framework?

Not necessarily. RSC support and framework APIs vary. Concepts may transfer, but routing, data fetching, directives, streaming, caching, and deployment examples may need adaptation.

### 33. How should I handle errors from Server Component data loading?

Handle expected missing or invalid data in the data layer or component flow, and use the framework's supported error boundary and route error conventions for unexpected failures. Do not expose internal exception details to users.

### 34. When should I choose a Client Component?

Choose one when the UI needs browser interaction: local state, effects, event handlers, browser APIs, or client-only libraries. Keep the boundary focused and pass it only the safe data and rendered content it needs.

### 35. When should I choose a Server Component?

Choose one for content and composition that benefit from server-side data access, reduced client JavaScript, or keeping implementation code off the client. It still must follow the framework's server environment and security rules.

## Quick distinctions

### 36. What is the difference between a Server Component and a Server Function?

A Server Component renders a part of the UI in the server component environment. A Server Function is a callable server operation. They can be used together, but one does not imply the other.

### 37. What is the difference between a Server Component and a Client Component rendered with SSR?

A Server Component's implementation remains server-side. A Client Component has client code for its interactive behavior, even if a framework also renders its initial HTML on the server.

### 38. What is the safest mental model for the server/client boundary?

Treat the server as a place to load and prepare UI data, and the client as the place for browser interaction. Anything rendered or passed across the boundary may be visible; authorization and data minimization still matter.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="082-server-functions-and-form-actions.md">082 - Server Functions and form Actions - Q&amp;A</a></td><td><a href="../lessons/083-react-server-components.md">Lesson</a></td><td><a href="../examples/083-react-server-components.md">Example</a></td><td><a href="084-client-server-boundaries-and-serialization.md">084 - Client and server boundaries, serialization, and security - Q&amp;A</a></td></tr>
  </tbody>
</table>
