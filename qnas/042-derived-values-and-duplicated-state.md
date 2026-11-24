# 042. Derived values and avoiding duplicated state - Q&A

## Source state and derived values

### 1. What is a derived value?

A value calculated from current props or state during rendering. Examples include a full name from name fields, a filtered list, or a count calculated from an array.

### 2. What is source state?

Information the component needs to remember between renders and that changes independently, such as input text, loaded records, a selected ID, or whether an operation is running.

### 3. Why avoid storing a value that can be calculated from other state?

The stored copy can become inconsistent with its source. Calculating it during rendering gives the UI a value based on the current inputs without requiring synchronization code.

### 4. Should I store firstName, lastName, and fullName as three state values?

Usually no. Store firstName and lastName, then calculate fullName during rendering.

### 5. Where should I calculate a full name?

In the component body, after reading the state values:

~~~jsx
const fullName = firstName + ' ' + lastName;
~~~

It is recalculated when React renders the component.

### 6. What is wrong with updating a fullName state variable in every name handler?

Every handler must remember to update it correctly. If one handler misses that update, the fields and displayed fullName can disagree.

### 7. Does calculating a value during render mean it persists by itself?

No. A derived value is recalculated from the current props and state on each render. It does not need independent persistence because its sources already persist.

## Effects and derived data

### 8. Should I use an Effect to copy firstName and lastName into fullName state?

No. Calculate fullName during rendering. An Effect adds a render with stale fullName, then a second update and render to synchronize it.

### 9. Should I use an Effect to update visibleItems when items or query changes?

Usually no. Calculate visibleItems from the current items and query during rendering.

### 10. Why can an Effect-based derived value briefly be stale?

React first renders with the source update and the old copied value. The Effect runs after that render, sets the copied value, and causes another render.

### 11. When is an Effect appropriate?

Use an Effect to synchronize with an external system, such as a browser API, a subscription, or a non-React widget. A calculation using only current props and state is not external synchronization.

### 12. Can I call a setter during render to synchronize derived state?

That is a rare specialized pattern, not a default way to derive values. First check whether the value can be calculated during rendering or whether state should reset through a changed component key.

## Lists, selection, and summaries

### 13. How should I derive a filtered list?

Keep the source list and filter controls in state, then use filter during rendering:

~~~jsx
const visibleItems = items.filter((item) =>
  item.name.toLowerCase().includes(query.trim().toLowerCase())
);
~~~

### 14. Is visibleItems state if it is stored in useState?

Yes, but if it is always computed from items and query, that state is redundant and must be synchronized. Prefer a local calculation.

### 15. Should I store the number of items separately from the array?

Not if it is always the array's length. Calculate items.length during render.

### 16. How should I calculate the number of available products?

Calculate it from the current products:

~~~jsx
const availableCount = products.filter((product) => product.inStock).length;
~~~

Do not maintain a second count that every add, remove, and stock update must keep synchronized.

### 17. How can I store which product is selected without duplicating its data?

Store the selected product's stable ID, then find the current item:

~~~jsx
const selectedProduct =
  products.find((product) => product.id === selectedProductId) ?? null;
~~~

### 18. What if the selected ID is not in the current list?

The lookup returns null. Render an appropriate empty or unavailable state instead of relying on a stale copy of the old object.

### 19. Why can storing the whole selected object separately cause a bug?

If the source list changes but selectedProduct does not, the list and details can show different versions of the same item. Deriving the item from its ID and the current list avoids that duplicate copy.

### 20. How can I sort a list for display without mutating state?

Copy the array before sorting:

~~~jsx
const sortedItems = [...items].sort(compareItems);
~~~

Do not call sort directly on the state array because sort mutates it.

### 21. Should I store a sorted copy of the list?

Usually no. Store the source list and, if the user chooses a sort mode, store that mode. Derive the sorted view from the list and mode.

### 22. Can I derive a value from a derived value?

Yes, as long as the calculations are pure and clear. For example, filter products, then compute the sum of the visible products. Avoid storing intermediate results that can be recalculated.

## Props and intentional drafts

### 23. Should I copy a prop into state so I can display it?

No, if the value should always follow the parent. Read the prop directly.

### 24. Why does useState(propValue) not keep the state in sync with the prop?

The argument initializes state for that component instance. Later renders with a changed prop do not automatically reset the state variable.

### 25. When is keeping an initial prop in local state intentional?

When the component treats it as a starting value for a user-edited draft and intentionally allows that draft to differ from later prop values. Naming the prop initialTitle or defaultValue can communicate that behavior.

### 26. What if the draft should reset whenever the edited record changes?

If the entire component's state should reset for a new record, give the child a key based on that record's identity so React creates a new instance. Avoid manually mirroring every prop into state.

### 27. Is a user's selected tab derived from another value?

Usually no. The selected tab changes through user interaction and is a genuine state value. Other UI such as the tab's label or content can be derived from it.

## Memoization and performance

### 28. Should I wrap every derived value in useMemo?

No. Start with a plain calculation. useMemo adds complexity and is mainly useful when an expensive calculation runs often and profiling shows that caching helps.

### 29. Does useMemo make a value state?

No. It caches the result of a calculation as a performance optimization. The value is still derived from its dependencies.

### 30. Can application correctness depend on React preserving a useMemo cache?

No. The component should still be correct if React recalculates the value. useMemo is not a persistent source of truth.

### 31. What belongs in the useMemo dependency list?

Every reactive value read by the calculation, such as the source array, query, or category. If a dependency changes, React needs to recalculate the result.

### 32. Does useMemo fix an incorrect or impure calculation?

No. The calculation still needs to be correct and pure. Memoization does not make mutating the source array safe.

## State design decisions

### 33. Are all values that can be calculated forbidden from state?

Not absolutely. A value can intentionally be editable, persisted independently, or represent a historical snapshot. The question is whether it is truly independent or whether it is a duplicate that must track another value.

### 34. Is a draft copy of a server value always redundant?

No. If a user is editing a draft that may differ from the saved value, the draft is meaningful independent state. Make the distinction clear and decide what should happen when the source record changes.

### 35. Should I store canSubmit as state?

Usually not if it is fully determined by current inputs and isSubmitting. Derive it:

~~~jsx
const canSubmit =
  email.trim() !== '' &&
  password.length >= 8 &&
  !isSubmitting;
~~~

### 36. Should I store isSubmitting as a derived value?

No. It usually represents whether an actual operation is in progress, so it changes when an operation starts or finishes. It is genuine state.

### 37. How do I avoid storing contradictory booleans?

Prefer one clear source or a single status value when only certain combinations make sense. For example, a status string can represent 'idle', 'saving', 'saved', or 'error' rather than several booleans that could conflict.

### 38. Should I derive data that arrives asynchronously?

The fetched result itself is usually stored because it comes from an external operation. Values calculated from that result, such as a filtered view or count, can be derived during rendering.

## Check your understanding

### 39. Which values should be state here?

~~~jsx
const [items, setItems] = useState(initialItems);
const [query, setQuery] = useState('');
const [visibleItems, setVisibleItems] = useState(initialItems);
const [itemCount, setItemCount] = useState(initialItems.length);
~~~

Usually items and query are state. Calculate visibleItems from items and query, and calculate itemCount as items.length.

### 40. Why is this Effect unnecessary?

~~~jsx
useEffect(() => {
  setVisibleItems(items.filter((item) => item.active));
}, [items]);
~~~

visibleItems is calculable from items. Derive it during rendering rather than scheduling a second state update and render.

### 41. What should happen if a selected product's price changes?

If the component stores the selected product ID and derives the selected product from the current array, the selected details use the updated product automatically.

### 42. What should I ask before adding a state variable?

Ask whether the value changes independently, whether it can be calculated from current props or state, and whether keeping a second copy would require synchronization.

## Summary

- Store source values that must persist; calculate dependent values during rendering.
- Avoid duplicate state that can become stale or inconsistent.
- Do not use an Effect to copy a derived value into state.
- Store selected IDs and derive current objects from the current collection.
- Use props directly when they should track the parent; use local draft state only when divergence is intentional.
- useMemo is an optional performance cache, not a source of truth.
