# Styling Strategies, CSS Modules, and Theming: Questions and Answers

## React and styling fundamentals

### 1. Does React include a styling system?
React exposes DOM properties such as `className` and `style`, but it does not require a particular way to organize CSS. The framework or build tool determines how stylesheet files are loaded and whether features such as CSS Modules are supported.

### 2. Why do JSX elements use `className` instead of `class`?
JSX uses the DOM property name `className` for the element's class attribute:

```jsx
<div className="notice">Saved</div>
```

### 3. What is the difference between `className` and `style`?
`className` connects the element to CSS rules in a stylesheet. `style` receives an object of CSS declarations, useful for individual values that come from data or measurement.

### 4. When should I use inline styles?
Use them for a truly dynamic value, such as a measured width or a color selected by the user. Keep reusable styles, pseudo-classes, responsive rules, and animations in CSS.

### 5. How are CSS properties written in the React `style` object?
Most multiword properties use camel case, such as `backgroundColor` and `marginTop`. Values are strings or numbers. Many numeric length values are treated as pixels; unitless CSS properties remain unitless.

### 6. Can an inline style express `:hover` or a media query?
No. An inline style object represents declarations on one element. Use a stylesheet for pseudo-classes, media queries, keyframes, and other selector-based rules.

## CSS Modules

### 7. What is a CSS Module?
It is a CSS file that supported tooling exposes as a mapping from local class names to generated class names. A component imports the mapping and uses its properties in `className`.

### 8. Is CSS Modules built into React?
No. React does not implement CSS Modules. A framework or build tool must support them and define details such as file naming, import behavior, and generated class names.

### 9. How do I use a module class in JSX?
Name the file using the project's convention, often `Panel.module.css`, import it, then access its local class mapping:

```tsx
import styles from "./Panel.module.css";

<div className={styles.panel}>Content</div>
```

### 10. What problem do CSS Modules solve?
They reduce accidental collisions between class names in separate component stylesheets. Two modules can both call a class `.title` and still map to different generated names.

### 11. Do CSS Modules stop CSS inheritance or the cascade?
No. They scope local class names but do not isolate a component from inherited values, custom properties, browser defaults, media queries, or the cascade.

### 12. Can a CSS Module contain a global selector?
Some toolchains provide a `:global(...)` syntax, but the exact syntax is tool-specific. Global rules should be deliberate; place resets and document-level styles in a clearly global stylesheet when possible.

### 13. What if TypeScript rejects a CSS Module import?
The type checker needs declarations for CSS imports. Check the framework or build tool setup and its supported CSS Module typings before adding custom declarations. Do not use an overly broad declaration that hides unrelated import mistakes.

### 14. Should global styles also be CSS Modules?
Usually document defaults, resets, font declarations, and shared tokens belong in an intentional global stylesheet. Component-specific selectors are good candidates for modules.

### 15. Is a generated CSS Module class a security boundary?
No. It helps avoid naming collisions. It does not sanitize content or prevent a component's styles from affecting layout and inherited values.

## Class names and component APIs

### 16. How can a component select a style variant?
Give it a semantic prop such as `tone="warning"` or `size="small"` and map that value to a class. This keeps callers from needing to know the component's internal class names.

### 17. How do I combine classes conditionally?
For a small set of conditions, join the relevant class names with a template string or an array filtered for false values. If a project already has a class-name helper, follow its convention.

### 18. Should every reusable component accept a `className` prop?
Not necessarily. A public `className` prop is useful when consumers need controlled customization. For a private component, semantic props and local styling may be enough. Define whether a supplied class extends or replaces defaults.

### 19. Is `className` a good place to encode application state?
A class can select a visual variant, but it should not be the only representation of an important state. Keep state in React and expose meaning with appropriate text or ARIA semantics when needed.

## Tokens and themes

### 20. What is a design token?
A named reusable design value, such as a semantic text color, spacing step, border radius, or shadow. Tokens make shared decisions easier to change consistently.

### 21. Why use semantic token names?
Names such as `--color-surface` describe a role. A theme can change the actual color without every component needing to know whether that surface is white, gray, or another color.

### 22. What are CSS custom properties?
CSS properties whose names begin with `--`, for example `--color-surface`. They participate in the cascade, can inherit, and are consumed with `var(--color-surface)`.

### 23. Where should shared tokens be declared?
Often on `:root` or a stable application ancestor. Components can then read them. A local override can scope a different value to a subtree when that is intentional.

### 24. What is a straightforward way to implement light and dark themes?
Define semantic CSS variables for the default theme and override them on an ancestor with an attribute such as `data-theme="dark"`. Components consume the same variables in both themes.

### 25. What does the CSS `color-scheme` property do?
It tells the browser which color schemes an element supports so browser-provided controls and other UI can better match. It does not automatically choose colors for every part of the page; CSS still needs to style the rest.

### 26. How do I respect the operating-system theme preference?
Use the `prefers-color-scheme` media feature when the system preference should control appearance. If the user can choose a theme manually, define and implement a clear precedence between system preference and the saved choice.

### 27. Should theme selection live in React state?
If the theme toggle is interactive, React state or an existing shared preference store can own the choice and set a DOM attribute. A tiny page may keep it local; distant components should share one source rather than duplicate state.

### 28. How can I avoid a theme flash or server/client mismatch?
Use the rendering framework's supported strategy and ensure the initial server and client markup agree. Do not read browser-only storage during server rendering. A system-driven CSS theme can avoid some client-side initialization, while a persisted manual preference needs deliberate initialization.

## Cascade, responsiveness, and accessibility

### 29. Why did a CSS rule lose even though it is inside a CSS Module?
Local class-name mapping does not override the cascade. Another applicable rule may win due to specificity, source order, origin, cascade layer, inline styles, or inheritance. Inspect the computed style and fix the ownership or selector clearly.

### 30. Are cascade layers required for a React app?
No. They can help large style systems establish ordering between reset, base, component, and utility rules. Use them only when their precedence is understood and the tooling order is deliberate.

### 31. Should JavaScript calculate responsive breakpoints?
Usually no. CSS media and container queries are designed to adapt layout to available space. Use JavaScript only when the application behavior itself, not merely presentation, depends on a measured condition.

### 32. What should a theme do about color contrast?
Keep text and controls readable in every theme. Test the foreground and background combinations and do not rely on color alone to communicate important status.

### 33. How should focus indicators be styled?
Provide a visible keyboard focus treatment, often with `:focus-visible`. Do not remove the browser outline unless a clear replacement is supplied.

### 34. How do I handle animation in a theme or component stylesheet?
Use CSS transitions or animations for visual changes, and respect `prefers-reduced-motion` for nonessential movement. A simple color transition may be disabled for visitors who request reduced motion.

### 35. Should I put all styles into one stylesheet?
Not necessarily. A global file is useful for shared foundations; component styles can be colocated or module-scoped. Choose a layout the team can navigate and keep shared tokens and component-specific rules distinct.

### 36. Can CSS Modules replace a design system?
No. They address local class naming and stylesheet organization. Shared tokens, component APIs, accessibility requirements, and visual conventions still need deliberate design.

### 37. Can CSS-in-JS and CSS Modules be used in the same app?
Technically, many projects can combine them, but doing so adds more conventions and possible duplication. Prefer the existing project approach unless a specific need justifies another system.

### 38. What should I inspect when a CSS Module style is missing?
Confirm the import path, module filename convention, exported class name, `className={styles.name}` usage, and build tool support. Then inspect the rendered class and browser computed styles.

### 39. Why is `!important` usually a poor first fix?
It can hide unclear style ownership and make later variants harder to override. Check cascade order, specificity, layers, and whether a semantic variant or token should own the value.

### 40. Which styling strategy should I choose?
Start from the project's current toolchain and team conventions. Global CSS works well for foundations, CSS Modules for component-local selectors, inline styles for isolated dynamic values, and utility classes or CSS-in-JS when their trade-offs fit the project.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="091-typescript-with-react.md">091 - TypeScript with React: Questions and Answers</a></td><td><a href="../lessons/092-styling-and-theming.md">Lesson</a></td><td><a href="../examples/092-styling-and-theming.md">Example</a></td><td><a href="093-animation-and-motion-preferences.md">093 - Animation, Motion Preferences, and View Transitions: Questions and Answers</a></td></tr>
  </tbody>
</table>
