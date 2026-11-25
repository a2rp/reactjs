# Styling Strategies, CSS Modules, and Theming in React

## Learning goals

After this lesson, you should be able to:

- Explain how React connects JSX elements to CSS without prescribing a styling system.
- Choose between global stylesheets, CSS Modules, inline styles, utility classes, and CSS-in-JS for a project.
- Use a CSS Module to scope component class names through a supported build tool.
- Compose conditional class names without creating fragile style logic.
- Build themes from CSS custom properties and a small semantic token system.
- Connect a theme choice to React state or a DOM attribute accessibly.
- Understand how CSS Modules interact with the regular cascade, specificity, responsive rules, and accessibility.

## 1. React and CSS have separate jobs

React describes the UI tree and updates DOM properties when the rendered result changes. CSS controls presentation: layout, color, spacing, typography, animation, and responsive behavior.

React does not require one styling architecture. A React element uses `className` to set the DOM `class` attribute, and its `style` prop accepts a JavaScript object for inline declarations. CSS can live in a normal stylesheet, a CSS Module, generated styles, or another system supported by the app's tooling.

```jsx
function Notice() {
  return <aside className="notice">Saved successfully.</aside>;
}
```

```css
.notice {
  border-inline-start: 0.25rem solid rebeccapurple;
  padding: 1rem;
}
```

Keeping the responsibilities clear helps: React chooses which content and state to render; CSS describes how that content should look.

## 2. Styling options

### Global CSS

A regular stylesheet makes selectors available throughout the document. It is a good fit for resets, font declarations, shared design tokens, and broad element rules.

```css
:root {
  --color-text: #182033;
  --color-surface: #fff;
}

body {
  margin: 0;
  color: var(--color-text);
  background: var(--color-surface);
}
```

Global selectors are easy to share, but broad names such as `.card`, `.title`, or `button` can accidentally affect unrelated components. Name global classes deliberately and keep resets and shared tokens intentional.

### CSS Modules

A CSS Module is a stylesheet whose class names are exposed as local mappings by a build tool. A common naming convention is `ComponentName.module.css`:

```text
ProfileCard.tsx
ProfileCard.module.css
```

```css
/* ProfileCard.module.css */
.card {
  padding: 1rem;
  border-radius: 0.75rem;
}

.title {
  margin: 0;
}
```

```tsx
import styles from "./ProfileCard.module.css";

export function ProfileCard() {
  return (
    <article className={styles.card}>
      <h2 className={styles.title}>Profile</h2>
    </article>
  );
}
```

The tooling maps local class names to generated names so unrelated modules can both define `.title`. React itself does not implement CSS Modules; support and details come from the framework or build tool. A TypeScript project also needs the toolchain's CSS Module type declarations so imports are understood by the type checker.

CSS Modules reduce accidental class-name collisions. They do not create a complete style sandbox: declarations still participate in CSS inheritance, the cascade, custom properties, media queries, and browser layout.

### Inline styles

The `style` prop accepts an object with JavaScript property names:

```jsx
function Swatch({ color, size }) {
  return (
    <span
      className="swatch"
      style={{ backgroundColor: color, width: size, height: size }}
    />
  );
}
```

This is useful when a value genuinely comes from data, such as a measured width or a user-selected color. React converts many numeric values to pixel lengths, while unitless CSS properties remain unitless. CSS property names are usually camel-cased, such as `backgroundColor`.

Inline styles are less suitable for the whole visual system. They do not naturally express pseudo-classes such as `:hover`, media queries, keyframes, or a large set of component variants. Put those rules in CSS. Do not repeatedly build a stylesheet as a string during rendering.

### Utility classes and CSS-in-JS

Utility-first systems provide small classes for common declarations. They can make layout changes visible next to markup, but they still need conventions and a coherent design system.

CSS-in-JS libraries generate or manage styles from JavaScript. They can offer component-oriented APIs and dynamic variants, with trade-offs in runtime work, build setup, debugging, server rendering, and team conventions. Check the active framework and project requirements before choosing a library.

There is no styling option that is best for every app. A project can combine options, but a clear division of responsibilities prevents duplicate ways to express the same styles.

## 3. Component classes and conditional styles

A component can add a variant class based on its props or state:

```jsx
function Alert({ tone, children }) {
  const className = tone === "warning"
    ? "alert alert--warning"
    : "alert alert--info";

  return <div className={className}>{children}</div>;
}
```

With a CSS Module, use the exported mapping:

```jsx
const className = tone === "warning"
  ? `${styles.alert} ${styles.warning}`
  : `${styles.alert} ${styles.info}`;
```

For several independent boolean variants, build the class list from an array:

```jsx
const className = [
  styles.button,
  primary && styles.primary,
  disabled && styles.disabled,
]
  .filter(Boolean)
  .join(" ");
```

This pattern is adequate for a small component. If the project uses a class-name helper, follow that established convention instead of introducing another dependency just for one conditional.

Keep component APIs semantic. A `tone="warning"` prop communicates intent better than exposing an arbitrary class-name string to every caller. An explicit `className` escape hatch can still be useful for reusable library components, but document whether caller styles extend or replace defaults.

## 4. Separate shared tokens from local details

A token is a named design value, such as a surface color or spacing size. CSS custom properties are a natural way to share tokens and let descendants inherit values.

```css
:root {
  --space-1: 0.5rem;
  --space-2: 1rem;
  --radius-card: 0.75rem;

  --color-text: #182033;
  --color-muted: #596579;
  --color-page: #f3f6fb;
  --color-surface: #fff;
  --color-border: #d9e0eb;
  --color-accent: #3157d5;
}

.card {
  padding: var(--space-2);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-card);
  color: var(--color-text);
  background: var(--color-surface);
}
```

Prefer semantic tokens such as `--color-surface` and `--color-text` over a token named only after a raw color such as `--blue-500`. Semantic names tell components what a value means; a theme can change the actual color without changing component rules.

Keep the token source stable and global, then consume tokens from local CSS Modules. A component should not redefine the entire design system independently.

## 5. Implement a theme with an attribute and tokens

One simple pattern places the selected theme on a stable ancestor:

```css
.app-shell {
  --page: #f3f6fb;
  --surface: #fff;
  --text: #182033;
  --muted: #596579;
  --border: #d9e0eb;
  --accent: #3157d5;

  color-scheme: light;
  min-height: 100vh;
  color: var(--text);
  background: var(--page);
}

.app-shell[data-theme="dark"] {
  --page: #111827;
  --surface: #1c2636;
  --text: #edf2fa;
  --muted: #a9b5c7;
  --border: #354257;
  --accent: #a9bcff;

  color-scheme: dark;
}

.card {
  border: 1px solid var(--border);
  color: var(--text);
  background: var(--surface);
}
```

A React component can keep the choice in state and set the attribute:

```jsx
function App() {
  const [theme, setTheme] = useState("light");

  return (
    <main className="app-shell" data-theme={theme}>
      <button
        type="button"
        onClick={() => setTheme(theme === "light" ? "dark" : "light")}
        aria-pressed={theme === "dark"}
      >
        {theme === "dark" ? "Use light theme" : "Use dark theme"}
      </button>
    </main>
  );
}
```

`color-scheme` lets browser-provided controls match the active theme. The CSS custom properties provide the colors for the rest of the interface.

For a small app, local component state may be enough. For a larger app, store the theme in a shared provider or another existing app-wide state solution so distant components can read the same preference. Avoid duplicating theme state in multiple components.

### Respecting the operating-system preference

A site can use `prefers-color-scheme` when it wants the browser or operating system to select a theme:

```css
:root {
  color-scheme: light dark;
}

@media (prefers-color-scheme: dark) {
  :root {
    --page: #111827;
    --surface: #1c2636;
    --text: #edf2fa;
  }
}
```

If the app provides a manual choice, define the priority clearly: system preference by default, with an explicit saved light/dark choice taking precedence. During server rendering, avoid reading browser-only storage to produce different initial markup on the server and client. Choose a hydration-safe initial value, then apply a saved preference through the framework's supported strategy.

## 6. CSS Modules still use the cascade

CSS Modules scope class names, but normal CSS rules still apply. The computed style can be affected by:

- selector specificity and source order;
- inherited properties and custom properties;
- media and container queries;
- cascade layers;
- inline declarations and browser defaults.

A module class does not automatically outrank every other selector. Avoid escalating specificity or adding `!important` as the first fix. Prefer a clear component variant or a token change at the correct boundary.

Cascade layers can establish broad ordering for larger stylesheets:

```css
@layer reset, base, components, utilities;

@layer base {
  body {
    margin: 0;
  }
}
```

Layer behavior depends on the order and whether rules are layered. Introduce layers deliberately and ensure framework or third-party styles are placed where intended.

Some CSS Module systems offer a `:global(...)` escape hatch. Its exact syntax is tool-specific and makes a selector global again; use it sparingly. Put truly global reset and document styles in a clearly global stylesheet.

## 7. Responsive and accessible styling

CSS handles responsive presentation with media or container queries. It is usually clearer to keep layout adaptation in CSS instead of reading viewport dimensions during every React render.

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
}

@media (max-width: 48rem) {
  .card-grid {
    grid-template-columns: 1fr;
  }
}
```

Keep meaning in semantic HTML and interaction state in React; CSS should not be the only way information is conveyed. A visual theme must retain readable contrast. Interactive controls need a visible keyboard focus indicator. Do not remove outlines without providing a clear replacement.

Respect reduced-motion preferences for nonessential transitions and animation:

```css
.theme-transition {
  transition: color 160ms ease, background-color 160ms ease;
}

@media (prefers-reduced-motion: reduce) {
  .theme-transition {
    transition: none;
  }
}
```

Avoid using color alone to show an important state. Pair color with text, an icon with an accessible name, or another clear cue.

## 8. Avoid common styling mistakes

- **Using `class` in JSX:** use `className`.
- **Assuming CSS Modules are a React feature:** they require project tooling support.
- **Putting every declaration in `style`:** use CSS for responsive rules, pseudo-classes, and reusable variants.
- **Treating module classes as isolated from inheritance:** custom properties and inherited values can still cross component boundaries.
- **Using raw colors throughout components:** centralize shared values as semantic tokens.
- **Creating multiple independent theme states:** keep the choice at one shared boundary.
- **Using a CSS class to communicate state without accessible markup:** express the state in semantic text or ARIA where appropriate.
- **Using `!important` to fight unclear ownership:** simplify the cascade or provide a variant.
- **Reading browser storage during server rendering:** browser APIs do not exist on the server and may cause different initial markup.
- **Naming selectors too generically in global CSS:** use module-local classes or deliberate project naming.

## 9. A practical selection guide

| Need | A reasonable starting point |
| --- | --- |
| Document defaults, resets, shared tokens | Global stylesheet |
| Component-specific classes with fewer naming collisions | CSS Modules, if supported by the toolchain |
| A value that is directly data-driven, such as measured width | Inline `style` for that value |
| A few fixed component variants | Semantic props mapped to classes |
| A team-wide utility convention | Utility classes supported by the project |
| Dynamic style generation with library-specific features | CSS-in-JS, after reviewing its runtime and build trade-offs |

The simplest setup that supports the app's layout, reuse, theming, and team workflow is usually a good starting point. Keep the choice consistent and document any project-specific conventions.

## Summary

React connects UI elements to CSS through attributes such as `className` and the `style` object, while leaving stylesheet organization to the project. CSS Modules provide local class mappings when supported by the build tool; they reduce naming collisions without replacing the cascade. CSS custom properties make shared semantic tokens and light/dark themes practical. React state or a shared preference can select a theme, while CSS handles responsive layout, focus styles, contrast, and motion preferences.
