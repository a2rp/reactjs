# React DevTools, Debugging, and Common Warnings

## Learning goals

After this lesson, you should be able to:

- Use React Developer Tools to inspect a component tree and its current inputs.
- Record and interpret a React Profiler session without guessing at performance problems.
- Follow an error from the browser console to the responsible component and source code.
- Recognize common React warnings and correct their underlying causes.
- Understand why Strict Mode may render components or run Effects more than once in development.
- Know which errors an Error Boundary can and cannot catch.
- Avoid hiding warnings instead of fixing the code that caused them.

## 1. React DevTools and browser DevTools answer different questions

React Developer Tools adds two panels to browser developer tools:

- **Components** shows the React component tree and lets you inspect the selected component's props, state, context, and Hooks.
- **Profiler** records React rendering work so you can find which components rendered during an interaction and which commits took longer.

The browser's built-in developer tools answer related questions about the DOM, styles, JavaScript execution, requests, storage, and layout. Use both. A React component tree is not the DOM tree: a component can return several DOM nodes, no DOM node, or another component.

Install the React Developer Tools browser extension for a supported browser. Keep the extension and application versions compatible. A standalone `react-devtools` package is also available for setups where an extension is not practical.

## 2. Inspect a component with the Components panel

Open the page, open browser developer tools, and select **Components**. Find the component in the React tree, then inspect its current props, state, context, and Hooks. Selecting a component can also help locate the corresponding rendered UI in the page.

Use the panel to answer a concrete question:

- Did the parent pass the value that this component expects?
- Is state still the old value, or did the event update it?
- Which context value did the component receive?
- Is the component mounted more than once?
- Which parent rendered the component with this prop?

DevTools is useful for examining the current render. It does not replace following the application logic, and editing an inspected value is a temporary debugging action, not a change to the source or a persisted update.

## 3. Profile an interaction before optimizing

Use the Profiler panel when an interaction feels slow or when a component renders more often than expected:

1. Reproduce the specific interaction in the app.
2. Start recording in **Profiler**.
3. Perform the interaction once or a small number of times.
4. Stop the recording and select the commit related to that interaction.
5. Inspect which components rendered and compare their work.
6. Trace the cause before changing memoization or state placement.

A **commit** is a batch of React work that React applied to the screen. The flamegraph helps identify components that participated in a selected commit; the ranked view helps find expensive components. A component rendering does not automatically mean there is a performance bug. It may need to render because its props or state changed.

Profile the work you care about and compare before and after in the same environment. Development instrumentation and Strict Mode can add extra work. For browser-wide performance, also inspect the browser Performance panel: React Profiler focuses on React rendering and does not explain every layout, paint, script, or network cost. Profiling instrumentation has overhead, and the standard production build does not include all profiling data by default.

React also exposes a `<Profiler>` component for programmatic measurements around part of a tree. Use it when you need measurements in code; do not wrap every component without a reason.

## 4. Follow a repeatable debugging process

When something is wrong, use evidence in a consistent order:

1. **Reproduce it.** Record the route, user action, input, and whether it occurs in development or production.
2. **Read the first relevant console error or warning.** Later messages may be consequences of the first failure.
3. **Use the component stack.** Find the named component in the Components panel and inspect the props, state, and context for that render.
4. **Trace the data.** Follow the value from the source, through the parent and props, into the component that displays or changes it.
5. **Pause at the code.** Use a source breakpoint at the event handler, effect, render calculation, or API response that changes the value.
6. **Check browser behavior.** Inspect the DOM and computed styles, request and response, storage, or Performance recording when the issue crosses those boundaries.
7. **Make the smallest correction.** Fix the cause instead of adding a conditional workaround that only hides the symptom.
8. **Repeat the same steps.** Confirm that the original failure is gone and that the intended behavior still works.

Source maps connect bundled or transformed code in browser tools back to authored source files. Keep them available in development. Production source maps and uploaded stack traces should follow the application's security and deployment policy.

## 5. Read warnings as clues

A React warning usually points to a broken assumption about identity, data, or rendering. Fix the assumption where it begins; do not globally suppress the message.

| Console message or symptom | What to inspect | Typical correction |
| --- | --- | --- |
| A child in a list needs a unique `key` | The JSX returned from `map()` | Use a stable identifier from the data that is unique among siblings. |
| Invalid Hook call | Hook call location, React/renderer versions, and dependency tree | Call Hooks at the top level of a function component or custom Hook; align React and renderer versions and check for duplicate React copies. |
| Uncontrolled input becomes controlled | Initial `value` or `checked` may be `undefined` or `null` | Keep the input controlled for its full lifetime with an initial string or boolean and update it synchronously in `onChange`. |
| State update during render or an infinite render loop | A setter runs while computing JSX | Derive values during render; update state in an event or synchronization effect when appropriate. |
| Effect appears to run twice after mount in development | Whether Strict Mode is enabled and whether setup has cleanup | Make setup and cleanup a reversible pair; do not add a “run once” ref to hide a missing cleanup. |
| Server and client output differ during hydration | Initial data, locale/date formatting, random values, browser-only conditions, and HTML nesting | Make the initial server and client render agree, and pass changing initial data consistently. |

A warning can originate far from the line where the incorrect value was first created. Use the component stack and data flow to trace it back.

## 6. Common warning: list keys and item identity

A key lets React match an item in one list between renders. Use a stable ID from the item:

```jsx
function TaskList({ tasks }) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>{task.title}</li>
      ))}
    </ul>
  );
}
```

An array index can identify an item only when the list is static and items are never inserted, removed, or reordered. If a list changes, index keys can make component state or input values appear on the wrong item. A generated key such as `Math.random()` changes every render and makes React treat each item as new.

Keys are used by React and are not passed as ordinary props. If a child needs the ID, pass it separately:

```jsx
<TaskRow key={task.id} taskId={task.id} task={task} />
```

## 7. Common warning: invalid Hook call

Hooks must be called at the top level of a function component or a custom Hook. Do not call a Hook in a condition, loop, event handler, nested callback, or after an early return:

```jsx
function Profile({ userId }) {
  const [tab, setTab] = useState('overview');

  if (!userId) {
    return <p>Select a user.</p>;
  }

  // Continue rendering with the Hook already called in the same order.
  return <ProfileTabs userId={userId} tab={tab} onTabChange={setTab} />;
}
```

If the call site follows the Rules of Hooks, check that `react` and its renderer such as `react-dom` use compatible versions. Also check whether the application has accidentally installed or bundled more than one React copy:

```sh
npm ls react react-dom
```

A linked local library or an incorrect dependency declaration can cause duplicate React copies. Fix the dependency arrangement instead of changing Hook call sites that are already valid.

## 8. Common warning: controlled and uncontrolled inputs

An uncontrolled input stores its current value in the DOM. A controlled input receives its value from React and needs an `onChange` handler to update that value. Do not switch an input between these modes during its lifetime.

A controlled text field should begin with a string, even before data has loaded:

```jsx
function NameField({ initialName }) {
  const [name, setName] = useState(initialName ?? '');

  return (
    <label>
      Name
      <input
        value={name}
        onChange={(event) => setName(event.target.value)}
      />
    </label>
  );
}
```

For a checkbox, use a boolean `checked` value and update it from `event.target.checked`. When a form does not need React to own the value on every edit, use an uncontrolled input with `defaultValue` instead. Choose one approach intentionally.

## 9. Strict Mode is a development check

When Strict Mode is enabled, React runs certain development checks more than once. A component may render an extra time to reveal an impure render. Effects receive an additional setup and cleanup cycle to reveal missing cleanup, and callback refs receive an extra cycle to reveal incomplete ref cleanup. These checks do not mean production runs the same extra development cycle.

If an Effect connects to a socket, subscribes to an event, starts a timer, or otherwise sets up an external resource, its cleanup should undo that setup:

```jsx
useEffect(() => {
  const connection = connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

Do not disable Strict Mode simply because a request or log appears twice. Check whether render has side effects, whether the Effect needs cleanup, and whether a request should be triggered by an event instead of by rendering. A non-idempotent operation should not be launched by an Effect without a deliberate duplicate-request strategy.

## 10. Hydration and production-only errors

Server-rendered HTML must match the initial client render during hydration. Mismatches can come from values that vary between server and browser, such as the current time, random values, locale-specific formatting, browser-only branches, external data that changed, or invalid HTML nesting.

Use the development build to get the full warning and component context. Compare the server's initial data and markup with the client's first render. Do not silence the warning or force a client-only branch without understanding the mismatch.

A production React build may show a compact error code instead of the full development message. Reproduce the issue in a development build when possible, and preserve the production code and stack trace for investigation. Minified errors do not by themselves identify the root cause; check the component tree and the failing boundary.

## 11. Know Error Boundary limits

An Error Boundary can show fallback UI when a descendant throws while rendering or in supported lifecycle work. It does not automatically catch errors thrown by event handlers, ordinary asynchronous callbacks such as a timer, server rendering, or the boundary itself. Handle event and request failures at those boundaries, and use an Error Boundary for rendering failures where fallback UI is useful.

A visible fallback should explain that something went wrong and offer a recovery path when one exists. Keep diagnostic details in logs or monitoring rather than exposing private error data to users.

## 12. Common debugging mistakes

- **Changing many things at once:** make one targeted change so the evidence stays clear.
- **Fixing the console message instead of its cause:** follow the component stack and data to the source.
- **Logging state immediately after a setter:** the current handler still sees the state snapshot from its render; inspect the next render or log the next value explicitly.
- **Disabling Strict Mode to hide repeated work:** fix render purity or effect cleanup.
- **Adding `key={index}` to silence a missing-key warning:** choose stable identity from the data.
- **Suppressing warnings globally:** keep useful diagnostics visible and correct the offending component.
- **Optimizing before profiling:** record the interaction and identify the work first.
- **Treating React Profiler as a complete browser profiler:** use browser Performance, Network, and rendering panels for work outside React.
- **Relying only on production minified errors:** use the development build to expose actionable messages while retaining production evidence.

## Summary

React DevTools helps inspect component inputs and profile React commits. Browser developer tools help inspect the DOM, source, requests, and overall runtime. Debug by reproducing the issue, reading the first useful message, inspecting the component stack and data, and correcting the source. Treat warnings as evidence: stable keys preserve identity, Hooks follow their call rules, controlled values stay defined, render remains pure, Effects clean up, and server and client initial output must agree.
