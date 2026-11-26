# TypeScript with React: Questions and Answers

## Concepts and setup

### 1. Why use TypeScript with React?
TypeScript checks many mistakes while code is being edited or built. It can catch incompatible props, incorrect event values, and incomplete reducer actions before the application runs. It does not validate data received at runtime by itself.

### 2. What is the difference between `.ts` and `.tsx`?
Use `.tsx` for TypeScript files that contain JSX. Use `.ts` for TypeScript files without JSX, such as data types, reducers, and utility functions.

### 3. Does TypeScript replace React?
No. React still renders and updates the interface. TypeScript adds a static type checker and is removed during the build process.

### 4. What does `@types/react` provide?
It provides TypeScript declarations for React APIs and JSX. React TypeScript projects generally use the React type packages provided by their setup or installed as development dependencies.

### 5. What does the `jsx` compiler option do?
It tells TypeScript how to process JSX syntax. A React project’s tooling normally configures this for the selected runtime and build system.

## Props and components

### 6. How do I describe a component’s props?
Declare an object type or interface and use it for the component parameter:

```tsx
type GreetingProps = {
  name: string;
  enthusiastic?: boolean;
};

function Greeting({ name, enthusiastic = false }: GreetingProps) {
  return <p>{enthusiastic ? `Hello, ${name}!` : `Hello, ${name}`}</p>;
}
```

### 7. Is `type` or `interface` better for props?
Both can describe object-shaped props. Choose the form that fits the project’s conventions. `type` is convenient for unions and intersections; interfaces can be extended and merged.

### 8. How do I make a prop optional?
Add `?` to its property name, then provide a default or handle the value being absent:

```ts
type PanelProps = {
  title: string;
  compact?: boolean;
};
```

### 9. What type should I use for `children`?
Use `ReactNode` when the component accepts typical renderable React content, such as elements, strings, numbers, or arrays of content:

```tsx
import type { ReactNode } from "react";

type CardProps = { children: ReactNode };
```

Use a narrower type when the component intentionally accepts only a specific child shape.

### 10. Should every function component use `React.FC`?
No. A function with typed props is enough. `React.FC` is an available type, but it does not need to be added to every component and does not make optional props required. Type the props directly when that is clearer.

### 11. Can TypeScript infer a component’s return type?
Usually. JSX tooling understands the returned JSX, so explicitly writing a return type is optional in most components. Add one when it improves clarity or enforces a specific API.

### 12. How do I type a callback prop?
Describe the arguments and return value in the props type:

```ts
type SaveButtonProps = {
  onSave: (id: number) => void;
};
```

Use `void` when the caller does not need a returned value.

## State, events, and refs

### 13. Does `useState` infer the state type?
Often. For a value such as `useState(0)`, TypeScript infers a numeric state. Add a type argument when the initial value is `null`, an empty collection, or one case in a wider union.

### 14. How do I type state that can be one of several values?
Use a union:

```ts
type Status = "idle" | "loading" | "success" | "error";
const [status, setStatus] = useState<Status>("idle");
```

This prevents unrelated strings from being assigned accidentally.

### 15. Why can an empty array cause inference problems?
An empty array provides no element from which TypeScript can infer the intended item type. State the type explicitly:

```ts
type Message = { id: number; text: string };
const [messages, setMessages] = useState<Message[]>([]);
```

### 16. How should I type an input change event?
React provides event types such as `ChangeEvent<HTMLInputElement>`. JSX often infers the event type from the element and handler position. Add an explicit type when a handler is declared separately:

```tsx
import type { ChangeEvent } from "react";

function handleChange(event: ChangeEvent<HTMLInputElement>) {
  console.log(event.currentTarget.value);
}
```

### 17. What is the difference between `target` and `currentTarget`?
`target` is the element where the event originated and can be a descendant. `currentTarget` is the element whose handler is running. When reading a known input’s value, `currentTarget` is often the clearer choice.

### 18. How do I type a form submit handler?
Use `FormEvent<HTMLFormElement>` for a named form handler. Call `preventDefault()` if the browser’s normal form submission should not occur.

### 19. How do I type a ref to an input element?
Use the corresponding DOM element type and include `null` when the ref starts without a mounted element:

```tsx
const inputRef = useRef<HTMLInputElement | null>(null);
```

Check that `current` is non-null before using the element.

### 20. Should I use state or a ref for a value?
Use state when changing the value should update rendered UI. Use a ref for a mutable value that should persist between renders without itself triggering a render, or for access to a DOM node.

## Reducers, context, and reusable types

### 21. Why are discriminated unions useful for reducer actions?
An action union with a shared literal `type` field lets TypeScript narrow the action to the matching shape in each branch:

```ts
type Action =
  | { type: "item/added"; title: string }
  | { type: "item/removed"; id: number };
```

### 22. How can I make a reducer exhaustive?
Use a `never` check after handling every action type. If a new action is added but not handled, the type checker can point to the missing branch:

```ts
function assertNever(value: never): never {
  throw new Error(`Unexpected action: ${String(value)}`);
}
```

### 23. How do I type a React context?
Give `createContext` the actual value type. If no valid default exists, use a nullable context and expose it through a custom hook that throws a clear error when used outside its provider.

### 24. Why is `createContext(null)` sometimes a problem?
Without a type argument, TypeScript may infer a context that only accepts `null`. Declare the intended context value type, such as `User | null`, so the provider and consumers have the correct contract.

### 25. How can I reuse props from a native element?
React exports utility types such as `ComponentPropsWithoutRef<"button">`. They let a wrapper accept standard button properties while excluding a ref when the wrapper does not forward one. Use the matching utility for the element and ref behavior you support.

### 26. Is the `key` prop available inside my component?
No. React consumes `key` when reconciling a list. If the component needs the same identifier as data, pass it as a separate prop, for example `itemId`.

### 27. When should I make a component generic?
Use a generic when a reusable component or helper should preserve a relationship between an input type and its output, such as a list callback receiving the same item type used by the list. Avoid generics when the component only has one concrete data shape.

## Safety and workflow

### 28. Is `any` the same as `unknown`?
No. `any` disables many checks for that value. `unknown` can hold any value, but code must narrow or validate it before using it as a specific type. Prefer `unknown` for untrusted input.

### 29. Does a type assertion validate JSON from an API?
No. An assertion such as `payload as User` only changes what the compiler believes. It does not inspect the value at runtime. Validate external data before relying on its shape.

### 30. Does TypeScript make runtime errors impossible?
No. TypeScript catches many statically visible problems, but it cannot prevent every runtime failure, network error, invalid external value, or browser limitation.

### 31. What is the difference between transpiling and type-checking?
Transpiling converts TypeScript and JSX to JavaScript. Some build tools transpile without checking all types. Run the project’s type-check command as part of development or continuous integration.

### 32. What does `strict` mode do?
It enables a set of stronger checks, including stricter treatment of nullable values and function types. Exact behavior is controlled by the TypeScript compiler options in the project.

### 33. What should I do when a library has no type declarations?
Check whether the library supplies its own declarations or has a maintained declaration package. If necessary, add a small local declaration that accurately describes the API you use; do not silence type errors broadly without understanding them.

### 34. Why does a callback sometimes get an implicit `any` error?
The callback may be outside a context where TypeScript can infer its parameter type, or the surrounding value may itself be untyped. Type the callback parameter or improve the type of the value that supplies it.

### 35. What should I check when JSX gives a type error?
Check the file extension (`.tsx` for JSX), JSX compiler configuration, React declarations, component prop names, and whether the component is imported and used with the expected type.

## Practical review

### 36. What is a good way to type a component API?
Start with the smallest public contract: required and optional props, callback signatures, and renderable children if needed. Keep internal state types near the state logic and use unions for finite states and actions.

### 37. When should I avoid adding a type annotation?
When the type is already inferred clearly at the point of use, extra annotations can repeat information. Add annotations where they establish an API boundary, clarify a non-obvious union, or fix an inference gap.

### 38. How do I safely read an optional value?
Use a guard, optional chaining, or a default appropriate to the behavior. Do not use the non-null assertion operator just to suppress a warning unless the invariant is actually guaranteed.

### 39. Can React props be mutated?
Treat props as read-only inputs. Create new values when transforming data and use state setters or reducer actions when changing component-owned state.

### 40. What should I learn next?
Practice typing forms and reusable components, then move on to data fetching, runtime validation, testing typed UI behavior, and application-wide state patterns.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="090-global-state-and-usesyncexternalstore.md">090 - Global State, `useSyncExternalStore`, and External Stores - Q&amp;A</a></td><td><a href="../lessons/091-typescript-with-react.md">Lesson</a></td><td><a href="../examples/091-typescript-with-react.md">Example</a></td><td><a href="092-styling-and-theming.md">092 - Styling Strategies, CSS Modules, and Theming: Questions and Answers</a></td></tr>
  </tbody>
</table>
