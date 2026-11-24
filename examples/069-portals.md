# Example: A Portaled Help Panel

This runnable React app demonstrates a non-modal help panel rendered outside a clipped card:

- The help panel is physically inserted into a sibling DOM target named `portal-root`.
- The panel escapes an in-tree container with `overflow: hidden` and a transformed ancestor.
- It reads a React context value from its original React ancestors.
- Clicking a control inside the portal still bubbles through the React tree to the app shell.
- The help panel has an explicit close button and is not presented as a modal dialog.

This example is a small portal demonstration, not a complete modal implementation. A modal also needs correct dialog semantics and keyboard focus management.

## Create the project

~~~sh
npm create vite@latest portal-help-demo -- --template react
cd portal-help-demo
npm install
npm run dev
~~~

Replace or add the files below, then open the local address printed by Vite.

## Project files

~~~text
portal-help-demo/
  index.html
  src/
    main.jsx
    App.jsx
    styles.css
~~~

## index.html

The portal target is a sibling of the main React root. Both nodes exist in the document before the module script runs.

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="theme-color" content="#f4f6fb" />
    <title>React Portal Help Panel</title>
  </head>
  <body>
    <div id="root"></div>
    <div id="portal-root"></div>
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

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## src/App.jsx

~~~jsx
import { createContext, useContext, useEffect, useState } from "react";
import { createPortal } from "react-dom";

const HelpThemeContext = createContext("default");

function QuickHelpPanel({ onClose }) {
  const theme = useContext(HelpThemeContext);

  return (
    <aside
      className="floating-help"
      id="quick-help-panel"
      role="region"
      aria-label="Quick help panel"
    >
      <div className="help-heading">
        <div>
          <p className="eyebrow">Quick help</p>
          <h2>Working with portals</h2>
        </div>
        <button
          className="icon-button"
          type="button"
          aria-label="Close quick help"
          onClick={onClose}
        >
          Ãƒâ€”
        </button>
      </div>

      <p>
        This panel is placed in a separate DOM node so it can escape the
        clipped card.
      </p>

      <div className="context-readout">
        <span className="context-swatch" aria-hidden="true" />
        <span>
          Context from the React parent: <strong>{theme}</strong>
        </span>
      </div>

      <p className="help-footnote">
        This is a non-modal help region. Focus is not trapped here.
      </p>
    </aside>
  );
}

function SummaryCard({ label, value, note }) {
  return (
    <article className="summary-card">
      <p>{label}</p>
      <strong>{value}</strong>
      <span>{note}</span>
    </article>
  );
}

export default function App() {
  const [portalTarget, setPortalTarget] = useState(null);
  const [helpOpen, setHelpOpen] = useState(false);
  const [shellClickCount, setShellClickCount] = useState(0);

  useEffect(() => {
    setPortalTarget(document.getElementById("portal-root"));
  }, []);

  function handleShellClick() {
    setShellClickCount((count) => count + 1);
  }

  return (
    <HelpThemeContext.Provider value="violet">
      <div className="page-shell" onClick={handleShellClick}>
        <header className="topbar">
          <a className="brand" href="#dashboard">
            <span className="brand-mark" aria-hidden="true">N</span>
            Northstar
          </a>
          <span className="topbar-label">Project workspace</span>
          <span className="avatar" aria-label="Signed in as Ash">A</span>
        </header>

        <main className="page-content" id="dashboard">
          <section className="page-heading" aria-labelledby="page-title">
            <p className="eyebrow">Team dashboard</p>
            <h1 id="page-title">Project overview</h1>
            <p>
              Open quick help to see a portal escape a clipped layout while
              staying in this React tree.
            </p>
          </section>

          <section className="summary-grid" aria-label="Project summary">
            <SummaryCard
              label="Active projects"
              value="08"
              note="Across 3 teams"
            />
            <SummaryCard
              label="Completed tasks"
              value="124"
              note="This month"
            />
            <SummaryCard
              label="Next milestone"
              value="Friday"
              note="Release review"
            />
          </section>

          <section className="portal-demo" aria-labelledby="portal-demo-title">
            <div className="demo-heading">
              <div>
                <p className="eyebrow">Portal placement</p>
                <h2 id="portal-demo-title">A clipped card</h2>
              </div>
              <button
                className="button button--primary"
                type="button"
                aria-expanded={helpOpen}
                disabled={!portalTarget}
                onClick={() => setHelpOpen((isOpen) => !isOpen)}
              >
                {helpOpen ? "Hide quick help" : "Open quick help"}
              </button>
            </div>

            <p className="demo-description">
              The dashed card clips its children. The help panel is rendered in
              the document's separate portal target, outside this card.
            </p>

            <div className="clip-demo">
              <div className="clip-demo-content">
                <span className="clip-label">overflow: hidden</span>
                <p>
                  This container has a fixed height and clips content that
                  extends beyond its edges.
                </p>
                <div className="inline-tooltip">
                  An in-tree tooltip would be clipped here.
                </div>
              </div>
            </div>
          </section>

          <section className="event-demo" aria-labelledby="event-demo-title">
            <div>
              <p className="eyebrow">React event propagation</p>
              <h2 id="event-demo-title">The React parent still receives clicks</h2>
              <p>
                This counter is updated by a click handler on the page shell.
                Click the portal panel's close button and watch it increment.
              </p>
            </div>
            <output className="click-count" aria-live="polite">
              {shellClickCount}
              <span> shell clicks</span>
            </output>
          </section>

          <footer className="page-footer">
            Portal target is outside the main React root in the DOM.
          </footer>
        </main>

        {helpOpen &&
          portalTarget &&
          createPortal(
            <QuickHelpPanel onClose={() => setHelpOpen(false)} />,
            portalTarget,
          )}
      </div>
    </HelpThemeContext.Provider>
  );
}
~~~

`useEffect` resolves the portal target after the component mounts, so this example does not read `document` at module scope. The target node in `index.html` already exists by then.

`QuickHelpPanel` reads `HelpThemeContext` from the provider even though its DOM is placed outside `.page-shell`. Its close button's click still reaches the shell's React handler, so the click counter increments. If that propagation were unwanted in an application, the portal handler could call `event.stopPropagation()`.

## src/styles.css

~~~css
:root {
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  color: #182230;
  background: #f4f6fb;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  --ink: #182230;
  --muted: #667085;
  --line: #e3e7ef;
  --brand: #514dd4;
  --brand-soft: #efefff;
  --surface: #ffffff;
  --shadow: 0 16px 42px rgb(31 45 74 / 8%);
}

*,
*::before,
*::after {
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

button:focus-visible,
a:focus-visible {
  outline: 3px solid #a5a3ff;
  outline-offset: 3px;
}

.page-shell {
  min-height: 100vh;
}

.topbar {
  display: flex;
  min-height: 70px;
  align-items: center;
  gap: 1rem;
  padding: 0 clamp(1rem, 5vw, 4.5rem);
  border-bottom: 1px solid var(--line);
  background: var(--surface);
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 0.65rem;
  color: var(--ink);
  font-weight: 750;
  text-decoration: none;
}

.brand-mark {
  display: grid;
  width: 34px;
  height: 34px;
  place-items: center;
  border-radius: 11px;
  background: var(--brand);
  color: white;
  font-weight: 800;
}

.topbar-label {
  color: var(--muted);
  font-size: 0.88rem;
}

.avatar {
  display: grid;
  width: 34px;
  height: 34px;
  margin-left: auto;
  place-items: center;
  border: 1px solid #d8d7ff;
  border-radius: 50%;
  background: var(--brand-soft);
  color: var(--brand);
  font-size: 0.85rem;
  font-weight: 750;
}

.page-content {
  width: min(100% - 2rem, 1040px);
  margin: 0 auto;
  padding: clamp(2.5rem, 7vw, 5rem) 0 2rem;
}

.page-heading {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin: 0 0 0.5rem;
  color: var(--brand);
  font-size: 0.72rem;
  font-weight: 800;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.page-heading h1 {
  margin: 0;
  color: var(--ink);
  font-size: clamp(2rem, 5vw, 3.1rem);
  letter-spacing: -0.055em;
}

.page-heading > p:last-child,
.demo-description,
.event-demo p:last-child {
  color: var(--muted);
  line-height: 1.65;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.summary-card {
  display: grid;
  gap: 0.55rem;
  min-height: 128px;
  align-content: start;
  padding: 1.1rem;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.summary-card p,
.summary-card span {
  margin: 0;
  color: var(--muted);
  font-size: 0.82rem;
}

.summary-card strong {
  color: var(--ink);
  font-size: 1.75rem;
  letter-spacing: -0.04em;
}

.portal-demo,
.event-demo {
  margin-top: 1rem;
  padding: clamp(1.2rem, 4vw, 1.75rem);
  border: 1px solid var(--line);
  border-radius: 16px;
  background: var(--surface);
  box-shadow: var(--shadow);
}

.demo-heading,
.event-demo {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1.25rem;
}

.demo-heading h2,
.event-demo h2 {
  margin: 0;
  color: var(--ink);
  font-size: 1.2rem;
  letter-spacing: -0.025em;
}

.button {
  min-height: 42px;
  flex: 0 0 auto;
  padding: 0.65rem 0.9rem;
  border: 1px solid transparent;
  border-radius: 9px;
  cursor: pointer;
  font-size: 0.86rem;
  font-weight: 700;
}

.button:focus-visible,
.icon-button:focus-visible {
  outline: 3px solid #a5a3ff;
  outline-offset: 3px;
}

.button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.button--primary {
  background: var(--brand);
  color: white;
}

.button--primary:hover:not(:disabled) {
  background: #403bb9;
}

.demo-description {
  margin: 0.8rem 0 1.25rem;
  font-size: 0.9rem;
}

.clip-demo {
  position: relative;
  min-height: 190px;
  overflow: hidden;
  isolation: isolate;
  border: 2px dashed #aaa8e8;
  border-radius: 12px;
  background:
    radial-gradient(circle at 1px 1px, #d9daf2 1px, transparent 0) 0 0 / 18px 18px,
    #fafaff;
  transform: translateZ(0);
}

.clip-demo-content {
  position: relative;
  min-height: 190px;
  padding: 1rem;
}

.clip-label {
  display: inline-flex;
  padding: 0.35rem 0.55rem;
  border-radius: 6px;
  background: #e9e8ff;
  color: #3936a8;
  font-size: 0.75rem;
  font-weight: 750;
}

.clip-demo-content > p {
  max-width: 430px;
  color: #555f72;
  font-size: 0.88rem;
  line-height: 1.55;
}

.inline-tooltip {
  position: absolute;
  top: 158px;
  left: 1rem;
  width: min(340px, calc(100% - 2rem));
  padding: 0.85rem 1rem;
  border: 1px solid #c9c7ff;
  border-radius: 10px;
  background: white;
  box-shadow: var(--shadow);
  color: #3936a8;
  font-size: 0.84rem;
  font-weight: 650;
}

.event-demo {
  align-items: center;
}

.event-demo > div {
  max-width: 700px;
}

.event-demo p:last-child {
  margin: 0.55rem 0 0;
  font-size: 0.88rem;
}

.click-count {
  display: grid;
  min-width: 115px;
  min-height: 75px;
  align-content: center;
  justify-items: center;
  padding: 0.7rem;
  border: 1px solid #d9d8ff;
  border-radius: 12px;
  background: var(--brand-soft);
  color: #3936a8;
  font-size: 1.5rem;
  font-weight: 800;
}

.click-count span {
  color: #5b5a7a;
  font-size: 0.72rem;
  font-weight: 650;
}

.page-footer {
  padding: 1.25rem 0;
  color: var(--muted);
  font-size: 0.82rem;
}

.floating-help {
  position: fixed;
  z-index: 1000;
  right: clamp(1rem, 4vw, 2rem);
  bottom: clamp(1rem, 4vw, 2rem);
  width: min(390px, calc(100vw - 2rem));
  padding: 1.2rem;
  border: 1px solid #dcdaf9;
  border-radius: 16px;
  background: var(--surface);
  box-shadow: 0 22px 70px rgb(23 32 51 / 20%);
  color: var(--ink);
}

.help-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
}

.help-heading h2 {
  margin: 0;
  font-size: 1.15rem;
  letter-spacing: -0.025em;
}

.icon-button {
  display: grid;
  width: 36px;
  height: 36px;
  flex: 0 0 36px;
  place-items: center;
  border: 1px solid var(--line);
  border-radius: 9px;
  background: white;
  color: #475467;
  cursor: pointer;
  font-size: 1.3rem;
}

.floating-help > p:not(.eyebrow) {
  color: #475467;
  font-size: 0.88rem;
  line-height: 1.6;
}

.context-readout {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  padding: 0.8rem;
  border: 1px solid var(--line);
  border-radius: 10px;
  background: #fafaff;
  color: #475467;
  font-size: 0.82rem;
}

.context-readout strong {
  color: var(--brand);
}

.context-swatch {
  width: 16px;
  height: 16px;
  flex: 0 0 16px;
  border-radius: 50%;
  background: var(--brand);
}

.floating-help .help-footnote {
  margin-bottom: 0;
  color: var(--muted);
  font-size: 0.76rem;
}

@media (max-width: 680px) {
  .summary-grid {
    grid-template-columns: 1fr;
  }

  .summary-card {
    min-height: 105px;
  }

  .demo-heading,
  .event-demo {
    align-items: flex-start;
    flex-direction: column;
  }

  .demo-heading .button {
    width: 100%;
  }

  .click-count {
    width: 100%;
    min-height: 62px;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
~~~

## What to observe

1. The dotted tooltip inside the dashed card is clipped by the card's overflow.
2. Select **Open quick help**. The panel is inserted under `#portal-root` instead of inside the clipped card and appears at the bottom-right of the viewport.
3. The panel displays the value `violet` from `HelpThemeContext` even though its DOM is outside `#root`.
4. The shell click counter increases when **Open quick help** is clicked. Open the panel, then select its close button; the counter increases again. React events follow the React tree.
5. Inspect the DOM. The help panel is under `#portal-root`, while its React parent is still `App`.
6. Notice that styles for `.floating-help` are defined as a global class. A selector that requires it to be a DOM descendant of `.page-shell` would not match this portaled element.

The portal target is resolved in an Effect so the code does not access `document` during module initialization. For a modal dialog, add correct dialog semantics and manage its complete keyboard and focus behavior; this non-modal help panel does not trap focus.
