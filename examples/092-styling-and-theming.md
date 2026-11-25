# React Styling and Theming: CSS Modules Dashboard Example

This example builds a responsive learning dashboard using a CSS Module for component classes, global CSS for document foundations, CSS custom properties for semantic color tokens, and React state for a light/dark theme toggle.

Use the files in an existing React + TypeScript project whose build tool supports CSS Modules. CSS Module import typing must also be available to the TypeScript checker. React itself does not provide CSS Module processing.

## File structure

```text
src/
├── App.module.css
├── App.tsx
├── main.tsx
└── styles/
    └── global.css
```

## `src/main.tsx`

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";
import "./styles/global.css";

const rootElement = document.getElementById("root");
if (!rootElement) {
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

The global stylesheet is imported once by the entry point. The component stylesheet is imported by the component that uses its classes.

## `src/App.tsx`

```tsx
import { useState } from "react";
import styles from "./App.module.css";

type Theme = "light" | "dark";

type DashboardStat = {
  label: string;
  value: string;
  detail: string;
  featured: boolean;
};

const stats: DashboardStat[] = [
  {
    label: "Lessons completed",
    value: "18",
    detail: "Across four learning tracks",
    featured: true,
  },
  {
    label: "Current streak",
    value: "6 days",
    detail: "A little practice each day",
    featured: false,
  },
  {
    label: "Saved for later",
    value: "12",
    detail: "Ready when you are",
    featured: false,
  },
];

const recentLessons = [
  { title: "CSS Modules and local styles", status: "Completed" },
  { title: "Semantic design tokens", status: "In progress" },
  { title: "Responsive component layouts", status: "Up next" },
];

export default function App() {
  const [theme, setTheme] = useState<Theme>("light");

  function toggleTheme() {
    setTheme((currentTheme) =>
      currentTheme === "light" ? "dark" : "light",
    );
  }

  return (
    <main className={styles.page} data-theme={theme}>
      <div className={styles.dashboard}>
        <header className={styles.topBar}>
          <a className={styles.brand} href="#overview" aria-label="React Notes home">
            <span className={styles.brandMark} aria-hidden="true">
              RN
            </span>
            <span>React Notes</span>
          </a>

          <button
            className={styles.themeButton}
            type="button"
            onClick={toggleTheme}
            aria-pressed={theme === "dark"}
          >
            <span className={styles.themeIndicator} aria-hidden="true" />
            Dark theme
          </button>
        </header>

        <section className={styles.intro} id="overview" aria-labelledby="page-title">
          <p className={styles.eyebrow}>Your learning space</p>
          <h1 className={styles.title} id="page-title">
            Good progress adds up.
          </h1>
          <p className={styles.subtitle}>
            Pick up where you left off and keep building your React skills.
          </p>
        </section>

        <section className={styles.statsGrid} aria-label="Learning overview">
          {stats.map((stat) => (
            <article
              className={
                stat.featured
                  ? `${styles.statCard} ${styles.featured}`
                  : styles.statCard
              }
              key={stat.label}
            >
              <p className={styles.statLabel}>{stat.label}</p>
              <p className={styles.statValue}>{stat.value}</p>
              <p className={styles.statDetail}>{stat.detail}</p>
            </article>
          ))}
        </section>

        <section
          className={styles.recentSection}
          id="recent-lessons"
          aria-labelledby="recent-title"
        >
          <div className={styles.sectionHeading}>
            <div>
              <p className={styles.eyebrow}>Keep moving</p>
              <h2 className={styles.sectionTitle} id="recent-title">
                Recent lessons
              </h2>
            </div>
            <span className={styles.sectionCount}>3 lessons</span>
          </div>

          <ul className={styles.lessonList}>
            {recentLessons.map((lesson) => (
              <li className={styles.lessonRow} key={lesson.title}>
                <span className={styles.lessonTitle}>{lesson.title}</span>
                <span className={styles.lessonStatus}>{lesson.status}</span>
              </li>
            ))}
          </ul>
        </section>

        <footer className={styles.footer}>
          Theme: <span className={styles.themeName}>{theme}</span>
        </footer>
      </div>
    </main>
  );
}
```

### What the component is doing

- `Theme` restricts the selected theme to two supported values.
- `useState` owns the current theme; a functional state update uses the previous value safely.
- `data-theme` marks the theme at one ancestor, so its custom properties inherit through the page.
- `styles.statCard`, `styles.featured`, and the other imported properties refer to local CSS Module classes.
- The list uses stable lesson titles as keys because the sample data has unique titles.
- The theme button keeps a stable name and reports its selected state with `aria-pressed`.

## `src/styles/global.css`

```css
@layer reset, base;

@layer reset {
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  html {
    min-width: 320px;
    min-height: 100%;
  }

  body {
    min-width: 320px;
    min-height: 100vh;
    margin: 0;
  }
}

@layer base {
  body {
    font-family:
      Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
      sans-serif;
    font-synthesis: none;
    text-rendering: optimizeLegibility;
  }

  button,
  input,
  select,
  textarea {
    font: inherit;
  }

  a {
    color: inherit;
  }
}
```

Global CSS is kept small: it defines box sizing, document defaults, and shared typography. The dashboard's layout and appearance stay in its local module.

## `src/App.module.css`

```css
.page {
  --page: #f4f6fb;
  --surface: #fff;
  --surface-raised: #f8f9fc;
  --text: #182033;
  --muted: #667187;
  --border: #e0e5ef;
  --accent: #3157d5;
  --accent-soft: #e8edff;
  --featured-text: #fff;
  --featured-muted: #e1e8ff;
  --featured-border: #3157d5;
  --shadow: 0 18px 55px rgb(24 42 82 / 8%);

  color-scheme: light;
  min-height: 100vh;
  padding: clamp(1.25rem, 4vw, 3.5rem);
  color: var(--text);
  background: var(--page);
  transition: color 180ms ease, background-color 180ms ease;
}

.page[data-theme="dark"] {
  --page: #101722;
  --surface: #192434;
  --surface-raised: #202d40;
  --text: #f0f3fa;
  --muted: #aeb9ca;
  --border: #344258;
  --accent: #b6c6ff;
  --accent-soft: #29385b;
  --featured-text: #101722;
  --featured-muted: #273351;
  --featured-border: #cad5ff;
  --shadow: 0 18px 55px rgb(0 0 0 / 22%);

  color-scheme: dark;
}

.dashboard {
  width: min(100%, 1040px);
  margin-inline: auto;
}

.topBar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: clamp(3rem, 8vw, 5.5rem);
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 0.7rem;
  color: var(--text);
  font-size: 0.95rem;
  font-weight: 800;
  text-decoration: none;
}

.brandMark {
  display: grid;
  width: 2.4rem;
  aspect-ratio: 1;
  place-items: center;
  border-radius: 0.75rem;
  color: #fff;
  background: #3157d5;
  font-size: 0.75rem;
  letter-spacing: 0.04em;
}

.themeButton {
  display: inline-flex;
  min-height: 2.75rem;
  align-items: center;
  gap: 0.55rem;
  padding: 0.55rem 0.85rem;
  border: 1px solid var(--border);
  border-radius: 999px;
  color: var(--text);
  background: var(--surface);
  font-size: 0.9rem;
  font-weight: 700;
  cursor: pointer;
}

.themeButton:hover {
  border-color: var(--accent);
}

.themeIndicator {
  width: 0.7rem;
  aspect-ratio: 1;
  border: 2px solid var(--accent);
  border-radius: 50%;
  background: var(--accent);
  box-shadow: inset -0.2rem -0.1rem 0 var(--surface);
}

.themeButton[aria-pressed="true"] .themeIndicator {
  border-color: var(--accent);
  background: var(--accent);
  box-shadow: none;
}

.intro {
  max-width: 680px;
  margin-bottom: 2.2rem;
}

.eyebrow {
  margin: 0 0 0.6rem;
  color: var(--accent);
  font-size: 0.75rem;
  font-weight: 850;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.title {
  margin: 0;
  font-size: clamp(2.4rem, 7vw, 5rem);
  letter-spacing: -0.06em;
  line-height: 1.02;
}

.subtitle {
  max-width: 38rem;
  margin: 1rem 0 0;
  color: var(--muted);
  font-size: clamp(1rem, 2vw, 1.15rem);
  line-height: 1.7;
}

.statsGrid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
  margin-bottom: clamp(3rem, 7vw, 5rem);
}

.statCard {
  min-height: 178px;
  padding: 1.25rem;
  border: 1px solid var(--border);
  border-radius: 1rem;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.statCard.featured {
  border-color: var(--featured-border);
  color: var(--featured-text);
  background: var(--accent);
}

.statLabel,
.statValue,
.statDetail {
  margin: 0;
}

.statLabel {
  color: var(--muted);
  font-size: 0.88rem;
  font-weight: 700;
}

.featured .statLabel,
.featured .statDetail {
  color: var(--featured-muted);
}

.statValue {
  margin-top: 1rem;
  font-size: clamp(1.8rem, 3vw, 2.4rem);
  font-weight: 850;
  letter-spacing: -0.05em;
}

.statDetail {
  margin-top: 0.35rem;
  color: var(--muted);
  font-size: 0.86rem;
}

.recentSection {
  padding: clamp(1.1rem, 3vw, 1.75rem);
  border: 1px solid var(--border);
  border-radius: 1rem;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.sectionHeading {
  display: flex;
  align-items: end;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.2rem;
}

.sectionHeading .eyebrow {
  margin-bottom: 0.35rem;
}

.sectionTitle {
  margin: 0;
  font-size: clamp(1.35rem, 3vw, 1.8rem);
  letter-spacing: -0.04em;
}

.sectionCount {
  color: var(--muted);
  font-size: 0.85rem;
}

.lessonList {
  display: grid;
  gap: 0.65rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.lessonRow {
  display: flex;
  min-height: 3.75rem;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.8rem 1rem;
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  background: var(--surface-raised);
}

.lessonTitle {
  font-weight: 700;
}

.lessonStatus {
  flex-shrink: 0;
  padding: 0.35rem 0.6rem;
  border-radius: 999px;
  color: var(--accent);
  background: var(--accent-soft);
  font-size: 0.75rem;
  font-weight: 800;
}

.footer {
  padding: 1.2rem 0 0;
  color: var(--muted);
  font-size: 0.85rem;
  text-align: right;
  text-transform: capitalize;
}

.themeName {
  color: var(--text);
  font-weight: 750;
}

.page :focus-visible {
  outline: 3px solid var(--accent);
  outline-offset: 3px;
}

@media (max-width: 700px) {
  .statsGrid {
    grid-template-columns: 1fr;
  }

  .statCard {
    min-height: 150px;
  }
}

@media (max-width: 460px) {
  .topBar {
    align-items: flex-start;
  }

  .themeButton {
    min-height: 2.5rem;
    padding-inline: 0.7rem;
  }

  .lessonRow {
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (prefers-reduced-motion: reduce) {
  .page {
    transition: none;
  }
}
```

## How the theme works

1. `theme` is typed as `"light" | "dark"`, so unrelated strings cannot be selected accidentally.
2. The toggle updates React state; React writes the current value to `data-theme` on the page root.
3. `.page[data-theme="dark"]` replaces the semantic custom properties and sets `color-scheme: dark` for browser controls.
4. Child cards and text use the same custom properties in either theme, so they do not need separate dark-mode selectors.
5. The toggle is a real button and `aria-pressed` reports whether dark theme is active. Keyboard focus remains visible.
6. The CSS Module maps its classes locally. The global stylesheet remains responsible for document-wide defaults.

## Try it

1. Activate **Dark theme** using a mouse, Enter, or Space. Confirm that cards, text, borders, and the browser color scheme all change.
2. Activate the button again. Confirm the light theme returns and the button's pressed state changes.
3. Narrow the viewport. The three summary cards should stack, and lesson rows should become vertical on a small screen.
4. Navigate to the theme button with the keyboard and check that its focus indicator is visible.
5. Enable reduced motion in the operating system and change the theme. The color transition should be disabled.
6. Rename `.statCard` in the module and its `styles.statCard` use together. The change stays limited to this module's mappings.
7. Temporarily use a generic global class with the same local name in another component. The CSS Module mapping should prevent a class-name collision, while inherited tokens still flow from `.page`.

## Key takeaways

- Global CSS owns document foundations; the CSS Module owns this dashboard's local classes.
- React state controls the selected theme; CSS custom properties control the visual values.
- A semantic token such as `--surface` makes a theme change reusable across components.
- CSS Modules reduce class-name collisions but do not replace CSS inheritance or the cascade.
- Responsive layout, keyboard focus, and reduced-motion behavior stay in CSS.
