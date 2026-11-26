# 022. Designing reusable component APIs - Example

## Goal

Build two small components and review them from their call sites. The example demonstrates:

- An optional prop with a useful default.
- `children` for the main content and a named JSX prop for an action region.
- A controlled interaction with a current value and a change callback.
- A component that uses semantic native elements and exposes state accessibly.
- Keeping each reusable component focused on presentation and interaction while the parent owns application decisions.

## Files

In an existing React project, create or update:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required. The `data-tone` attribute is an explicit styling hook; add CSS rules for it if you want to change the appearance.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
import { useState } from 'react';

function Notice({ tone = 'info', title, actions, children }) {
  return (
    <aside data-tone={tone}>
      <h2>{title}</h2>
      <div>{children}</div>
      {actions && <div>{actions}</div>}
    </aside>
  );
}

function Disclosure({ id, title, isOpen, onOpenChange, children }) {
  return (
    <section>
      <h2>{title}</h2>
      <button
        type="button"
        aria-expanded={isOpen}
        aria-controls={id}
        onClick={() => onOpenChange(!isOpen)}
      >
        {isOpen ? 'Hide details' : 'Show details'}
      </button>
      <div id={id} hidden={!isOpen}>
        {children}
      </div>
    </section>
  );
}

export default function App() {
  const [deliveryOpen, setDeliveryOpen] = useState(false);
  const [message, setMessage] = useState('');

  function handleViewProfile() {
    setMessage('The profile action was requested.');
  }

  return (
    <main>
      <h1>Designing reusable component APIs</h1>
      <p>
        Clear props make it easy to understand what each component needs and
        what callers can customize.
      </p>

      <Notice
        tone="success"
        title="Profile saved"
        actions={
          <button type="button" onClick={handleViewProfile}>
            View profile
          </button>
        }
      >
        Your profile changes have been saved.
      </Notice>
      <p role="status">{message}</p>

      <Notice title="Tip">
        This notice uses the default <code>info</code> tone.
      </Notice>

      <section aria-labelledby="delivery-heading">
        <h2 id="delivery-heading">Controlled component</h2>
        <p>
          The parent owns whether the details are open and passes both the
          current value and the callback.
        </p>
        <Disclosure
          id="delivery-details"
          title="Delivery details"
          isOpen={deliveryOpen}
          onOpenChange={setDeliveryOpen}
        >
          <p>Orders usually arrive within three to five business days.</p>
        </Disclosure>
      </section>
    </main>
  );
}
~~~

## Read the API at the call site

The `Notice` calls use the same component in two ways. The first supplies a named `tone` variant and an `actions` slot. The second omits `tone`, so `Notice` uses its `info` default. Both put the primary message inside `children`.

The action button is supplied as JSX by the caller. That keeps the `Notice` component independent of what �view profile� means. `App` owns the handler and decides what to do when the action is selected.

`Disclosure` has a controlled contract:

- `isOpen` is the current value from the parent.
- `onOpenChange` reports the next requested value.
- `id` connects the button's `aria-controls` to the details region.
- `title` and `children` provide the component's content.

When the button is activated, `Disclosure` asks for the opposite value by calling `onOpenChange`. The parent passes `setDeliveryOpen`, which updates its state. On the next render, `Disclosure` receives the new `isOpen` value and updates both the button label and `aria-expanded`. The `hidden` attribute keeps the controlled region in the document while hiding its content when closed.

The parent uses `useState` to make the interaction runnable. The point of this chapter is the contract: the child does not keep a competing copy of `isOpen`, and the parent decides who owns the value.

## Try these changes

1. Remove `tone="success"` from the first `Notice` and observe that the default is `info`.
2. Put two elements inside `children`, such as a paragraph and a link, and see that the component can contain caller-provided markup.
3. Change the `actions` value to two buttons. The component does not need a new prop for each action.
4. Change the `Disclosure` title and content without changing its interaction contract.
5. Add a second disclosure with a unique `id` and a separate state value. Consider whether each disclosure should open independently or whether the parent should allow only one open item at a time.
6. Add CSS selectors for `[data-tone="success"]` and `[data-tone="info"]`. The component uses an explicit `data-*` styling hook instead of forwarding an arbitrary prop to the DOM.

## API review

| Component | API choice | Why it helps |
| --- | --- | --- |
| `Notice` | `tone = 'info'` | The optional visual variant has a predictable fallback. |
| `Notice` | `children` | Callers provide the primary message and can compose markup. |
| `Notice` | `actions` | A named region makes it clear where action content appears. |
| `Disclosure` | `isOpen` with `onOpenChange` | The parent owns the current value and receives requested changes. |
| `Disclosure` | `id` | The button can reference the matching content region accessibly. |
| `Disclosure` | Native `button`, `aria-expanded`, and `hidden` | The interaction exposes its state and uses built-in browser semantics. |

The API avoids separate props for every possible action button, an ambiguous `state` prop, and a private open state that could disagree with the parent.

## Practice

1. Add a `Notice` call with a JSX link in `children` and a button in `actions`.
2. Add a `compact` visual variant only if the UI needs it. Decide whether a named `size` prop or CSS belongs in this component's contract.
3. Change `Disclosure` so its callback receives an object like `{ nextOpen }`. Decide whether that extra wrapper makes the API clearer for this component.
4. Sketch an uncontrolled disclosure using an initial `defaultOpen` prop. Write down how its contract differs from the controlled version before implementing it.
5. Review every prop and explain what caller decision it represents. Remove any prop that has no clear role.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="021-pure-components-and-predictable-rendering.md">021 - Pure components and predictable rendering - Example</a></td><td><a href="../lessons/022-designing-reusable-component-apis.md">Lesson</a></td><td><a href="../qnas/022-designing-reusable-component-apis.md">Q&amp;A</a></td><td><a href="023-thinking-in-react-and-component-trees.md">023 - Thinking in React: from a design to a component tree - Example</a></td></tr>
  </tbody>
</table>
