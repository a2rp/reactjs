# React Server Components: lesson list with one interactive island

## Goal

This example uses an RSC-capable framework's App Router to render a lesson list on the server and hydrate one small interactive counter in the browser.

The lesson list stays in a Server Component. Only the like button needs client-side state and an event handler. The example uses in-memory sample data so it runs without a database or external service.

## Project files

Use these files inside an existing Next.js App Router project:

```text
app/
├── like-button.jsx
└── page.jsx
```

Keep the `app/layout.jsx` file created by the framework's project scaffold. In this framework's App Router, `page.jsx` is a Server Component by default.

## `app/page.jsx`

```jsx
import LikeButton from "./like-button";

async function getLessons() {
  // Replace this sample array with a database or other server-side data source.
  return [
    {
      id: "components",
      title: "Components and JSX",
      summary: "Describe the interface with reusable React components.",
    },
    {
      id: "props",
      title: "Props and composition",
      summary: "Pass data into components and compose larger interfaces.",
    },
    {
      id: "state",
      title: "State and interaction",
      summary: "Keep interactive UI state in the component that owns it.",
    },
  ];
}

export default async function Page() {
  const lessons = await getLessons();

  return (
    <main>
      <h1>React course lessons</h1>
      <p>This list is rendered by a Server Component.</p>

      <ul>
        {lessons.map((lesson) => (
          <li key={lesson.id}>
            <h2>{lesson.title}</h2>
            <p>{lesson.summary}</p>
          </li>
        ))}
      </ul>

      <section aria-labelledby="like-heading">
        <h2 id="like-heading">Interactive Client Component</h2>
        <p>The list above stays server-rendered. This button needs browser state.</p>
        <LikeButton initialCount={0} />
      </section>
    </main>
  );
}
```

The page can be an async Server Component in the App Router. It awaits its data before returning the UI. The sample data is local to the server module; in a real app, replace the function body with a server-side data-layer call supported by the application.

## `app/like-button.jsx`

```jsx
"use client";

import { useState } from "react";

export default function LikeButton({ initialCount }) {
  const [count, setCount] = useState(initialCount);

  return (
    <button
      type="button"
      onClick={() => setCount((currentCount) => currentCount + 1)}
    >
      Likes: {count}
    </button>
  );
}
```

The `"use client"` directive makes this module a Client Component entry point. It uses state and an event handler, so its JavaScript must be available in the browser.

## What happens

1. The framework renders `Page` in its Server Component environment.
2. `Page` awaits `getLessons()` and creates the lesson list markup.
3. The framework includes the Client Component boundary and its supported props in the rendered result.
4. On an initial page load, the framework can send HTML for the page and button.
5. The browser loads the Client Component JavaScript and hydrates the button.
6. Clicking the button updates its local state without re-rendering the lesson data source.

The exact transport, HTML generation, streaming, and navigation details belong to the framework.

## Why the boundary is small

The lesson headings and summaries do not need browser state or event handlers, so they can remain in the Server Component. The button does need state, so only its module is marked as client code.

If the entire page were moved into a Client Component, the browser would need its implementation and all client-side dependencies. A focused boundary keeps the interactive portion clear and avoids making unrelated UI client-side.

## Passing data safely

The example passes only a number to the button. If the lesson data came from a private database, render only the fields meant for the visitor. Never pass credentials, private tokens, full database clients, or internal-only records as Client Component props.

A server-rendered result is still visible to the person who receives it. Check authorization before loading protected lessons, and keep private values out of the rendered UI.

## Try these changes

1. Change the initial count from `0` to `5`. The button should start at five and increment from there.
2. Add one more lesson to the array. It should appear without changing the Client Component.
3. Add a `category` string to each lesson and render it in the Server Component list.
4. Remove `"use client"` from `like-button.jsx`. The component will no longer be allowed to use client state and an event handler in this setup.
5. Keep `"use client"` on the button and add unrelated static content to `page.jsx`. That content can remain in the Server Component.

## Common mistakes

- Importing a database client into `like-button.jsx`. The button is part of the client module graph; keep server data access in server modules.
- Removing `"use client"` while retaining `useState` or `onClick`.
- Passing a secret as a prop because the parent is server-rendered.
- Marking the whole page as a Client Component just to support one button.
- Assuming this file layout works in a plain client-only React setup. The App Router and RSC support are framework features.

## Expected result

The page displays three lesson summaries and a button labeled `Likes: 0`. Each click increments the number. The lesson list is rendered by the Server Component, while the button's state and click behavior are provided by the Client Component.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="082-server-functions-and-form-actions.md">082 - Server Functions and form Actions - Example</a></td><td><a href="../lessons/083-react-server-components.md">Lesson</a></td><td><a href="../qnas/083-react-server-components.md">Q&amp;A</a></td><td><a href="084-client-server-boundaries-and-serialization.md">084 - Client/server boundaries: safe course cards and local interaction</a></td></tr>
  </tbody>
</table>
