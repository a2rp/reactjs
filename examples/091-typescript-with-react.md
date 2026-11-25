# TypeScript with React: Reading List Example

This example builds a small reading-list interface with typed component props, controlled form state, a DOM ref, a filter union, and an exhaustive reducer action union.

Use these files inside an existing React project configured for TypeScript and JSX. Files containing JSX use the `.tsx` extension; the type and reducer modules use `.ts`. The project entry point should render `App` from `src/main.tsx`.

## File structure

```text
src/
├── App.tsx
├── App.css
├── main.tsx
├── types.ts
└── books/
    ├── BookForm.tsx
    ├── BookList.tsx
    ├── ReadingList.tsx
    ├── bookReducer.ts
    └── initialState.ts
```

## `src/types.ts`

```ts
export type BookFilter = "all" | "reading" | "finished";

export interface Book {
  id: number;
  title: string;
  finished: boolean;
}

export interface BookState {
  books: Book[];
  nextId: number;
}

export type BookAction =
  | { type: "book/added"; title: string }
  | { type: "book/toggled"; id: number }
  | { type: "book/removed"; id: number };
```

The filter and action `type` fields are finite unions. A misspelled filter or unknown action becomes a type error.

## `src/books/initialState.ts`

```ts
import type { BookState } from "../types";

export const initialBookState: BookState = {
  books: [
    { id: 1, title: "A book to start with", finished: false },
    { id: 2, title: "A finished book", finished: true },
  ],
  nextId: 3,
};
```

## `src/books/bookReducer.ts`

```ts
import type { BookAction, BookState } from "../types";

function assertNever(value: never): never {
  throw new Error(`Unhandled book action: ${String(value)}`);
}

export function bookReducer(
  state: BookState,
  action: BookAction,
): BookState {
  switch (action.type) {
    case "book/added": {
      const title = action.title.trim();
      if (title.length === 0) return state;

      return {
        books: [
          ...state.books,
          { id: state.nextId, title, finished: false },
        ],
        nextId: state.nextId + 1,
      };
    }

    case "book/toggled":
      return {
        ...state,
        books: state.books.map((book) =>
          book.id === action.id
            ? { ...book, finished: !book.finished }
            : book,
        ),
      };

    case "book/removed":
      return {
        ...state,
        books: state.books.filter((book) => book.id !== action.id),
      };

    default:
      return assertNever(action);
  }
}
```

The reducer returns new state and does not mutate its input. If another action is added to `BookAction` without a matching `case`, the `never` check points out the missing handling.

## `src/books/BookForm.tsx`

```tsx
import { useRef, useState } from "react";
import type { FormEvent } from "react";

type BookFormProps = {
  onAdd: (title: string) => void;
};

export function BookForm({ onAdd }: BookFormProps) {
  const [draft, setDraft] = useState("");
  const [error, setError] = useState("");
  const inputRef = useRef<HTMLInputElement | null>(null);

  function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    const title = draft.trim();
    if (title.length === 0) {
      setError("Enter a title before adding a book.");
      inputRef.current?.focus();
      return;
    }

    onAdd(title);
    setDraft("");
    setError("");
    inputRef.current?.focus();
  }

  return (
    <form className="book-form" onSubmit={handleSubmit}>
      <label htmlFor="book-title">Book title</label>
      <div className="form-row">
        <input
          id="book-title"
          ref={inputRef}
          value={draft}
          onChange={(event) => {
            setDraft(event.currentTarget.value);
            if (error) setError("");
          }}
          placeholder="For example, The Hobbit"
          aria-describedby={error ? "book-title-error" : undefined}
        />
        <button type="submit">Add book</button>
      </div>
      {error && (
        <p className="form-error" id="book-title-error" role="alert">
          {error}
        </p>
      )}
    </form>
  );
}
```

The input event is inferred from its JSX position. The named submit handler uses `FormEvent<HTMLFormElement>`. The ref’s type describes both the DOM element and its initial `null` value.

## `src/books/BookList.tsx`

```tsx
import type { Book } from "../types";

type BookListProps = {
  books: Book[];
  onToggle: (id: number) => void;
  onRemove: (id: number) => void;
};

export function BookList({ books, onToggle, onRemove }: BookListProps) {
  if (books.length === 0) {
    return <p className="empty-state">No books match this filter.</p>;
  }

  return (
    <ul className="book-list" aria-live="polite">
      {books.map((book) => (
        <li className="book-row" key={book.id}>
          <span className={book.finished ? "book-title finished" : "book-title"}>
            {book.title}
          </span>
          <div className="book-actions">
            <button
              type="button"
              className="secondary-button"
              onClick={() => onToggle(book.id)}
              aria-label={`${book.finished ? "Mark as reading" : "Mark as finished"}: ${book.title}`}
            >
              {book.finished ? "Mark reading" : "Mark finished"}
            </button>
            <button
              type="button"
              className="remove-button"
              onClick={() => onRemove(book.id)}
              aria-label={`Remove ${book.title}`}
            >
              Remove
            </button>
          </div>
        </li>
      ))}
    </ul>
  );
}
```

`key` helps React track list items; it is not passed into `BookList` as a regular prop. The book’s `id` is passed to callbacks as application data.

## `src/books/ReadingList.tsx`

```tsx
import { useReducer, useState } from "react";
import type { ChangeEvent } from "react";
import type { BookFilter } from "../types";
import { bookReducer } from "./bookReducer";
import { initialBookState } from "./initialState";
import { BookForm } from "./BookForm";
import { BookList } from "./BookList";

function isBookFilter(value: string): value is BookFilter {
  return value === "all" || value === "reading" || value === "finished";
}

export function ReadingList() {
  const [state, dispatch] = useReducer(bookReducer, initialBookState);
  const [filter, setFilter] = useState<BookFilter>("all");

  function handleFilterChange(event: ChangeEvent<HTMLSelectElement>) {
    const value = event.currentTarget.value;
    if (isBookFilter(value)) setFilter(value);
  }

  const visibleBooks = state.books.filter((book) => {
    if (filter === "reading") return !book.finished;
    if (filter === "finished") return book.finished;
    return true;
  });

  const finishedCount = state.books.filter((book) => book.finished).length;

  return (
    <section className="reading-list" aria-labelledby="reading-list-title">
      <div className="list-heading">
        <div>
          <h2 id="reading-list-title">Your books</h2>
          <p>
            {finishedCount} of {state.books.length} finished
          </p>
        </div>
        <label className="filter-control">
          <span>Show</span>
          <select value={filter} onChange={handleFilterChange}>
            <option value="all">All books</option>
            <option value="reading">Still reading</option>
            <option value="finished">Finished</option>
          </select>
        </label>
      </div>

      <BookForm
        onAdd={(title) => dispatch({ type: "book/added", title })}
      />

      <BookList
        books={visibleBooks}
        onToggle={(id) => dispatch({ type: "book/toggled", id })}
        onRemove={(id) => dispatch({ type: "book/removed", id })}
      />
    </section>
  );
}
```

The browser select returns a string. `isBookFilter` narrows that string before it is stored as `BookFilter`, so a value outside the union cannot enter the state through this handler.

## `src/App.tsx`

```tsx
import { ReadingList } from "./books/ReadingList";
import "./App.css";

export default function App() {
  return (
    <main className="app-shell">
      <header className="page-heading">
        <p className="eyebrow">A small React + TypeScript project</p>
        <h1>Reading list</h1>
        <p>Keep track of what you are reading and what you have finished.</p>
      </header>
      <ReadingList />
    </main>
  );
}
```

## `src/main.tsx`

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";

const rootElement = document.getElementById("root");
if (!rootElement) {
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

## `src/App.css`

```css
:root {
  font-family: Inter, ui-sans-serif, system-ui, sans-serif;
  color: #172033;
  background: #f3f6fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
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

button {
  min-height: 42px;
  padding: 0.6rem 0.9rem;
  border: 0;
  border-radius: 0.65rem;
  background: #3157d5;
  color: #fff;
  font-weight: 700;
  cursor: pointer;
}

button:hover {
  filter: brightness(0.94);
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #98b5ff;
  outline-offset: 2px;
}

.app-shell {
  width: min(100% - 2rem, 760px);
  margin: 0 auto;
  padding: 4rem 0;
}

.page-heading {
  margin-bottom: 2rem;
}

.page-heading h1 {
  margin: 0.3rem 0;
  font-size: clamp(2.4rem, 7vw, 4rem);
  letter-spacing: -0.05em;
}

.page-heading p {
  color: #5b6578;
  line-height: 1.6;
}

.page-heading .eyebrow {
  margin: 0;
  color: #3157d5;
  font-size: 0.8rem;
  font-weight: 800;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.reading-list {
  padding: clamp(1.1rem, 4vw, 2rem);
  border: 1px solid #dce3ef;
  border-radius: 1.2rem;
  background: #fff;
  box-shadow: 0 18px 50px rgb(24 42 82 / 8%);
}

.list-heading,
.form-row,
.book-row,
.book-actions {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.8rem;
}

.list-heading {
  margin-bottom: 1.4rem;
}

.list-heading h2,
.list-heading p {
  margin: 0;
}

.list-heading p,
.filter-control {
  color: #687286;
  font-size: 0.9rem;
}

.filter-control {
  display: grid;
  gap: 0.35rem;
}

.filter-control select,
.book-form input {
  min-height: 42px;
  padding: 0.55rem 0.7rem;
  border: 1px solid #ccd5e4;
  border-radius: 0.6rem;
  background: #fff;
  color: #172033;
}

.book-form {
  display: grid;
  gap: 0.5rem;
  margin-bottom: 1.3rem;
  font-weight: 700;
}

.book-form input {
  flex: 1;
  min-width: 0;
  font-weight: 400;
}

.form-error {
  margin: 0;
  color: #b42318;
  font-size: 0.9rem;
  font-weight: 500;
}

.book-list {
  display: grid;
  gap: 0.65rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.book-row {
  min-height: 68px;
  padding: 0.75rem;
  border: 1px solid #e1e6ef;
  border-radius: 0.8rem;
}

.book-title {
  overflow-wrap: anywhere;
  font-weight: 650;
}

.book-title.finished {
  color: #747e90;
  text-decoration: line-through;
}

.book-actions {
  flex-shrink: 0;
}

.secondary-button {
  background: #e8edff;
  color: #2445b5;
}

.remove-button {
  background: #fff0ed;
  color: #a52d20;
}

.empty-state {
  margin: 1rem 0 0;
  padding: 1.4rem;
  border: 1px dashed #ccd5e4;
  border-radius: 0.8rem;
  color: #687286;
  text-align: center;
}

@media (max-width: 560px) {
  .app-shell {
    padding: 2.5rem 0;
  }

  .list-heading,
  .book-row {
    align-items: stretch;
    flex-direction: column;
  }

  .filter-control {
    width: 100%;
  }

  .form-row,
  .book-actions {
    align-items: stretch;
    flex-direction: column;
  }
}
```

## Try it

1. Add a book with leading and trailing spaces; the saved title is trimmed.
2. Submit an empty or spaces-only title; the form shows an error and focuses the input.
3. Mark a book finished, then switch between all three filters.
4. Remove a book and confirm the finished count updates.
5. Add a new action to `BookAction` but temporarily omit its reducer case; the exhaustive check should produce a type error.

## What this example demonstrates

- Props and callback signatures are declared at component boundaries.
- State is inferred when its initial value is sufficient and explicitly typed when it represents a union.
- The reducer’s action union narrows safely by its discriminant.
- The input ref accounts for the element being absent before it mounts.
- The select’s string value is checked before it becomes a `BookFilter`.
- TypeScript checks the program’s declared shapes; external data would still need runtime validation.
