# Example: React 19.3 View Transitions, Fragment refs, and browser-only rendering

This example set demonstrates the React 19.3 APIs in a small product catalog:

1. the runnable catalog uses a React Transition, View Transition Classes, transition types, and a shared thumbnail-to-detail image;
2. a Fragment ref observes sibling cards without adding a wrapper; and
3. a server-rendering snippet uses `use(browser())` for a value that exists only in the browser.

The Trusted Types section shows how to verify the React integration without pretending that React sanitizes data.

## Requirements

- React and React DOM 19.3 or later, using compatible versions.
- A React DOM app with the modern JSX transform.
- A browser that supports the View Transition API for the animation. The React UI must still make sense when motion is unavailable or reduced.

For a new Vite app:

~~~sh
npm create vite@latest react-19-3-demo -- --template react
cd react-19-3-demo
npm install
npm install react@^19.3 react-dom@^19.3
npm run dev
~~~

Keep the two React packages compatible. If your package manager resolves a different version range, inspect the installed versions before using these APIs.

## Part 1: A catalog with page and shared-element transitions

### 1. Create `src/App.jsx`

~~~jsx
import {
  ViewTransition,
  addTransitionType,
  startTransition,
  useState,
} from "react";
import "./App.css";

const products = [
  {
    id: "moss",
    name: "Moss Notebook",
    description: "A durable notebook for field notes and daily ideas.",
    color: "#688b68",
    label: "Soft green cover",
  },
  {
    id: "sunrise",
    name: "Sunrise Planner",
    description: "A weekly planner with room for priorities and reflections.",
    color: "#d78a55",
    label: "Warm orange cover",
  },
  {
    id: "ocean",
    name: "Ocean Sketchbook",
    description: "A lay-flat sketchbook for drawings and studies.",
    color: "#4c7e9c",
    label: "Deep blue cover",
  },
];

function imageTransitionName(productId) {
  return "react-demo-product-image-" + productId;
}

function ProductArtwork({ product }) {
  return (
    <div
      className="product-artwork"
      style={{ "--artwork-color": product.color }}
      role="img"
      aria-label={product.label}
    >
      <span aria-hidden="true">{product.name.slice(0, 1)}</span>
    </div>
  );
}

function ProductCard({ product, onOpen }) {
  return (
    <article className="product-card">
      <button
        className="product-card__button"
        type="button"
        onClick={onOpen}
        aria-label={"View " + product.name}
      >
        <ViewTransition
          name={imageTransitionName(product.id)}
          share="product-image-share"
          default="none"
        >
          <ProductArtwork product={product} />
        </ViewTransition>

        <span className="product-card__name">{product.name}</span>
        <span className="product-card__hint">View details</span>
      </button>
    </article>
  );
}

function CatalogPage({ onOpen }) {
  return (
    <ViewTransition
      enter={{ backward: "page-enter-from-left" }}
      exit={{ forward: "page-exit-to-left" }}
      default="none"
    >
      <main className="page" aria-labelledby="catalog-title">
        <p className="eyebrow">React 19.3 demo</p>
        <h1 id="catalog-title">Field goods</h1>
        <p className="intro">
          Choose an item to see its details. The image is shared between the
          catalog card and the detail page.
        </p>

        <div className="product-grid">
          {products.map((product) => (
            <ProductCard
              key={product.id}
              product={product}
              onOpen={() => onOpen(product.id)}
            />
          ))}
        </div>
      </main>
    </ViewTransition>
  );
}

function ProductPage({ product, onBack }) {
  return (
    <ViewTransition
      enter={{ forward: "page-enter-from-right" }}
      exit={{ backward: "page-exit-to-right" }}
      default="none"
    >
      <main className="page" aria-labelledby="product-title">
        <button className="back-button" type="button" onClick={onBack}>
          <span aria-hidden="true">←</span> All products
        </button>

        <article className="product-detail">
          <ViewTransition
            name={imageTransitionName(product.id)}
            share="product-image-share"
            default="none"
          >
            <ProductArtwork product={product} />
          </ViewTransition>

          <div>
            <p className="eyebrow">Notebook collection</p>
            <h1 id="product-title">{product.name}</h1>
            <p className="intro">{product.description}</p>
            <p className="detail-note">
              The selected artwork uses the same View Transition name as its
              catalog thumbnail.
            </p>
          </div>
        </article>
      </main>
    </ViewTransition>
  );
}

export default function App() {
  const [selectedId, setSelectedId] = useState(null);
  const selectedProduct = products.find((product) => product.id === selectedId);

  function openProduct(id) {
    startTransition(() => {
      addTransitionType("forward");
      setSelectedId(id);
    });
  }

  function returnToCatalog() {
    startTransition(() => {
      addTransitionType("backward");
      setSelectedId(null);
    });
  }

  return (
    <div className="app-shell">
      {selectedProduct ? (
        <ProductPage product={selectedProduct} onBack={returnToCatalog} />
      ) : (
        <CatalogPage onOpen={openProduct} />
      )}
    </div>
  );
}
~~~

The page components return their `<ViewTransition>` first, so their enter/exit boundaries can be detected. The artwork boundaries are nested deeper because they connect the matching visual element across the catalog and detail trees.

### 2. Create `src/App.css`

~~~css
:root {
  color: #17211c;
  background: #f5f4ee;
  font-family: system-ui, sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button {
  font: inherit;
}

.app-shell {
  min-height: 100vh;
}

.page {
  width: min(100% - 2rem, 68rem);
  margin-inline: auto;
  padding-block: clamp(2rem, 7vw, 5rem);
}

.eyebrow {
  margin: 0 0 0.5rem;
  color: #526a58;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

h1 {
  margin: 0;
  font-size: clamp(2.25rem, 6vw, 4rem);
  letter-spacing: -0.05em;
}

.intro {
  max-width: 42rem;
  color: #4b5750;
  font-size: 1.1rem;
  line-height: 1.65;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 14rem), 1fr));
  gap: 1.25rem;
  margin-top: 2.5rem;
}

.product-card {
  min-width: 0;
  overflow: hidden;
  border: 1px solid #d9ddd5;
  border-radius: 1rem;
  background: #fff;
}

.product-card__button {
  display: grid;
  width: 100%;
  gap: 0.75rem;
  padding: 0.75rem;
  border: 0;
  background: transparent;
  color: inherit;
  text-align: left;
  cursor: pointer;
}

.product-card__button:focus-visible,
.back-button:focus-visible {
  outline: 3px solid #215d43;
  outline-offset: 3px;
}

.product-artwork {
  display: grid;
  min-height: 12rem;
  place-items: center;
  overflow: hidden;
  border-radius: 0.65rem;
  background:
    radial-gradient(circle at 70% 25%, rgb(255 255 255 / 42%), transparent 30%),
    linear-gradient(145deg, var(--artwork-color), #263b31);
  color: #fff;
}

.product-artwork span {
  display: grid;
  width: 5rem;
  aspect-ratio: 0.72;
  place-items: center;
  border: 1px solid rgb(255 255 255 / 65%);
  border-radius: 0.3rem;
  background: rgb(255 255 255 / 15%);
  font-size: 3rem;
  font-weight: 700;
  box-shadow: 0 1rem 2rem rgb(0 0 0 / 18%);
}

.product-card__name {
  font-size: 1.05rem;
  font-weight: 700;
}

.product-card__hint,
.detail-note {
  color: #526a58;
  font-size: 0.9rem;
}

.back-button {
  margin-bottom: 2rem;
  padding: 0.65rem 0.9rem;
  border: 1px solid #c7d0c7;
  border-radius: 0.5rem;
  background: #fff;
  color: #17211c;
  cursor: pointer;
}

.product-detail {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 20rem), 1fr));
  align-items: center;
  gap: clamp(1.5rem, 5vw, 4rem);
}

.product-detail .product-artwork {
  min-height: 26rem;
}

.detail-note {
  max-width: 32rem;
  line-height: 1.6;
}

::view-transition-new(.page-enter-from-right) {
  animation: page-enter-from-right 180ms ease-out both;
}

::view-transition-new(.page-enter-from-left) {
  animation: page-enter-from-left 180ms ease-out both;
}

::view-transition-old(.page-exit-to-left) {
  animation: page-exit-to-left 140ms ease-in both;
}

::view-transition-old(.page-exit-to-right) {
  animation: page-exit-to-right 140ms ease-in both;
}

::view-transition-group(.product-image-share) {
  animation-duration: 260ms;
  animation-timing-function: cubic-bezier(0.2, 0.75, 0.25, 1);
}

@keyframes page-enter-from-right {
  from {
    opacity: 0;
    transform: translateX(1rem);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes page-enter-from-left {
  from {
    opacity: 0;
    transform: translateX(-1rem);
  }

  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes page-exit-to-left {
  to {
    opacity: 0;
    transform: translateX(-0.75rem);
  }
}

@keyframes page-exit-to-right {
  to {
    opacity: 0;
    transform: translateX(0.75rem);
  }
}

@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(.page-enter-from-right),
  ::view-transition-group(.page-enter-from-left),
  ::view-transition-group(.page-exit-to-left),
  ::view-transition-group(.page-exit-to-right),
  ::view-transition-group(.product-image-share) {
    animation-duration: 1ms;
  }
}

@media (max-width: 38rem) {
  .product-detail .product-artwork {
    min-height: 18rem;
  }
}
~~~

### 3. Try the interaction

1. Click or keyboard-activate a product card.
2. The catalog and detail page enter/exit as a non-urgent React Transition.
3. The product artwork keeps the same `name` in both trees, so React can animate it as a shared element.
4. Use the back button to transition to the catalog.
5. Enable the operating system's reduced-motion setting and repeat the interaction.

The layout, text, and buttons work independently of the animation. The thumbnail artwork is a CSS illustration, so the example does not depend on a remote image host.

### 4. Add a transition event when CSS is insufficient

The following optional replacement for the page boundary demonstrates an imperative enter animation. Use this pattern only when CSS classes cannot express the required effect:

~~~jsx
<ViewTransition
  enter="auto"
  onEnter={(instance) => {
    if (window.matchMedia("(prefers-reduced-motion: reduce)").matches) {
      return () => {};
    }

    const animation = instance.new.animate(
      [{ opacity: 0, transform: "translateY(12px)" }, { opacity: 1, transform: "translateY(0)" }],
      { duration: 180, easing: "ease-out" }
    );

    return () => animation.cancel();
  }}
>
  <main className="page">...</main>
</ViewTransition>
~~~

If the animation is driven by JavaScript, check the user's reduced-motion preference before starting it and return a cleanup function so an interrupted transition can cancel it.

## Part 2: Observe sibling cards with a Fragment ref

A Fragment ref lets the observer act on sibling `<article>` elements without introducing a wrapper. Put this in `src/ObservedCards.jsx`:

~~~jsx
import { Fragment, useEffect, useRef } from "react";

export default function ObservedCards({ items }) {
  const cardsRef = useRef(null);

  useEffect(() => {
    const cards = cardsRef.current;
    if (!cards) return;

    const observer = new IntersectionObserver((entries) => {
      for (const entry of entries) {
        entry.target.dataset.visible = String(entry.isIntersecting);
      }
    }, { threshold: 0.15 });

    cards.observeUsing(observer);

    return () => {
      cards.unobserveUsing(observer);
      observer.disconnect();
    };
  }, []);

  function focusFirstCard() {
    cardsRef.current?.focus();
  }

  return (
    <>
      <button type="button" onClick={focusFirstCard}>
        Focus the first card
      </button>

      <Fragment ref={cardsRef}>
        {items.map((item) => (
          <article className="observed-card" key={item.id} tabIndex={-1}>
            <h2>{item.title}</h2>
            <p>{item.description}</p>
          </article>
        ))}
      </Fragment>
    </>
  );
}
~~~

Add a small visual change to `src/App.css`:

~~~css
.observed-card {
  margin-block: 1rem;
  padding: 1.25rem;
  border: 1px solid #d9ddd5;
  border-radius: 0.75rem;
  background: #fff;
  opacity: 0.55;
  transition: opacity 160ms ease;
}

.observed-card[data-visible="true"] {
  opacity: 1;
}

@media (prefers-reduced-motion: reduce) {
  .observed-card {
    transition-duration: 1ms;
  }
}
~~~

Render it from a component and pass a few items:

~~~jsx
<ObservedCards
  items={[
    { id: "one", title: "Field notes", description: "Observe the first card." },
    { id: "two", title: "Daily ideas", description: "Observe the second card." },
  ]}
/>
~~~

`observeUsing` observes first-level DOM children, so here it observes each rendered `article`. The `focus` method searches descendants for a focusable element; `tabIndex={-1}` makes each article programmatically focusable without adding it to the normal Tab sequence. In a production interface, manage focus according to the interaction and provide a visible focus style.

## Part 3: Render a browser-only value in an SSR app

This example is for a server-rendered React framework. In a client-only Vite app, the server bailout has no useful role.

Create a Client Component such as `src/LocalDraft.jsx`:

~~~jsx
"use client";

import { use, useState } from "react";
import { browser } from "react-dom";

export default function LocalDraft() {
  use(browser("The draft is stored in localStorage."));

  const [draft, setDraft] = useState(
    () => localStorage.getItem("react-demo-draft") ?? ""
  );

  function updateDraft(event) {
    const nextDraft = event.target.value;
    setDraft(nextDraft);
    localStorage.setItem("react-demo-draft", nextDraft);
  }

  return (
    <label className="draft-field">
      Draft (saved in this browser)
      <textarea value={draft} onChange={updateDraft} rows={5} />
    </label>
  );
}
~~~

Render it beneath Suspense. A Server Component may render a Client Component and a Suspense boundary in frameworks that support React Server Components:

~~~jsx
import { Suspense } from "react";
import LocalDraft from "./LocalDraft";

export default function SettingsPage() {
  return (
    <section>
      <h1>Local settings</h1>
      <Suspense fallback={<p>Loading browser settings…</p>}>
        <LocalDraft />
      </Suspense>
    </section>
  );
}
~~~

During SSR, the fallback is rendered for the browser-only component. After hydration, React renders the draft from `localStorage`. This component intentionally does not show the saved draft in the initial server HTML. If the app can provide a stable initial draft from the server, prefer passing that value and rendering useful HTML on the server.

## Part 4: Check Trusted Types integration safely

React 19.3 preserves Trusted Types objects when passing values to protected DOM APIs. Confirm the application uses an actual sanitizer-backed policy and an enforcing Content Security Policy, for example:

~~~http
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types app-html
~~~

The sanitizer and policy belong in application-owned security code. The rendering pattern should receive a value that has already been sanitized and converted to `TrustedHTML`:

~~~jsx
function ImportedArticle({ untrustedMarkup }) {
  // Application-owned helper must sanitize before it creates TrustedHTML.
  const safeHtml = sanitizeToTrustedHTML(untrustedMarkup);

  return (
    <article
      className="imported-article"
      dangerouslySetInnerHTML={{ __html: safeHtml }}
    />
  );
}
~~~

`sanitizeToTrustedHTML` is intentionally shown as an application security helper, not implemented here. Connect it to a vetted sanitizer and the site's Trusted Types policy; never implement it by simply wrapping an untrusted string in `trustedTypes.createPolicy`. React's compatibility support does not sanitize strings or make `dangerouslySetInnerHTML` safe by itself.

## Troubleshooting

### No animation appears

- Confirm both React packages are 19.3 or later.
- Confirm the state update is a Transition, a Suspense reveal, or a `useDeferredValue` update.
- For enter/exit, confirm the View Transition boundary is the first component inserted or deleted and is returned before other DOM nodes in its own component subtree.
- Confirm the CSS selector uses the exact class passed to the boundary.
- Check that another named boundary is not mounted with the same name.

### The shared image fades instead of moving

- Confirm the old and new `<ViewTransition>` components have exactly the same name.
- Confirm both trees change in one React Transition; an intermediate Suspense fallback can prevent a shared pair from existing in the same transition.
- Keep that name unique elsewhere in the mounted application.
- Verify the browser supports the View Transition API.

### The browser-only component fails during SSR

- Confirm it is a Client Component.
- Confirm the server-rendered tree includes a `<Suspense>` boundary.
- Call `use(browser())`; calling `browser()` alone does not opt out of SSR.
- Keep browser globals such as `localStorage` after the bailout call, not in module scope or before it.

### The Fragment observer sees no targets

- Confirm the Fragment contains element children, not only text.
- Remember that observer methods target first-level DOM children. If a child component renders a wrapper element, the wrapper is the observed target.
- Confirm the observer remains connected and is not disconnected before the effect runs.

## What this example demonstrates

- A View Transition only runs for an eligible React Transition.
- Transition types can choose direction-specific CSS classes.
- Matching `name` values connect a shared visual across two trees.
- Fragment refs coordinate DOM APIs across siblings without an extra wrapper.
- `use(browser())` renders a Suspense fallback on the server and browser-dependent content after hydration.
- Trusted Types requires an application-managed sanitizer and policy; React only preserves the typed values.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="118-react-19-2-features.md">118 - Example: Preserve Panel State and Profile React 19.2 Work</a></td><td><a href="../lessons/119-react-19-3-features.md">Lesson</a></td><td><a href="../qnas/119-react-19-3-features.md">Q&amp;A</a></td><td><a href="120-react-quick-reference-and-glossary.md">120 - React quick reference and glossary: examples</a></td></tr>
  </tbody>
</table>
