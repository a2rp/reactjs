# Testing Strategy and Testable Component Design

## Learning goals

After this lesson, you should be able to:

- Choose a test level based on the behavior and risk you need to verify.
- Explain unit, component integration, and browser end-to-end tests.
- Write assertions around visible behavior and accessible semantics.
- Design component boundaries that make behavior straightforward to exercise.
- Keep tests deterministic without mocking away the behavior under test.
- Understand why coverage and snapshots are useful signals, not proof of correctness.

## 1. Tests protect behavior

A React test should answer a practical question: if someone uses this interface, does it respond correctly? A useful test gives confidence about behavior that matters, catches a meaningful regression, and explains the expected result when it fails.

Tests are executable documentation. A test named “shows an error when saving fails” communicates an important product rule. A test named “sets state variable to true” mostly describes an implementation choice that may change during a refactor.

Testing is not a substitute for manual accessibility checks, browser checks, or careful design. It is one part of a feedback loop that helps changes remain safe.

## 2. Choose the test level that fits the question

Different tests provide different kinds of confidence:

| Test level | Best for | Example |
| --- | --- | --- |
| Pure unit test | A deterministic function with no React or browser dependency | Formatting a price or clamping a number |
| Component test | Rendered UI, component state, and interaction with nearby UI | Clicking a quantity button updates the displayed quantity |
| Integration test | Several real parts working together across a meaningful boundary | A form submits, calls an API adapter, and presents success or failure |
| Browser end-to-end test | A critical workflow in a real browser and application build | A person completes checkout and sees confirmation |
| Manual check | Behavior that depends on real assistive technology, browser rendering, or a device | Confirming focus and announcements with a screen reader |

These categories overlap. A component test that exercises a user flow through rendered controls is also an integration test at a small scale. The label matters less than the question the test answers.

Do not make every test an end-to-end test. Browser tests cover valuable paths but take longer to run and can involve more setup. Do not make every test a tiny isolated unit test either: an isolated test can pass while the rendered interface is unusable. Use a mix based on risk, complexity, and how quickly the team needs feedback.

## 3. Test the public contract

A component's public contract is what a user or parent component can observe: its text, accessible roles and names, form values, enabled or disabled controls, focus behavior, navigation, and documented callbacks.

For rendered React UI, prefer queries that reflect that contract:

1. Find buttons, links, headings, and other controls by role and accessible name.
2. Find form fields by their associated label.
3. Find visible explanatory or status text when that is what the user sees.
4. Use a test identifier only when no meaningful user-facing or semantic query fits.

For example, a test can find the button named “Save changes,” activate it, and assert that “Changes saved” appears. It should not need to inspect a private state variable, component instance, internal hook call, or CSS class name.

If a test cannot find an important control by role or label, that can reveal a missing accessible name or label in the interface. Fixing the UI can improve both usability and test clarity.

## 4. Keep the test close to how people use the interface

A component test should render the component into a DOM and interact with the resulting controls. Use an interaction helper that models a user action, such as clicking a visible button or typing into a labeled field. Then assert on the resulting page.

A test usually follows three steps:

1. Arrange the initial component and its inputs.
2. Act through a user-visible control or documented integration boundary.
3. Assert on the visible result or a relevant callback contract.

A test should not reproduce React's implementation. Avoid calling event handlers directly when the event can be triggered through the UI, setting a component's state from the test, or asserting the exact component tree. These shortcuts can make a test pass without proving the user path works.

For a pure function, direct input and output tests are appropriate because the function itself is the public unit. For example, a quantity clamp function can be tested with a value below the minimum, within range, and above the maximum.

## 5. Design components with clear boundaries

Testability usually follows from clear responsibilities. A component is easier to reason about when it:

- Receives changing data through explicit props.
- Renders output from its props and state without hidden global dependencies.
- Sends user intent through named event callbacks when a parent owns the decision.
- Keeps side effects such as network access, storage, and timers at recognizable boundaries.
- Separates reusable domain calculations from rendering when the calculation has meaningful independent rules.

For example, an order form can receive its initial values and an onSubmit callback. The form owns typing and validation display, while a parent or service layer owns sending the order. A test can verify the form's visible behavior and that submitting valid data calls the documented callback with expected values.

This does not mean every line needs a new component or helper. Do not extract code only to make a test easier to reach. A useful boundary should clarify responsibility, reuse, data flow, or side effects as well as testing.

## 6. Mock boundaries, not the behavior under test

A mock replaces a dependency with a controlled substitute. It is useful for an external or nondeterministic boundary, such as a network response, current time, browser storage, or a third-party service.

Keep the component, React state updates, and user-facing logic real when those are what the test is meant to verify. If a component test mocks every child component, hook, and interaction, it may prove only that the mocks agree with one another.

Use realistic success, empty, and failure responses for external boundaries. Make the expected response explicit in the test. Prefer a request-level mock or a small injected adapter over rewriting the internals of the UI. The right seam is the one that keeps the behavior under test intact while controlling the outside world.

## 7. Make tests deterministic and independent

A reliable test has a known starting state and does not depend on execution order. Each test should create its own user session and render what it needs. Reset shared mocks, fake timers, and storage between tests when they are used. Clean up pending timers or subscriptions when the component unmounts.

Async UI needs async assertions. Wait for the user-visible result rather than adding an arbitrary sleep. A fixed delay can make a test slow when the UI is fast and flaky when the UI is slower than expected.

Use fake time only when time itself is part of the behavior, such as a debounce or countdown. When simulating typed input with fake timers, connect the interaction helper to the test runner's timer advancement API as required by that helper.

Avoid tests that depend on network access, the machine clock, random values, or a specific order unless that dependency is deliberately controlled.

## 8. Treat snapshots and coverage as supporting tools

A snapshot records rendered output for comparison. A small snapshot can help review a stable, meaningful structure, but a large snapshot can change for incidental reasons and be accepted without understanding the difference. A snapshot does not explain whether a button is usable, whether an error is clear, or whether a keyboard user can complete the task.

Coverage reports which code ran while tests executed. High line coverage does not prove that assertions are meaningful or that important states were checked. A test can execute a line without checking its result.

Use coverage to find untested branches and decide whether missing cases matter. Prioritize behavior and risk: boundaries, invalid inputs, loading and failure states, permissions, and important user journeys. A smaller suite of clear, behavior-focused tests is often more valuable than a large suite of brittle implementation checks.

## 9. A practical planning method

Before writing a test, state the behavior in one sentence. Then answer:

- Who uses this behavior, and what can they observe?
- What is the normal path?
- What boundary or failure path would be costly if broken?
- Which dependencies must be controlled for a repeatable result?
- Which test level can prove this behavior with the least unnecessary setup?

For a quantity selector, the behavior might be: “A person can increase the quantity up to the allowed maximum, and the control cannot go beyond that maximum.” A component test can render the selector, click its named button, and verify both the displayed quantity and the disabled boundary. There is no need to inspect its state hook.

For a critical payment workflow, a component test alone may not be enough. Add a small number of browser tests to verify that the built application, routing, form, and payment result work together. Keep the remaining business rules covered by faster tests.

## Common mistakes

- Testing private state, component instances, hook call counts, or exact internal markup.
- Reaching into the DOM by CSS class when a user-facing role or label is available.
- Mocking the component's own logic and treating the test as proof of that logic.
- Using snapshots as the only assertion for an interactive screen.
- Treating a coverage percentage as a measure of test quality.
- Adding arbitrary sleeps to async tests.
- Sharing mutable setup between tests and depending on their order.
- Extracting tiny components solely to make tests possible.
- Using only isolated unit tests for UI that must work as a complete interaction.
- Using only slow browser tests for every small calculation and edge case.

## Summary

Choose tests by the behavior and risk they need to cover. Test pure logic as pure logic, and test rendered React components through their accessible, user-facing controls. Keep important dependencies real, control external boundaries, and make setup deterministic. Clear component responsibilities and explicit data flow make behavior easier to test and maintain. Use snapshots and coverage as clues, then rely on meaningful assertions and manual checks for the confidence a feature needs.

## Practice checklist

1. Write one sentence describing a component's user-visible contract.
2. Identify one pure rule and test its boundary values directly.
3. Render the component and locate controls by accessible role and name.
4. Perform an interaction and assert the visible result.
5. Identify an external dependency and choose a seam to control it.
6. Add one failure or boundary case that matters to the user.
7. Remove any assertion that depends on private state or incidental markup.
