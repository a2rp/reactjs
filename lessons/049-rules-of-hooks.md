# Rules of Hooks and Why They Matter

## Why React has Hook rules

Hooks let function components use state, context, Effects, and other React features. React needs to know which Hook call belongs to which piece of state or behavior on every render. A component must therefore call its Hooks in a predictable order while React renders it.

The rules are not style preferences. They let React associate Hook calls with the correct component instance and let lint tools and the React Compiler analyze component logic.

## Learning goals

After this lesson, you should be able to:

- call Hooks only from function components and custom Hooks;
- keep ordinary Hook calls at the top level and in the same order on every render;
- recognize invalid calls in conditions, loops, callbacks, event handlers, classes, and module scope;
- move conditions inside a Hook or split conditional UI into components;
- use custom Hooks without confusing shared logic with shared state;
- recognize the special rules for React's use API.

## What counts as a Hook?

A Hook is a React API used to access React features from a function component or another Hook. Built-in examples include useState, useReducer, useContext, useEffect, useMemo, useRef, and useId.

A custom Hook is an ordinary JavaScript function that uses one or more Hooks to package reusable React logic. Its name starts with use followed by a capital letter, such as useOnlineStatus or useSearchFilter.

~~~jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  function increment() {
    setCount((current) => current + 1);
  }

  return { count, increment };
}
~~~

The name is a convention that makes Hook usage visible to readers and static analysis tools. A regular helper that does not call Hooks should normally use an ordinary name, such as getSortedItems.

## Rule 1: Call Hooks only from React functions

Call ordinary Hooks from:

1. the top level of a React function component; or
2. the top level of a custom Hook.

Do not call them from regular utility functions, event handlers, class components, module-level code, or functions passed to other Hooks.

~~~jsx
function SearchBox() {
  const [query, setQuery] = useState("");

  function handleChange(event) {
    setQuery(event.target.value);
  }

  return <input value={query} onChange={handleChange} />;
}
~~~

Calling the state setter in handleChange is correct. The Hook call itself remains in SearchBox while React renders it.

This is invalid:

~~~jsx
function SearchBox() {
  function handleChange() {
    const [query, setQuery] = useState("");
  }

  return <button onClick={handleChange}>Search</button>;
}
~~~

An event handler runs in response to an interaction, outside the render phase. React cannot attach a new Hook call to the component's render-time Hook sequence there.

### Hooks cannot be called from regular functions

A regular helper can calculate and return data, but it cannot call Hooks:

~~~jsx
function getCurrentTheme() {
  const theme = useContext(ThemeContext);
  return theme;
}
~~~

Rename and structure it as a custom Hook, then call it from a component or another custom Hook:

~~~jsx
function useCurrentTheme() {
  return useContext(ThemeContext);
}

function Toolbar() {
  const theme = useCurrentTheme();
  return <div className={theme}>Toolbar</div>;
}
~~~

### Hooks cannot be called from class components

Hooks are for function components and custom Hooks. In a class component, use the class APIs supported by that component model or convert it to a function component when appropriate.

### Do not call a component as an ordinary function

React must control when a component runs so it can track identity, state, and rendering. Render a component through JSX:

~~~jsx
function Page() {
  return <ProfileCard />;
}
~~~

Do not call it directly:

~~~jsx
function Page() {
  return ProfileCard();
}
~~~

A direct function call makes ProfileCard an ordinary part of Page's execution rather than an independently rendered component. React cannot manage it as a separate component boundary.

## Rule 2: Call ordinary Hooks at the top level

Call Hooks at the top level of a component or custom Hook, before any conditional early return. Call the same ordinary Hooks in the same order on every render.

~~~jsx
function Dashboard({ user }) {
  const [activeTab, setActiveTab] = useState("overview");
  const theme = useContext(ThemeContext);
  const isOnline = useOnlineStatus();

  if (!user) {
    return <p>Please sign in.</p>;
  }

  return (
    <main className={theme}>
      <h1>{user.name}</h1>
      <p>{isOnline ? "Online" : "Offline"}</p>
      <button onClick={() => setActiveTab("settings")}>
        Settings
      </button>
      <p>Selected tab: {activeTab}</p>
    </main>
  );
}
~~~

All three Hooks run before the early return, whether or not user exists. The return may be conditional; Hook calls before it are still made in a stable order.

## Why conditional Hook calls break

React associates the results of ordinary Hook calls with their positions in the sequence for a component instance. The exact internal representation is an implementation detail, but the important rule is that each render must present the same ordinary Hook call sequence.

This conditional call is invalid:

~~~jsx
function Profile({ isAdmin }) {
  if (isAdmin) {
    const [permissions, setPermissions] = useState([]);
  }

  const [name, setName] = useState("");
  return <p>{name}</p>;
}
~~~

On one render, React sees useState for permissions followed by useState for name. On another, it sees only useState for name. A Hook result can then be associated with the wrong call, and React may report that more or fewer Hooks were rendered than expected.

Call the Hook unconditionally and make its initial value conditional:

~~~jsx
function Profile({ isAdmin }) {
  const [permissions, setPermissions] = useState(() =>
    isAdmin ? ["read", "write"] : ["read"]
  );
  const [name, setName] = useState("");

  return <p>{name} has {permissions.length} permissions.</p>;
}
~~~

Remember that the initializer runs when this component instance is created. If isAdmin changes later and permissions should be recalculated, define an explicit state update or ownership strategy rather than expecting the initializer to run again.

## Conditions belong inside the Hook's logic

If an Effect should perform work only under a condition, call useEffect at the top level and put the condition inside its callback:

~~~jsx
function UserPanel({ isSignedIn, userId }) {
  useEffect(() => {
    if (!isSignedIn) {
      return;
    }

    const subscription = subscribeToUser(userId);

    return () => subscription.unsubscribe();
  }, [isSignedIn, userId]);

  return <p>{isSignedIn ? "Account details" : "Sign in to continue"}</p>;
}
~~~

Do not conditionally call the Effect:

~~~jsx
function UserPanel({ isSignedIn, userId }) {
  if (isSignedIn) {
    useEffect(() => {
      const subscription = subscribeToUser(userId);
      return () => subscription.unsubscribe();
    }, [userId]);
  }

  return <p>Account details</p>;
}
~~~

The conditional version changes the Hook sequence. The valid version always calls useEffect; the Effect callback decides whether to start a subscription and its cleanup releases it when dependencies change or the component unmounts.

## Put different Hook-using UI in different components

Do not call a Hook only for one branch of a component. Move that branch into a child component so React can mount or unmount the child as a whole:

~~~jsx
function AccountPage({ user }) {
  return user ? <AccountDetails user={user} /> : <SignInMessage />;
}

function AccountDetails({ user }) {
  const [expanded, setExpanded] = useState(false);

  return (
    <section>
      <h2>{user.name}</h2>
      <button onClick={() => setExpanded((value) => !value)}>
        {expanded ? "Show less" : "Show more"}
      </button>
      {expanded && <p>{user.biography}</p>}
    </section>
  );
}
~~~

AccountDetails owns its Hook calls. When the user is absent, React does not render that child; when a user becomes available, React mounts it as a component. The Hook sequence within each mounted component remains valid.

## Hooks cannot be called in loops

This is invalid because the number of Hook calls can change with the array length:

~~~jsx
function ItemList({ items }) {
  const itemStates = items.map((item) => useState(item.selected));
  return <p>{itemStates.length} items</p>;
}
~~~

Use a component for each item instead:

~~~jsx
function ItemList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <ItemRow key={item.id} item={item} />
      ))}
    </ul>
  );
}

function ItemRow({ item }) {
  const [selected, setSelected] = useState(item.selected);

  return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={selected}
          onChange={(event) => setSelected(event.target.checked)}
        />
        {item.name}
      </label>
    </li>
  );
}
~~~

Each ItemRow is a separate component instance with its own stable Hook sequence. A key lets React match each row with its item when the collection changes.

## Hooks cannot be called inside callbacks

Do not call a Hook inside a callback passed to useMemo, useReducer, useEffect, or another function:

~~~jsx
const theme = useMemo(() => useContext(ThemeContext), []);
~~~

Instead, call the Hook at the top level, then use its result:

~~~jsx
const theme = useContext(ThemeContext);
const style = useMemo(() => createStyle(theme), [theme]);
~~~

For an Effect, call useEffect at the top level and use ordinary functions inside its callback. Do not call another Hook from that callback.

## Hooks cannot be called in async functions or try/catch blocks

An async function, event callback, or error-handling block is not a component render or a custom Hook call. Do not place ordinary Hook calls there.

~~~jsx
async function loadProfile() {
  const [profile, setProfile] = useState(null);
  // Invalid: this function runs asynchronously, not as a component render.
}
~~~

Instead, declare state in the component and start asynchronous work from an event handler or Effect, then update the declared state with its setter.

~~~jsx
function ProfileLoader() {
  const [profile, setProfile] = useState(null);

  async function handleLoad() {
    const nextProfile = await fetchProfile();
    setProfile(nextProfile);
  }

  return (
    <button onClick={handleLoad}>
      {profile ? profile.name : "Load profile"}
    </button>
  );
}
~~~

Do not call ordinary Hooks inside try, catch, or finally. Keep Hook calls outside those blocks and handle failures in normal JavaScript code at the appropriate level.

## Custom Hooks follow the same rules

A custom Hook is not a loophole. Its own Hook calls must also stay at the top level and in a stable order. A custom Hook can call other Hooks because React reaches it through a component render or another custom Hook.

~~~jsx
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
    }

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return width;
}
~~~

Call it unconditionally from a component:

~~~jsx
function ResponsiveMessage() {
  const width = useWindowWidth();

  return <p>{width < 600 ? "Narrow layout" : "Wide layout"}</p>;
}
~~~

If a behavior should be conditional, pass the condition as data to the custom Hook and let it handle that condition internally. Do not call the custom Hook from an if statement.

Each call to a custom Hook has its own state. Calling useWindowWidth in two components shares the logic, not one shared width state. To share the state itself, lift it to a common owner or use an appropriate shared store.

## The special use API exception

React's use API is a special case. Unlike ordinary Hooks such as useState and useEffect, use may be called conditionally and inside loops. It must still be called from a component or custom Hook and cannot be called inside try/catch/finally.

Do not generalize this exception to names such as useState, useContext, useEffect, or custom Hooks like useOnlineStatus. Those ordinary Hook calls still follow the top-level and same-order rules.

~~~jsx
function Message({ promise, shouldShow }) {
  if (shouldShow) {
    const message = use(promise);
    return <p>{message}</p>;
  }

  return <p>Message hidden</p>;
}
~~~

This example applies only when using React's use API with a supported resource. It is not a pattern for conditionally calling other Hooks.

## Let tooling catch violations

The React Hooks ESLint plugin can flag many invalid Hook calls, including conditional calls, calls after early returns, and calls from callbacks. Keep the plugin enabled and treat its warnings as useful feedback about component structure.

Linting does not replace understanding the rules. If a Hook call is hidden inside an ambiguously named function, static analysis may not recognize the intended custom Hook boundary. Use clear component names and the use prefix for actual custom Hooks.

## Common mistakes

### Calling useState from a click handler

Declare state while rendering the component. In the handler, call the state setter to update it.

### Calling an Effect only when a prop is true

Call the Effect unconditionally and branch inside its callback. Include reactive values that the Effect reads in its dependency list.

### Putting a Hook after an early return

Move ordinary Hook calls above the conditional return so every render reaches them in the same order.

### Calling a Hook for each array item

Render a child component for each item and put the Hook in the child. Give each child a stable key.

### Hiding Hooks inside ordinary helper names

If a function uses Hooks, name it as a custom Hook and call it only from a component or another custom Hook. If it does not use Hooks, keep it a normal helper.

### Treating every function starting with use as a Hook

The prefix is a convention that signals Hook behavior. Do not add it to a helper that does not use Hooks without a clear reason; doing so restricts where callers can safely use that helper.

### Treating the special use API as permission for other Hooks

The conditional and loop exception applies specifically to React's use API. It does not apply to useState, useEffect, or custom Hooks.

## Practice

1. Why must ordinary Hook calls appear in the same order on every render?
2. Where may a Hook be called?
3. Is calling a state setter from an event handler allowed?
4. Why is calling useState from inside an event handler invalid?
5. What should happen to an Effect that only needs to subscribe when a condition is true?
6. Why should Hooks appear before conditional early returns?
7. How can a list render one stateful Hook call per item correctly?
8. Why is a custom Hook not allowed inside a condition?
9. Can a custom Hook call another custom Hook?
10. Does calling the same custom Hook in two components share its state?
11. Why should a component be rendered as JSX instead of called like an ordinary function?
12. Can React's use API be called in a condition?
13. Does the use API exception apply to useEffect?
14. Which React lint plugin helps find Hook rule violations?

## Recap

- Call ordinary Hooks only while React renders a function component or a custom Hook.
- Call ordinary Hooks at the top level, in the same order on every render, before any early return.
- Conditions may control the logic inside an Effect or custom Hook, but must not control whether an ordinary Hook is called.
- Put Hook-using repeated or conditional UI in separate child components.
- Custom Hooks share reusable logic; each call has independent state.
- React's use API has a limited conditional and loop exception; ordinary Hooks do not.
- Use the React Hooks ESLint plugin to help detect mistakes.
