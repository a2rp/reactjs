# Memoization with memo and Optimization Decisions

## Learning objectives

After this lesson, you should be able to:

- Explain what React memoization can and cannot skip.
- Use memo to skip some renders when a component receives unchanged props.
- Distinguish memo, useMemo, and useCallback.
- Recognize when object and function identity defeats a memoization boundary.
- Decide whether an optimization is useful by profiling the real interaction.
- Avoid relying on memoization for correctness.

## 1. Start with normal rendering

A React component describes UI from its current props, state, and context. When a component updates, React may render its descendants to determine what the UI should look like. This is normal behavior; a render does not necessarily mean the browser rewrites every DOM node.

Most components are inexpensive to render. Start with clear component boundaries and ordinary React code. Add manual memoization only when it addresses work that matters.

## 2. What memo does

memo wraps a component and returns a memoized component. When its parent renders, React can skip rendering the wrapped component if its props compare equal to the previous props.

    import { memo } from 'react';

    const Greeting = memo(function Greeting({ name }) {
      return <p>Hello, {name}.</p>;
    });

By default, React compares each prop with Object.is. This is a shallow, per-prop comparison:

- Equal primitive values such as the same string or number compare equal.
- Two different object references compare different, even when their contents look identical.
- Two separately created functions compare different.

memo is an optimization hint, not a promise that React will never render the component. The component can still render when its own state changes, when a context it reads changes, or when React needs to render it for another reason.

A component must be correct without memo. Do not use memo to hide mutation, impure rendering, or a missing state update.

## 3. When a memoized component can help

A memo boundary may help when both conditions are true:

1. A component often receives the same props while its parent renders for unrelated reasons.
2. Rendering that component is costly enough that skipping it matters.

For example, a product preview might be expensive to render while a parent updates a small search field. If the preview receives stable props, memo may avoid repeating its work.

It usually does not help when:

- The component is already fast.
- Its props change on every parent render.
- The parent and child always update together.
- The extra code makes the component harder to understand without a measured benefit.

Do not wrap every component in memo as a default habit.

## 4. Keep props stable and focused

A memoized component can only skip work when its props compare equal. Prefer passing the smallest useful values instead of a large object when that makes the component boundary clearer.

    // A specific primitive is easy to compare.
    <ProfileCard name={person.name} isOnline={person.isOnline} />

    // The whole object is compared by reference.
    <ProfileCard person={person} />

Passing an object is perfectly valid. It simply means the object reference must remain the same for the default comparison to consider that prop unchanged.

Avoid creating needless new objects and functions in a frequently rendering parent when a measured memo boundary depends on stable props. First consider whether the component boundary or state placement should change. If stability is still useful, use the appropriate Hook or let the React Compiler handle eligible code.

## 5. memo, useMemo, and useCallback

These APIs address different values:

| API | What it memoizes | Typical use |
| --- | --- | --- |
| memo | A component's rendered work for equal props | A measured, often-rendering child with stable props |
| useMemo | A calculation result | An expensive pure calculation or a value whose stable identity is required |
| useCallback | A function definition | A callback whose stable identity is useful to a memoized child or another Hook |

### useMemo for a calculation

    import { useMemo } from 'react';

    function SearchResults({ records, query }) {
      const visibleRecords = useMemo(
        () => filterRecords(records, query),
        [records, query]
      );

      return <ResultList records={visibleRecords} />;
    }

useMemo calls the calculation during render when needed and reuses its previous result while dependencies compare equal with Object.is. Include every reactive value used by the calculation. The calculation must be pure.

Do not use useMemo for cheap calculations without a reason. Do not use it as persistent storage or as a replacement for state. React may discard a memoized cache in specific situations; correctness must not depend on the cache.

### useCallback for a function

    import { useCallback, useState } from 'react';

    function Counter() {
      const [count, setCount] = useState(0);

      const increment = useCallback(() => {
        setCount((currentCount) => currentCount + 1);
      }, []);

      return <button onClick={increment}>Count: {count}</button>;
    }

useCallback returns a function and can preserve its identity while its dependencies stay equal. It does not call the function for you.

A callback can often be simpler without useCallback. Use it when a stable function identity helps a measured memoized child or is needed as a dependency of another Hook. Keep the dependency list complete. An incomplete list can capture stale values and cause bugs.

When a callback only needs the previous state to calculate the next state, an updater function can avoid reading that state in the callback, as in the example above.

## 6. Object and function identity

JavaScript creates a new object or function expression when that expression runs. Therefore, these props can change by reference on every render:

    <Chart options={{ showLegend: true }} onSelect={(id) => select(id)} />

Even when the values appear equivalent, the child receives new references. A memoized child will normally render again because its props changed.

Possible responses:

1. Check whether memo is needed at all.
2. Keep state closer to the component that uses it.
3. Pass smaller primitive props when that is a natural interface.
4. Use useMemo or useCallback if stable identity is useful and profiling supports it.
5. Use React Compiler where available; it can optimize eligible components, values, and functions automatically.

Do not restructure every object or callback preemptively. Optimizations add code and should earn their complexity.

## 7. Custom prop comparison

memo accepts an optional arePropsEqual function. It receives previous props and next props, and must return true only when the component would render the same output and behave the same way with either set.

    const Chart = memo(
      ChartView,
      (previousProps, nextProps) =>
        previousProps.points === nextProps.points &&
        previousProps.width === nextProps.width
    );

Custom comparisons are easy to get wrong:

- Every prop that can affect output or behavior must be considered.
- Function props must be considered. A changed callback can close over new parent values.
- Returning true for changed behavior can leave the component using stale data.
- A deep comparison can cost more than rendering the component.
- Data structures that change shape can make a hand-written comparison fragile.

Start with the default comparison. Add a custom comparator only after profiling proves it is useful and after checking that it compares all relevant props.

## 8. State and context are separate from props

memo only compares props passed from a parent. It does not block updates from the component's own state or context.

If a component reads a context whose value changes, it must update to receive the new context. If a large component reads context but only a small part of its UI depends on that value, split the component: read context in a small outer component, then pass a focused prop to a memoized inner component if profiling shows a benefit.

Prefer local state for interactions that affect only one part of the UI. Keeping transient form, hover, or disclosure state near its consumer can prevent unrelated parts of a tree from rendering.

## 9. Prefer structural improvements first

Before adding memoization, consider these simpler changes:

- Keep transient state close to where it is used.
- Split a large component so unrelated sections have separate state and work.
- Pass JSX as children to visual wrapper components when appropriate.
- Keep render calculations pure and reasonably small.
- Avoid Effects that derive state unnecessarily.
- Use stable keys for list items.
- Reduce the amount of work or data the component must process.

These changes can improve the component design even when no memoization is needed.

## 10. React Compiler and manual memoization

React Compiler can automatically apply memoization to eligible components and values, reducing the need for manual memo, useMemo, and useCallback calls. Follow the compiler setup and diagnostics for the project's toolchain.

When the compiler is enabled:

- Write straightforward React code first.
- Do not mechanically add manual memoization to every component.
- Existing manual memoization remains supported.
- Keep an existing optimization if it has a clear purpose; remove or simplify it only when behavior and profiling support that change.
- Remember that the compiler does not make incorrect or impure code correct.

Without the compiler, the same principles apply: begin with simple code, find a measured bottleneck, then choose the smallest optimization that addresses it.

## 11. A practical optimization workflow

1. Reproduce the slow interaction.
2. Use React Developer Tools Profiler or browser performance tools to find the expensive work.
3. Check whether the cost is repeated calculation, cascading renders, DOM work, network activity, or something else.
4. Make one focused change, such as moving state, splitting a component, memoizing a component, or caching an expensive calculation.
5. Repeat the same interaction and compare measurements.
6. Keep the change only if it improves the relevant result without making the code needlessly difficult to maintain.

Measure realistic data and interactions. Development behavior can differ from production, and Strict Mode may intentionally repeat some render-related work to expose impurities.

## Key takeaways

- memo can skip some parent-caused renders when props are unchanged.
- The default prop comparison uses Object.is for each prop.
- State and context updates can still render a memoized component.
- useMemo caches a calculation result; useCallback caches a function identity.
- Manual memoization is for performance, never correctness.
- Profile first, choose the simplest useful optimization, and measure again.
- React Compiler can handle many optimizations automatically for eligible code.
