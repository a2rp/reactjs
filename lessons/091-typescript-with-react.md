# TypeScript with React Components and Hooks

## Learning goals

After this lesson, you should be able to:

- Write React components in `.tsx` files with typed props.
- Let TypeScript infer common JSX, event, and Hook types.
- Describe state that may be empty or in different loading states.
- Type reducers with discriminated action unions.
- Type children, native element props, refs, and context values.
- Recognize where compile-time types stop protecting an application.

## 1. What TypeScript adds to a React project

TypeScript checks JavaScript code against types before it runs. With React, it can catch mistakes in component props, event handling, and state updates, and provide better editor hints.

TypeScript does not change React's runtime behavior. Types are removed when the code is emitted. A type such as `Book` does not validate JSON received from a server while the application is running.

A React Web project needs React's type definitions and a TypeScript configuration that supports the project's JSX transform. Frameworks and build tools often provide setup conventions, so follow the existing project before adding a separate configuration.

## 2. Use `.tsx` for files containing JSX

A TypeScript file containing JSX uses the `.tsx` extension:

```text
ProfileCard.tsx
```

A file with only types, reducers, or utilities can use `.ts`:

```text
profileTypes.ts
profileReducer.ts
```

The TypeScript configuration must enable a JSX mode supported by the project's build tool. Strict type checking is useful because it makes missing and uncertain values visible earlier.

## 3. Give component props a clear object type

Props describe the values a parent may pass to a component. Use an `interface` or a `type` for a non-trivial props object:

```tsx
interface GreetingProps {
  name: string;
  subtitle?: string;
}

function Greeting({ name, subtitle }: GreetingProps) {
  return (
    <section>
      <h2>Hello, {name}</h2>
      {subtitle && <p>{subtitle}</p>}
    </section>
  );
}
```

An optional prop may be omitted, so code should account for `undefined`. A default value can make the component's behavior explicit:

```tsx
interface ResultsProps {
  pageSize?: number;
}

function Results({ pageSize = 20 }: ResultsProps) {
  return <p>Showing up to {pageSize} results.</p>;
}
```

Use either `interface` or `type` consistently. Both can describe object props; `type` is also useful for unions and aliases.

## 4. Use unions for a limited set of choices

When a prop should accept only a few named values, describe that set as a union:

```tsx
type ButtonVariant = "primary" | "secondary" | "danger";

interface ButtonProps {
  variant: ButtonVariant;
  children: React.ReactNode;
}
```

Now `variant="warning"` is caught as a type error rather than silently becoming an unsupported value.

For state with distinct shapes, use a discriminated union:

```tsx
type LoadState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };
```

The `status` property lets TypeScript narrow the available fields:

```tsx
function Message<T>({ state }: { state: LoadState<T> }) {
  switch (state.status) {
    case "idle":
      return <p>Not started.</p>;
    case "loading":
      return <p>Loading…</p>;
    case "success":
      return <p>Loaded successfully.</p>;
    case "error":
      return <p role="alert">{state.error.message}</p>;
  }
}
```

The success case has `data`; the error case has `error`. The union prevents representing contradictory states such as “loading and failed” unless that is explicitly part of the model.

## 5. Type children when the component accepts them

Use React's `ReactNode` type when a component accepts general renderable children:

```tsx
import type { ReactNode } from "react";

interface PanelProps {
  title: string;
  children: ReactNode;
}

function Panel({ title, children }: PanelProps) {
  return (
    <section>
      <h2>{title}</h2>
      {children}
    </section>
  );
}
```

Do not add `children` to a props type if the component does not use children. Be specific when the component needs one particular child shape, but avoid restricting children to `JSX.Element` when text, fragments, or multiple nodes should also be accepted.

## 6. Let JSX infer event types

Inline handlers usually get their event type from the element:

```tsx
<input
  value={name}
  onChange={(event) => setName(event.currentTarget.value)}
/>
```

For a named handler, provide the element-specific React event type when inference cannot determine it:

```tsx
import type { FormEvent } from "react";

function handleSubmit(event: FormEvent<HTMLFormElement>) {
  event.preventDefault();
}
```

The generic identifies the element whose event is being handled. Use the target property that matches the task: `currentTarget` refers to the element with the handler, while `target` can refer to a nested event source.

Common event types include:

- `ChangeEvent<HTMLInputElement>` for a named input-change handler.
- `FormEvent<HTMLFormElement>` for a form submission handler.
- `MouseEvent<HTMLButtonElement>` for a button mouse handler.

You do not need to annotate every inline callback. Let JSX inference do the routine work.

## 7. Type Hook state when inference needs help

When the initial state clearly identifies its type, TypeScript can infer it:

```tsx
const [isOpen, setIsOpen] = useState(false);
// isOpen is boolean
```

Use an explicit type argument when the initial value is not enough to describe future values:

```tsx
const [selectedId, setSelectedId] = useState<string | null>(null);
const [books, setBooks] = useState<Book[]>([]);
```

An empty array alone does not show which item type the array should contain, so type it when inference would be too narrow or unclear.

Avoid overly broad types such as `any`. If a value may be absent, represent that fact with `null` or `undefined` in the type and handle it in the UI.

## 8. Type reducers with action unions

A reducer is a good place to use a discriminated union. Each action type carries exactly the fields it needs:

```tsx
type Action =
  | { type: "book/added"; title: string }
  | { type: "book/toggled"; id: number }
  | { type: "book/removed"; id: number };
```

Type the reducer's state, action, and result:

```tsx
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "book/added":
      return addBook(state, action.title);
    case "book/toggled":
      return toggleBook(state, action.id);
    case "book/removed":
      return removeBook(state, action.id);
    default:
      return assertNever(action);
  }
}

function assertNever(value: never): never {
  throw new Error("Unhandled action");
}
```

If a new action is added but not handled, the `never` check helps TypeScript report the missing case.

## 9. Type context without hiding a missing provider

The context value type is inferred from the value passed to `createContext`. If consumers must be inside a provider, a nullable default plus a checked custom Hook makes that requirement explicit:

```tsx
import { createContext, useContext } from "react";

interface Session {
  userId: string;
  displayName: string;
}

const SessionContext = createContext<Session | null>(null);

function useSession(): Session {
  const session = useContext(SessionContext);

  if (session === null) {
    throw new Error("useSession must be used inside SessionProvider");
  }

  return session;
}
```

This avoids pretending that a context always has a value when it may not. Do not silence the possibility with `as Session` unless the runtime guarantee is enforced elsewhere.

## 10. Type refs by what they reference

A ref to a DOM input can be typed as:

```tsx
const inputRef = useRef<HTMLInputElement | null>(null);
```

The value begins as `null`, because the input has not been attached yet. Check it before use:

```tsx
inputRef.current?.focus();
```

A ref for mutable data is different from a DOM ref. Type it according to the value it stores, and remember that changing a ref does not trigger a render.

## 11. Extend native element props carefully

A reusable component can accept native props for the element it renders. For example:

```tsx
import type { ComponentPropsWithoutRef } from "react";

type ButtonProps = ComponentPropsWithoutRef<"button"> & {
  variant?: "primary" | "secondary";
};

function Button({ variant = "primary", ...buttonProps }: ButtonProps) {
  return (
    <button
      {...buttonProps}
      className={`button button--${variant}`}
    />
  );
}
```

This preserves native properties such as `type`, `disabled`, and `onClick`. Do not forward props that your component consumes as styling or behavior options onto the DOM unless they are valid native attributes.

If you wrap an element but intentionally do not expose refs, `ComponentPropsWithoutRef` makes that choice explicit. Ref APIs vary with React and type-definition versions; use the pattern expected by the installed React version.

## 12. Generic components and Hooks

Generics express relationships between values without losing their specific types. A simple typed list can keep each item's data and render callback aligned:

```tsx
import type { ReactNode } from "react";

interface ListProps<T> {
  items: T[];
  getKey: (item: T) => string | number;
  renderItem: (item: T) => ReactNode;
}

function List<T>({ items, getKey, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={getKey(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

Use generics when the component truly works with different data types. A generic component with too many type parameters can make normal use harder instead of safer.

## 13. Type external data at runtime

A TypeScript annotation does not validate a network response:

```tsx
const book = (await response.json()) as Book;
```

That assertion only tells the compiler to trust the programmer; it does not check the response. Treat data from an API, storage, URL, or user input as untrusted. Parse and validate it at runtime, using checks or a runtime schema validator, before treating it as a `Book`.

Use `unknown` for a value whose shape is not yet known, then narrow it. Avoid `any`, which disables useful checks and can spread through the application.

## 14. Type annotations are not runtime behavior

Types can catch incorrect values during type checking, but they do not:

- Validate server responses.
- Enforce required props at runtime.
- Sanitize user input.
- Prevent a caller from bypassing the type checker.
- Replace accessibility or behavioral tests.

Keep runtime validation for data that can be malformed and runtime UI behavior for states such as loading, empty, and error.

## 15. Common mistakes

- Naming a JSX TypeScript file `.ts` rather than `.tsx`.
- Writing types for values TypeScript already infers clearly.
- Leaving an empty array state without an item type when inference is ambiguous.
- Making every prop optional and checking them all at runtime without a reason.
- Using `any` or broad assertions to suppress useful errors.
- Assuming a type assertion validates JSON.
- Typing all children as a single `JSX.Element` when text or multiple nodes are valid.
- Manually annotating every inline event handler.
- Forgetting that `key` is a React JSX attribute, not a normal prop passed to the component.
- Creating impossible reducer states with unrelated boolean flags.
- Returning a fake default value from Context to silence a possible missing provider.
- Assuming the bundler's transpilation step also performs full type checking.

## Key takeaways

- Use `.tsx` for JSX and `.ts` for TypeScript without JSX.
- Describe component props with object types and use unions for constrained choices.
- Let TypeScript infer common JSX events and Hook values.
- Use explicit types for empty or nullable state, reducers, refs, and context boundaries.
- Treat `unknown` external data as unvalidated until runtime checks establish its shape.
- Types improve correctness and editor support; they do not replace runtime validation.
