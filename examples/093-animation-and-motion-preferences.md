# React Animation and View Transitions: Interactive Example

This example compares three approaches in one small motion playground:

1. A CSS transition for a state-driven panel.
2. React 19.3's `<ViewTransition>` for a card entering and exiting.
3. One optional Web Animations API pulse that is skipped when the visitor prefers reduced motion.

Use it in an existing React 19.3+ TypeScript project. The CSS is a regular stylesheet so its view-transition pseudo-element rules stay global. The project entry point imports that stylesheet once.

## File structure

```text
src/
├── App.tsx
├── App.css
└── main.tsx
```

## `src/main.tsx`

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";
import "./App.css";

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

## `src/App.tsx`

```tsx
import {
  startTransition,
  useEffect,
  useRef,
  useState,
  useSyncExternalStore,
  ViewTransition,
} from "react";

const reducedMotionQuery = "(prefers-reduced-motion: reduce)";

function subscribeToMotionPreference(onChange: () => void) {
  if (typeof window === "undefined" || typeof window.matchMedia !== "function") {
    return () => {};
  }

  const mediaQuery = window.matchMedia(reducedMotionQuery);
  mediaQuery.addEventListener("change", onChange);
  return () => mediaQuery.removeEventListener("change", onChange);
}

function getMotionPreference() {
  if (typeof window === "undefined" || typeof window.matchMedia !== "function") {
    return false;
  }

  return window.matchMedia(reducedMotionQuery).matches;
}

function getServerMotionPreference() {
  return false;
}

function usePrefersReducedMotion() {
  return useSyncExternalStore(
    subscribeToMotionPreference,
    getMotionPreference,
    getServerMotionPreference,
  );
}

export default function App() {
  const [isPanelOpen, setIsPanelOpen] = useState(false);
  const [isCardVisible, setIsCardVisible] = useState(false);
  const [status, setStatus] = useState("");
  const prefersReducedMotion = usePrefersReducedMotion();
  const pulseButtonRef = useRef<HTMLButtonElement | null>(null);
  const pulseAnimationRef = useRef<Animation | null>(null);

  useEffect(() => {
    return () => pulseAnimationRef.current?.cancel();
  }, []);

  useEffect(() => {
    if (prefersReducedMotion) {
      pulseAnimationRef.current?.cancel();
      pulseAnimationRef.current = null;
    }
  }, [prefersReducedMotion]);

  function toggleCard() {
    startTransition(() => {
      setIsCardVisible((visible) => !visible);
    });
  }

  function runPulse() {
    if (prefersReducedMotion) {
      setStatus("Reduced motion is on, so the pulse was skipped.");
      return;
    }

    const button = pulseButtonRef.current;
    if (!button) return;

    pulseAnimationRef.current?.cancel();
    pulseAnimationRef.current = button.animate(
      [
        { transform: "scale(1)" },
        { transform: "scale(1.04)" },
        { transform: "scale(1)" },
      ],
      {
        duration: 220,
        easing: "ease-out",
      },
    );
    setStatus("The one-time pulse animation played.");
  }

  return (
    <main className="app-shell">
      <header className="page-heading">
        <p className="eyebrow">React motion lab</p>
        <h1>Small motion, clear state</h1>
        <p>
          Try CSS transitions, React View Transitions, and an optional
          JavaScript animation.
        </p>
      </header>

      <section className="demo-panel" aria-labelledby="css-transition-title">
        <div className="section-heading">
          <div>
            <p className="step-label">01 · CSS transition</p>
            <h2 id="css-transition-title">Open a details panel</h2>
          </div>
          <button
            className="secondary-button"
            type="button"
            aria-expanded={isPanelOpen}
            aria-controls="panel-details"
            onClick={() => setIsPanelOpen((open) => !open)}
          >
            {isPanelOpen ? "Close details" : "Open details"}
          </button>
        </div>

        <div
          className={isPanelOpen ? "details-panel is-open" : "details-panel"}
          id="panel-details"
          aria-hidden={!isPanelOpen}
        >
          <p>
            React changes the class from state. CSS transitions the panel's
            opacity and height-related presentation.
          </p>
        </div>
      </section>

      <section className="demo-panel" aria-labelledby="view-transition-title">
        <div className="section-heading">
          <div>
            <p className="step-label">02 · React View Transition</p>
            <h2 id="view-transition-title">Reveal a card</h2>
          </div>
          <button
            className="primary-button"
            type="button"
            aria-expanded={isCardVisible}
            aria-controls={isCardVisible ? "feature-card" : undefined}
            onClick={toggleCard}
          >
            {isCardVisible ? "Remove card" : "Add card"}
          </button>
        </div>

        {isCardVisible && (
          <ViewTransition
            enter="card-enter"
            exit="card-exit"
            default="none"
          >
            <article className="feature-card" id="feature-card">
              <span className="card-icon" aria-hidden="true">
                ✓
              </span>
              <div>
                <h3>Transition work completed</h3>
                <p>
                  React coordinated this boundary with the browser's view
                  transition.
                </p>
              </div>
            </article>
          </ViewTransition>
        )}
      </section>

      <section className="demo-panel" aria-labelledby="web-animation-title">
        <div className="section-heading">
          <div>
            <p className="step-label">03 · Web Animations API</p>
            <h2 id="web-animation-title">Run a one-time pulse</h2>
          </div>
          <button
            className="secondary-button pulse-button"
            ref={pulseButtonRef}
            type="button"
            onClick={runPulse}
          >
            Play pulse
          </button>
        </div>

        <p className="motion-note">
          Reduced motion preference: <strong>{prefersReducedMotion ? "on" : "off"}</strong>
        </p>
        <p className="status-message" role="status" aria-live="polite">
          {status}
        </p>
      </section>

      <p className="closing-note">
        All three controls still work when motion is reduced or unavailable.
      </p>
    </main>
  );
}
```

### How the TypeScript component works

- `isPanelOpen` is urgent UI state; a regular state update toggles it immediately.
- `isCardVisible` is changed inside React's `startTransition` so its `<ViewTransition>` boundary can animate.
- `usePrefersReducedMotion` subscribes to the browser media query with `useSyncExternalStore`. Its server snapshot is stable and does not access `window`.
- The Web Animations API is invoked only from the button handler, never during render.
- The active `Animation` is canceled when replaced, when reduced motion becomes active, and when the component unmounts.
- The live status message reports whether the optional pulse ran or was skipped.

## `src/App.css`

```css
@layer reset, base;

@layer reset {
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
}

@layer base {
  body {
    color: #182033;
    background: #f3f6fb;
    font-family: Inter, ui-sans-serif, system-ui, sans-serif;
    line-height: 1.55;
  }

  button {
    font: inherit;
  }
}

.app-shell {
  width: min(100% - 2rem, 820px);
  margin-inline: auto;
  padding: clamp(2rem, 7vw, 5rem) 0;
}

.page-heading {
  margin-bottom: 2rem;
}

.page-heading h1 {
  margin: 0;
  font-size: clamp(2.2rem, 7vw, 3.8rem);
  letter-spacing: -0.055em;
  line-height: 1.05;
}

.page-heading > p:last-child {
  max-width: 42rem;
  color: #596579;
}

.eyebrow,
.step-label {
  margin: 0 0 0.45rem;
  color: #3157d5;
  font-size: 0.75rem;
  font-weight: 850;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.demo-panel {
  margin-block: 1rem;
  padding: clamp(1rem, 4vw, 1.5rem);
  border: 1px solid #d9e0eb;
  border-radius: 1rem;
  background: #fff;
  box-shadow: 0 12px 36px rgb(24 42 82 / 7%);
}

.section-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.section-heading h2 {
  margin: 0;
  font-size: clamp(1.15rem, 3vw, 1.4rem);
}

.primary-button,
.secondary-button {
  min-height: 2.7rem;
  flex-shrink: 0;
  padding: 0.6rem 0.9rem;
  border: 1px solid transparent;
  border-radius: 0.7rem;
  font-weight: 750;
  cursor: pointer;
}

.primary-button {
  color: #fff;
  background: #3157d5;
}

.secondary-button {
  border-color: #d9e0eb;
  color: #24324a;
  background: #f8f9fc;
}

.primary-button:hover,
.secondary-button:hover {
  filter: brightness(0.95);
}

button:focus-visible {
  outline: 3px solid #7898ff;
  outline-offset: 3px;
}

.details-panel {
  display: grid;
  grid-template-rows: 0fr;
  margin-top: 0;
  padding-inline: 0.85rem;
  overflow: hidden;
  border: 0 solid transparent;
  border-radius: 0.75rem;
  color: #596579;
  background: #f8f9fc;
  opacity: 0;
  transition:
    grid-template-rows 180ms ease,
    opacity 180ms ease,
    margin-top 180ms ease,
    padding-block 180ms ease;
}

.details-panel > p {
  min-height: 0;
  margin: 0;
  overflow: hidden;
}

.details-panel.is-open {
  grid-template-rows: 1fr;
  margin-top: 1rem;
  padding-block: 0.85rem;
  opacity: 1;
}

.feature-card {
  display: flex;
  align-items: flex-start;
  gap: 0.9rem;
  margin-top: 1.25rem;
  padding: 1rem;
  border: 1px solid #cdd8ff;
  border-radius: 0.85rem;
  background: #f1f4ff;
}

.card-icon {
  display: grid;
  width: 2rem;
  aspect-ratio: 1;
  flex-shrink: 0;
  place-items: center;
  border-radius: 50%;
  color: #fff;
  background: #3157d5;
  font-weight: 850;
}

.feature-card h3,
.feature-card p {
  margin: 0;
}

.feature-card p {
  margin-top: 0.25rem;
  color: #596579;
}

.motion-note {
  margin: 1rem 0 0;
  color: #596579;
}

.status-message {
  min-height: 1.5em;
  margin: 0.35rem 0 0;
  color: #3157d5;
  font-weight: 700;
}

.closing-note {
  margin-top: 1.5rem;
  color: #596579;
  text-align: center;
}

@keyframes card-enter {
  from {
    opacity: 0;
    transform: translateY(0.65rem) scale(0.985);
  }

  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes card-exit {
  to {
    opacity: 0;
    transform: translateY(-0.3rem) scale(0.99);
  }
}

::view-transition-new(.card-enter) {
  animation: card-enter 220ms ease-out both;
}

::view-transition-old(.card-exit) {
  animation: card-exit 150ms ease-in both;
}

@media (max-width: 520px) {
  .section-heading {
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (prefers-reduced-motion: reduce) {
  .details-panel {
    transition: none;
  }

  ::view-transition-group(.card-enter),
  ::view-transition-group(.card-exit),
  ::view-transition-new(.card-enter),
  ::view-transition-old(.card-exit) {
    animation-duration: 1ms !important;
  }
}
```

The CSS transition animates the details panel's presentation, while React still determines whether its content is expanded. The `prefers-reduced-motion` rule removes that transition and reduces the custom view-transition animation to a negligible duration.

## Try it

1. Open and close the details panel. Its `aria-expanded` value follows React state while CSS transitions the visual change.
2. Add and remove the feature card. The state update uses React transition work, and the View Transition class selects its enter or exit animation.
3. Press **Play pulse**. The Web Animations API plays one short pulse and replaces any previous pulse.
4. Enable reduced motion in the operating-system accessibility settings and repeat all three actions. The panel transition stops, the view transition is reduced, and the JavaScript pulse is skipped with a status message.
5. Change the reduced-motion preference while the page is open during a pulse. The subscription updates and cancels an active animation.
6. Disable or remove the browser's View Transition support. The same card should still appear and disappear without the snapshot animation.
7. Use the keyboard to operate each button. Focus remains visible and the disclosure/card controls remain available.

## Important distinction

The example uses two APIs with similar names for different work:

- `startTransition` schedules the React state update as non-urgent transition work.
- `<ViewTransition>` coordinates that React update with a browser view transition.
- `document.startViewTransition()` is not called here; React owns coordination for this React update.

## Key takeaways

- Use CSS for ordinary state-driven transitions and keyframes.
- Use `<ViewTransition>` in React 19.3+ when old/new UI snapshots need continuity.
- Use the Web Animations API only when imperative control is useful, and cancel its `Animation` handle.
- Honor `prefers-reduced-motion` in CSS and in JavaScript-started animation.
- Keep the interface functional when animation is disabled or unsupported.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="092-styling-and-theming.md">092 - React Styling and Theming: CSS Modules Dashboard Example</a></td><td><a href="../lessons/093-animation-and-motion-preferences.md">Lesson</a></td><td><a href="../qnas/093-animation-and-motion-preferences.md">Q&amp;A</a></td><td><a href="094-internationalization-and-localization.md">094 - React Internationalization and RTL: Locale Dashboard Example</a></td></tr>
  </tbody>
</table>
