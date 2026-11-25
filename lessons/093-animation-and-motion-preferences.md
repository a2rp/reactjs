# Animation, Motion Preferences, and View Transitions in React

## Learning goals

After this lesson, you should be able to:

- Choose between a CSS transition, CSS keyframes, the Web Animations API, and a view transition.
- Use React state to select an animation state without manually changing the DOM.
- Respect the visitor's reduced-motion preference for both CSS and JavaScript-driven animation.
- Explain the browser View Transition API and how React 19.3's `<ViewTransition>` coordinates it.
- Use `startTransition` when an update should participate in React's View Transition animation.
- Keep content understandable and usable when motion is reduced or unavailable.

## 1. Why add motion?

Motion can show that an item was added, connect a changed layout to its previous state, or help a visitor understand where content moved. It should support the task rather than compete with it.

An animation is decoration; the application's state and content must still be correct if the animation is skipped. Keep controls responsive, preserve focus, and communicate meaningful state changes with text or semantics instead of motion alone.

## 2. Let React render state; let CSS animate it

A React event handler should update state. React renders the resulting classes and markup, and CSS animates the changed visual properties:

```tsx
import { useState } from "react";

export function DetailsToggle() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <section className={isOpen ? "details is-open" : "details"}>
      <button
        type="button"
        aria-expanded={isOpen}
        onClick={() => setIsOpen((open) => !open)}
      >
        {isOpen ? "Hide details" : "Show details"}
      </button>
      {isOpen && <p>The details are rendered from React state.</p>}
    </section>
  );
}
```

The `aria-expanded` value exposes the disclosure state. React controls whether the content exists; a CSS class may style its presentation. Avoid using refs to repeatedly set `element.style` for ordinary state-driven UI because it creates a second source of truth outside React.

## 3. CSS transitions and keyframe animations

### Transitions

A transition animates a property when its computed value changes between states:

```css
.details {
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: white;
  transition: border-color 160ms ease, background-color 160ms ease;
}

.details.is-open {
  border-color: #3157d5;
  background: #f4f6ff;
}
```

Transitions are useful for simple changes between two visual states, such as color, opacity, or a small transform. A transition does not create a new state; it interpolates the CSS property after the state changes.

### Keyframes

A keyframe animation describes a sequence of values and can run when a class is applied:

```css
@keyframes toast-enter {
  from {
    opacity: 0;
    transform: translateY(0.5rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.toast.is-entering {
  animation: toast-enter 180ms ease-out both;
}
```

Keyframes suit a multi-step effect or an animation that runs independently of a property changing between two classes. Avoid looping or attention-grabbing animation unless it has a clear purpose and an accessible way to stop or reduce it.

### Which CSS properties?

Prefer animating `transform` and `opacity` for common movement and fade effects. Animating layout-affecting properties such as `top`, `left`, `width`, or `height` can cause repeated layout work. Browser rendering varies, so profile important animations instead of assuming every transform is free.

Choose durations that keep the interface responsive. Use easing to make motion feel natural, but keep the timing consistent across related components.

## 4. Respect reduced-motion preferences

A visitor's device can request less nonessential motion. CSS can adapt transitions and animations with `prefers-reduced-motion`:

```css
.card {
  transition: transform 180ms ease, box-shadow 180ms ease;
}

.card:hover {
  transform: translateY(-0.2rem);
}

@media (prefers-reduced-motion: reduce) {
  .card {
    transition: none;
  }

  .card:hover {
    transform: none;
  }

  .toast.is-entering {
    animation: none;
  }
}
```

Reducing motion does not require removing all feedback. A changed border, text, or icon can still make the state clear. Avoid replacing a large movement with another movement that causes the same discomfort.

A broad project-level rule can reduce durations, but targeted rules make it easier to preserve essential feedback and avoid accidentally affecting unrelated animations. Test the interface with the operating-system preference enabled.

### JavaScript-started animations

For an animation created with the Web Animations API, check the preference before starting motion. If the preference can change while the page is open, subscribe to the media query and cancel or adjust active animations when it changes. CSS media rules alone cannot prevent JavaScript from starting an imperative animation.

Use imperative animation only when CSS cannot reasonably express the behavior. Retain and cancel the returned `Animation` object when the component unmounts or the effect is superseded. Keep animation work out of render; rendering should stay pure.

## 5. The browser View Transition API

The View Transition API captures a representation of the old and new document states and animates between them. It is useful for a page or element that is replaced or moved while the visitor should retain a sense of continuity.

The browser also offers `document.startViewTransition()` for imperative same-document transitions. Browser support depends on the visitor's browser, so the page should remain correct without the transition. The feature is visual enhancement, not a prerequisite for navigation or state updates.

Do not confuse the browser's `document.startViewTransition()` with React's `startTransition()`:

- `document.startViewTransition()` is a browser API that coordinates a DOM update with a visual snapshot transition.
- React's `startTransition()` marks React state work as non-urgent.

They have related names but different jobs.

## 6. React's `<ViewTransition>` component

React 19.3 makes `<ViewTransition>` a stable DOM API for coordinating UI boundaries with the browser's View Transition API. It can animate a boundary entering, exiting, changing, or being shared between locations.

```tsx
import { startTransition, useState, ViewTransition } from "react";

export function RevealCard() {
  const [isVisible, setIsVisible] = useState(false);

  function reveal() {
    startTransition(() => {
      setIsVisible((visible) => !visible);
    });
  }

  return (
    <section>
      <button type="button" onClick={reveal}>
        {isVisible ? "Hide card" : "Show card"}
      </button>

      {isVisible && (
        <ViewTransition>
          <article className="card">
            <h2>Welcome back</h2>
            <p>This card enters as part of a React transition.</p>
          </article>
        </ViewTransition>
      )}
    </section>
  );
}
```

The state update is wrapped in `startTransition`, so React can treat it as transition work and activate the boundary's view transition. Urgent updates made with a normal state setter do not automatically activate `<ViewTransition>`.

The default effect is a browser-provided cross-fade. React selects enter, exit, update, or shared-element behavior from how the boundary changes. The image-based snapshot may move and scale the boundary as a whole; it does not animate every descendant as an independent layout object.

### Customizing React view transitions

In React 19.3, transition class props can select CSS classes for different activation types:

```tsx
<ViewTransition enter="card-enter" exit="card-exit" default="none">
  <article className="card">...</article>
</ViewTransition>
```

The CSS View Transition pseudo-elements can target the generated class:

```css
::view-transition-new(.card-enter) {
  animation: card-enter 180ms ease-out both;
}

::view-transition-old(.card-exit) {
  animation: card-exit 140ms ease-in both;
}

@keyframes card-enter {
  from {
    opacity: 0;
    transform: translateY(0.5rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes card-exit {
  to {
    opacity: 0;
  }
}
```

React applies its View Transition class when that type of transition activates. Use the CSS view-transition pseudo-elements to style the captured old or new image; ordinary component selectors do not style those snapshots directly.

Only use a shared `name` when connecting an element removed from one location to an element added elsewhere in the same transition. Names need to be unique among simultaneously active boundaries. For ordinary enter and exit animations, let React generate unique names.

### Motion preferences still apply

React does not automatically turn off `<ViewTransition>` animations when the visitor requests reduced motion. Adapt or disable the corresponding view-transition animation in CSS:

```css
@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(.card-enter),
  ::view-transition-group(.card-exit),
  ::view-transition-old(.card-exit),
  ::view-transition-new(.card-enter) {
    animation-duration: 1ms;
  }
}
```

For a simpler experience, choose no view-transition class in the reduced-motion state or use an appropriate `none` setting where the React boundary supports it. Keep all content and controls available even when the animation is skipped.

React's built-in `<ViewTransition>` should coordinate React-driven view transitions. Avoid independently starting `document.startViewTransition()` around the same React update; two owners can interrupt or conflict with one another.

## 7. Choosing the right tool

| Need | Starting point |
| --- | --- |
| Hover, focus, or simple state change between CSS values | CSS transition |
| Multi-step or entrance effect controlled by a class | CSS keyframes |
| Custom imperative effect not practical in CSS | Web Animations API with cleanup |
| Continuity between replaced or moved UI snapshots | View Transition API |
| React 19.3 DOM update coordinated with a view transition | React `<ViewTransition>` with transition work |

Do not animate every update. A text field value or urgent confirmation should normally update immediately. Keep motion for changes where it helps the visitor understand the result.

## 8. Common mistakes

- **Confusing the two transition APIs:** React `startTransition` schedules non-urgent React work; `document.startViewTransition` is a browser DOM API.
- **Expecting a regular state update to activate `<ViewTransition>`:** React view transitions require transition work, Suspense reveal, or another supported transition source.
- **Ignoring reduced motion:** CSS and React View Transitions need an explicit reduced-motion treatment.
- **Animating essential content into invisibility:** Motion can be skipped; the final state must remain readable and usable.
- **Starting animations during render:** Start animation in response to state/effects, not while calculating JSX.
- **Leaving Web Animations running:** Cancel imperative animations on cleanup or when replacing them.
- **Using duplicate shared names:** Simultaneously active view-transition names must not collide.
- **Using large continuous movement for decoration:** It can distract and cause discomfort; choose brief, purposeful movement.
- **Making animation the only status cue:** Pair it with semantic state, text, or another non-motion indicator.
- **Calling the browser API around React's own ViewTransition work:** Let one system coordinate the same DOM update.

## Summary

React state determines the UI state; CSS transitions and keyframes animate visual changes. `prefers-reduced-motion` lets CSS tone down or remove nonessential effects, while JavaScript-started animations need their own preference handling and cleanup. The browser View Transition API animates snapshots between DOM states. React 19.3's stable `<ViewTransition>` coordinates React transitions with that browser capability; it does not animate ordinary urgent updates automatically and does not honor reduced motion unless the app styles for it.
