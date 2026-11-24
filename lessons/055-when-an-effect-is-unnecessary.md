# When an Effect Is Unnecessary

## The short rule

An Effect is for synchronizing a committed React component with something outside React. If the work can be expressed as rendering from current props and state, or as a response to a specific user action, an Effect usually adds an unnecessary second step.

Before writing an Effect, ask:

> What external system is this component synchronizing with?

If the answer is “none,” first try to solve the problem with render logic, state design, or an event handler.

Removing unnecessary Effects often removes duplicate state, extra renders, stale intermediate UI, and difficult-to-trace data flow.

## Learning goals

After this lesson, you should be able to:

- distinguish render calculations, user events, and external synchronization;
- derive display values directly from props and state;
- use memoization only when a measured calculation needs caching;
- avoid mirroring props or derived values into state;
- reset a component subtree with a key when its identity changes;
- choose an event handler or lifted state instead of an Effect;
- recognize the rare render-time state adjustment pattern and its limits;
- identify cases where an Effect or an external-store Hook is appropriate.

## Three different causes of work

React applications have work with different causes. Keeping those causes separate makes a component easier to reason about.

| Cause | Example | Typical place |
| --- | --- | --- |
| Rendering current inputs | Filter products using the current query | Component body |
| A particular user action | Submit an order when the form is submitted | Event handler |
| React must synchronize with an external system | Subscribe to a browser event or control a third-party widget | Effect, or a purpose-built Hook |

An Effect runs because React committed a render whose dependencies changed, or because the component mounted. It does not remember which user action caused the render. That distinction is the key to deciding whether an Effect belongs.

## Derive values during rendering

A value that can be calculated from current props or state is usually not independent state. Calculate it in the component body:

~~~jsx
function SearchableProducts({ products }) {
  const [query, setQuery] = useState("");

  const normalizedQuery = query.trim().toLowerCase();
  const visibleProducts = products.filter((product) =>
    product.name.toLowerCase().includes(normalizedQuery)
  );
  const visibleCount = visibleProducts.length;

  return (
    <>
      <label>
        Search products
        <input
          value={query}
          onChange={(event) => setQuery(event.target.value)}
        />
      </label>

      <p>{visibleCount} products shown</p>

      <ul>
        {visibleProducts.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </>
  );
}
~~~

The query is the changing input, so it belongs in state. The filtered list and its count are consequences of that input and the products, so they do not need their own state variables.

An Effect-based version would first render with an old or empty list, commit that UI, run an Effect, set the filtered list, and render again. Calculating it during render gives React the current answer in the first render.

### Avoid redundant state

This adds synchronization work without adding information:

~~~jsx
const [fullName, setFullName] = useState("");

useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
~~~

Instead:

~~~jsx
const fullName = firstName + " " + lastName;
~~~

The same principle applies to totals, counts, filtered collections, validation summaries, labels, and booleans that can be determined from existing state. Store the smallest set of values that can independently change. Derive the rest.

Do not store both a selected object and its selected identifier if the object can be found from the identifier. Keeping duplicate representations creates opportunities for them to disagree.

### Use memoization only for a performance reason

A normal calculation should stay simple:

~~~jsx
const visibleProducts = products.filter(matchesQuery);
~~~

If profiling shows that a calculation is expensive and repeating it is a real cost, cache the calculation with **useMemo**:

~~~jsx
const visibleProducts = useMemo(
  () => products.filter(matchesQuery),
  [products, normalizedQuery]
);
~~~

**useMemo** is a performance optimization, not a correctness requirement. Start with the direct calculation. Do not introduce an Effect that copies the calculated result into state; that still creates redundant state and another render. For large collections, choose an appropriate search strategy or measure the actual work before adding caching.

## Do not mirror a prop into state by accident

This pattern captures only the initial prop value:

~~~jsx
function Greeting({ name }) {
  const [editableName, setEditableName] = useState(name);
  // ...
}
~~~

If the parent later passes a different name, **editableName** remains the original state. Adding an Effect to overwrite it on every prop change can be just as confusing: it is unclear whether the user or the parent owns the value.

Choose the ownership that matches the interface:

- If the value is read-only, render the prop directly.
- If the parent owns edits, pass a value and an event callback so the component is controlled.
- If the child owns a draft, make that ownership explicit and decide what should happen when the edited record changes.

A prop named **initialName** or **defaultName** can communicate that it is only used to initialize local state. Later prop changes are intentionally ignored until the component is reset.

## Reset state when identity changes

Suppose a profile editor has a local draft. When the selected profile changes, the previous draft should not carry into the new profile. Give the editor a key based on the identity of the edited profile:

~~~jsx
<ProfileEditor key={profile.id} profile={profile} />
~~~

~~~jsx
function ProfileEditor({ profile }) {
  const [draft, setDraft] = useState(profile.name);

  return (
    <label>
      Display name
      <input
        value={draft}
        onChange={(event) => setDraft(event.target.value)}
      />
    </label>
  );
}
~~~

When the key changes, React treats this as a new editor instance and initializes its local state from the new profile. This also resets state in that editor's descendants. Use a stable identity, not a value that changes on every render.

A key is suitable when the whole keyed subtree belongs to the new identity and should reset. It also discards unsaved local state, which should be an intentional product decision.

## When only part of local state needs adjustment

Sometimes a component should preserve most local state when a prop changes but clear or adjust one particular value. First consider whether the value can be derived or whether the state should be moved to the parent.

React also permits a conditional update to the same component's state during rendering. React discards that render's result and retries the component before rendering its children:

~~~jsx
function SelectionList({ items }) {
  const [previousItems, setPreviousItems] = useState(items);
  const [selection, setSelection] = useState(null);

  if (items !== previousItems) {
    setPreviousItems(items);
    setSelection(null);
  }

  // Render the list using the current selection.
}
~~~

This is a specialized pattern, not a general replacement for Effects. The condition must make progress and stop being true after the update. Only update the currently rendering component; do not update a parent, sibling, or another component during render. This approach is harder to understand than deriving values or resetting a keyed subtree, so use it only when those clearer options do not fit.

## Respond to user actions in their handlers

If the user clicks Buy, submits a form, selects a file, or presses Save, put the action's direct consequences in the handler for that action.

Avoid recording an action in state and then watching that state with an Effect just to perform the action:

~~~jsx
// Avoid this indirect flow:
const [shouldSubmit, setShouldSubmit] = useState(false);

useEffect(() => {
  if (shouldSubmit) {
    sendOrder();
  }
}, [shouldSubmit]);
~~~

Prefer the event itself:

~~~jsx
async function handleSubmit(event) {
  event.preventDefault();
  await sendOrder();
}
~~~

The handler knows what happened and has access to the submitted values. An Effect only sees that a render occurred. Putting a purchase request or notification in an Effect can repeat it after restoring state, remounting, or changing an unrelated dependency.

Network synchronization can be an Effect when the desired external result is genuinely defined by current rendered inputs, such as keeping displayed search results synchronized with a query. For a one-time command caused by a button press, use the handler. For application data loading, also consider the framework's data-loading facilities rather than building a full fetching system inside a component Effect.

## Update related state in one event

Do not use an Effect to tell a parent that child state changed:

~~~jsx
// Avoid synchronizing the parent after the child has already rendered:
useEffect(() => {
  onChange(isOn);
}, [isOn, onChange]);
~~~

If the event can update both values, do both in the event handler:

~~~jsx
function handleToggle() {
  const nextIsOn = !isOn;
  setIsOn(nextIsOn);
  onChange(nextIsOn);
}
~~~

If the parent is the natural owner, lift the state and let the child receive the current value and a callback. Both patterns keep the state transition connected to the event that caused it. Avoid maintaining two independent copies that must later be synchronized.

## Avoid chains of state-setting Effects

A chain such as “selected item changes, Effect calculates price, another Effect calculates tax, another Effect creates a summary” makes intermediate states part of the application. It can cause multiple commits and makes it difficult to find the source of a value.

Prefer one of these:

- calculate a display result from the current state during rendering;
- update related source state together in the event handler;
- use a reducer when a single user action makes a complex, coordinated state transition.

Effects should connect React to an external system; they should not form an internal pipeline for ordinary application data.

## Effects still have an important job

Effects are appropriate when a component must synchronize with something React does not own, for example:

- registering and removing a browser event listener;
- starting and stopping a connection, timer, observer, or animation;
- controlling a non-React widget;
- updating an external browser API such as the document title;
- synchronizing with an external service whose current state follows rendered inputs.

For each Effect, name the external system and describe how setup keeps it synchronized. If there is no such system, reconsider the Effect.

For a mutable external store or browser value that can change outside React, prefer **useSyncExternalStore** when it fits. It accepts a subscription function and a snapshot reader, and React uses them to keep the rendered value current. Do not make an Effect subscription and mirror the store value into state when the dedicated Hook is the right integration.

## A practical decision process

Before adding an Effect, work through these questions:

1. Can this value be calculated from current props or state? Calculate it during render.
2. Is the calculation genuinely expensive? Measure it, then consider **useMemo**.
3. Is the work caused by a specific interaction? Put it in that event handler.
4. Are multiple components keeping the same fact? Choose one owner or lift the state up.
5. Does a change of identity mean the entire subtree should start fresh? Use a stable key.
6. Is only a narrow piece of local state affected? Reconsider the state model before using the specialized render-time adjustment pattern.
7. Is there a mutable external store? Consider **useSyncExternalStore**.
8. Is there still an external system to synchronize? If yes, an Effect may be appropriate; define dependencies and cleanup for that synchronization.

## Common mistakes

### Storing a value that is fully determined elsewhere

A duplicate state variable can become stale or require an Effect to keep it synchronized. Store the source of truth and derive the display value.

### Using **useMemo** as a default

Memoization has its own complexity. Use it only when there is a meaningful performance reason, and keep the calculation correct without the cache.

### Putting a command in an Effect

Effects can run again as dependencies change. Commands tied to a click or submission belong to the event that initiated them.

### Resetting every prop-derived value with an Effect

An Effect may let the old value render before it resets. Decide who owns the value, derive it, or use a key when a new identity should reset the whole subtree.

### Using a key for every small change

A changed key resets all state in the keyed subtree. This can lose drafts, focus, or other intentionally preserved UI state. Key the component only when a new identity should truly begin a new state lifetime.

### Replacing every Effect with render logic

Some work is truly external synchronization. Keep Effects for that job, make their dependencies accurate, and clean up resources when their synchronization ends.

## Key takeaways

- Effects synchronize React with external systems; they are not a general state synchronization mechanism.
- Calculate derived values during rendering instead of mirroring them into state.
- Add memoization only to address a measured performance cost.
- Put interaction-specific actions in their event handlers.
- Lift shared state to one owner instead of synchronizing duplicate copies.
- Use a stable key to reset a component subtree when its identity changes.
- Reserve render-time state adjustment for narrow cases that cannot be expressed more clearly.
- Use a purpose-built external-store Hook when integrating with a mutable store.
