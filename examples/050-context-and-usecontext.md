# Example: Context and useContext

This runnable Vite example shows how to create a context, provide a changing value, read it several levels below the provider, and override it for one nested branch. It also demonstrates a default value and a custom Hook that catches a missing required provider.

The theme toggle changes local React state. No network requests or persistent storage are used.

## Project files

~~~text
context-theme-demo/
├── index.html
└── src/
    ├── App.jsx
    ├── main.jsx
    ├── styles.css
    └── contexts/
        └── theme-context.jsx
~~~

Use these files in an existing Vite React project. From the project root, install dependencies if needed and start the development server:

~~~sh
npm install
npm run dev
~~~

## src/contexts/theme-context.jsx

~~~jsx
import {
  createContext,
  useCallback,
  useContext,
  useMemo,
  useState,
} from "react";

export const ThemeContext = createContext(null);
export const GreetingContext = createContext("Default context greeting");

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  const toggleTheme = useCallback(() => {
    setTheme((currentTheme) =>
      currentTheme === "light" ? "dark" : "light"
    );
  }, []);

  const value = useMemo(
    () => ({ theme, toggleTheme }),
    [theme, toggleTheme]
  );

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);

  if (context === null) {
    throw new Error("useTheme must be used inside ThemeProvider.");
  }

  return context;
}
~~~

ThemeContext is created once at module scope. ThemeProvider owns the changing theme state and exposes both the current theme and an action to toggle it.

The memoized value object stays the same when ThemeProvider re-renders for an unrelated reason. It changes when theme changes. This is useful if the provider has unrelated state; for a small app, start without memoization and add it only when it helps.

GreetingContext has a default value and is intentionally not given a provider in this demo. Its consumer will read the default.

## src/App.jsx

~~~jsx
import { useContext, useMemo } from "react";
import {
  GreetingContext,
  ThemeContext,
  ThemeProvider,
  useTheme,
} from "./contexts/theme-context.jsx";

function ThemeButton() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button
      className="theme-button"
      type="button"
      onClick={toggleTheme}
    >
      Switch to {theme === "light" ? "dark" : "light"} theme
    </button>
  );
}

function ThemeCard({ title, children }) {
  const { theme } = useTheme();

  return (
    <article className={"theme-card theme-card-" + theme}>
      <p className="eyebrow">Read by useTheme</p>
      <h3>{title}</h3>
      <div>{children}</div>
      <p className="small-note">This card reads the {theme} theme.</p>
    </article>
  );
}

function DeepToolbar() {
  return (
    <div className="toolbar">
      <div>
        <p className="eyebrow">Intermediate component</p>
        <h3>Toolbar</h3>
      </div>
      <ThemeButton />
    </div>
  );
}

function Workspace() {
  return (
    <section className="workspace">
      <DeepToolbar />
      <div className="card-grid">
        <ThemeCard title="Account panel">
          <p>The theme arrives through context, not through Workspace props.</p>
        </ThemeCard>
        <ThemeCard title="Help panel">
          <p>This distant consumer reads the same nearest provider value.</p>
        </ThemeCard>
      </div>
    </section>
  );
}

function NestedDarkPreview() {
  const { toggleTheme } = useTheme();

  const darkPreviewValue = useMemo(
    () => ({
      theme: "dark",
      toggleTheme,
    }),
    [toggleTheme]
  );

  return (
    <ThemeContext.Provider value={darkPreviewValue}>
      <ThemeCard title="Nested override">
        <p>
          This card stays dark even when the surrounding page uses light mode.
        </p>
      </ThemeCard>
    </ThemeContext.Provider>
  );
}

function DefaultGreeting() {
  const greeting = useContext(GreetingContext);

  return (
    <div className="default-card">
      <p className="eyebrow">No GreetingContext provider</p>
      <h3>{greeting}</h3>
      <p className="small-note">
        This consumer reads the static default value from createContext.
      </p>
    </div>
  );
}

function PageContent() {
  const { theme } = useTheme();

  return (
    <main className={"page page-" + theme}>
      <header className="page-header">
        <div>
          <p className="eyebrow">React context demo</p>
          <h1>Share a value with descendants</h1>
          <p className="intro">
            Toggle the theme, inspect the nested override, and compare it with
            the consumer that uses a context default.
          </p>
        </div>
        <ThemeButton />
      </header>

      <section className="demo-section" aria-labelledby="provider-heading">
        <p className="eyebrow">Provider above consumers</p>
        <h2 id="provider-heading">Current page theme: {theme}</h2>
        <p className="muted">
          ThemeProvider owns the theme state. ThemeButton and ThemeCard read it
          through useTheme, even though Workspace does not receive a theme prop.
        </p>
        <Workspace />
      </section>

      <section className="demo-section" aria-labelledby="override-heading">
        <p className="eyebrow">Nearest provider wins</p>
        <h2 id="override-heading">Override one branch</h2>
        <p className="muted">
          The page uses the selected theme, while the nested preview supplies
          dark only to its own descendants.
        </p>
        <NestedDarkPreview />
      </section>

      <section className="demo-section" aria-labelledby="default-heading">
        <p className="eyebrow">Static fallback</p>
        <h2 id="default-heading">Read a context default</h2>
        <DefaultGreeting />
      </section>

      <footer className="page-footer">
        <p>
          Theme changes remain in React state and are cleared when the page is
          refreshed.
        </p>
      </footer>
    </main>
  );
}

export default function App() {
  return (
    <ThemeProvider>
      <PageContent />
    </ThemeProvider>
  );
}
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./styles.css";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
~~~

## src/styles.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
  color: #202a3a;
  background: #f3f6fb;
}

button {
  font: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #86aaff;
  outline-offset: 3px;
}

.page {
  min-height: 100vh;
  padding: 3rem max(1rem, calc((100vw - 960px) / 2)) 4rem;
  transition:
    color 160ms ease,
    background-color 160ms ease;
}

.page-light {
  color: #202a3a;
  background: #f3f6fb;
}

.page-dark {
  color: #edf2fa;
  background: #141b27;
}

.page-header,
.section-heading,
.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header > div {
  max-width: 68ch;
}

.eyebrow {
  margin: 0 0 0.4rem;
  color: #4770c1;
  font-size: 0.74rem;
  font-weight: 800;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

.page-dark .eyebrow {
  color: #9cb8ff;
}

h1,
h2,
h3,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 6vw, 3.4rem);
  letter-spacing: -0.045em;
}

h2 {
  margin-bottom: 0.45rem;
  font-size: clamp(1.25rem, 3vw, 1.7rem);
  letter-spacing: -0.025em;
}

h3 {
  margin-bottom: 0.5rem;
  font-size: 1.08rem;
}

.intro,
.muted,
.small-note,
.page-footer {
  color: #59677a;
  line-height: 1.65;
}

.page-dark .intro,
.page-dark .muted,
.page-dark .small-note,
.page-dark .page-footer {
  color: #bac6d7;
}

.intro {
  margin-bottom: 0;
  font-size: 1.05rem;
}

.theme-button {
  flex: 0 0 auto;
  min-height: 2.75rem;
  padding: 0.65rem 0.9rem;
  border: 1px solid #315db5;
  border-radius: 0.6rem;
  color: #fff;
  background: #315db5;
  font-weight: 750;
}

.theme-button:hover {
  background: #244b9a;
}

.demo-section {
  margin-top: 1rem;
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d6dfeb;
  border-radius: 1rem;
  background: rgb(255 255 255 / 78%);
  box-shadow: 0 10px 28px rgb(32 53 84 / 5%);
}

.page-dark .demo-section {
  border-color: #354257;
  background: #202a39;
}

.demo-section > .muted {
  max-width: 72ch;
  margin-bottom: 1rem;
}

.workspace {
  padding: 1rem;
  border: 1px solid #dce4ef;
  border-radius: 0.8rem;
  background: rgb(255 255 255 / 56%);
}

.page-dark .workspace {
  border-color: #43516a;
  background: rgb(10 16 25 / 28%);
}

.toolbar {
  margin-bottom: 1rem;
  padding-bottom: 0.9rem;
  border-bottom: 1px solid #dce4ef;
}

.page-dark .toolbar {
  border-color: #43516a;
}

.toolbar h3,
.toolbar .eyebrow {
  margin-bottom: 0;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.8rem;
}

.theme-card,
.default-card {
  padding: 1rem;
  border: 1px solid #d6dfeb;
  border-radius: 0.75rem;
}

.theme-card-light,
.default-card {
  color: #202a3a;
  background: #fff;
}

.theme-card-dark {
  border-color: #34445f;
  color: #f0f4fb;
  background: #202d43;
}

.theme-card .small-note,
.default-card .small-note {
  margin: 0.8rem 0 0;
  font-size: 0.86rem;
}

.theme-card p:last-child,
.default-card p:last-child {
  margin-bottom: 0;
}

.page-footer {
  margin-top: 1.4rem;
  text-align: center;
  font-size: 0.88rem;
}

@media (max-width: 680px) {
  .page {
    padding-block: 2rem 3rem;
  }

  .page-header,
  .section-heading {
    align-items: flex-start;
    flex-direction: column;
  }

  .card-grid {
    grid-template-columns: 1fr;
  }

  .theme-button {
    width: 100%;
  }
}
~~~

## index.html

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="An interactive React example for context providers and useContext."
    />
    <title>React Context and useContext Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
~~~

## How to explore the example

### Read context through intermediate components

1. The App component renders ThemeProvider around PageContent.
2. PageContent renders Workspace.
3. Workspace renders DeepToolbar and several ThemeCard components.
4. DeepToolbar does not receive a theme prop. ThemeButton reads the theme through useTheme.
5. Each ThemeCard independently reads the nearest ThemeContext value.

The intermediate components do not need to forward theme data.

### Update the provider value

1. Select Switch to dark theme in the header or toolbar.
2. ThemeProvider updates its theme state.
3. The provider receives a new memoized value object.
4. Components that read ThemeContext update to use the new value.

The theme and toggle action are packaged in one context value. The callback is stable, and the object is memoized until the theme changes.

### Compare the nested provider

The NestedDarkPreview component reads the outer toggle action, then provides a value with theme set to dark around its child card. That nested provider overrides the outer theme only within its own branch. The rest of the page still follows the selected theme.

### Check the default value

DefaultGreeting reads GreetingContext. No GreetingContext provider wraps it, so it receives the default string passed to createContext.

ThemeContext uses null as a sentinel and the useTheme custom Hook throws if a theme consumer is rendered outside ThemeProvider.

## React version compatibility

The example uses ThemeContext.Provider syntax, which works in React 18 and React 19. In React 19 and later, you can instead render ThemeContext directly as the provider:

~~~jsx
<ThemeContext value={value}>
  {children}
</ThemeContext>
~~~

## What to notice

- Context must be created once so provider and consumers share the same context object.
- useContext reads the closest provider above the calling component.
- A provider returned by a component cannot supply context to that same component's useContext call.
- Context distributes the theme; ThemeProvider's state owns updates.
- The nested provider changes only its own branch.
- The default applies only when a matching provider is absent.
- Memoization helps keep object value identity stable, but should be used when useful rather than added automatically.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="049-rules-of-hooks.md">049 - Example: Following the Rules of Hooks</a></td><td><a href="../lessons/050-context-and-usecontext.md">Lesson</a></td><td><a href="../qnas/050-context-and-usecontext.md">Q&amp;A</a></td><td><a href="051-context-provider-design.md">051 - Example: Designing a Scoped Context Provider</a></td></tr>
  </tbody>
</table>
