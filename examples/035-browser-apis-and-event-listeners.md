# 035. Working with browser APIs and external event listeners - Example

This example displays the browser viewport dimensions and the current reduced-motion preference. It subscribes to the window resize event and a media query change event, then removes both listeners in their Effect cleanup functions. A separate button uses a regular React onClick handler.

## Project files

Create these files in an existing React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

Start the development server with `npm run dev`.

## src/App.jsx

~~~jsx
import { useEffect, useState } from 'react';

function App() {
  const [viewport, setViewport] = useState(null);
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(null);
  const [clickCount, setClickCount] = useState(0);

  useEffect(() => {
    function updateViewport() {
      setViewport({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', updateViewport);
    updateViewport();

    return () => {
      window.removeEventListener('resize', updateViewport);
    };
  }, []);

  useEffect(() => {
    const motionQuery = window.matchMedia(
      '(prefers-reduced-motion: reduce)',
    );

    function updateMotionPreference(event) {
      setPrefersReducedMotion(event.matches);
    }

    motionQuery.addEventListener('change', updateMotionPreference);
    setPrefersReducedMotion(motionQuery.matches);

    return () => {
      motionQuery.removeEventListener('change', updateMotionPreference);
    };
  }, []);

  function handleClick() {
    setClickCount((count) => count + 1);
  }

  return (
    <main className="page">
      <header className="page-header">
        <p className="eyebrow">React browser API example</p>
        <h1>Browser state and event lifetimes</h1>
        <p>
          Resize the browser or change its motion preference to see external
          browser events update this interface.
        </p>
      </header>

      <section className="card" aria-labelledby="viewport-heading">
        <h2 id="viewport-heading">Viewport size</h2>

        {viewport === null ? (
          <p>Reading the browser viewport…</p>
        ) : (
          <dl className="measurements">
            <div>
              <dt>Width</dt>
              <dd>{viewport.width} CSS pixels</dd>
            </div>
            <div>
              <dt>Height</dt>
              <dd>{viewport.height} CSS pixels</dd>
            </div>
          </dl>
        )}

        <p className="explanation">
          The window resize listener updates React state. Its Effect removes
          that same listener when this component is removed.
        </p>
      </section>

      <section className="card" aria-labelledby="motion-heading">
        <h2 id="motion-heading">Reduced motion preference</h2>

        {prefersReducedMotion === null ? (
          <p>Checking the browser preference…</p>
        ) : (
          <p className="preference">
            {prefersReducedMotion
              ? 'The browser currently requests reduced motion.'
              : 'The browser currently has no reduced-motion request.'}
          </p>
        )}

        <p className="explanation">
          The media query provides the initial value and sends a change event
          when the browser preference changes.
        </p>
      </section>

      <section className="card" aria-labelledby="click-heading">
        <h2 id="click-heading">A React event prop</h2>
        <p>
          This button uses onClick because the update is caused by a click on
          this React-rendered button.
        </p>
        <button className="button" type="button" onClick={handleClick}>
          Record a click
        </button>
        <p className="click-count">
          Button activated {clickCount} {clickCount === 1 ? 'time' : 'times'}.
        </p>
      </section>

      <footer className="page-footer">
        <p>
          Resize this window. If your operating system exposes a reduced-motion
          setting, change it while the page is open to see the media query
          update.
        </p>
      </footer>
    </main>
  );
}

export default App;
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

StrictMode is included so development checks can exercise the Effects' setup and cleanup. After the check, the component has one active resize listener and one active media query listener.

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  color: #172033;
  background: #f2f5f9;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

body {
  min-width: 20rem;
  min-height: 100vh;
}

button {
  font: inherit;
}

.page {
  width: min(100% - 2rem, 52rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin-bottom: 0.5rem;
  color: #4057a5;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  max-width: 18ch;
  margin-bottom: 0.75rem;
  font-size: clamp(2rem, 6vw, 3.25rem);
}

h2 {
  margin-bottom: 1rem;
  font-size: 1.25rem;
}

.page-header > p:last-child,
.explanation,
.page-footer {
  color: #4b5567;
  line-height: 1.6;
}

.card {
  margin-block: 1rem;
  padding: 1.25rem;
  border: 1px solid #d7deea;
  border-radius: 0.9rem;
  background: #fff;
  box-shadow: 0 0.25rem 1rem rgb(23 32 51 / 6%);
}

.measurements {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(10rem, 1fr));
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.measurements > div {
  padding: 0.9rem;
  border-radius: 0.65rem;
  background: #f2f5f9;
}

.measurements dt {
  margin-bottom: 0.35rem;
  color: #4b5567;
  font-size: 0.9rem;
}

.measurements dd,
.preference {
  font-size: 1.1rem;
  font-weight: 700;
}

.preference {
  margin-bottom: 1rem;
}

.button {
  margin-top: 1rem;
  padding: 0.7rem 1rem;
  border: 0;
  border-radius: 0.55rem;
  color: #fff;
  background: #304db3;
  cursor: pointer;
}

.button:hover {
  background: #243d96;
}

.button:focus-visible {
  outline: 3px solid #8096f5;
  outline-offset: 3px;
}

.click-count {
  margin-top: 0.75rem;
}

.page-footer {
  margin-top: 1.5rem;
  font-size: 0.95rem;
}
~~~

## What to observe

1. When the component appears, the viewport Effect registers its listener and reads an initial size.
2. Resizing the browser invokes the listener, which updates React state and rerenders the dimensions.
3. Changing the operating system's reduced-motion preference causes the media query's change listener to update the displayed preference.
4. Clicking the button runs its React onClick handler. No browser-level click subscription is needed.
5. When the component unmounts, both Effects remove the exact listener callbacks they registered.
6. Strict Mode may run setup, cleanup, and setup during development. The page should still have only one active listener for each browser source.

## Variations

- Extract viewport tracking into a custom Hook if several components need the same viewport value.
- Add an Escape key listener only while a dialog or panel is open, and remove it when the UI closes.
- Use an IntersectionObserver to show when a particular element enters the viewport.
- Add a timer in an Effect and clear it in cleanup.
- Start with another neutral state and keep browser API reads inside Effects if you adapt this example for server rendering.

## Expected behavior

The initial page can display a short reading message until the Effects run. The dimensions then match the current browser viewport and change after resizing. The motion message reflects the browser's current preference and updates when that preference changes. The click count changes only when the button is activated.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="034-file-inputs-and-upload-flows.md">034 - File inputs and upload flows - Example</a></td><td><a href="../lessons/035-browser-apis-and-event-listeners.md">Lesson</a></td><td><a href="../qnas/035-browser-apis-and-event-listeners.md">Q&amp;A</a></td><td><a href="036-accessible-interactive-form-project.md">036 - Build an accessible interactive form feature - Example</a></td></tr>
  </tbody>
</table>
