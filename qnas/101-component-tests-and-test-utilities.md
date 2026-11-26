# Component Tests and React Test Utilities: Questions and Answers

## Rendering and querying

### 1. What does React Testing Library's render function do?

It renders a React element into a DOM container attached to the test document and returns helpers for the rendered result. This lets the test inspect actual DOM output instead of a React component instance.

### 2. What is screen?

screen contains DOM Testing Library queries already bound to document.body. It is usually the clearest way to find content in the rendered page and can also find content rendered into the body through a portal.

### 3. Should I use screen or the queries returned by render?

Prefer screen for ordinary page-level queries. The render result still has bound queries, but screen makes it clear that the test is looking for something a user can find in the document.

### 4. What is the getBy query family for?

Use getBy when the matching element should exist now. It returns the element or throws a helpful error if there is no match or more than one match when only one is expected.

### 5. What is the queryBy query family for?

Use queryBy when the element may be absent. It returns null when no element matches, so it is useful for checking that content is not present. It still reports an error if there are multiple matches for a single-element query.

### 6. What is the findBy query family for?

Use findBy when an element should appear asynchronously, such as after a request or deferred state update. It returns a promise that resolves when a match appears or rejects after the query's retry period.

### 7. Should I use findBy just to slow down a test?

No. If content should already be present, use getBy. If it should remain absent, use queryBy. Use findBy only when the behavior actually includes an asynchronous appearance.

### 8. Why do accessible queries matter in component tests?

They search the interface through roles, names, and labels that correspond to the accessibility tree. They often make tests more resilient and can expose a missing accessible name or label in the component.

### 9. When should I use a test ID?

Use one when an element has no useful role, label, or user-visible text that fits the test. It is a fallback, not the normal selector for every element.

### 10. Should I query with container.querySelector?

Usually not. Prefer role, label, and text queries because those describe the rendered contract. Raw container selectors can be appropriate for special DOM structures or low-level integration, but should not replace semantic queries for normal controls.

### 11. What does screen.debug do?

It prints the current DOM to help diagnose why a query or assertion is failing. It is a temporary debugging aid, not a test assertion or a permanent workaround.

## Providers and reusable setup

### 12. How do I test a component that needs context?

Render it with the provider it uses in the application. The render function accepts a wrapper component, which can supply a theme, router, localization context, or another required provider.

### 13. What is a custom render helper?

It is a small application test function that calls the regular render helper with common provider setup. It reduces repeated boilerplate while keeping the component and its test inputs visible.

### 14. Should every test use one global wrapper with every application provider?

No. Add only the providers required by the component or scenario. An oversized wrapper can hide dependencies, add unnecessary setup, and make failures harder to diagnose.

### 15. How can a provider's value vary between tests?

Let the custom render helper accept the value as an input, then pass the relevant value for each test. Keep the default explicit and make special cases clear at the call site.

### 16. How can I test a portal rendered outside the component container?

Use screen, which searches document.body. The default render queries also use a base element that normally includes the body, so portal output can be queried without selecting from the component's inner container.

## Rerender, unmount, and cleanup

### 17. What does rerender do?

It renders another React element into the existing test root. It can check how an already rendered component responds when its input props change.

### 18. When should I use rerender?

Use it when replacing props on the rendered component is specifically the behavior under test. If the behavior is that a parent updates those props, render and test the parent-child path so the real data flow is covered.

### 19. Does rerender simulate a parent changing props?

It updates the rendered element directly. It does not test the application's actual parent logic. It is useful for the child component's response to new props, but parent ownership needs a parent-level test.

### 20. What does unmount do?

It removes the rendered React tree. Use it when the test needs to verify cleanup behavior, such as removing a subscription or stopping a resource when the component leaves the page.

### 21. What does cleanup do?

It unmounts rendered trees and removes their test containers from the document. This prevents DOM and effect state from one test leaking into another.

### 22. Is cleanup automatic?

React Testing Library attempts automatic cleanup when the test framework exposes a global afterEach hook. If your runner does not expose it globally, register cleanup explicitly in the test setup.

### 23. Does cleanup reset all shared test state?

No. It cleans up rendered React trees and DOM containers. Reset mocks, fake timers, storage, modules, or other shared state separately if the test uses them.

### 24. Should I call cleanup after every test manually?

Not if your configured runner and library already perform automatic cleanup. If automatic cleanup is unavailable, add one explicit afterEach cleanup registration in shared setup rather than repeating it in every test body.

## Hooks and renderHook

### 25. Can I call a Hook directly in a test?

No. Hooks must run while React renders a component. Calling a Hook as an ordinary function violates the Rules of Hooks and does not model its React lifecycle.

### 26. What does renderHook do?

It provides a small test component that calls a Hook and exposes its latest return value to the test. It can be helpful for a reusable Hook library whose exported Hook is itself the API.

### 27. Should I use renderHook for every application Hook?

No. For most application Hooks, test a component that consumes the Hook and assert on the resulting UI. That proves how the Hook affects real use and avoids treating an internal return shape as the only contract.

### 28. Can renderHook also use a provider wrapper?

Yes. It accepts a wrapper option for context providers just like render. If the behavior matters to a consuming component, also consider testing through that component.

## React updates and act

### 29. What does React act do?

It groups a unit of UI work and flushes the React updates related to it before assertions. This helps tests observe a committed result instead of an intermediate state.

### 30. Do I need to wrap every render in act?

No. React Testing Library wraps its normal render and common event helpers so ordinary component tests usually do not need explicit act calls.

### 31. When might I need explicit act?

When a test causes React updates outside the helpers that already handle them, such as directly resolving a deferred operation or advancing a timer that updates component state. Use async act and await the work before asserting.

### 32. Which act import should modern React tests use?

Use act from react or the wrapper exported by React Testing Library. Do not use act from react-dom/test-utils, which is deprecated.

### 33. What does an act warning mean?

A React update happened outside the test's expected update boundary or the test finished before related work settled. Check whether the test needs to await a visible result or wrap a deliberately direct update in act. Do not silence the warning without understanding it.

### 34. Does fireEvent need manual act around it?

React Testing Library's fireEvent helpers are wrapped with act. For ordinary user interaction, user-event is generally a closer model of how an interaction occurs. Avoid adding redundant act wrappers around helpers that already manage updates.

### 35. Does RTL setup the React act environment flag?

React Testing Library configures its supported test environment for act. A warning about the environment can indicate that React is being tested outside the normal library setup or that versions and configuration do not match.

## Choosing utilities

### 36. When should I inspect the raw DOM container?

Only when the behavior requires a low-level DOM operation or a structure that semantic queries cannot express. For ordinary content, prefer screen queries; direct container inspection often couples the test to markup details.

### 37. Do component tests need a real browser?

A DOM emulator is sufficient for many component rendering and interaction tests. It does not fully reproduce browser rendering, layout, or assistive technology behavior. Use real-browser tests and manual checks for behavior that depends on those environments.

### 38. What should I do if a test helper becomes complicated?

Keep the test setup near the test until repetition is real. Extract a small helper for repeated provider or fixture setup, but do not build a separate abstraction layer that hides the component, props, or expected behavior.

### 39. How do I know which utility to use?

Use render to mount a component, screen to query the document, and an appropriate getBy/queryBy/findBy family for the expected timing. Add a provider wrapper for required context. Use rerender, unmount, renderHook, and act only when their specific behavior is part of the question.

### 40. What is the main rule for these utilities?

Use helpers that make the test look like a clear account of rendered UI and its lifecycle. Keep the component visible in the test, use semantic queries, and let the test runner handle routine setup and cleanup where configured.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="100-testing-strategy-and-component-design.md">100 - Testing Strategy and Testable Component Design: Questions and Answers</a></td><td><a href="../lessons/101-component-tests-and-test-utilities.md">Lesson</a></td><td><a href="../examples/101-component-tests-and-test-utilities.md">Example</a></td><td><a href="102-testing-user-interactions.md">102 - Testing User Interactions and Complete Flows: Questions and Answers</a></td></tr>
  </tbody>
</table>
