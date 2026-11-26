# URL State, Search Parameters, and Navigation - Examples

This example builds a small course catalog whose search text, category, sort order, and page are stored in the URL. A link can reproduce the same view, reload keeps the controls selected, and Back and Forward restore earlier selections.

The code uses React Router v7 imports from `react-router`. In a React Router v6 project, import the corresponding APIs from `react-router-dom`.

## 1. Create a React app and add the router

In an existing Vite React app, install React Router:

```sh
npm install react-router
```

Use these files:

```text
src/
  main.jsx
  styles.css
```

## 2. Build the URL-backed catalog

Create `src/main.jsx`:

```jsx
import { StrictMode, useEffect } from "react";
import { createRoot } from "react-dom/client";
import {
  BrowserRouter,
  Link,
  Navigate,
  Route,
  Routes,
  useLocation,
  useSearchParams,
} from "react-router";
import "./styles.css";

const courses = [
  {
    id: "components",
    title: "Thinking in Components",
    category: "foundations",
    summary: "Break an interface into reusable pieces and describe each piece with JSX.",
    popularity: 96,
  },
  {
    id: "props",
    title: "Props and Component Data",
    category: "foundations",
    summary: "Pass information through a component tree with clear, predictable interfaces.",
    popularity: 88,
  },
  {
    id: "state",
    title: "State and User Interaction",
    category: "components",
    summary: "Model changing UI values and update them in response to user actions.",
    popularity: 93,
  },
  {
    id: "effects",
    title: "Effects and External Systems",
    category: "components",
    summary: "Synchronize a component with browser APIs and other external systems.",
    popularity: 79,
  },
  {
    id: "routing",
    title: "Routes and Navigation",
    category: "routing",
    summary: "Connect application URLs to pages, links, and navigation behavior.",
    popularity: 85,
  },
  {
    id: "forms",
    title: "Accessible Forms",
    category: "routing",
    summary: "Build forms with labels, useful controls, validation, and clear feedback.",
    popularity: 81,
  },
  {
    id: "styling",
    title: "Styling a React Interface",
    category: "styling",
    summary: "Organize styles and build a responsive interface with reusable patterns.",
    popularity: 74,
  },
];

const categories = [
  { value: "all", label: "All categories" },
  { value: "foundations", label: "Foundations" },
  { value: "components", label: "Components" },
  { value: "routing", label: "Routing" },
  { value: "styling", label: "Styling" },
];

const allowedCategories = new Set(categories.map(({ value }) => value));
const allowedSorts = new Set(["popular", "title"]);
const pageSize = 3;

function readPositiveInteger(value, fallback = 1) {
  const parsed = Number(value);

  return Number.isInteger(parsed) && parsed > 0 ? parsed : fallback;
}

function CourseCatalog() {
  const [searchParams, setSearchParams] = useSearchParams();
  const location = useLocation();

  const queryValue = searchParams.get("q") ?? "";
  const query = queryValue.trim().toLocaleLowerCase();

  const requestedCategory = searchParams.get("category") ?? "all";
  const category = allowedCategories.has(requestedCategory)
    ? requestedCategory
    : "all";

  const requestedSort = searchParams.get("sort") ?? "popular";
  const sort = allowedSorts.has(requestedSort) ? requestedSort : "popular";

  const matchingCourses = courses.filter((course) => {
    const matchesQuery =
      query.length === 0 ||
      `${course.title} ${course.summary}`.toLocaleLowerCase().includes(query);

    const matchesCategory =
      category === "all" || course.category === category;

    return matchesQuery && matchesCategory;
  });

  const sortedCourses = [...matchingCourses].sort((first, second) => {
    if (sort === "title") {
      return first.title.localeCompare(second.title);
    }

    return second.popularity - first.popularity;
  });

  const totalPages = Math.max(1, Math.ceil(sortedCourses.length / pageSize));
  const requestedPage = readPositiveInteger(searchParams.get("page"));
  const page = Math.min(requestedPage, totalPages);

  const visibleCourses = sortedCourses.slice(
    (page - 1) * pageSize,
    page * pageSize,
  );

  // Normalize malformed or out-of-range pages without adding a history step.
  useEffect(() => {
    const rawPage = searchParams.get("page");

    if (rawPage !== null && String(page) !== rawPage) {
      const next = new URLSearchParams(searchParams);

      if (page === 1) {
        next.delete("page");
      } else {
        next.set("page", String(page));
      }

      setSearchParams(next, { replace: true });
    }
  }, [page, searchParams, setSearchParams]);

  function handleSearchSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);
    const nextQuery = String(formData.get("q") ?? "").trim();
    const next = new URLSearchParams(searchParams);

    if (nextQuery) {
      next.set("q", nextQuery);
    } else {
      next.delete("q");
    }

    // A changed search can have a different number of pages.
    next.delete("page");
    setSearchParams(next);
  }

  function handleCategoryChange(event) {
    const next = new URLSearchParams(searchParams);
    const nextCategory = event.target.value;

    if (nextCategory === "all") {
      next.delete("category");
    } else {
      next.set("category", nextCategory);
    }

    next.delete("page");
    setSearchParams(next);
  }

  function handleSortChange(event) {
    const next = new URLSearchParams(searchParams);
    const nextSort = event.target.value;

    if (nextSort === "popular") {
      next.delete("sort");
    } else {
      next.set("sort", nextSort);
    }

    next.delete("page");
    setSearchParams(next);
  }

  function goToPage(nextPage) {
    const next = new URLSearchParams(searchParams);

    if (nextPage <= 1) {
      next.delete("page");
    } else {
      next.set("page", String(nextPage));
    }

    setSearchParams(next);
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <p className="eyebrow">React learning library</p>
        <h1>Course catalog</h1>
        <p>
          Search, filter, and sort courses. The address bar records the current
          view, so you can reload it or share it.
        </p>
      </header>

      <section className="controls" aria-label="Course search and filters">
        <form
          key={queryValue}
          className="search-form"
          role="search"
          onSubmit={handleSearchSubmit}
        >
          <label htmlFor="course-query">Search courses</label>
          <div className="search-row">
            <input
              id="course-query"
              name="q"
              type="search"
              defaultValue={queryValue}
              placeholder="Try components or forms"
            />
            <button type="submit">Search</button>
          </div>
        </form>

        <div className="select-row">
          <label>
            Category
            <select value={category} onChange={handleCategoryChange}>
              {categories.map((item) => (
                <option key={item.value} value={item.value}>
                  {item.label}
                </option>
              ))}
            </select>
          </label>

          <label>
            Sort by
            <select value={sort} onChange={handleSortChange}>
              <option value="popular">Most popular</option>
              <option value="title">Title</option>
            </select>
          </label>
        </div>

        <Link className="clear-link" to="/courses">
          Clear all filters
        </Link>
      </section>

      <section id="results" className="results" aria-labelledby="results-title">
        <div className="results-heading">
          <div>
            <h2 id="results-title">Courses</h2>
            <p aria-live="polite">
              {sortedCourses.length} result{sortedCourses.length === 1 ? "" : "s"}
              {sortedCourses.length > 0
                ? ` · Page ${page} of ${totalPages}`
                : ""}
            </p>
          </div>
          <code className="current-url">
            {location.pathname}
            {location.search}
          </code>
        </div>

        {visibleCourses.length > 0 ? (
          <ul className="course-grid">
            {visibleCourses.map((course) => (
              <li className="course-card" key={course.id}>
                <p className="course-category">{course.category}</p>
                <h3>{course.title}</h3>
                <p>{course.summary}</p>
                <p className="popularity">
                  Popularity score: {course.popularity}
                </p>
              </li>
            ))}
          </ul>
        ) : (
          <p className="empty-state">
            No courses match these options. Try a different search or clear the
            filters.
          </p>
        )}

        <nav className="pagination" aria-label="Course result pages">
          <button
            type="button"
            onClick={() => goToPage(page - 1)}
            disabled={page <= 1}
          >
            Previous
          </button>
          <span>
            Page {page} of {totalPages}
          </span>
          <button
            type="button"
            onClick={() => goToPage(page + 1)}
            disabled={page >= totalPages}
          >
            Next
          </button>
        </nav>
      </section>
    </main>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/courses" element={<CourseCatalog />} />
        <Route path="*" element={<Navigate to="/courses" replace />} />
      </Routes>
    </BrowserRouter>
  );
}

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

## 3. Add the styles

Create `src/styles.css`:

```css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #172033;
  background: #f4f6fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button,
input,
select {
  font: inherit;
}

button,
select {
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.page-shell {
  width: min(100% - 2rem, 980px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.page-header {
  max-width: 680px;
  margin-bottom: 2rem;
}

.eyebrow,
.course-category {
  color: #4b5ec7;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2,
h3,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.75rem;
  font-size: clamp(2rem, 5vw, 3.25rem);
  line-height: 1.08;
}

.page-header > p:last-child {
  color: #586174;
  line-height: 1.65;
}

.controls,
.course-card {
  border: 1px solid #dce1ec;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 8px 24px rgb(25 39 74 / 5%);
}

.controls {
  display: grid;
  gap: 1.25rem;
  padding: 1.25rem;
}

.search-form label,
.select-row label {
  display: grid;
  gap: 0.45rem;
  font-size: 0.9rem;
  font-weight: 650;
}

.search-row,
.select-row {
  display: flex;
  gap: 0.75rem;
}

.search-row input,
.select-row select {
  min-width: 0;
  min-height: 44px;
  border: 1px solid #bac3d4;
  border-radius: 8px;
  padding: 0.6rem 0.75rem;
  background: #fff;
  color: inherit;
}

.search-row input {
  flex: 1;
}

.search-row button,
.pagination button {
  min-height: 44px;
  border: 0;
  border-radius: 8px;
  padding: 0.6rem 1rem;
  background: #3448b5;
  color: #fff;
  font-weight: 700;
}

.select-row > label {
  flex: 1;
}

.select-row select {
  width: 100%;
}

.clear-link {
  width: fit-content;
  color: #3448b5;
}

.results {
  margin-top: 2.5rem;
}

.results-heading {
  display: flex;
  align-items: end;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
}

.results-heading h2 {
  margin-bottom: 0.25rem;
}

.results-heading p {
  margin-bottom: 0;
  color: #586174;
}

.current-url {
  max-width: 50%;
  overflow-wrap: anywhere;
  border-radius: 6px;
  padding: 0.4rem 0.55rem;
  background: #e8ebf4;
  color: #35405a;
  font-size: 0.8rem;
}

.course-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 250px), 1fr));
  gap: 1rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.course-card {
  padding: 1.2rem;
}

.course-card h3 {
  margin-bottom: 0.55rem;
}

.course-card > p:not(.course-category, .popularity) {
  color: #586174;
  line-height: 1.55;
}

.course-category {
  margin-bottom: 0.65rem;
  font-size: 0.7rem;
}

.popularity {
  margin-bottom: 0;
  color: #586174;
  font-size: 0.85rem;
}

.empty-state {
  border: 1px dashed #aeb8cc;
  border-radius: 12px;
  padding: 1.25rem;
  color: #586174;
}

.pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-top: 1.5rem;
}

.pagination span {
  color: #586174;
  font-size: 0.9rem;
}

@media (max-width: 600px) {
  .page-shell {
    padding-top: 2rem;
  }

  .search-row,
  .select-row,
  .results-heading {
    align-items: stretch;
    flex-direction: column;
  }

  .current-url {
    max-width: none;
  }

  .pagination {
    gap: 0.65rem;
  }
}
```

## 4. Try the URL state

1. Open `/courses`. The default sort is popularity, and defaults are omitted from the query string.
2. Search for `forms`. The submitted search appears in the URL as `q=forms`.
3. Choose a category or sort order. Its value is added to the query string while existing values remain.
4. Move to the next result page. The URL records `page=2`.
5. Copy the address or reload. The query, selected controls, and current page are reconstructed from the URL.
6. Change a filter, then use Back and Forward. The controls and result list follow the restored URL.
7. Enter a malformed or out-of-range `page` value in the address. The example falls back to a valid page and replaces the invalid history entry.
8. Select a filter combination with no matches. The page shows an empty state and keeps the selected values visible.

Example URLs include:

```text
/courses?q=forms
/courses?q=forms&category=routing
/courses?q=forms&category=routing&sort=title&page=2
```

## 5. Why the handlers clone the current URL

Each handler starts with:

```js
const next = new URLSearchParams(searchParams);
```

It changes only the parameter it owns, then sends one complete next value to `setSearchParams`. Search, category, and sort changes remove `page`, because the previous page number might not exist in the new result set.

The search form uses a local, unsubmitted draft. Submitting the form commits it to the URL and creates a history entry. Its `key` changes with the committed query, so Back and Forward reset the draft input to match the restored URL.

## 6. Optional variations

### Replace history for debounced typing

If you update the URL while the user types, debounce the updates and replace the current entry so Back does not visit every character. The router setter accepts a replacement option:

```js
setSearchParams(next, { replace: true });
```

Keep the submitted-search pattern above if a search should create a useful Back step.

### Use a plain GET form

A server-rendered search page can use a standard form:

```html
<form action="/courses" method="get">
  <label for="q">Search courses</label>
  <input id="q" name="q" type="search">
  <button type="submit">Search</button>
</form>
```

The browser puts the named value in the query string. The server must parse and validate it before using it.

### Add repeated filters

For multiple selected tags, read all values with `getAll("tag")` and append each selected value when building the next search parameters. Remove only the tags that were deselected; preserve the other parameters.

### Verify framework-specific behavior

This client-side example uses React Router. A server-rendered React framework may expose query values through route props or another API. Adapt the parsing code to that framework and validate values on the server as well.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="086-nested-routes-layouts-and-errors.md">086 - Nested routes, layouts, and route errors: course browser</a></td><td><a href="../lessons/087-url-state-and-navigation.md">Lesson</a></td><td><a href="../qnas/087-url-state-and-navigation.md">Q&amp;A</a></td><td><a href="088-organizing-react-applications.md">088 - Organizing Components, Features, and Application Code - Examples</a></td></tr>
  </tbody>
</table>
