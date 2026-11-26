# React Server Components and server rendering

## Learning goals

After this lesson, you should be able to:

- Explain what a React Server Component (RSC) is and where it can run.
- Distinguish Server Components from Client Components.
- Explain how Server Components relate to server-side rendering (SSR).
- Choose a useful boundary for interactive Client Components.
- Understand what can cross the server-to-client boundary.
- Recognize framework, data, and security constraints.

## What is a Server Component?

A Server Component is a React component that a React Server Components-enabled framework or build tool renders in a server environment. Depending on the application, it can run while the application is built or while a request is being handled.

A Server Component can read server-side data directly. For example, a framework may let it call a database layer or read files without first creating an HTTP endpoint for the browser to call. The component can be asynchronous and wait for that data during rendering.

```jsx
async function LessonList() {
  const lessons = await getLessonsFromDataLayer();

  return (
    <ul>
      {lessons.map((lesson) => (
        <li key={lesson.id}>{lesson.title}</li>
      ))}
    </ul>
  );
}
```

The function above is a conceptual example: `getLessonsFromDataLayer` must be implemented for the chosen application and must only be called in a supported server environment.

The browser does not receive the Server Component's implementation as client-side component code. The framework sends the rendered result and any data required to integrate it with Client Components. Frameworks can also produce HTML for the initial page.

## React Server Components need framework support

Server Components are an architecture supported by a framework or compatible build tool. They are not enabled by adding a directive to an ordinary React component, and a typical client-only React setup does not automatically provide them.

Frameworks own important details such as routing, data loading, caching, streaming, bundling, and deployment. Those details differ by framework and version. Use the framework's supported conventions; do not assume that an example for one framework works unchanged in another.

In a framework that supports the model, files in a designated server-rendered tree may be Server Components by default. For example, the Next.js App Router uses Server Components by default in its app tree. The exact defaults belong to that framework.

## Server Components and Client Components

Server Components and Client Components solve different problems.

| Capability | Server Component | Client Component |
| --- | --- | --- |
| Runs as part of the server component render | Yes | Its code also runs in the browser |
| Read server-only data directly | Yes, within the framework's server environment | No; call a server endpoint or pass safe data in |
| Use state, effects, or event handlers | No | Yes |
| Use browser-only APIs such as `window` | No | Yes, in the browser |
| Send its component implementation to the browser | No | Yes, when required by the client bundle |

A Server Component is a good place for content, server-side data access, and composition. A Client Component is needed for interactive behavior such as a menu that opens, a counter, an input with local state, or an effect that synchronizes with the browser.

## The `use client` boundary

In frameworks that use this convention, put the `"use client"` directive at the top of a module that needs client capabilities:

```jsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

The directive marks a client entry point. The modules it imports become part of the client-side module graph, so keep this boundary near the interactive UI where possible. A large client boundary can send more JavaScript to the browser than needed.

A Client Component is not necessarily rendered only in the browser. On an initial page load, a framework may also render its HTML on the server, then load the required JavaScript and hydrate it in the browser. The client code is needed for its state, effects, and event handlers.

A Server Component can render a Client Component and pass it data:

```jsx
export default async function Page() {
  const profile = await getPublicProfile();

  return <ProfileCard name={profile.name} />;
}
```

Only pass values supported by the framework's server-to-client transport. Do not pass database connections, server functions that are not explicitly supported, or secrets as ordinary props.

## Compose server-rendered UI with client interaction

A Client Component can receive server-rendered JSX through a prop such as `children`. This lets an interactive wrapper control a small client-side behavior while the Server Component supplies content.

```jsx
// Server Component
<Disclosure title="Lesson details">
  <LessonDetails lesson={lesson} />
</Disclosure>
```

Here, `Disclosure` can be a Client Component and `LessonDetails` can remain a Server Component. The Client Component should render the supplied `children`; it should not import the Server Component into its own client module.

This composition pattern helps keep data access and mostly static content on the server while making the required interaction work in the browser.

## Server Components are not the same as SSR

Server Components and server-side rendering describe related but different parts of rendering:

- **Server Components** describe where a component runs and how its output participates in the component tree.
- **Server-side rendering (SSR)** describes producing HTML on a server for a page response.
- **Hydration** is the browser attaching React behavior to HTML that was rendered earlier.

A framework can combine these techniques. It can render initial HTML on the server, use Server Components for server-owned parts of the tree, and hydrate Client Components for interaction. A Client Component can still be included in server-rendered initial HTML; it remains a Client Component because its JavaScript provides browser behavior.

Do not assume that every Server Component response is simply an HTML string. Frameworks use a representation that lets React and the framework coordinate server-rendered output with client-side components. The framework may use that output alongside HTML, streaming, or later navigation updates.

## Data loading, streaming, and errors

An asynchronous Server Component can wait for data as part of rendering. Frameworks may stream ready parts of a page while slower parts are still pending. Suspense boundaries and loading conventions can provide fallback UI, but the exact routing, caching, revalidation, and error behavior is framework-specific.

Keep data-loading rules explicit:

1. Decide whether data is needed at build time or for each request.
2. Read it through the server data source supported by the framework.
3. Handle missing records and expected failures.
4. Return only the data the rendered UI needs.
5. Follow the framework's caching and revalidation rules.

Rendering on a server does not automatically mean data is fresh on every request. A framework may cache a result or pre-render a route. Configure freshness according to the framework and the data's needs.

## Security and data boundaries

Server Components can access private resources, but rendered output is not private merely because it was produced on the server. Any data rendered into HTML or sent through the framework's component payload can reach the user.

- Do not render passwords, private tokens, or internal-only fields.
- Authorize access before reading or returning user-specific data.
- Pass a minimal view model to Client Components instead of an entire database record.
- Do not assume that hiding a value in a Client Component makes it secret.
- Use the framework's server-only boundary checks where available.
- Keep request-specific data scoped to the current request unless the framework documents safe caching.

The server/client boundary is an architectural boundary, not an authorization system.

## Context and browser APIs

Server Components cannot use client React context, state, effects, or browser globals. If a subtree needs context, put the context provider in a Client Component and render the server-created children through it where the framework supports that composition.

Browser APIs such as `window`, `document`, local storage, and event listeners belong in Client Components. Access them in an effect or an event handler when appropriate; they do not exist during server rendering.

## Common mistakes

- **Thinking Server Components and Server Functions are the same:** Components render UI; Server Functions are callable server operations.
- **Adding `"use client"` everywhere:** This expands the client module graph and can increase browser JavaScript.
- **Assuming a Client Component only renders in the browser:** A framework may server-render its initial HTML and hydrate it later.
- **Importing a Server Component from a Client Component:** Keep server-owned composition in the Server Component and pass rendered UI through supported props.
- **Sending all server data to the client:** Return only the safe fields needed for the UI.
- **Assuming server execution is authorization:** Check identity and permissions before reading protected data.
- **Treating caching as universal React behavior:** Rendering and caching policies are supplied by the framework.
- **Expecting RSC to work in every React setup:** Choose a framework or build tool that explicitly supports it.

## Summary

A Server Component runs in a server environment managed by an RSC-capable framework. It can load data and render UI without shipping its implementation to the browser. A Client Component provides state, effects, browser APIs, and event handlers. Use Server Components for server-owned data and composition, add focused Client Component boundaries for interaction, and treat all rendered or passed data as potentially visible to the user. SSR, hydration, and Server Components can work together, but they describe different parts of the system.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="082-server-functions-and-form-actions.md">082 - Server Functions and form Actions</a></td><td><a href="../qnas/083-react-server-components.md">Q&amp;A</a></td><td><a href="../examples/083-react-server-components.md">Example</a></td><td><a href="084-client-server-boundaries-and-serialization.md">084 - Client and server boundaries, serialization, and security</a></td></tr>
  </tbody>
</table>
