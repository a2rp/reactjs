# Nested routes, layouts, and route errors: course browser

## Goal

This example uses React Router Data Mode to build a small course browser with:

- A shared application layout.
- A nested courses layout with its own navigation.
- An index route at `/courses`.
- A dynamic course detail route.
- A route loader that reports a missing course.
- A local error boundary that handles the missing course while preserving the parent layout.
- A top-level catch-all route for unknown paths.

The course data is an in-memory array. No database, server rendering, or authentication is included.

## Setup and files

Use an existing Vite React project and install React Router:

```sh
npm install react-router
```

Create these files:

```text
src/
├── main.jsx
└── router.jsx
```

The project should have an HTML document containing `<div id="root"></div>`.

## `src/main.jsx`

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { RouterProvider } from "react-router";
import { router } from "./router";

const rootElement = document.getElementById("root");

if (!rootElement) {
  throw new Error('Expected an element with id="root".');
}

createRoot(rootElement).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
);
```

## `src/router.jsx`

```jsx
import {
  createBrowserRouter,
  isRouteErrorResponse,
  Link,
  NavLink,
  Outlet,
  useLoaderData,
  useRouteError,
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
    description: "How routes compose layouts and child pages.",
  },
];

async function loadCourses() {
  return courses.map(({ id, title }) => ({ id, title }));
}

async function loadCourse({ params }) {
  const course = courses.find((item) => item.id === params.courseId);

  if (!course) {
    throw new Response("Course not found", {
      status: 404,
      statusText: "Not Found",
    });
  }

  return course;
}

function RootLayout() {
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

function CoursesLayout() {
  return (
    <section>
      <h1>Courses</h1>
      <nav aria-label="Course section">
        <Link to="/courses">All courses</Link>
      </nav>

      <Outlet />
    </section>
  );
}

function HomePage() {
  return (
    <section>
      <h1>React course browser</h1>
      <p>Choose a course to see its details.</p>
      <Link to="/courses">Browse courses</Link>
    </section>
  );
}

function CourseIndexPage() {
  const courseList = useLoaderData();

  return (
    <section>
      <h2>All courses</h2>
      <ul>
        {courseList.map((course) => (
          <li key={course.id}>
            <Link to={"/courses/" + course.id}>{course.title}</Link>
          </li>
        ))}
      </ul>
    </section>
  );
}

function CourseDetailsPage() {
  const course = useLoaderData();

  return (
    <article>
      <h2>{course.title}</h2>
      <p>{course.description}</p>
      <Link to="/courses">Back to all courses</Link>
    </article>
  );
}

function CourseErrorBoundary() {
  const error = useRouteError();

  if (isRouteErrorResponse(error) && error.status === 404) {
    return (
      <section role="alert">
        <h2>Course not found</h2>
        <p>No course matches that course ID.</p>
        <Link to="/courses">Return to all courses</Link>
      </section>
    );
  }

  return (
    <section role="alert">
      <h2>We could not load this course</h2>
      <p>Please return to the course list and try again.</p>
      <Link to="/courses">Return to all courses</Link>
    </section>
  );
}

function RootErrorBoundary() {
  return (
    <main role="alert">
      <h1>Something went wrong</h1>
      <p>The page could not be displayed. Please try again later.</p>
      <Link to="/">Go home</Link>
    </main>
  );
}

function NotFoundPage() {
  return (
    <section>
      <h1>Page not found</h1>
      <p>This path does not match a course-browser page.</p>
      <Link to="/">Go home</Link>
    </section>
  );
}

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    ErrorBoundary: RootErrorBoundary,
    children: [
      {
        index: true,
        element: <HomePage />,
      },
      {
        path: "courses",
        element: <CoursesLayout />,
        children: [
          {
            index: true,
            loader: loadCourses,
            element: <CourseIndexPage />,
          },
          {
            path: ":courseId",
            loader: loadCourse,
            element: <CourseDetailsPage />,
            ErrorBoundary: CourseErrorBoundary,
          },
        ],
      },
      {
        path: "*",
        element: <NotFoundPage />,
      },
    ],
  },
]);
```

## Route tree

```text
/
├── index                         HomePage
├── courses                       CoursesLayout
│   ├── index                     CourseIndexPage
│   └── :courseId                 CourseDetailsPage
└── *                             NotFoundPage
```

Both the root route and the courses route render an `<Outlet />`. The detail route renders in the courses outlet, and the courses layout renders in the root outlet.

## What to open

| Path | Expected result |
| --- | --- |
| `/` | Home page inside the root layout |
| `/courses` | Course index inside both layouts |
| `/courses/react-basics` | Course details inside both layouts |
| `/courses/not-a-course` | Course error boundary inside the courses layout |
| `/unknown` | Catch-all not-found page inside the root layout |

The missing-course URL matches the `:courseId` route. Its loader cannot find a record and throws a 404 response. The nearest route boundary, `CourseErrorBoundary`, displays a safe message in the course section. The shared application and courses layouts remain around it.

An unknown path does not match a course route, so the catch-all route renders `NotFoundPage`. A missing record and an unmatched URL are different cases.

## Why the boundaries are placed here

- `RootErrorBoundary` is the application-level fallback.
- `CourseErrorBoundary` handles a failure specific to loading or rendering a course detail.
- The normal catch-all component handles an unmatched path.
- Expected content such as a course not existing is represented as a 404 route response.
- Internal exception details are not rendered to the visitor.

If the course loader throws an unexpected error instead, the course boundary shows a generic fallback. For production reporting, send sanitized diagnostic details through the application's protected logging system.

## Try these changes

1. Add another item to `courses`. It should appear on the index route and have a detail route.
2. Open `/courses/not-a-course`. Confirm that the course error appears while the shared navigation remains visible.
3. Open `/something-else`. Confirm that the route catch-all displays the not-found page.
4. Temporarily make `loadCourses` throw an error. The root boundary should handle it because the index route has no more-specific boundary.
5. Move `CourseErrorBoundary` to the courses parent route. Compare which route failures it then handles.

## Direct requests and production hosting

This uses a browser router in a client-rendered app. The hosting server must serve the application entry point for paths such as `/courses/react-basics`; otherwise a direct request or refresh may fail before React Router loads. Configure the SPA fallback for the chosen host. A server-rendered framework handles direct route requests according to its own conventions.

## Security note

The sample catalog is public and stored in the client-side example for simplicity. In a real application, protected course data should be loaded in a trusted server or API layer. Authenticate and authorize each read or mutation there. A nested route, hidden link, or error boundary is not an access-control system.

## Expected result

The app has persistent top-level navigation and a nested course section. The course index and detail views render inside the courses layout. An unknown course ID renders a local 404 fallback without replacing the whole application shell; an unrelated URL renders the route's not-found page.
