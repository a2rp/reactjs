# Client/server boundaries: safe course cards and local interaction

## Goal

This example shows an RSC-enabled Next.js App Router page reading data in a server-only module and passing a small serializable DTO to a Client Component. The browser-side component adds an item to a local reading list.

The sample catalog is public. It deliberately leaves internal fields out of the rendered result. For private or user-specific data, add authentication and authorization in the server-side data layer before returning a DTO.

## Files

Add these files to an existing Next.js App Router project. Keep the framework-generated root layout:

```text
app/
├── favorite-button.jsx
└── page.jsx
lib/
└── courses.js
```

In this example, the App Router treats `app/page.jsx` as a Server Component by default. The `server-only` marker is a Next.js-supported guard that helps catch imports of the data module from the client module graph.

## `lib/courses.js`

```js
import "server-only";

// Simulated server-side records. These internal fields must not reach the UI.
const courseRecords = [
  {
    id: "react-fundamentals",
    title: "React fundamentals",
    summary: "Build interfaces from components, props, and state.",
    internalReviewNote: "Needs another editorial review.",
    supplierKey: "private-example-key",
  },
  {
    id: "react-rendering",
    title: "React rendering",
    summary: "Understand how React calculates and updates the UI.",
    internalReviewNote: "Draft is approved for the public catalog.",
    supplierKey: "another-private-example-key",
  },
];

export async function getPublicCourses() {
  // In a real app, query the data source on the server.
  // Select only the fields that this public page is allowed to display.
  return courseRecords.map(({ id, title, summary }) => ({
    id,
    title,
    summary,
  }));
}
```

The function returns plain objects containing strings. Those values are supported and easy to inspect at the server/client boundary. The internal review notes and supplier keys are never placed in the DTO.

## `app/page.jsx`

```jsx
import { getPublicCourses } from "../lib/courses";
import FavoriteButton from "./favorite-button";

export default async function Page() {
  const courses = await getPublicCourses();

  return (
    <main>
      <h1>React courses</h1>
      <p>The course cards are rendered by a Server Component.</p>

      <ul>
        {courses.map((course) => (
          <li key={course.id}>
            <article>
              <h2>{course.title}</h2>
              <p>{course.summary}</p>
              <FavoriteButton courseId={course.id} />
            </article>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

The page runs in the Server Component environment. It can await data from the server-only module and passes only `courseId` to the Client Component. The browser does not need the complete server record.

## `app/favorite-button.jsx`

```jsx
"use client";

import { useState } from "react";

export default function FavoriteButton({ courseId }) {
  const [saved, setSaved] = useState(false);

  return (
    <button
      type="button"
      aria-pressed={saved}
      onClick={() => setSaved((currentValue) => !currentValue)}
    >
      {saved ? "Saved" : "Save"} course
    </button>
  );
}
```

This Client Component needs state and an event handler, so it starts the client module graph with `"use client"`. The click handler is created inside client code. The Server Component does not try to pass a normal JavaScript callback across the boundary.

The `courseId` prop is a string. It is serializable, but it is not proof that the user is allowed to modify a server record with that ID. If this button later saves a favorite to a database, the server operation must authenticate the caller, validate the ID, and check authorization independently.

## What crosses the boundary?

For each course, the Server Component passes this value:

```js
{
  courseId: "react-fundamentals"
}
```

The transport is React's Server Component protocol, not necessarily plain JSON. This example intentionally uses a simple plain-object shape with a string field.

These server-only fields do not cross the boundary:

```js
{
  internalReviewNote: "Needs another editorial review.",
  supplierKey: "private-example-key"
}
```

Keeping them out of the DTO is the protection. Merely placing a value in a Server Component does not make it secret if the value is later rendered, returned, or passed to client code.

## Data flow

1. The framework imports `lib/courses.js` from the server page.
2. The server-only module returns a filtered public DTO for each course.
3. The Server Component renders the title and summary.
4. It passes a string ID to the Client Component boundary.
5. The browser runs the button's state and event handler.
6. The click changes the local saved state; it does not write to a server database.

Because the example changes only local UI state, it needs no Server Function. If persistent favorites are added later, use the framework's supported mutation flow and enforce permissions on the server.

## Private-data adaptation

For a private profile or tenant-specific record, do not simply return a raw database row. Adapt the data function so it identifies the current viewer, checks access to the requested resource, and then returns only approved fields:

```js
import "server-only";

export async function getPrivateProfileDTO(profileId) {
  const viewer = await requireAuthenticatedViewer();
  const profile = await database.profile.findById(profileId);

  if (!profile || profile.ownerId !== viewer.id) {
    return null;
  }

  return {
    displayName: profile.displayName,
    email: profile.email,
  };
}
```

`requireAuthenticatedViewer` and `database` represent application-specific authentication and data access modules. Implement them using the project's actual trusted authentication and database layers. Never replace the trusted viewer lookup with a client-controlled ID, query parameter, or form field.

If a Client Component receives this DTO, the displayed fields are intentionally disclosed to that authorized viewer. Do not include passwords, tokens, internal flags, or unrelated record fields.

## Try these changes

1. Add a `duration` string to each course record, include it in `getPublicCourses`, and render it in `page.jsx`.
2. Add `internalReviewNote` to the returned DTO temporarily and display it. Observe that anything rendered becomes visible. Remove it again.
3. Remove the `"use client"` directive from `favorite-button.jsx`. The state Hook and click handler need a Client Component in this setup.
4. Add a function prop such as `onSaved={() => console.log("saved")}` from `page.jsx`. A locally created callback is not an ordinary serializable Server-to-Client prop; put browser event behavior in the client module instead.
5. Inspect the imports used by `favorite-button.jsx`. It should not import `lib/courses.js`, a database driver, or private environment configuration.

## Expected result

The page displays two course cards. Each card has a button that changes between “Save course” and “Saved course” when clicked. The course content comes from server-side code; the button's local interaction runs in the browser. Private fields in the simulated server records are omitted from the DTO and never rendered.

## Important limitation

This example demonstrates a public catalog and local-only favorites. It does not implement login, a database, or persistent saves. The private-data adaptation shows where the application's real authorization check belongs; it must use a genuine trusted identity and data layer before being used for protected records.
