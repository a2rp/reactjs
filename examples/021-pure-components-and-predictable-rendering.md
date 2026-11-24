# 021. Pure components and predictable rendering - Example

## Goal

Build a recipe page where each component calculates its output from props. The example demonstrates:

- Rendering the same component twice with the same props.
- Changing output by changing an input prop.
- Treating the ingredients array as read-only.
- Mutating a temporary array created locally during render.
- Keeping component definitions at the top level of the file.

## Files

In an existing Vite React project, create or update these files:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required.

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
const ingredients = [
  { id: 'water', name: 'Water', amountPerServing: 250, unit: 'ml' },
  { id: 'tea', name: 'Black tea', amountPerServing: 1, unit: 'spoon' },
  { id: 'milk', name: 'Milk', amountPerServing: 100, unit: 'ml' },
];

function IngredientList({ ingredients, servings }) {
  const rows = [];

  for (const ingredient of ingredients) {
    const amount = ingredient.amountPerServing * servings;
    rows.push(
      <li key={ingredient.id}>
        {amount} {ingredient.unit} {ingredient.name}
      </li>,
    );
  }

  return <ul>{rows}</ul>;
}

function RecipeCard({ title, ingredients, servings }) {
  return (
    <article>
      <h2>{title}</h2>
      <p>Servings: {servings}</p>
      <IngredientList ingredients={ingredients} servings={servings} />
    </article>
  );
}

export default function App() {
  return (
    <main>
      <h1>Predictable recipe rendering</h1>
      <p>Each recipe is calculated from its props.</p>

      <section aria-labelledby="two-serving-recipes">
        <h2 id="two-serving-recipes">The same inputs</h2>
        <RecipeCard
          title="Tea recipe A"
          ingredients={ingredients}
          servings={2}
        />
        <RecipeCard
          title="Tea recipe B"
          ingredients={ingredients}
          servings={2}
        />
      </section>

      <section aria-labelledby="four-serving-recipe">
        <h2 id="four-serving-recipe">A different input</h2>
        <RecipeCard
          title="Tea for a group"
          ingredients={ingredients}
          servings={4}
        />
      </section>
    </main>
  );
}
~~~

## What to notice

- `RecipeCard` and `IngredientList` are declared at the top level, outside `App`.
- The ingredient data is passed as a prop and only read. The component never changes the source objects.
- Both two-serving cards receive the same `ingredients` and `servings` values, so they calculate the same ingredient amounts. A different title changes only the title output.
- The four-serving card receives a different `servings` prop, so its calculated amounts change predictably.
- `rows` is a fresh array created inside `IngredientList`. Filling that local array with `push()` is safe because it is not shared with another render or component.
- `main.jsx` wraps the app in Strict Mode. The list remains predictable even when React performs extra development rendering checks.

## Compare with an unsafe pattern

Do not add this component to the runnable app. It changes shared data while rendering, so each call can produce a different result:

~~~jsx
let guestNumber = 0;

function ImpureGuestLabel() {
  guestNumber += 1;
  return <p>Guest {guestNumber}</p>;
}
~~~

Pass the value into a pure component instead:

~~~jsx
function GuestLabel({ guestNumber }) {
  return <p>Guest {guestNumber}</p>;
}
~~~

## Practice

1. Change one recipe's `servings` prop and calculate how its ingredient quantities change.
2. Add an ingredient record with a stable `id` and confirm the list includes it.
3. Explain why changing `ingredient.amountPerServing` inside `IngredientList` would be unsafe.
4. Replace the local `rows` array loop with `map()` and keep the same keys and output.
5. Add a button with an `onClick` handler that calls a passed-in `onSave` function. Keep the call inside the handler, not in the render body.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [020 - Component boundaries and responsibility](../lessons/020-component-boundaries-and-responsibility.md) | [021 - Pure components and predictable rendering](../lessons/021-pure-components-and-predictable-rendering.md) | [Open Q&A](../qnas/021-pure-components-and-predictable-rendering.md) | [022 - Designing reusable component APIs](../lessons/022-designing-reusable-component-apis.md) |