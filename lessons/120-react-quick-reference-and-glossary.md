# 120. React quick reference and glossary

This final chapter is a desk reference for the React course. Use it to recall a pattern or find the API family that fits a task; return to the linked lesson when you need the full explanation. The repository's [ReactJS summary](../reactjs-summary.md) gives a longer narrative review, and [ReactJS keywords and concepts](../reactjs-keywords.md) is the broader glossary.

## Version scope

This reference describes stable React **19.3** as documented on November 25, 2026. Check your installed React version before copying version-specific APIs. Keep **react** and **react-dom** compatible. Features marked with a release number were introduced or made stable in that release.

React's reference site follows the latest React version. Framework behavior, Server Component bundler protocols, and third-party packages have their own versioning and upgrade rules.

## 1. The shortest useful mental model

A React component is a JavaScript function that describes UI from its current inputs. React calls components, reconciles the returned descriptions, and commits the necessary changes to the host environment.

~~~text
props + state + context
          |
          v
       render
          |
          v
      React tree
          |
          v
       commit
          |
          v
  browser DOM / other renderer
~~~

The practical rules are:

1. **Describe UI; do not manually keep React's DOM in sync.** Change props or state and let React render the result.
2. **Keep rendering pure.** For the same props, state, and context, return the same UI and avoid observable side effects.
3. **Treat props and state as read-only snapshots.** Make new arrays and objects when updating state.
4. **Keep state minimal and near its users.** Derive values that can be calculated; lift state only when components need to coordinate.
5. **Use Effects to synchronize with external systems.** Use event handlers for work caused by an interaction and render-time calculations for derived UI.
6. **Use semantic HTML first.** Native controls bring keyboard and accessibility behavior that a generic element does not.
7. **Measure before optimizing.** Memoization and scheduling APIs address specific costs; they are not general correctness fixes.

## 2. Component and JSX patterns

### A small component

~~~jsx
function Greeting({ name }) {
  return <h1>Hello, {name}</h1>;
}

export default function App() {
  return <Greeting name="Mina" />;
}
~~~

- Component names begin with a capital letter; built-in HTML tags are lowercase.
- A component returns a React node. It may return an element, text, a number, a Fragment, an array of nodes, or an empty value such as null.
- JSX is transformed by a build tool. It is not raw HTML and needs a JSX-capable toolchain.
- Put JavaScript expressions in braces: {name}.
- Use className, htmlFor, camel-cased DOM props, and a style object such as style={{ backgroundColor: "tomato" }}.
- Use Fragment or <>...</> to group siblings without adding a DOM wrapper. Use explicit <Fragment key={id}> for keyed Fragments; the shorthand cannot take a key or ref.
- A component should not be defined inside another component unless there is a specific reason; a new function identity can reset the nested component's state.

### Lists, keys, and conditions

~~~jsx
<ul>
  {people.map((person) => (
    <li key={person.id}>{person.name}</li>
  ))}
</ul>
~~~

Use a stable key that identifies the item among its siblings. Avoid array indexes when items can be inserted, removed, or reordered, and do not generate a fresh random key during rendering. A key helps React preserve identity; it is not a regular prop.

Use normal JavaScript control flow:

~~~jsx
{isLoading ? <Spinner /> : <Results items={items} />}

{hasError && <p role="alert">{errorMessage}</p>}

{items.length === 0 ? <p>No results.</p> : <ItemList items={items} />}
~~~

Remember that 0 && <Panel /> evaluates to 0, which React renders. For counts, use an explicit condition such as count > 0 && ....

## 3. State and updates

### Pick the right representation

| Need | Use |
| --- | --- |
| A value that changes and affects UI | useState or useReducer |
| A value that can be calculated from props/state | Calculate it during render |
| A mutable value that should not trigger rendering | useRef |
| A value shared through many levels | Context, after considering composition |
| A coordinated set of state transitions | A pure reducer |
| State shared across separate application areas | Consider a suitable external store or framework |

A state setter queues a future render. It does not change the state snapshot used by the currently running component or event handler.

~~~jsx
const [count, setCount] = useState(0);

function incrementTwice() {
  setCount((current) => current + 1);
  setCount((current) => current + 1);
}
~~~

Use a functional updater when the next value depends on the previous queued value. Replace objects and arrays instead of mutating them:

~~~jsx
setUser((current) => ({ ...current, name: nextName }));
setItems((current) => current.filter((item) => item.id !== removedId));
~~~

### Avoid duplicate or derived state

~~~jsx
const [query, setQuery] = useState("");
const visibleItems = items.filter((item) =>
  item.name.toLowerCase().includes(query.toLowerCase())
);
~~~

Do not copy visibleItems into state and synchronize it with an Effect. That creates a second source of truth and an extra render.

### State identity and resetting

React preserves state according to a component's type, position in the tree, and key. A different key deliberately creates a new identity and resets the state for that subtree:

~~~jsx
<Chat key={recipient.id} recipient={recipient} />
~~~

Use this when the entire state belongs to the keyed entity. Do not change keys just to force arbitrary updates.

## 4. Events, forms, and accessibility

Pass a function to an event prop; do not call the handler during render:

~~~jsx
<button type="button" onClick={save}>Save</button>
<button type="button" onClick={() => removeItem(item.id)}>Remove</button>
~~~

Use a real button for an action and a link for navigation. A clickable div does not automatically become keyboard accessible.

### Controlled input

~~~jsx
const [email, setEmail] = useState("");

<label htmlFor="email">Email</label>
<input
  id="email"
  type="email"
  value={email}
  onChange={(event) => setEmail(event.target.value)}
/>
~~~

A controlled text input has a string value and synchronously updates it in onChange. For checkboxes, control checked. Avoid switching an input from controlled to uncontrolled during its lifetime.

### Form submission

~~~jsx
function SearchForm({ onSearch }) {
  const [query, setQuery] = useState("");

  function handleSubmit(event) {
    event.preventDefault();
    onSearch(query.trim());
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="search">Search</label>
      <input
        id="search"
        value={query}
        onChange={(event) => setQuery(event.target.value)}
      />
      <button type="submit">Search</button>
    </form>
  );
}
~~~

Use browser defaults unless the app has a reason to replace them. Associate labels with fields, describe errors in text, connect hints and errors to their inputs, and expose pending/success/error feedback.

### React form Actions

React 19 supports functions as form actions. useActionState can hold a result and pending state; useFormStatus reads the status of a parent form from a descendant component. These APIs do not automatically create a backend or make a function run on a server.

Use 'use server' only for a Server Function in a framework/runtime that supports it. It does not label a Server Component.

## 5. Hook and API finder

Call Hooks at the top level of a function component or custom Hook, following the Rules of Hooks. The React use API has special conditional-call behavior; do not generalize that exception to other Hooks. A name beginning with use does not make an arbitrary function a Hook.

### React Hooks

| Hook | Use it for | Quick caution |
| --- | --- | --- |
| useState | Local state with direct setter updates | State is a render snapshot; use updater functions when needed |
| useReducer | Related state updates described by actions | Keep the reducer pure |
| useContext | Read the nearest provider value | Context consumers update when that value changes |
| useRef | Store a DOM node or mutable value outside rendering | Changing .current does not re-render |
| useEffect | Synchronize with an external system after commit | Include dependencies and return cleanup when setup needs undoing |
| useLayoutEffect | Read or adjust layout before paint | Can block paint; prefer useEffect when possible |
| useInsertionEffect | Insert styles before layout Effects | Primarily for CSS-in-JS libraries |
| useEffectEvent | Read current values from logic called by an Effect | React 19.2; do not use it to conceal a real dependency |
| useMemo | Cache an expensive calculation between renders | Optimization only; code must be correct without the cache |
| useCallback | Cache a function identity for a memoized consumer | Optimization only; avoid adding it everywhere |
| useTransition | Mark updates as non-blocking and read pending status | Keep controlled text input updates urgent |
| useDeferredValue | Let a non-critical value lag behind urgent UI | It does not debounce or cancel a network request |
| useId | Create stable IDs for accessibility relationships | Not a list key or data identifier |
| useImperativeHandle | Customize a ref exposed by a component | Escape hatch; prefer declarative props |
| useSyncExternalStore | Subscribe to an external store | Use the store's subscription/snapshot contract |
| useDebugValue | Label a custom Hook in React DevTools | Mostly useful for custom Hook authors |
| useActionState | Track a form Action's result and pending state | React 19; an Action still needs a real implementation |
| useOptimistic | Show temporary optimistic state during an Action | The final result still must reconcile with authoritative data |
| use | Read a Promise or Context; in React 19.3, consume browser() | A pending Promise can suspend to the nearest Suspense boundary |

Custom Hooks are ordinary JavaScript functions that use Hooks to share stateful logic. Their state is not automatically shared between callers.

### Built-in React components

| Component | Purpose |
| --- | --- |
| <Fragment> | Group nodes without adding a wrapper; React 19.3 supports refs on explicit Fragments |
| <StrictMode> | Enable extra development-only checks |
| <Suspense> | Show a fallback while supported child work suspends |
| <Profiler> | Collect programmatic render timing information |
| <Activity> | Hide/restore a subtree while preserving its React state; stable in React 19.2 |
| <ViewTransition> | Coordinate browser View Transition animations with React Transitions; stable in React 19.3 |

Built-in HTML and SVG tags such as <button>, <form>, <input>, and <svg> are React DOM components too. Use their web platform semantics and React DOM prop conventions.

## 6. React package map

Import APIs from the package entry point that owns them. A framework may call top-level rendering APIs on your behalf.

| Import path | Common APIs | Typical owner |
| --- | --- | --- |
| react | Components, Hooks, createContext, lazy, memo, startTransition, addTransitionType | Application components |
| react-dom | createPortal, flushSync, requestFormReset, browser, resource preload APIs | Web-specific component integrations; rarely needed directly |
| react-dom/client | createRoot, hydrateRoot | Client entry point, unless a framework owns it |
| react-dom/server | renderToPipeableStream, renderToReadableStream, string renderers | Server entry point, usually framework-owned |
| react-dom/static | prerender, prerenderToNodeStream and resume APIs | Static prerendering, usually framework-owned |
| react-dom Hooks | useFormStatus | A descendant that needs its parent form's pending state |

### Less common React APIs

- createContext: define a context and its fallback value.
- lazy: load a component's module when it is first rendered; place the lazy component under Suspense.
- memo: optionally skip rendering when props compare equal; profile before adding it.
- startTransition / addTransitionType: mark non-urgent work and identify the Transition's cause.
- cache / cacheSignal: server-side cache lifetime APIs; use in the supported server rendering environment.
- act: test helper for flushing React updates before assertions.
- captureOwnerStack: development diagnostic API.

### React DOM APIs that need a reason

- createPortal: render a subtree in another DOM location while preserving its React ownership.
- flushSync: force pending React work to commit synchronously for an integration that requires it. It can hurt performance and interfere with transitions.
- Resource APIs such as preload, preloadModule, preconnect, prefetchDNS, preinit, and preinitModule: give the browser a resource hint when the app knows a resource will be needed. Frameworks often manage these.
- browser: with use(browser()), opt a Client Component out of SSR behind a Suspense fallback when server output is not meaningful.

### Client and server entry points

- createRoot(container): mount a client-rendered React tree in a DOM container.
- hydrateRoot(container, tree): attach React to matching HTML that was generated earlier. Keep the first client render consistent with the server output.
- Server stream APIs: generate HTML with streaming and Suspense support. Node.js and Web Stream environments have different entry points.
- Static prerender APIs: produce static HTML; use streaming SSR when the app needs request-time streaming or progressive Suspense reveals.

React 19 removed ReactDOM.render, ReactDOM.hydrate, findDOMNode, and unmountComponentAtNode. Use createRoot, hydrateRoot, refs, and root.unmount() instead. Older server stream APIs were also replaced by current server entry points.

## 7. Choosing a pattern

| Situation | First choice | Avoid by default |
| --- | --- | --- |
| Parent needs to configure a child | Props and callbacks | Mutating child internals |
| Deep descendants need a shared setting | Composition first; Context when it removes noisy plumbing | Putting every value in global context |
| Show a value computed from current inputs | Calculate it during render | Mirroring it into state with an Effect |
| Respond to a click or submit | Event handler or form Action | Effect that watches a flag set by the handler |
| Connect to a subscription or browser API | Effect with cleanup, or an appropriate external-store API | Starting subscriptions during render |
| Focus, measure, or call an imperative widget API | Ref and a narrowly scoped imperative handle | Querying/manipulating React-owned nodes throughout the tree |
| Show a lazy module or suspended resource | Suspense boundary with an intentional fallback | Assuming every Promise automatically suspends |
| Improve an interaction that blocks input | Transition or deferred rendering after identifying the slow work | Marking the controlled input itself non-urgent |
| Optimize a slow render | Profile; then memoize the measured work | Blanket memo, useMemo, or useCallback |
| Render in a new location in the DOM | Portal | Duplicating the component's state |
| Build navigation and data-loading conventions | A router or React framework | Assuming React itself provides routing |
| Share a server-rendered page and client interactivity | SSR/hydration and framework conventions | Reading browser-only values before hydration |
| Insert untrusted rich HTML | A vetted sanitizer and security policy | Treating React or Trusted Types as a sanitizer |

## 8. Server rendering and framework boundaries

- **Server rendering (SSR)** creates HTML outside the browser so a page can show content before client JavaScript is ready.
- **Hydration** attaches React's client behavior to compatible server-generated HTML. A hydration mismatch means the server and initial client output disagree.
- **Streaming** can send server-rendered content in chunks and reveal Suspense boundaries progressively.
- **Server Components (RSC)** render in a separate server/build environment. They can access server-only data but do not run interactive client Hooks.
- **Client Components** are included in the browser bundle and can use state, Effects, and browser APIs. In frameworks that use RSC, 'use client' marks a module boundary.
- **Server Functions** are callable server-side functions supported by a compatible framework. 'use server' marks such a function; it does not mean “this file is a Server Component.”
- **Frameworks** may own routing, bundling, SSR, caching, and RSC integration. Choose a framework because the app needs those conventions, not because JSX requires one.
- React's RSC implementation APIs used by bundlers/frameworks do not necessarily follow minor-version semver. Framework authors should follow their framework's support policy and React's official guidance.

Do not assume browser globals exist during server rendering. Keep initial server and client output deterministic, or isolate a truly browser-only subtree with the supported browser bailout pattern from the React 19.3 lesson.

## 9. Quality checklist

### Correctness

- [ ] Components and reducers are pure.
- [ ] State is replaced immutably and updates use functional updaters where needed.
- [ ] Lists use stable keys from data.
- [ ] Derived values are computed rather than duplicated.
- [ ] Effects include their real reactive dependencies and clean up subscriptions/timers.
- [ ] Async requests handle loading, errors, cancellation or stale responses, and empty results.
- [ ] Server and initial client output match during hydration.

### Accessibility and user experience

- [ ] Controls use appropriate native elements and have visible labels.
- [ ] Keyboard users can operate the interface and see focus.
- [ ] Validation errors and async status are announced in meaningful text.
- [ ] Focus is moved or restored when the interaction requires it.
- [ ] View Transitions honor reduced-motion preference; content remains usable without animation.
- [ ] Color, semantics, and status do not rely on one visual cue alone.

### Security, performance, and delivery

- [ ] Untrusted input is treated as text or sanitized before any HTML injection.
- [ ] Secrets and privileged data stay on the server.
- [ ] Performance work is based on a trace or measurement.
- [ ] Bundle size, loading behavior, and production errors are understood.
- [ ] The chosen React, React DOM, framework, and library versions are compatible.
- [ ] Build, deployment, monitoring, and maintenance steps are documented.

## 10. Glossary

| Term | Quick definition |
| --- | --- |
| Action | A function used by a form or a Transition to perform work and report a result; framework Server Functions are a separate server capability. |
| Batching | Processing several queued state updates together before rendering the result. |
| Boundary | A component-tree location that controls a behavior such as error recovery, Suspense fallback, or transition animation. |
| Client Component | A component included in browser JavaScript; in RSC frameworks, it is introduced through a client module boundary. |
| Commit | Applying a completed React render to the host environment and running the associated commit work. |
| Component | A reusable function or class that describes part of the UI. |
| Context | A channel for providing a value to descendants without passing it through every component in between. |
| Controlled input | A form control whose current value is driven by React props/state. |
| Custom Hook | A function that composes Hooks to reuse stateful logic; each call has its own Hook state unless it connects to shared state. |
| Derived state | A value stored separately even though it can be calculated from current props or state; often an avoidable duplicate. |
| Effect | Post-commit synchronization between React and an external system. |
| Element | A description of a component or host element for React to render. |
| Hydration | Attaching React to compatible HTML previously generated outside the client render. |
| Identity | How React matches a component instance across renders using type, position, and key. |
| Key | A stable sibling identifier used during reconciliation; it is not passed to the child as a prop. |
| Memoization | Reusing a previously computed value or render result when inputs compare equal. |
| Node (React node) | A value React can render as a child, including elements, text, numbers, arrays, portals, or empty values. |
| One-way data flow | The usual pattern of passing props down and sending change requests back through callbacks or shared state. |
| Portal | A React subtree rendered into a different DOM location while remaining part of the same React tree. |
| Props | Read-only inputs passed from a parent to a child component. |
| Reconciliation | Comparing a new rendered description with the previous tree to decide what must change. |
| Render | Calling components to calculate the next UI description. A render does not always cause DOM changes. |
| RSC | React Server Components, rendered in a server/build environment separate from the client app. |
| Server Component | A component rendered by an RSC-capable environment; it cannot use client-only state and Effects. |
| Server Function | A function marked for remote/server execution by a framework or compatible runtime. |
| State | Data whose changes can affect the rendered interface. |
| State snapshot | The state values visible to one particular render and the handlers created by it. |
| Strict Mode | A development-only mode that runs extra checks to help detect impure rendering and missing cleanup. |
| Suspense | A boundary that displays a fallback while supported child work is not ready. |
| Transition | Non-urgent React work that can be interrupted by urgent updates. |
| Uncontrolled input | A form control whose current value is primarily kept by the DOM. |
| Hydration mismatch | A difference between server HTML and the initial client-rendered output. |
| View Transition | A browser snapshot animation that React 19.3 can coordinate with eligible React Transitions. |

### Commonly confused pairs

| Pair | Difference |
| --- | --- |
| Props vs state | Props are parent-provided inputs; state is data owned by a component or state system. |
| State vs ref | State affects rendering; changing a ref does not request a render. |
| Render vs commit | Render calculates UI; commit applies accepted changes to the host environment. |
| Event handler vs Effect | An event handler runs because a user/action triggers it; an Effect keeps an external system synchronized with committed UI. |
| Effect vs derived value | An Effect synchronizes externally; a derived value belongs in render if it is calculable from current inputs. |
| Controlled vs uncontrolled input | A controlled value comes from React; an uncontrolled value remains in the DOM after its initial value. |
| useMemo vs useCallback | useMemo caches a calculation result; useCallback caches a function identity. Both are optimizations. |
| useTransition vs useDeferredValue | useTransition marks a state update; useDeferredValue lets a value used by a subtree lag behind. |
| Suspense vs Error Boundary | Suspense handles supported pending work; Error Boundaries handle rendering errors in a descendant tree. |
| SSR vs RSC | SSR produces HTML for a page; RSC renders components in a separate environment and can feed a client tree. They solve related but different problems. |
| Server Component vs Server Function | A Server Component describes server-rendered UI; a Server Function is server-side logic invoked through a supported framework mechanism. |
| Fragment vs DOM element | A Fragment groups children without a DOM node; an element has browser semantics, attributes, and a layout box. |

## 11. Find the full lesson

Use the roadmap in [README.md](../README.md) to find the detailed chapter for a topic. The final course chapters consolidate integration and production work:

- [Build and ship a complete accessible React application](116-react-capstone-project.md)
- [Upgrading React and handling version changes](117-react-upgrades-and-version-changes.md)
- [React 19.2: Activity and performance tooling](118-react-19-2-features.md)
- [React 19.3: View Transitions, Fragment refs, and browser APIs](119-react-19-3-features.md)

Then return to the root [ReactJS summary](../reactjs-summary.md) for a longer recap or [ReactJS keywords and concepts](../reactjs-keywords.md) for definitions across the whole roadmap.

## Key reminders

- React is a UI library; the surrounding stack supplies many app-level conventions.
- Make render a pure description of the current UI.
- Model state once, derive what can be calculated, and respond to user actions with event handlers or Actions.
- Use Effects only when React must synchronize with something external.
- Preserve semantics, accessibility, security, and hydration correctness while building.
- Optimize measured bottlenecks and keep version-sensitive features in view.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="119-react-19-3-features.md">119 - React 19.3: View Transitions, Fragment refs, and browser APIs</a></td><td><a href="../qnas/120-react-quick-reference-and-glossary.md">Q&amp;A</a></td><td><a href="../examples/120-react-quick-reference-and-glossary.md">Example</a></td><td>End of course</td></tr>
  </tbody>
</table>
