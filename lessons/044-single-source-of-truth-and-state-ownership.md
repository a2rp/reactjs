# Single Source of Truth and State Ownership

## Why ownership matters

React applications often show the same changing fact in more than one place. A selected product may appear in a picker, a details panel, and a summary. A message may appear in an editor and a preview. Those views need to agree.

If each component keeps its own copy of the same fact, an update can change one copy while leaving another stale. Instead, choose one owner for each unique piece of state. That owner keeps the current value and passes it to every component that needs it.

**Single source of truth** means one authoritative state value for one fact within the part of the application coordinating it. It does not mean putting every state value in the root component. Independent facts can have different owners.

## Learning goals

After this lesson, you should be able to:

- identify when two state values represent the same fact;
- choose one owner for a shared value;
- pass that value to every consumer that needs it;
- derive display values instead of storing redundant copies;
- keep independent state near the component that uses it;
- recognize when two similar-looking values are actually different facts;
- avoid synchronizing duplicate state with Effects.

## One fact, one owner

Suppose a page lets the user select a product. The selected product appears in three places:

- the picker shows which product is selected;
- a details panel shows its description and price;
- a summary repeats its name and price.

These are three views of one selection. If each component owns a separate selected product, the values can disagree. Instead, their common parent owns one selected product ID and passes the resulting value to its children.

~~~text
ProductPage
  state: selectedProductId
  ├── ProductPicker receives selectedProductId
  ├── ProductDetails receives selectedProduct
  └── SelectionSummary receives selectedProduct
~~~

The parent is the owner. The children are consumers. A child that lets the user make a selection reports the requested ID through a callback.

## Find the owner for a piece of state

Use these questions when deciding where a value belongs:

1. What exact fact does this value represent?
2. Which components need to read it?
3. Which component or interaction can change it?
4. Which components must stay consistent after it changes?
5. What is the closest common parent that can coordinate those consumers?
6. Can the value be derived from another state value or prop instead of stored separately?

Put shared state in the closest component that can coordinate all consumers. Keep unrelated state closer to the component that uses it.

This is a decision for each piece of state, not a rule that every value must share one owner.

## Pass one value to multiple consumers

A parent can pass the same value to several children:

~~~jsx
function ProductPage() {
  const [selectedProductId, setSelectedProductId] = useState("notebook");

  const selectedProduct = products.find(
    (product) => product.id === selectedProductId
  );

  return (
    <>
      <ProductPicker
        products={products}
        selectedProductId={selectedProductId}
        onSelectProduct={setSelectedProductId}
      />
      <ProductDetails product={selectedProduct} />
      <SelectionSummary product={selectedProduct} />
    </>
  );
}
~~~

The picker uses the selected ID as its value and reports a new selection. The details and summary receive the product derived from that ID. When the ID changes, all three children render from the same source.

A callback such as onSelectProduct can make the child's role clear. The picker does not need to know which state hook or data structure the parent uses.

## Store the minimal source value

When one value can be derived from another, store only the source value when practical.

For the product page, the selected product can be found from the product list and the selected ID. Storing both selectedProductId and selectedProduct creates two pieces of state that must be kept synchronized:

~~~jsx
// Usually redundant if products and selectedProductId already exist.
const [selectedProductId, setSelectedProductId] = useState("notebook");
const [selectedProduct, setSelectedProduct] = useState(products[0]);
~~~

If the selected ID changes but the selected object does not, the UI can show conflicting choices. Derive the object instead:

~~~jsx
const selectedProduct = products.find(
  (product) => product.id === selectedProductId
);
~~~

Other common examples of derived values include:

- a total calculated from cart items;
- a filtered list calculated from items and a query;
- whether a button is disabled based on a form's values;
- a character count calculated from a message;
- whether an item is selected based on an active ID.

Derived values should normally be calculated while rendering. Do not add an Effect just to copy a value from one state variable into another.

## A shared value can have several consumers

A consumer does not need to own the value. The owner can distribute it through props:

~~~jsx
function SelectionSummary({ product }) {
  if (!product) {
    return <p>No product selected.</p>;
  }

  return <p>{product.name}: {"$"}{product.price}</p>;
}
~~~

If the parent passes a product prop, the summary should render that prop. It should not create its own selectedProduct state from the prop. That would create a second owner for what is supposed to be one fact.

A component can still own state for a different fact. For example, a picker might keep its search query locally if only the picker uses it, while the page owns the selected product ID.

## Keep independent state local

Consider the product picker search field. If the query only filters options inside ProductPicker, that query can belong to ProductPicker:

~~~jsx
function ProductPicker({ products, selectedProductId, onSelectProduct }) {
  const [query, setQuery] = useState("");

  const visibleProducts = products.filter((product) =>
    product.name.toLowerCase().includes(query.toLowerCase())
  );

  // Render the local search field and the controlled selection.
}
~~~

There are two different facts here:

- selectedProductId is shared by the page, picker, details, and summary;
- query is used only by the picker.

They should not be forced into the same owner just because they appear in one part of the interface.

## Similar values may represent different facts

Avoid duplicate copies of the **same** fact, but do not collapse values that represent different stages or meanings.

For example, an edit form may have:

- savedProfile: the last profile received from or saved to a server;
- draftProfile: the user's unsaved edits.

These values may initially match, but they serve different purposes. The draft can change before the user saves. A reset action may replace the draft with the saved profile; a successful save may update both according to the application's behavior.

This is not redundant state when the distinction is intentional and the UI needs both meanings. Name values according to their roles so their ownership is clear.

Other legitimate pairs include:

- current route and a draft route being edited;
- server results and a locally edited copy;
- original text and an unsaved draft;
- current selection and a temporary hover target.

When deciding whether two values are duplicates, ask whether they represent the same fact at the same point in the workflow.

## Ownership and updates

The owner is responsible for the authoritative value and the rules for changing it. Consumers can request updates through callback props.

~~~jsx
<ProductPicker
  selectedProductId={selectedProductId}
  onSelectProduct={(productId) => setSelectedProductId(productId)}
/>
~~~

The callback acts as the boundary between the child interaction and the parent's state. The parent may validate the request, select a fallback, update related values, or ignore it. A child should not need to know those details.

In a small component, passing a state setter directly may be acceptable:

~~~jsx
onSelectProduct={setSelectedProductId}
~~~

For reusable components or meaningful domain actions, a named callback such as onSelectProduct communicates intent and keeps the parent's state representation private.

## Do not use Effects to keep copies aligned

This pattern stores the same selection twice:

~~~jsx
const [selectedProductId, setSelectedProductId] = useState("notebook");
const [selectedProduct, setSelectedProduct] = useState(products[0]);

useEffect(() => {
  setSelectedProduct(
    products.find((product) => product.id === selectedProductId)
  );
}, [products, selectedProductId]);
~~~

It introduces an extra update step. The ID can change before the Effect updates the object, and the code has to maintain two values that can be calculated together. Prefer deriving selectedProduct from selectedProductId during rendering.

Effects are for synchronizing with external systems, such as a browser API or network connection, not for copying one piece of React state into another when it can be derived.

## The owner does not have to be the root

Choosing a single source of truth does not mean building one enormous state object in App. The owner should be the nearest component that can coordinate the consumers.

For a value used by siblings in a small section, a section component may own it. For a value shared by different distant branches, a higher ancestor or Context may be appropriate. The ownership decision answers “who controls this fact?” Context answers “how can descendants receive it without passing props through every layer?” They are related design questions, not the same question.

## Common mistakes

### Each sibling keeps its own copy

The components can display different values after an update. Move the shared value to their closest common parent and pass it to each consumer.

### The parent stores both an ID and the matching object

If one is reliably derived from the other and the source data, storing both invites inconsistency. Store the minimal source and derive the rest.

### Every state value is moved to one component

Independent state does not need a shared owner. Keep local state near its only consumer to avoid unnecessary props and coupling.

### A child copies a prop into local state

The local copy will not automatically follow later prop changes. Render from the prop when it represents the controlled value. Use separate local state only when it represents a distinct fact, such as an editable draft.

### Effects synchronize values that could be derived

This adds an asynchronous or later update step where a direct calculation would do. Derive data during rendering when it is based on current props and state.

### “Single source of truth” is interpreted as “one state object for everything”

The phrase applies to one fact within its coordination boundary. A page can have many state values and owners, as long as each shared fact has a clear authoritative owner.

## Practice

1. A selected category appears in a dropdown, a heading, and a filtered result list. What should be the source of truth?
2. A product ID and a product object are both stored, even though the object can be found in a product list. What is the risk?
3. A search query is used only by a single picker. Does it need to be owned by the page?
4. A profile editor has a saved profile and an unsaved draft. Are these necessarily duplicate state?
5. Several siblings need to display the same cart total. Should each calculate and store its own copy?
6. A prop is copied to state so a child can display it. What should you check before keeping that local state?
7. Does single source of truth require that all application state live at the root?
8. What is the difference between ownership and distribution of a value?

Suggested answers:

1. One category value owned by the closest common parent that coordinates the dropdown, heading, and results.
2. The ID and object can disagree after an update. Derive the object from the ID and product list when practical.
3. No. Keep it in the picker if no other component needs it.
4. No. Saved data and unsaved edits represent different stages, so both may be needed.
5. Usually no. Keep the cart data in one owner and derive the total for each consumer.
6. Decide whether the local state represents a genuinely different fact, such as a draft. If it is the same controlled value, render from the prop.
7. No. Each piece of state can have the nearest useful owner.
8. Ownership is which component controls the authoritative value. Distribution is how consumers receive it, often through props or Context.

## Summary

- Choose one authoritative owner for each shared fact.
- Put shared state in the closest component that can coordinate its consumers.
- Pass the current value to every consumer that needs it.
- Derive values from existing state and props instead of maintaining redundant copies.
- Keep independent state local.
- Preserve separate values when they represent different facts, such as saved data and an unsaved draft.
- Use callbacks for children to request changes; let the owner apply the update rules.
