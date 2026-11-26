# Routing concepts: a small React course browser

## Goal

This example uses React Router's Declarative mode in a client-rendered React app. It maps URLs to a home page, course list, course detail, and not-found screen.

The example focuses on route matching and navigation. It does not add data loaders, server rendering, authentication, or query-string state; those depend on the application's chosen architecture and are covered elsewhere in the course.

## Setup

Use an existing Vite React project and install React Router:

```sh
npm install react-router
```

Create these files:

```text
src/
├── App.jsx
└── main.jsx
```

The project should already have an HTML document containing a `<div id="root"></div>` element.

## `src/main.jsx`

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router";
import App from "./App";

const rootElement = document.getElementById("root");

if (!rootElement) {
  throw new Error('Expected an element with id="root".');
}

createRoot(rootElement).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
```

Mount one `BrowserRouter` around the part of the application that uses router APIs. It reads the browser location and coordinates client-side navigation with browser history.

## `src/App.jsx`

```jsx
import {
  Link,
  NavLink,
  Outlet,
  Route,
  Routes,
  useParams,
} from "react-router";

const courses = [
  {
    id: "react-basics",
    title: "React basics",
    description: "Components, JSX, props, and state.",
  },
  {
    id: "react-rendering",
    title: "React rendering",
    description: "How React calculates and updates the interface.",
  },
  {
    id: "react-routing",
    title: "React routing",
    description: "How URLs select pages and how users navigate.",
  },
];

function AppLayout() {
  return (
    <>
      <header>
        <nav aria-label="Main navigation">
          <NavLink to="/" end>
            Home
          </NavLink>{" "}
          <NavLink to="/courses">
            Courses
          </NavLink>
        </nav>
      </header>

      <main>
        <Outlet />
      </main>
    </>
  );
}

function HomePage() {
  return (
    <section>
      <h1>React course browser</h1>
      <p>Choose a course to view its details.</p>
      <Link to="/courses">Browse courses</Link>
    </section>
  );
}

function CourseListPage() {
  return (
    <section>
      <h1>Courses</h1>
      <ul>
        {courses.map((course) => (
          <li key={course.id}>
            <Link to={"/courses/" + course.id}>{course.title}</Link>
          </li>
        ))}
      </ul>
    </section>
  );
}

function CourseDetailsPage() {
  const { courseId } = useParams();
  const course = courses.find((item) => item.id === courseId);

  if (!course) {
    return (
      <section>
        <h1>Course not found</h1>
        <p>No course matches the requested course ID.</p>
        <Link to="/courses">Return to all courses</Link>
      </section>
    );
  }

  return (
    <article>
      <h1>{course.title}</h1>
      <p>{course.description}</p>
      <Link to="/courses">Back to courses</Link>
    </article>
  );
}

function NotFoundPage() {
  return (
    <section>
      <h1>Page not found</h1>
      <p>This URL does not match a course-browser route.</p>
      <Link to="/">Go home</Link>
    </section>
  );
}

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<AppLayout />}>
        <Route index element={<HomePage />} />
        <Route path="courses" element={<CourseListPage />} />
        <Route path="courses/:courseId" element={<CourseDetailsPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Route>
    </Routes>
  );
}
```

## Routes created

| URL | Result |
| --- | --- |
| `/` | Home page |
| `/courses` | Course list |
| `/courses/react-basics` | Details for the matching course |
| `/courses/not-a-course` | Course-level not-found message |
| `/something-else` | Route-level not-found page |

The route with `:courseId` matches one path segment and makes its value available through `useParams()`. The application looks up that value in its data and handles a missing course instead of assuming that every URL is valid.

## How navigation works here

- `BrowserRouter` reads the current browser location.
- `Routes` selects the route that matches the URL.
- `Route` connects the path to a component.
- `NavLink` and `Link` render links for in-app navigation.
- `Outlet` is the position where the selected child route appears inside the shared layout.
- `useParams()` reads the dynamic path value.

Clicking a router link updates the route while keeping the shared app shell mounted. The browser history still records navigation, so Back and Forward remain useful.

## Direct URL requests

Test the detail URL by opening it directly or refreshing it after navigation. A development server usually provides the needed SPA fallback. Production hosting must also be configured to serve the app entry point for client-side routes. If the host returns a file-not-found response for `/courses/react-basics` before React starts, change the host's routing configuration or choose a server-rendered framework that handles the route requests.

## Extend the example

1. Add a course record to the `courses` array. It should appear in the list and have a detail route.
2. Type an unknown course ID after `/courses/`. The route matches, but the lookup should display “Course not found”.
3. Open an unrelated path. The catch-all route should display “Page not found”.
4. Use the browser Back and Forward buttons after clicking several course links.
5. Replace the local array with route-loader or server-side data using the selected router's supported data model.

## Limits and security

This is a client-only example with static sample data. A route match and a hidden navigation link are not access control. For private course data, validate the requested ID and check the visitor's permissions in the server or API layer that returns that data.

The catch-all component provides a not-found UI in the client app. Whether a deployment also returns an HTTP 404 status depends on the server or framework configuration.

## Expected result

The app shows a home page with a link to the course list. Each course link opens a detail URL. The shared navigation remains visible, the dynamic ID selects the matching course, and unknown URLs render a helpful not-found message.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="084-client-server-boundaries-and-serialization.md">084 - Client/server boundaries: safe course cards and local interaction</a></td><td><a href="../lessons/085-routing-concepts-and-router-choice.md">Lesson</a></td><td><a href="../qnas/085-routing-concepts-and-router-choice.md">Q&amp;A</a></td><td><a href="086-nested-routes-layouts-and-errors.md">086 - Nested routes, layouts, and route errors: course browser</a></td></tr>
  </tbody>
</table>
