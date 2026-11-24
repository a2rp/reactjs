# Designing Context Providers and Avoiding Overuse

## Context is a boundary, not a default transport

Context makes values available to descendants without forwarding them through every intermediate component. That is useful when many distant components need the same value. It can also hide dependencies and make updates harder to trace if it is used for every prop.

Good context design answers four questions:

1. Which part of the tree needs this value?
2. Which component owns and updates it?
3. Which components should be able to read or change it?
4. How will consumers detect that the provider is missing?

This lesson builds on context and useContext by focusing on where providers belong, how to expose a small API, and how to avoid turning context into an unstructured application-wide store.

## Learning goals

After this lesson, you should be able to:

- choose between props, composition, and context based on the data flow;
- place a provider near the smallest subtree that needs its value;
- encapsulate context creation and access behind a provider and custom Hooks;
- use a sentinel default and report missing required providers clearly;
- separate frequently changing state from stable actions when useful;
- combine context with useReducer for shared state transitions;
- reason about context value identity and update scope.

## Start with props and composition

Props are explicit. A component's inputs can be understood by reading its function signature:

~~~jsx
function TaskRow({ task, onToggle }) {
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={() => onToggle(task.id)}
      />
      {task.text}
    </label>
  );
}
~~~

Passing props through a few layers is often a good trade. It documents which component uses which value and keeps a component easy to reuse.

Composition can avoid forwarding data through layout-only components. A component can accept already-created content as children:

~~~jsx
function PageLayout({ children }) {
  return <main className="page-layout">{children}</main>;
}

function TaskPage({ tasks }) {
  return (
    <PageLayout>
      <TaskList tasks={tasks} />
    </PageLayout>
  );
}
~~~

PageLayout does not need to know about tasks. If explicit props and composition both remain awkward because many distant descendants need the same shared value, context may be a good fit.

## Place the provider near its consumers

A provider should usually wrap the smallest subtree that needs its value and include the component that owns the state.

~~~jsx
function App() {
  return (
    <>
      <PublicHeader />
      <TaskProvider>
        <TaskWorkspace />
      </TaskProvider>
      <PublicFooter />
    </>
  );
}
~~~

Here the task state is available within TaskWorkspace without becoming a dependency of the public header and footer.

A provider can live higher when the whole application needs its value, such as a site-wide theme or routing state. Avoid placing every provider at the root simply because it is possible. Provider scope is part of the design: it says which part of the UI shares that value.

## Give each context a clear purpose

Prefer names that communicate the kind of value:

~~~jsx
const ThemeContext = createContext(null);
const AccountContext = createContext(null);
const TasksStateContext = createContext(undefined);
const TasksDispatchContext = createContext(undefined);
~~~

A context should not become a bag of unrelated application data. Group values that share ownership, meaning, and update patterns. Separate values when different subtrees need different providers or update rates.

Create context objects at module scope. A context object is an identity shared by its provider and consumers; recreating it during render breaks that relationship.

## Encapsulate context behind a provider and custom Hooks

Keep context details in one module instead of making every component import the raw context:

~~~jsx
const TasksContext = createContext(undefined);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={{ tasks, dispatch }}>
      {children}
    </TasksContext.Provider>
  );
}

export function useTasks() {
  const value = useContext(TasksContext);

  if (value === undefined) {
    throw new Error("useTasks must be used inside TasksProvider.");
  }

  return value;
}
~~~

The provider owns the state and exposes it through a deliberate boundary. Consumers call useTasks instead of knowing how the context was created.

For React 19 and later, the context object can be rendered directly as the provider. The Provider property form shown above also works in React 19 and supports earlier React versions.

### Treat a required provider as required

If a context only makes sense inside a provider, use a sentinel default such as undefined or null, then check it in the custom Hook. A useful error points directly to the missing provider.

A meaningful default is appropriate when a consumer can work correctly outside a provider. For example, a standalone component might use a default theme of light. The default value is static; it is not the state that updates the application.

## Separate state from actions when it helps

A reducer provides both current state and a dispatch function:

~~~jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
~~~

You can provide them through separate contexts:

~~~jsx
const TasksStateContext = createContext(undefined);
const TasksDispatchContext = createContext(undefined);
~~~

~~~jsx
<TasksStateContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    {children}
  </TasksDispatchContext.Provider>
</TasksStateContext.Provider>
~~~

Components that only dispatch actions need not subscribe to the changing tasks value. Components that render the list can read the state context. The dispatch function returned by useReducer has stable identity.

Separate contexts do not prevent every render. A component can still render because its parent renders with changed props or children. They do make the context dependencies more focused and avoid sending a new state value to dispatch-only consumers.

This pattern is useful when it clarifies ownership or reduces unnecessary context updates. For a small feature with one or two consumers, a single context or ordinary props may be simpler.

## Keep transitions in the state owner

Context distributes state and actions; it does not decide how state changes. A reducer can define valid transitions in one place:

~~~jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case "task/added":
      return [...tasks, action.task];

    case "task/toggled":
      return tasks.map((task) =>
        task.id === action.id
          ? { ...task, done: !task.done }
          : task
      );

    case "task/deleted":
      return tasks.filter((task) => task.id !== action.id);

    default:
      throw new Error("Unknown action: " + action.type);
  }
}
~~~

The provider calls useReducer and publishes tasks and dispatch. Deep descendants can dispatch an action without passing event handlers through each intermediate component. The reducer remains the state-transition boundary.

Avoid moving every state value into a reducer and context automatically. Local input state can stay in the input component. Use shared context only when multiple descendants need that state or its actions.

## Context value identity and update design

React compares a provider's previous and next values with Object.is. A new object literal is a different value, even if its fields appear unchanged:

~~~jsx
<SettingsContext.Provider value={{ density, setDensity }}>
  {children}
</SettingsContext.Provider>
~~~

If the provider renders often, useCallback and useMemo can keep a function and object stable when their underlying data is unchanged:

~~~jsx
const updateDensity = useCallback((nextDensity) => {
  setDensity(nextDensity);
}, []);

const settings = useMemo(
  () => ({ density, updateDensity }),
  [density, updateDensity]
);
~~~

Treat these as performance tools, not requirements for every provider. Start with clear ownership and a simple value. Measure or identify avoidable updates before adding memoization.

Components that read a context receive updates when that context value changes. Wrapping a consumer in memo does not block new context values. If a component needs only one part of a large context, consider splitting contexts or using a small outer context-reading component that passes the needed value as a prop to a memoized child.

## Avoid making context a global mutable store

A context provider belongs to a particular rendered tree and owns values for its descendants. Do not mutate a module-level object to imitate shared state. Such mutation bypasses React's update model and can make UI changes unpredictable.

Context also is not a cache, server-state library, or security boundary. It can expose a state interface to a subtree, while specialized tools or trusted services handle concerns such as remote data caching and authorization.

## A provider can combine multiple contexts

A feature provider can keep state and action contexts together behind one wrapper:

~~~jsx
function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksStateContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksStateContext.Provider>
  );
}
~~~

Consumers do not need to know that two contexts exist:

~~~jsx
function useTasks() {
  const tasks = useContext(TasksStateContext);

  if (tasks === undefined) {
    throw new Error("useTasks must be used inside TasksProvider.");
  }

  return tasks;
}

function useTasksDispatch() {
  const dispatch = useContext(TasksDispatchContext);

  if (dispatch === undefined) {
    throw new Error("useTasksDispatch must be used inside TasksProvider.");
  }

  return dispatch;
}
~~~

This keeps call sites simple while preserving separate read and action subscriptions.

If several app-level providers are needed, a small AppProviders component can compose them. Keep that wrapper organized by feature and avoid turning it into a dumping ground for unrelated state.

## Common overuse patterns

### Putting every prop into context

A value used by one child is usually easier to pass directly. Context can hide where a dependency came from and couple a reusable component to a particular provider.

### Creating one enormous context

A broad object can make unrelated consumers update together and makes ownership unclear. Divide context by feature or update needs when the boundary is meaningful.

### Moving local state to the application root

A text field used by one form does not need application-wide context. Keep state close to the components that use and update it.

### Treating dispatch and state as the same subscription

A component that only sends actions may not need to read the changing state. Separate contexts when that distinction makes dependencies clearer or improves update behavior.

### Memoizing every provider immediately

Memoization adds code and dependency maintenance. Use it when object/function identity causes avoidable work, not as a default decoration.

### Using context to hide poor component boundaries

If layout components only forward props, consider passing children or extracting a component closer to the data owner. Context may not be needed.

### Assuming context is inherently global

A provider only affects its descendant branch. Use a provider's location to express the actual sharing scope.

### Using context as a security check

Client-side context can affect what a user sees, but trusted server code must enforce access rules.

## Practice

1. When are props clearer than context?
2. How can children composition avoid forwarding a prop through a layout component?
3. Where should a feature provider usually be placed?
4. Why should context be created outside components?
5. What does a custom context Hook add to the provider API?
6. When is a sentinel default useful?
7. How can state and dispatch be exposed through separate contexts?
8. Why can a dispatch-only component avoid subscribing to task state?
9. Do separate contexts prevent all ordinary React renders?
10. Why does a reducer still matter when state is exposed through context?
11. What can cause object-valued context to update consumers?
12. Does memo block a context consumer from receiving a changed value?
13. When might an AppProviders component be useful?
14. Why should one context not become an unstructured app-wide data bag?
15. Is client-side context an authorization boundary?
16. Which state should usually stay local to a form input?

## Recap

- Prefer props and composition until context solves a real sharing problem.
- Put a provider around the smallest subtree that needs its value.
- Encapsulate context behind a feature provider and custom Hooks.
- Use a sentinel and a clear error when a provider is required.
- Context distributes values; state and reducers define how they change.
- Separate state and dispatch contexts when it makes subscriptions clearer.
- Split contexts by ownership and update needs, not by arbitrary rules.
- Optimize context value identity when it matters, and keep local state local.
