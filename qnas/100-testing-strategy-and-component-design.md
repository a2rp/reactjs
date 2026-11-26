# Testing Strategy and Testable Component Design: Questions and Answers

## Purpose and test levels

### 1. What should a React test prove?

It should prove a behavior that matters to a user or to a documented component contract. It should begin from a known state, exercise the relevant path, and assert an outcome that would reveal a regression.

### 2. What is a unit test?

A unit test checks a small deterministic unit in isolation, often a pure function. For example, test that a price formatter handles zero, ordinary values, and invalid input according to its contract. A React component does not have to be forced into a tiny isolated test if its behavior depends on rendered UI.

### 3. What is a component test?

It renders a component into a DOM and checks its visible behavior. The test can find a labeled input, enter text, activate a button, and verify the rendered result. This exercises React rendering and interaction together.

### 4. What is an integration test?

It checks that multiple real parts work together across a meaningful boundary, such as a form, validation logic, and an API adapter. Component and integration tests can overlap; the goal is to verify the real behavior without unnecessary setup.

### 5. What is an end-to-end test?

It checks an important workflow through the running application, commonly in a real browser. It can catch issues involving routing, browser behavior, build configuration, and several connected screens. Keep these tests focused on high-value workflows because they usually cost more to run and maintain.

### 6. How many tests should be at each level?

There is no universal ratio. Consider risk, complexity, execution speed, and how much confidence a level provides. Use fast tests for pure rules, DOM-based component tests for UI behavior, and a smaller number of browser tests for critical complete journeys.

### 7. Should every component have its own test file?

Not automatically. Test behavior where it gives useful confidence. A reusable component with a meaningful public contract may deserve direct coverage. A tiny internal component may be better exercised as part of the larger interaction users actually experience.

### 8. Are component tests always unit tests?

No. A component test often combines rendering, state, event handling, and nearby components, so it can be an integration test at a small scale. The name is less important than the behavior and boundary being tested.

## Assertions and accessible queries

### 9. What is a component's public contract?

It is the behavior callers and users can observe: accessible controls, displayed content, form values, disabled states, focus behavior, navigation, and documented callbacks or events.

### 10. What should I query first in a rendered UI test?

Prefer accessible roles and names for controls such as buttons, links, and headings. For form fields, prefer their labels. Visible text is useful for non-interactive output. Use test IDs only when semantic or user-visible queries do not fit.

### 11. Why use role and accessible name queries?

They resemble how people locate controls, including people using assistive technology. They also reveal when an important control lacks a useful accessible name. A test that cannot find a button by its name may point to a usability issue in the component.

### 12. Are test IDs forbidden?

No. They are an escape hatch for a stable element that has no meaningful role, label, or visible text for the test to use. They should not be the default way to select every element because users cannot see or hear them.

### 13. Should a test assert on CSS class names?

Usually not. Class names are implementation details and can change while behavior stays the same. Assert on the visible outcome, accessible state, or semantic element. A visual regression test may be appropriate when appearance itself is the requirement.

### 14. Should I inspect React component state in a test?

Usually no. A user cannot inspect a state variable or call an internal handler. Render the component, use the UI, and assert on the resulting page. Directly testing a pure exported function is appropriate when that function itself is the unit under test.

### 15. When is a callback assertion appropriate?

When the callback is part of the component's documented contract, such as onSubmit or onChange. Exercise the visible interaction first, then check the values passed across that boundary. Avoid asserting incidental internal calls that callers do not rely on.

### 16. Is calling an event handler directly a good test?

Usually not for UI behavior. Calling it skips the DOM, event wiring, browser semantics, and React update path. Use a simulated user interaction on the rendered control. Direct function calls are suitable for a pure helper, not as a replacement for testing a user flow.

## Component design and dependency boundaries

### 17. What makes a component easier to test?

Clear inputs and outputs, explicit props, recognizable ownership of state, and limited hidden dependencies. A component with a clear responsibility is easier to render and reason about than one that reads global state, talks to several services, and controls unrelated screens.

### 18. Should I extract every calculation into a helper?

No. Extract a calculation when it has meaningful independent rules, is reused, or becomes clearer outside the component. Do not create many tiny helpers only to increase the number of isolated tests.

### 19. Should every UI fragment become a component?

No. Extract components when the boundary improves responsibility, reuse, data flow, or readability. A component tree created only to make tests reach internal details can make the application harder to understand.

### 20. Where should network requests and storage access live?

Keep external effects at recognizable boundaries, such as a service, adapter, custom hook, or parent that owns the workflow. The exact shape depends on the application. The goal is to control outside dependencies in a test while keeping the React behavior being verified real.

### 21. What should I mock?

Mock or control dependencies that are external or nondeterministic: network responses, time, random values, storage, or third-party services. Keep the component and its relevant user-facing logic real. If mocks replace the behavior you meant to test, the test gives little confidence.

### 22. Should every child component be mocked in a parent test?

Usually no. Mocking every child can hide integration bugs and make the test rely on assumptions encoded in the mocks. Keep real child components when they are part of the behavior being tested. Mock only a boundary whose real behavior is outside the test's scope.

### 23. What makes a good mock response?

It is explicit, realistic enough for the contract, and chosen to verify a meaningful state. Cover relevant success, empty, and failure outcomes. Avoid a generic mock that always succeeds if failure handling matters.

## Async behavior and reliability

### 24. How do I test asynchronous UI?

Wait for the user-visible outcome with an asynchronous query or wait utility. Do not assume a request completed immediately, and do not use an arbitrary long sleep. The test should wait for the condition a user would observe.

### 25. Why are arbitrary sleeps unreliable?

A sleep may waste time when the result appears quickly and still be too short on a slower machine. It hides the actual condition the test cares about. Waiting for the rendered state is both more direct and more resilient.

### 26. How can tests remain independent?

Give each test its own render and interaction session. Reset shared mocks, timers, and storage when used. Do not rely on another test to prepare state, and clean up subscriptions or timers that could affect later tests.

### 27. Should tests use the network?

Most component tests should not depend on a live network. Control the request boundary so results are predictable and the test is fast. A dedicated integration or end-to-end environment may verify the real service connection when that is an explicit goal.

### 28. When should I use fake timers?

Use them when elapsed time is part of the behavior, such as debounce, delayed feedback, or a countdown. Keep real time for ordinary tests. If an interaction helper works with fake timers, configure it to advance the test runner's clock as its documentation requires.

## Snapshots, coverage, and quality

### 29. Are snapshots bad?

No. A small, intentional snapshot can help review stable rendered output. A large snapshot that is the only assertion for an interactive screen can hide important behavior and encourage approving changes without understanding them.

### 30. Does high code coverage prove that the feature works?

No. Coverage says code ran; it does not say the test checked the result or exercised meaningful input. Use coverage to locate untested branches, then decide which user-visible or risk-sensitive cases need assertions.

### 31. What should I prioritize if time is limited?

Prioritize behavior that is important, likely to break, or costly when broken. Include normal use, important boundaries, validation, loading and failure states, permissions, and critical flows. Keep assertions clear so a failing test helps diagnose the issue.

### 32. Do automated tests replace accessibility checks?

No. Automated tests can verify roles, labels, focus behavior, and some state changes. They cannot fully reproduce every browser and assistive technology combination. Manually check important workflows with keyboard navigation and the assistive technologies your users rely on.

### 33. Why can a test pass while the application is broken?

The test may check an internal detail rather than the user-visible contract, mock away the faulty behavior, skip the real interaction, or omit a failure path. Review whether the test starts from realistic inputs, uses the actual UI path, and asserts on an outcome that matters.

### 34. What is a useful test name?

Describe the condition and observable result, such as “disables increase when the quantity reaches its maximum.” A reader should understand the product behavior from the name without knowing internal variable names.

### 35. What is the main design lesson?

Good testability usually comes from clear responsibilities and explicit boundaries, not special hooks added only for tests. When a behavior is difficult to exercise, first ask whether the component owns too many concerns or hides important dependencies.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="099-accessible-announcements-and-async-ui.md">099 - Accessible Announcements and Asynchronous Interfaces: Questions and Answers</a></td><td><a href="../lessons/100-testing-strategy-and-component-design.md">Lesson</a></td><td><a href="../examples/100-testing-strategy-and-component-design.md">Example</a></td><td><a href="101-component-tests-and-test-utilities.md">101 - Component Tests and React Test Utilities: Questions and Answers</a></td></tr>
  </tbody>
</table>
