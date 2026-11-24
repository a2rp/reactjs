# 042. Derived values and avoiding duplicated state

## Goal

If a value can be calculated from the current props or state, calculate it during rendering instead of storing a second copy in state. This keeps one source of truth and avoids synchronization bugs.

This chapter explains derived values, filtered and selected data, redundant state, prop mirroring, and when memoization is useful.

## Learning outcomes

After this chapter, you should be able to:

- Distinguish source state from values derived during rendering.
- Calculate totals, labels, validity, and filtered lists from current data.
- Store a selected item's ID instead of duplicating the whole item object.
- Explain why synchronizing derived state with an Effect adds work and can show stale UI.
- Recognize when copied props are intentional editable state rather than redundant state.
- Use useMemo only when an expensive calculation needs caching.
- Keep rendering calculations pure.

## Store the source; calculate what follows from it

State should hold information that the component needs to remember between renders, such as values entered by the user or records loaded from an operation.

A value that can be recalculated from props or existing state usually does not need another state variable. Calculate it as the component renders:

~~~jsx
function NameForm() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  return (
    <section>
      <label>
        First name
        <input
          value={firstName}
          onChange={(event) => setFirstName(event.target.value)}
        />
      </label>

      <label>
        Last name
        <input
          value={lastName}
          onChange={(event) => setLastName(event.target.value)}
        />
      </label>

      <p>Full name: {fullName.trim()}</p>
    </section>
  );
}
~~~

firstName and lastName are the source values. fullName is calculated from them. Whenever either input changes, React renders again and calculates fullName from the new values.

## Why duplicated state becomes inconsistent

This form stores firstName, lastName, and a separate fullName:

~~~jsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');
~~~

Every update handler would need to update the name field and fullName together. If one handler forgets, the UI displays values that disagree. A single source of truth avoids that synchronization problem.

Duplicated state can also become stale when updated in an Effect:

~~~jsx
// Avoid: a second render is needed to copy derived data into state.
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);
~~~

The first render after a name change still has the previous fullName. Then the Effect runs, calls another setter, and causes another render. Since fullName is calculable during render, use a local calculation instead.

## Derive filtered lists during rendering

Keep the original list and the filter controls as state. Calculate the visible list from them:

~~~jsx
const [products, setProducts] = useState(initialProducts);
const [query, setQuery] = useState('');
const [category, setCategory] = useState('all');

const visibleProducts = products.filter((product) => {
  const matchesQuery = product.name
    .toLowerCase()
    .includes(query.trim().toLowerCase());

  const matchesCategory =
    category === 'all' || product.category === category;

  return matchesQuery && matchesCategory;
});
~~~

Do not create filteredProducts as another state variable and synchronize it with an Effect. Each render can calculate the list that matches the current products, query, and category.

It is also fine to calculate small summaries during render:

~~~jsx
const productCount = products.length;
const availableCount = products.filter((product) => product.inStock).length;
~~~

Those values always match products because they come directly from the current array.

## Derive a selected object from its ID

If a list stores objects and the user selects one, store its stable ID rather than a second copy of the entire object:

~~~jsx
const [selectedProductId, setSelectedProductId] = useState(null);

const selectedProduct =
  products.find((product) => product.id === selectedProductId) ?? null;
~~~

When an item is selected:

~~~jsx
<button onClick={() => setSelectedProductId(product.id)}>
  View details
</button>
~~~

If the selected product changes in products, selectedProduct is recalculated from the updated list. If the product is removed, the lookup returns null rather than keeping a stale object copy.

Storing the selected object separately can duplicate its data:

~~~jsx
const [selectedProduct, setSelectedProduct] = useState(null);
~~~

If products is later updated but selectedProduct is not, the list and details can disagree. Store the ID and derive the object when possible.

## Derive validity and status

If a status can be determined from current inputs, derive it instead of setting a separate boolean:

~~~jsx
const [email, setEmail] = useState('');
const isEmailPresent = email.trim().length > 0;
~~~

For several conditions:

~~~jsx
const canSubmit =
  email.trim() !== '' &&
  password.length >= 8 &&
  !isSubmitting;
~~~

Only store isSubmitting if it represents a real ongoing operation. Do not separately store canSubmit when it is fully determined by other values.

A derived value should describe the current data. A state value should represent information that needs to persist independently or is changed by an interaction or external operation.

## Props are also a source for calculations

Use props directly for values that should track the parent:

~~~jsx
function Badge({ color }) {
  return <span className={'badge badge-' + color}>{color}</span>;
}
~~~

Do not copy a prop into state just to use it:

~~~jsx
function Badge({ color }) {
  const [badgeColor, setBadgeColor] = useState(color);
  return <span>{badgeColor}</span>;
}
~~~

useState uses color only for the initial value. If the parent later passes a different color, badgeColor will not automatically change.

Sometimes a prop is intentionally an initial value for a local editable draft. Make that intent clear:

~~~jsx
function EditableTitle({ initialTitle }) {
  const [draftTitle, setDraftTitle] = useState(initialTitle);
  // Later changes to initialTitle are intentionally not copied into draftTitle.
}
~~~

This is different from a value that should always reflect the current prop. If the component's local state should reset whenever the identity of the record changes, a parent can give the component a key based on that identity so React creates a fresh instance. The state preservation chapter covers component identity and keys.

## Sorting without storing a duplicate list

If the UI needs to display a sorted view, keep the source list and derive a sorted copy:

~~~jsx
const sortedProducts = [...products].sort((first, second) =>
  first.name.localeCompare(second.name)
);
~~~

sort mutates its array, so copy products first. Do not sort the state array directly. If the order itself is a user-controlled persistent choice, store the sort criterion, such as 'name' or 'price', then derive the ordered list from products and that criterion.

## When a calculation is expensive

Start by calculating derived values directly during rendering. If profiling shows a calculation is expensive and reruns often, use useMemo to cache the result:

~~~jsx
const visibleProducts = useMemo(() => {
  return filterProducts(products, query, category);
}, [products, query, category]);
~~~

The dependency list must include every reactive value used by the calculation. If a dependency changes, React recalculates the value.

useMemo is a performance optimization, not a source of truth. The component must still be correct if React recalculates the value. Do not use useMemo just to avoid storing redundant state; a direct calculation is simpler for ordinary work.

## Keep render calculations pure

A derived calculation should return a value without changing the source state or causing side effects:

~~~jsx
const completedTasks = tasks.filter((task) => task.done);
~~~

Do not mutate the source while deriving a view:

~~~jsx
// Avoid: sort changes the state array itself.
const sortedTasks = tasks.sort(compareTasks);
~~~

Instead, work with a copy:

~~~jsx
const sortedTasks = [...tasks].sort(compareTasks);
~~~

Do not send requests, update state, or edit the DOM while calculating derived values. User actions belong in event handlers; synchronization with an external system belongs in an Effect.

## When a value should be state

A value is a good candidate for state when it changes independently and must survive a render. Examples include:

- Text currently being edited in an input.
- The original records loaded from a request.
- A selected ID or current tab.
- Whether an operation is in progress.
- A user's chosen sort order.
- A draft value that is intentionally allowed to differ from its original prop.

A value is usually derived when it can be calculated from current props and state without losing user intent:

- Full name from first and last names.
- A filtered or sorted view of an existing list.
- The count of matching records.
- Whether a form can be submitted.
- The selected object found by a selected ID.
- A total calculated from line items.

Before adding state, ask whether the value can be calculated from data you already keep.

## A practical review checklist

For every state variable, ask:

1. Does this value change independently, or can I calculate it from other current props or state?
2. If I store it separately, which handlers must keep it synchronized?
3. Could a later update change the source but leave this copy stale?
4. Is this user-edited draft intentionally separate from the original value?
5. Is the calculation actually expensive enough to need useMemo?
6. Am I changing the source data while calculating the derived value?

## Practice

A component stores this state:

~~~jsx
const [items, setItems] = useState(initialItems);
const [query, setQuery] = useState('');
const [visibleItems, setVisibleItems] = useState(initialItems);
const [itemCount, setItemCount] = useState(initialItems.length);
~~~

Refactor the design:

- Which values should remain state?
- How should visibleItems be calculated?
- How should itemCount be calculated?
- Why would an Effect that copies items and query into visibleItems be unnecessary?

Then consider a list with selectedItem stored as a whole object. Change the state design to store selectedItemId and derive the selected object from the current items.

## Recap

- Store the source values that need to persist; calculate dependent values during rendering.
- Avoid duplicate state that must be synchronized.
- Do not use an Effect to copy derived data into state.
- Store a selected ID and derive the selected item from the current list.
- Use props directly when a value should track the parent.
- A deliberate editable draft can be separate state when it is meant to diverge.
- Use useMemo only when profiling justifies caching an expensive calculation.
- Keep render calculations pure and avoid mutating source data.
