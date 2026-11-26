# Context and useContext

## Why context exists

Props make data flow explicit from a parent to its child. But when a value is needed far down a component tree, every intermediate component may have to accept and pass a prop it does not use. This is often called prop drilling.

Context lets a parent provide a value to components below it without forwarding that value through every intermediate component. A descendant reads the value with useContext.

Context is useful for values that many components in a subtree need, such as a visual theme, current account, locale, or a shared state interface. It is not automatically better than props. Props remain easier to trace when only a few nearby components need a value.

## Learning goals

After this lesson, you should be able to:

- explain the problem context solves and when ordinary props are clearer;
- create a context with createContext;
- provide a value to descendants and read it with useContext;
- explain default values, nested providers, and nearest-provider lookup;
- update context by keeping changing data in state;
- understand how changed context values update consumers;
- avoid common provider placement and value identity mistakes.

## Props are explicit data flow

With props, each component that passes information down knows what it is forwarding:

~~~jsx
function App() {
  return <Page theme="dark" />;
}

function Page({ theme }) {
  return <Toolbar theme={theme} />;
}

function Toolbar({ theme }) {
  return <ThemedButton theme={theme} />;
}

function ThemedButton({ theme }) {
  return <button className={"button-" + theme}>Save</button>;
}
~~~

This is clear when each layer uses or meaningfully owns the value. It becomes repetitive when many intermediate components only forward the same prop.

## What context provides

A context object identifies a kind of value. A provider places a value in the tree. Descendant components can read the nearest matching value without receiving it as a prop at each layer.

~~~text
Theme provider
└── Page
    └── Toolbar
        └── ThemedButton reads ThemeContext
~~~

The intermediate components do not need to know about ThemeContext if they do not use it.

Context is scoped to a branch of the React tree. It does not send data to parents, siblings, or components outside the provider subtree.

## The three basic steps

### 1. Create a context

Create a context outside components, usually in a module dedicated to that context:

~~~jsx
import { createContext } from "react";

export const ThemeContext = createContext("light");
~~~

The argument is the default value. React uses it when a component has no matching provider above it. This fallback is static; it is not a place to store changing application state.

Choose a default that is safe and meaningful. If a provider is required, a sentinel such as null can help detect a missing provider:

~~~jsx
export const ThemeContext = createContext(null);
~~~

A custom Hook can then produce a clear error if the provider was forgotten.

### 2. Provide a value

In React 19 and later, render the context object as the provider and pass its value prop:

~~~jsx
<ThemeContext value="dark">
  <Toolbar />
</ThemeContext>
~~~

With React 18 and earlier, use the Provider property:

~~~jsx
<ThemeContext.Provider value="dark">
  <Toolbar />
</ThemeContext.Provider>
~~~

In either form, the provider makes its value available to components rendered inside it.

### 3. Read the value

Call useContext at the top level of a function component or custom Hook:

~~~jsx
import { useContext } from "react";

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return <button className={"button-" + theme}>Save</button>;
}
~~~

React searches upward from ThemedButton and returns the value from the closest matching provider. Any number of intermediary components can sit between the provider and the consumer.

## The closest provider wins

A provider can override a value for one branch of the tree. A nested provider supplies its value to its own descendants:

~~~jsx
<ThemeContext value="dark">
  <MainToolbar />
  <ThemeContext value="light">
    <PrintPreview />
  </ThemeContext>
</ThemeContext>
~~~

MainToolbar reads dark. PrintPreview reads light. Components after the nested provider closes return to the outer value.

Different context objects are independent. A ThemeContext provider does not override a LocaleContext value.

## A provider must be above the component that reads it

useContext searches upward from the component calling it. A provider returned by that same component is below that component and cannot affect its own useContext call:

~~~jsx
function ThemePage() {
  const theme = useContext(ThemeContext);

  return (
    <ThemeContext value="dark">
      <p>Theme read by ThemePage: {theme}</p>
      <Toolbar />
    </ThemeContext>
  );
}
~~~

ThemePage reads the closest provider above itself, or the default if there is none. Toolbar is inside the provider and reads dark.

If a component needs to both own a provider and read the value it provides, split the provider and consumer into separate components:

~~~jsx
function ThemePage() {
  return (
    <ThemeContext value="dark">
      <ThemeContent />
    </ThemeContext>
  );
}

function ThemeContent() {
  const theme = useContext(ThemeContext);
  return <p>Current theme: {theme}</p>;
}
~~~

## Defaults and missing providers

Suppose the context is created with a default of light:

~~~jsx
const ThemeContext = createContext("light");
~~~

If no matching provider is above a consumer, useContext returns light. If a provider exists and passes another value, the consumer gets that provided value.

A provider that passes undefined is still a provider. The consumer receives undefined; React does not substitute the default. The default is only used when there is no matching provider in the ancestor tree.

For a required context, use a sentinel and a custom Hook:

~~~jsx
const AccountContext = createContext(null);

function useAccount() {
  const account = useContext(AccountContext);

  if (account === null) {
    throw new Error("useAccount must be used inside AccountProvider.");
  }

  return account;
}
~~~

This makes provider omissions easier to diagnose than silently returning an incomplete value.

## Context can carry changing values

Context itself does not own state. To update a context value, keep the changing value in state and pass that state to the provider:

~~~jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  function toggleTheme() {
    setTheme((currentTheme) =>
      currentTheme === "light" ? "dark" : "light"
    );
  }

  return (
    <ThemeContext value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext>
  );
}
~~~

A consumer can read both the value and the action:

~~~jsx
function ThemeToggle() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <button onClick={toggleTheme}>
      Switch from {theme}
    </button>
  );
}
~~~

When state in the provider changes, it supplies a new value and consumers of that context receive the update.

## Context value identity and consumer updates

React compares the previous and next provider values with Object.is. If the value is different, React updates components that read that context below the provider.

Primitive values such as strings and booleans are easy to compare:

~~~jsx
<ThemeContext value={theme}>
  {children}
</ThemeContext>
~~~

Objects and functions have identity. This creates a new object on every provider render:

~~~jsx
<SettingsContext value={{ density, setDensity }}>
  {children}
</SettingsContext>
~~~

Even when density has not changed, a parent render creates a new object. Context readers see a different value and may update.

For a provider that re-renders often, use useMemo and useCallback when profiling or design shows that stable value identity is useful:

~~~jsx
function SettingsProvider({ children }) {
  const [density, setDensity] = useState("comfortable");

  const toggleDensity = useCallback(() => {
    setDensity((current) =>
      current === "comfortable" ? "compact" : "comfortable"
    );
  }, []);

  const value = useMemo(
    () => ({ density, toggleDensity }),
    [density, toggleDensity]
  );

  return (
    <SettingsContext value={value}>
      {children}
    </SettingsContext>
  );
}
~~~

Do not add memoization automatically. First keep the provider value simple. Optimize after identifying a real update cost.

Wrapping a context reader in memo does not stop it from receiving a changed context value. Memo can help with props, but a consumer still updates when the context it reads changes.

## Context and state ownership

The provider should usually be near the part of the tree that owns the state. Putting every context at the application root can make ownership unclear and expose values to branches that do not need them.

Use context to make a value available across a subtree. Use state or a reducer to define how that value changes. A provider can combine state and update functions into a context value, but the state still belongs to the provider component or another state owner.

When data and actions change independently or consumers need different subsets, consider separate contexts. For example, a frequently changing cursor position may not belong in the same context value as rarely changing account configuration.

## Context is not a replacement for all props

Use props when:

- one component passes data to a nearby child;
- the value is specific to one component relationship;
- explicit dependencies make the component easier to reuse;
- there are only a few layers and forwarding stays understandable.

Consider context when:

- many descendants in a subtree need the same value;
- intermediate components only forward it;
- the value represents an environment or shared capability for a subtree;
- it would be awkward to pass a stable service or update action through many layers.

Consider component composition when intermediate components can receive already-created JSX as children. This can keep ownership close to the components that know about the data without introducing a global-looking dependency.

## Common use cases

### Theme

Buttons, panels, and menus in a branch can read a shared theme without each intermediate component forwarding a theme prop.

### Current account

Navigation, profile controls, and other distant descendants can read the active account. Keep authorization checks on the server or at the trusted boundary as required; UI context alone is not a security boundary.

### Locale or formatting settings

Distant components can access a locale or formatting preference without every component accepting it as a prop.

### Shared state interface

A provider can expose state and actions from a reducer to a subtree. This is useful when many distant consumers need to read and update related state. Context distributes the interface; the reducer owns the transition logic.

## Common mistakes

### Creating a new context inside a component

Creating a context during render makes a new context object on each render. Define it at module scope so providers and consumers refer to the same object.

### Forgetting the provider

A consumer then receives the context's default value. If the default is null, the code may fail later in a confusing place. A guarded custom Hook can report the missing provider clearly.

### Putting the provider inside the consumer

The consumer cannot read a provider that is returned beneath it. Move the provider above the consumer or split the component into provider and child consumer.

### Expecting the default to replace an undefined provider value

A provider with value undefined is still the closest provider. Its consumers receive undefined. Set an explicit fallback in the provider if that is the intended value.

### Using context for every prop

This hides dependencies and makes components harder to reuse. Prefer ordinary props for local data flow.

### Treating context as state storage

The context object is a channel, not mutable storage. Put changing data in state or a reducer and pass the current value through a provider.

### Passing a new object without noticing its identity

A new object is a new context value even if its fields happen to be equal. Keep values simple and memoize only where the update behavior warrants it.

### Expecting memo to block context updates

A context consumer still receives changes from the context it reads. Split contexts or restructure provider values if updates need a narrower scope.

### Defining multiple unrelated values in one broad context

Unrelated changes can update consumers that only need one part. Separate contexts when values have meaningfully different ownership or update patterns.

## Practice

1. What problem does context solve?
2. Does context send a value to a component's parent or sibling?
3. What does createContext return?
4. When is a context's default value used?
5. What happens if a matching provider passes undefined?
6. Which provider value does a consumer receive when several matching providers are nested?
7. Can a component read a provider that it returns from its own render?
8. How can a component both provide and read a value?
9. Does context itself own or update state?
10. How can a context value change over time?
11. How does React compare old and new provider values?
12. Why can a new object in a provider value trigger consumer updates?
13. Does memo block a consumer from receiving a changed context value?
14. When are ordinary props clearer than context?
15. How can a custom Hook make a required provider easier to debug?
16. Why should a context object generally be created outside components?
17. When might two separate contexts be preferable to one large context?
18. Why is a context provider not a security boundary?

## Recap

- Context provides a value to descendants without forwarding it through every intermediate component.
- Create a context once, provide its value above consumers, and read it with useContext.
- The nearest matching provider wins; the default is used only when there is no provider above.
- A provider passing undefined does not cause React to use the default.
- Context distributes a value; state or a reducer owns changes to that value.
- React updates context consumers when the provider value changes according to Object.is.
- Prefer props for local data flow and use context for shared subtree needs.
- In React 19, render the context object as a provider; earlier versions use its Provider property.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="049-rules-of-hooks.md">049 - Rules of Hooks and Why They Matter</a></td><td><a href="../qnas/050-context-and-usecontext.md">Q&amp;A</a></td><td><a href="../examples/050-context-and-usecontext.md">Example</a></td><td><a href="051-context-provider-design.md">051 - Designing Context Providers and Avoiding Overuse</a></td></tr>
  </tbody>
</table>
