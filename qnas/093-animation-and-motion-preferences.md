# Animation, Motion Preferences, and View Transitions: Questions and Answers

## Animation fundamentals

### 1. What role should animation play in a React interface?
It should help explain a state change, continuity, or hierarchy. Content and controls must stay understandable if every animation is skipped.

### 2. Does React animate DOM changes automatically?
No. React updates the UI from state and props. CSS, the Web Animations API, a browser view transition, or a React animation API supplies the animation behavior.

### 3. How should a React event trigger a CSS animation?
Update state from the handler and render a class or element based on that state. CSS can then transition a property or start a keyframe animation. Avoid manually mutating DOM styles for ordinary state-driven UI.

### 4. What is a CSS transition?
A transition interpolates a CSS property between its old and new computed values, such as a color or opacity change when a class changes.

### 5. What is a CSS keyframe animation?
It defines one or more intermediate values over time. Apply it through a selector, often when a class is added or an element appears.

### 6. When should I use a transition instead of keyframes?
Use a transition for a simple change between states. Use keyframes for a multi-stage sequence or an effect that is not just interpolation between two states.

### 7. Which properties are common choices for movement?
`transform` and `opacity` are common choices because they often avoid repeated layout work. Actual rendering depends on the browser and page, so profile important animations.

### 8. Why can animating `width`, `height`, `top`, or `left` be expensive?
They can affect layout and cause more rendering work on each animation frame. Prefer a transform when it preserves the intended behavior; not every effect can be expressed that way.

### 9. Should every update be animated?
No. Urgent typing, status confirmation, and frequent updates should remain responsive. Animate only changes where motion helps explain the result.

### 10. How do I represent an animated component's state?
Keep the meaningful state in React, then derive classes, attributes, and rendered content from it. The animation should not become a separate source of truth for application state.

## Reduced motion and accessibility

### 11. What does `prefers-reduced-motion` mean?
It detects a device preference to remove, reduce, or replace nonessential motion. It is a cue from the visitor's accessibility settings.

### 12. How do I handle it in CSS?
Use `@media (prefers-reduced-motion: reduce)` to disable, shorten, or replace nonessential transitions and animations. Keep important state changes visible through non-motion cues too.

### 13. Should all transitions be set to a tiny duration globally?
Not necessarily. A broad override can affect unrelated effects and hide feedback. Target nonessential motion deliberately and test the whole interface with reduced motion enabled.

### 14. Does React's `<ViewTransition>` automatically honor reduced motion?
No. The app should provide a reduced-motion treatment for its view-transition effects, commonly through CSS media rules for the relevant transition pseudo-elements or classes.

### 15. Is a CSS media query enough for a Web Animations API effect?
No. JavaScript can still start an imperative animation. Check the preference before starting it and cancel or adapt active animations if the preference changes while the page is open.

### 16. Should a motion preference change the application state?
Usually it should change only presentation. The same content and actions should remain available; only the nonessential movement should be reduced.

### 17. Can animation be the only way to indicate success or failure?
No. Use text, an appropriate accessible status, or another persistent cue. A visitor may not see or may not be able to use the animation.

### 18. What should an animated disclosure include?
Use a real button and expose whether the controlled content is open, for example with `aria-expanded`. The control should still work when the animation is disabled.

### 19. How should keyboard focus behave during a transition?
Keep focus on the control that initiated the action unless the interaction's semantics require moving it. Do not move focus just to follow a decorative animation.

### 20. How can I avoid distracting animation?
Use short, purposeful movement, avoid unnecessary loops and large continuous motion, and respect reduced-motion settings. Provide a clear way to stop an essential long-running effect.

## Browser View Transition API

### 21. What does the browser View Transition API do?
It captures representations of an old and new DOM state and animates between them. It can create continuity when a document or part of a single-page app changes.

### 22. What is `document.startViewTransition()`?
It is a browser API that coordinates a DOM update with a view transition. It is distinct from React's state-scheduling API named `startTransition`.

### 23. What is the difference between `startTransition` and `startViewTransition`?
React's `startTransition` marks React state work as non-urgent. `document.startViewTransition` is a browser API that captures and animates DOM snapshots around an update.

### 24. Do view transitions work in every browser version?
No. Support depends on the browser. Treat the animation as an enhancement and make the no-transition path fully usable.

### 25. How do CSS view-transition pseudo-elements work?
They target the captured old/new images and groups used for a browser view transition. They are different from ordinary DOM elements, so regular component selectors do not directly style those snapshots.

### 26. What is `view-transition-name` used for?
It associates a rendered element with a named transition image. Names are most useful for connecting an old element with its replacement during a shared-element transition; names must not conflict among active elements.

### 27. Can I call `document.startViewTransition()` around a React state update?
It is possible to coordinate updates manually, but avoid having both React's built-in `<ViewTransition>` and direct browser API calls manage the same update. Multiple coordinators can interrupt or conflict with each other.

## React View Transitions

### 28. What is React's `<ViewTransition>`?
In React 19.3, it is a stable DOM component that opts a UI boundary into browser view-transition animations for supported React transitions. It is not the same as a CSS transition.

### 29. When did `<ViewTransition>` become stable?
It became stable in React 19.3. Earlier builds exposed it experimentally through React canary releases, so check the version and channel used by an older project.

### 30. What kinds of changes can a React view transition represent?
React can classify a boundary entering, exiting, updating, or sharing a name between removed and inserted subtrees. The boundary placement affects what snapshot participates.

### 31. Does an ordinary `setState` activate `<ViewTransition>`?
Not automatically. React view-transition animations are activated by supported transition work, such as an update wrapped with `startTransition`, a Suspense reveal, or `useDeferredValue`.

### 32. When should I wrap a state update in `startTransition`?
When the update is non-urgent and can be treated as transition work. Keep urgent input feedback immediate. A view transition is a visual effect attached to transition work, not a reason to delay every interaction.

### 33. What is the default `<ViewTransition>` animation?
By default, React uses the browser's smooth cross-fade behavior. The boundary is represented by a snapshot, so it may move, scale, and fade as a group rather than every descendant being individually layout-animated.

### 34. How can I customize a React view transition?
Pass the supported enter, exit, update, share, or default class props, then style the corresponding CSS view-transition pseudo-elements. React applies the selected class when that transition type activates.

### 35. Should I set a `name` on every `<ViewTransition>`?
No. React generates unique names for regular boundaries. Use an explicit shared name only when an old element and a newly rendered element should be matched across locations.

### 36. What happens if two active view transitions use the same explicit name?
The browser cannot unambiguously match them as intended. Keep names unique among simultaneously active boundaries and use names only for deliberate shared-element transitions.

### 37. Does React's `<ViewTransition>` work in React Native?
The stable API described in this lesson is for the DOM. Verify platform-specific availability rather than assuming web transition APIs are shared with native renderers.

### 38. Can a view transition animate every nested element's layout independently?
Not by default. The browser animates captured images for the boundary. Place boundaries carefully or use another animation technique when individual elements need continuous independent movement.

## Implementation and debugging

### 39. Why does my React view transition not run?
Check that the project uses a React version with the API, the update is transition work, the boundary mounts/unmounts or changes as expected, and the browser supports the necessary API. Then verify the CSS class and reduced-motion rules.

### 40. Why does a custom view-transition CSS rule not affect the component itself?
View-transition pseudo-elements style captured snapshots, not the normal component element. Use ordinary selectors for the live DOM and `::view-transition-*` selectors for the transition images.

### 41. What should I do when JavaScript starts a Web Animation?
Keep the returned `Animation` handle, cancel it on cleanup or replacement, and check the user's motion preference before starting. Avoid starting it during React render.

### 42. How should I test reduced-motion behavior?
Enable the operating-system or browser reduced-motion preference and inspect the same interactions. Confirm animations are removed or toned down and that the resulting state remains clear and usable.

### 43. How do I debug a flash or jump during an animation?
Check whether the element is mounted in the expected state, whether its start and end styles are both defined, whether a reduced-motion rule intentionally disables it, and whether another layout or transition rule overrides it.

### 44. What is a good fallback for unsupported View Transitions?
Render and update the same content normally. The feature should add continuity where available; it must not gate rendering, navigation, or access to the new state.

### 45. When should I choose CSS animation, the Web Animations API, or a view transition?
Use CSS for selector-driven state changes and keyframe sequences. Use the Web Animations API for a custom imperative effect that CSS cannot express conveniently. Use view transitions when the old and new UI snapshots need continuity.
