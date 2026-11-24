# Example: When an Effect Is Unnecessary

This self-contained React example demonstrates four different kinds of work:

- catalog filtering and cart totals are derived during rendering;
- creating a local order preview happens in the form submit handler;
- switching profiles resets that editor's draft with a stable key;
- one Effect synchronizes the browser tab title with the selected profile.

The example does not make network requests or depend on external assets. The order is explicitly a local preview, not a real purchase.

## Create the project

Create a React project with Vite:

~~~sh
npm create vite@latest unnecessary-effect-demo -- --template react
cd unnecessary-effect-demo
npm install
~~~

Replace the generated files with the following versions, then start the development server:

~~~sh
npm run dev
~~~

## Project files

~~~text
unnecessary-effect-demo/
  index.html
  src/
    App.jsx
    main.jsx
    styles.css
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f5f7fb" />
    <title>React Patterns</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

const rootElement = document.getElementById("root");

if (!rootElement) {
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

Strict Mode helps reveal Effects that do not clean up correctly. The title Effect below restores the title that was in place before its setup.

## src/App.jsx

~~~jsx
import { useEffect, useState } from "react";

const PRODUCTS = [
  { id: "notebook", name: "Field notebook", category: "Stationery", price: 8 },
  { id: "pen", name: "Everyday pen", category: "Stationery", price: 4 },
  { id: "bottle", name: "Steel water bottle", category: "Outdoor", price: 24 },
  { id: "lamp", name: "Reading lamp", category: "Home", price: 38 },
  { id: "mug", name: "Travel mug", category: "Outdoor", price: 18 },
];

const INITIAL_PROFILES = [
  { id: "alex", name: "Alex Morgan" },
  { id: "sam", name: "Sam Lee" },
];

const money = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

function ProfileEditor({ profile, onSave }) {
  const [draftName, setDraftName] = useState(profile.name);
  const [message, setMessage] = useState("");

  function handleSubmit(event) {
    event.preventDefault();

    const nextName = draftName.trim();

    if (!nextName) {
      setMessage("Enter a name before saving.");
      return;
    }

    onSave(nextName);
    setMessage("Profile name saved.");
  }

  return (
    <form className="profile-form" onSubmit={handleSubmit}>
      <label htmlFor="profile-name">Display name</label>
      <div className="inline-control">
        <input
          id="profile-name"
          value={draftName}
          onChange={(event) => setDraftName(event.target.value)}
        />
        <button type="submit">Save name</button>
      </div>
      <p className="form-message" aria-live="polite">
        {message}
      </p>
    </form>
  );
}

function ProductCard({ product, quantity, onAdd }) {
  return (
    <article className="product-card">
      <div>
        <p className="eyebrow">{product.category}</p>
        <h3>{product.name}</h3>
        <p className="price">{money.format(product.price)}</p>
      </div>
      <button type="button" onClick={() => onAdd(product)}>
        Add to cart
        <span className="button-count" aria-label={quantity + " in cart"}>
          {quantity}
        </span>
      </button>
    </article>
  );
}

export default function App() {
  const [query, setQuery] = useState("");
  const [category, setCategory] = useState("All");
  const [cart, setCart] = useState({});
  const [customerName, setCustomerName] = useState("");
  const [orders, setOrders] = useState([]);
  const [nextOrderId, setNextOrderId] = useState(1);
  const [orderMessage, setOrderMessage] = useState("");
  const [profiles, setProfiles] = useState(INITIAL_PROFILES);
  const [activeProfileId, setActiveProfileId] = useState(INITIAL_PROFILES[0].id);

  const normalizedQuery = query.trim().toLowerCase();
  const visibleProducts = PRODUCTS.filter((product) => {
    const matchesQuery = product.name.toLowerCase().includes(normalizedQuery);
    const matchesCategory = category === "All" || product.category === category;
    return matchesQuery && matchesCategory;
  });

  const cartProducts = PRODUCTS.filter((product) => cart[product.id] > 0);
  const cartItemCount = cartProducts.reduce(
    (count, product) => count + cart[product.id],
    0
  );
  const cartTotal = cartProducts.reduce(
    (total, product) => total + product.price * cart[product.id],
    0
  );

  const activeProfile =
    profiles.find((profile) => profile.id === activeProfileId) ?? profiles[0];

  useEffect(() => {
    const previousTitle = document.title;
    document.title = activeProfile.name + " | React patterns";

    return () => {
      document.title = previousTitle;
    };
  }, [activeProfile.name]);

  function handleAddToCart(product) {
    setCart((currentCart) => ({
      ...currentCart,
      [product.id]: (currentCart[product.id] ?? 0) + 1,
    }));
  }

  function handleSubmitOrder(event) {
    event.preventDefault();

    const trimmedName = customerName.trim();

    if (!trimmedName) {
      setOrderMessage("Enter your name to prepare the local order preview.");
      return;
    }

    if (cartItemCount === 0) {
      setOrderMessage("Add at least one product before preparing an order.");
      return;
    }

    const order = {
      id: nextOrderId,
      customer: trimmedName,
      itemCount: cartItemCount,
      total: cartTotal,
    };

    setOrders((currentOrders) => [order, ...currentOrders]);
    setNextOrderId((currentId) => currentId + 1);
    setCustomerName("");
    setCart({});
    setOrderMessage(
      "Order " + order.id + " was prepared locally. No request was sent."
    );
  }

  function handleSaveProfile(nextName) {
    setProfiles((currentProfiles) =>
      currentProfiles.map((profile) =>
        profile.id === activeProfileId
          ? { ...profile, name: nextName }
          : profile
      )
    );
  }

  return (
    <main className="page-shell">
      <header className="page-header">
        <div>
          <p className="eyebrow">React patterns · Chapter 055</p>
          <h1>When an Effect is unnecessary</h1>
          <p className="intro">
            Keep calculations in rendering, actions in event handlers, and
            Effects for synchronization with systems outside React.
          </p>
        </div>
        <span className="status-pill">Local demo</span>
      </header>

      <section className="panel" aria-labelledby="catalog-heading">
        <div className="section-heading">
          <div>
            <p className="eyebrow">Derived during render</p>
            <h2 id="catalog-heading">Product catalog</h2>
          </div>
          <p className="result-count">
            {visibleProducts.length}{" "}
            {visibleProducts.length === 1 ? "product" : "products"}
          </p>
        </div>

        <div className="filters">
          <label>
            Search products
            <input
              type="search"
              value={query}
              onChange={(event) => setQuery(event.target.value)}
              placeholder="Try notebook or mug"
            />
          </label>

          <label>
            Category
            <select
              value={category}
              onChange={(event) => setCategory(event.target.value)}
            >
              <option>All</option>
              <option>Stationery</option>
              <option>Outdoor</option>
              <option>Home</option>
            </select>
          </label>
        </div>

        {visibleProducts.length > 0 ? (
          <div className="product-grid">
            {visibleProducts.map((product) => (
              <ProductCard
                key={product.id}
                product={product}
                quantity={cart[product.id] ?? 0}
                onAdd={handleAddToCart}
              />
            ))}
          </div>
        ) : (
          <p className="empty-state">
            No products match those filters. Try a different search or category.
          </p>
        )}
      </section>

      <div className="lower-grid">
        <section className="panel" aria-labelledby="cart-heading">
          <div className="section-heading">
            <div>
              <p className="eyebrow">Derived during render</p>
              <h2 id="cart-heading">Cart preview</h2>
            </div>
            <span className="count-badge">{cartItemCount} items</span>
          </div>

          {cartProducts.length === 0 ? (
            <p className="empty-state">
              The cart is empty. Add a product to calculate a total.
            </p>
          ) : (
            <ul className="cart-list">
              {cartProducts.map((product) => (
                <li key={product.id}>
                  <span>
                    {product.name} × {cart[product.id]}
                  </span>
                  <strong>
                    {money.format(product.price * cart[product.id])}
                  </strong>
                </li>
              ))}
            </ul>
          )}

          <div className="cart-total">
            <span>Total</span>
            <strong>{money.format(cartTotal)}</strong>
          </div>

          <form className="order-form" onSubmit={handleSubmitOrder}>
            <label htmlFor="customer-name">Name for local order preview</label>
            <input
              id="customer-name"
              value={customerName}
              onChange={(event) => setCustomerName(event.target.value)}
              autoComplete="name"
              placeholder="Your name"
            />
            <button type="submit">Prepare order preview</button>
            <p className="form-message" aria-live="polite">
              {orderMessage}
            </p>
          </form>

          {orders.length > 0 && (
            <div className="order-history">
              <h3>Prepared previews</h3>
              <ul className="cart-list">
                {orders.map((order) => (
                  <li key={order.id}>
                    <span>
                      #{order.id} · {order.customer} · {order.itemCount} items
                    </span>
                    <strong>{money.format(order.total)}</strong>
                  </li>
                ))}
              </ul>
            </div>
          )}
        </section>

        <section className="panel" aria-labelledby="profile-heading">
          <div className="section-heading">
            <div>
              <p className="eyebrow">Reset by component identity</p>
              <h2 id="profile-heading">Profile draft</h2>
            </div>
          </div>

          <label htmlFor="profile-picker">Edit profile</label>
          <select
            id="profile-picker"
            value={activeProfile.id}
            onChange={(event) => setActiveProfileId(event.target.value)}
          >
            {profiles.map((profile) => (
              <option key={profile.id} value={profile.id}>
                {profile.name}
              </option>
            ))}
          </select>

          <ProfileEditor
            key={activeProfile.id}
            profile={activeProfile}
            onSave={handleSaveProfile}
          />

          <p className="explanation">
            Switch profiles after changing the draft. The new profile has a
            different key, so React creates a fresh editor with that profile's
            name. The browser tab title is synchronized by the one Effect in
            this demo.
          </p>
        </section>
      </div>

      <footer className="page-footer">
        <p>
          This demo stores everything in component state. Refreshing the page
          clears the cart, profiles, and prepared order previews.
        </p>
      </footer>
    </main>
  );
}
~~~

## src/styles.css

~~~css
:root {
  color: #192230;
  background: #f5f7fb;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  font-weight: 400;
  line-height: 1.5;
}

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  min-width: 320px;
  min-height: 100vh;
}

button,
input,
select {
  font: inherit;
}

button,
select {
  cursor: pointer;
}

button {
  border: 0;
}

button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 3px solid #77a7ff;
  outline-offset: 2px;
}

.page-shell {
  width: min(1120px, calc(100% - 40px));
  margin: 0 auto;
  padding: 54px 0 36px;
}

.page-header,
.section-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 24px;
}

.page-header {
  margin-bottom: 28px;
}

h1,
h2,
h3 {
  color: #172236;
  line-height: 1.2;
}

h1 {
  max-width: 720px;
  margin-top: 6px;
  font-size: clamp(2rem, 5vw, 3.25rem);
  letter-spacing: -0.045em;
}

h2 {
  font-size: 1.3rem;
  letter-spacing: -0.02em;
}

h3 {
  font-size: 1rem;
}

.intro {
  max-width: 680px;
  margin-top: 12px;
  color: #5d6879;
}

.eyebrow {
  color: #65748b;
  font-size: 0.75rem;
  font-weight: 750;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

.status-pill,
.count-badge,
.result-count {
  border-radius: 999px;
  background: #e8eefb;
  color: #344b78;
  font-size: 0.85rem;
  font-weight: 700;
  padding: 7px 12px;
  white-space: nowrap;
}

.panel {
  border: 1px solid #e0e5ee;
  border-radius: 18px;
  background: #fff;
  box-shadow: 0 12px 36px rgb(31 48 79 / 6%);
  padding: 24px;
}

.section-heading {
  align-items: center;
  margin-bottom: 20px;
}

.filters {
  display: grid;
  grid-template-columns: minmax(0, 2fr) minmax(180px, 1fr);
  gap: 14px;
  margin-bottom: 20px;
}

label {
  display: grid;
  gap: 7px;
  color: #364255;
  font-size: 0.88rem;
  font-weight: 650;
}

input,
select {
  width: 100%;
  min-height: 44px;
  border: 1px solid #cbd3df;
  border-radius: 9px;
  background: #fff;
  color: #192230;
  padding: 9px 12px;
}

input::placeholder {
  color: #8993a2;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
}

.product-card {
  display: flex;
  min-height: 160px;
  flex-direction: column;
  justify-content: space-between;
  gap: 18px;
  border: 1px solid #e5e9f0;
  border-radius: 12px;
  background: #fbfcfe;
  padding: 16px;
}

.product-card h3 {
  margin-top: 4px;
}

.price {
  margin-top: 8px;
  color: #4f5d72;
  font-weight: 700;
}

.product-card button,
.order-form button,
.profile-form button {
  display: inline-flex;
  min-height: 42px;
  align-items: center;
  justify-content: center;
  gap: 10px;
  border-radius: 9px;
  background: #264f9b;
  color: #fff;
  font-weight: 700;
  padding: 9px 13px;
}

.product-card button:hover,
.order-form button:hover,
.profile-form button:hover {
  background: #1e417f;
}

.button-count {
  display: grid;
  min-width: 22px;
  min-height: 22px;
  place-items: center;
  border-radius: 999px;
  background: rgb(255 255 255 / 20%);
  font-size: 0.76rem;
}

.lower-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  align-items: start;
  gap: 18px;
  margin-top: 18px;
}

.count-badge {
  background: #edf3ed;
  color: #316044;
}

.empty-state {
  border-radius: 10px;
  background: #f7f9fc;
  color: #687487;
  padding: 14px;
}

.cart-list {
  display: grid;
  gap: 11px;
  list-style: none;
}

.cart-list li,
.cart-total {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 16px;
}

.cart-list li {
  color: #4e5a6b;
  font-size: 0.92rem;
}

.cart-list strong,
.cart-total strong {
  color: #18253a;
  white-space: nowrap;
}

.cart-total {
  margin-top: 18px;
  border-top: 1px solid #e5e9f0;
  padding-top: 14px;
  font-weight: 700;
}

.order-form,
.profile-form {
  display: grid;
  gap: 11px;
  margin-top: 22px;
}

.form-message {
  min-height: 1.5em;
  color: #53647d;
  font-size: 0.86rem;
}

.order-history {
  margin-top: 22px;
}

.order-history h3 {
  margin-bottom: 12px;
}

.inline-control {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  gap: 9px;
}

.explanation {
  margin-top: 18px;
  color: #5d6879;
  font-size: 0.9rem;
}

.page-footer {
  margin-top: 20px;
  color: #687487;
  font-size: 0.85rem;
  text-align: center;
}

@media (max-width: 760px) {
  .page-shell {
    width: min(100% - 28px, 600px);
    padding-top: 32px;
  }

  .page-header {
    flex-direction: column;
  }

  .product-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .lower-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .panel {
    padding: 18px;
  }

  .filters,
  .product-grid {
    grid-template-columns: 1fr;
  }

  .section-heading {
    align-items: flex-start;
  }

  .result-count,
  .count-badge {
    font-size: 0.76rem;
    padding: 6px 9px;
  }

  .inline-control {
    grid-template-columns: 1fr;
  }
}
~~~

## What to notice

### Filtering and totals are not state

The visible products, cart item count, and cart total are calculated directly from the current query, category, product list, and cart quantities. There is no Effect copying those results into additional state.

### Cart and form work follow user actions

Adding a product updates the cart from its button handler. Preparing an order preview happens in the form's submit handler, where the entered name and current cart are available. The preview remains local and makes no request.

### The editor resets when its identity changes

The editor key is the profile ID. Switching to a different profile creates a new editor instance, so its draft begins with that profile's current name. Saving a name for the same profile keeps the key and therefore keeps the same editor instance.

### The document title is a real synchronization

The browser owns the document title, so the Effect synchronizes that external browser value with the selected profile. Its cleanup restores the previous title when the synchronization changes or the component is removed.

## Practice changes

1. Add a price range filter and derive its matching products during rendering.
2. Add a remove button for each cart line and update the cart in its event handler.
3. Add a cancel button that clears the current editor draft only when the user selects it.
4. Replace the local order preview with a server request initiated by the submit handler. Handle loading and errors in the form flow, and protect remote commands from accidental duplicates.
5. Remove the document-title Effect and explain which external browser value would stop staying synchronized.
