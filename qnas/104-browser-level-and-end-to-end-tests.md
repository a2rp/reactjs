# Browser-Level Tests and End-to-End Behavior: Questions and Answers

## Purpose and scope

### 1. What does an end-to-end browser test prove?

It shows that a selected user journey works through the running application in a real browser. Depending on the environment, it can cover the built client, browser events, routing, styles, and real services. Its evidence is limited to the path and conditions the test exercised.

### 2. Is a browser test the same as a component test?

No. A component test checks a smaller piece of React behavior quickly, often in a DOM emulator. A browser test starts the application and drives a real browser through a broader path. Use component tests for many state and input combinations, and browser tests for a small set of critical journeys and browser-specific behavior.

### 3. Does passing an end-to-end test prove that an app is accessible?

No. It proves only that the tested interactions and assertions passed. Include accessible semantics in the interface, test keyboard paths and focus behavior where required, use accessibility-specific checks, and perform manual review with assistive technology when appropriate.

### 4. How many journeys should be end-to-end tests?

Choose a small number based on user and business risk: the critical task, an important route or authentication flow, and selected recovery cases. Keep exhaustive validation and edge-case matrices in faster unit, component, and integration tests unless real browser behavior is essential.

### 5. What makes a useful end-to-end test name?

It describes the user action and expected result, such as “customer receives an order confirmation after checkout”. Names like “checkout works” do not communicate the contract being protected.

## Setup and browsers

### 6. What is Playwright Test?

It is a browser automation test runner with test fixtures, assertions, projects, parallel execution, and failure diagnostics. It can run tests in Chromium, Firefox, and WebKit.

### 7. Why does the project need a local web server?

The test must exercise the application as it runs in a browser. Playwright's web server configuration can start the app and wait for it to become ready before running tests. Use the server mode and build step appropriate for the framework.

### 8. Should browser tests use the development server or production build?

Choose based on the behavior you want to protect. A dev server gives quick feedback. A production build served locally can catch build, asset path, and deployment-mode issues. Many projects use both: a quick development smoke path and a production-like CI run.

### 9. Which browser engines should be covered?

Use the engines your users and support policy require. Start with the highest-value browser, then add Firefox and WebKit for critical paths or scheduled coverage. More browsers increase confidence but also increase runtime and maintenance, so choose coverage deliberately.

### 10. Can tests run in parallel?

Yes, when each test owns its page, browser state, accounts, and data. Shared mutable state can make parallel tests interfere with one another. Isolate or namespace records and avoid ordering dependencies.

## Locators and assertions

### 11. Which locator should I use for a button?

Prefer a role and accessible name, for example `page.getByRole('button', { name: 'Save profile' })`. It reflects how users and assistive technology identify the control and is less coupled to implementation details.

### 12. When should I use a label locator?

Use `getByLabel` for form controls associated with visible labels. It makes the test's intent clear and helps catch missing or incorrect form labels.

### 13. Are test IDs always wrong?

No. A test ID can be useful when an element has no meaningful user-facing role, name, or label. Prefer user-facing locators when they fit, and use test IDs as an intentional stable hook instead of brittle classes or positional selectors.

### 14. What if a locator matches two buttons?

Make it more specific using a section, a meaningful accessible name, or a locator scoped to the relevant part of the page. Also check whether duplicate controls are an interface problem. Do not select the first match simply to silence ambiguity.

### 15. Why should assertions be web-first?

An awaited assertion such as `toHaveText` retries while the page reaches the expected state. This handles ordinary rendering and network timing without assuming when React will finish an update.

### 16. Why should I avoid fixed sleeps?

A fixed delay is both wasteful and unreliable: it slows fast runs and may be too short on slower machines. Wait for a visible state or a specific observable result instead.

### 17. Can I put a click inside `waitFor` or a retrying assertion callback?

Avoid it. Retry callbacks may execute more than once, so a click or submission inside one could be repeated. Perform the action once, then retry only the assertion about its result.

### 18. What does Playwright auto-waiting do?

Before an action such as clicking, Playwright waits for actionability conditions, including that the target is uniquely identified, visible, stable, and able to receive events. This reduces timing races, but it cannot make a wrong locator or a broken application correct.

## State, data, and network

### 19. What is a browser context, and why does isolation matter?

A browser context is an isolated browser session with its own cookies and storage. Separate contexts help tests start without another test's login or client-side state. Tests should still isolate server-side records and accounts too.

### 20. How should a test create its data?

Use deterministic setup: seed a dedicated test store, create records with unique identifiers, or use a controlled service. Clean up only data owned by that test. Never depend on another test having run first.

### 21. Should tests sign in through the UI every time?

Test the sign-in journey through the UI in a focused test. For unrelated journeys, a controlled test account or saved test storage state can reduce duplication. Keep that state private, limited to test accounts, and out of version control.

### 22. When should an API response be mocked?

Mock or intercept the API when the test is about the browser UI and needs predictable success, rejection, or failure states. This makes the case repeatable, but it does not prove that the real API or database works.

### 23. When should the browser test call a real backend?

Use a dedicated test service when the test is meant to verify the integration across browser, client, API, and persistence. Seed and clean up known test data, and keep the environment separate from production and other users' work.

### 24. Should every end-to-end test call third-party services?

No. A live third-party dependency can add rate limits, outages, changing data, and cleanup problems. Control the boundary for UI tests and keep live integration coverage small and intentional.

### 25. How do I make tests independent when they modify server data?

Give each test its own data namespace or account, seed only what it needs, and clean up only what it created. If the environment cannot safely isolate concurrent writes, serialize the relevant suite or provide disposable environments.

## Browser coverage and failures

### 26. What should be tested in a real browser instead of only a DOM emulator?

Prioritize requirements that depend on actual browser behavior: native form validation, focus behavior, history and deep-link refreshes, CSS layout, keyboard navigation, file or clipboard APIs, and production-bundle startup. The exact cases depend on the app.

### 27. Do end-to-end tests replace visual regression tests?

No. An assertion that a heading is visible does not verify exact layout or pixel-level appearance. Use visual comparisons only when appearance itself is part of the requirement, with review for environment-specific rendering differences.

### 28. What should I inspect when a test fails in CI?

Inspect the failing step, trace, screenshot, HTML report, browser console, and relevant network activity. Determine whether the cause is a real application defect, bad expectation, ambiguous locator, failed server, leaked data, or unstable external dependency.

### 29. Should I increase retries if a test is flaky?

Retries can capture diagnostics, but passing only after a retry is evidence of instability. Investigate the cause-shared data, an uncontrolled service, a weak locator, or timing assumptions-and fix it instead of treating retries as the solution.

### 30. Is a failed end-to-end test always an application bug?

No. The test server may not be ready, the data may be wrong, a network dependency may be unavailable, or the test may have an invalid expectation. Use artifacts to separate test-environment failures from product failures.

### 31. Can a test simply check that a button is clickable?

That is usually not enough. Assert the result the user needs, such as a saved message, changed route, updated list, or confirmation. The interaction itself is meaningful only when its outcome is also correct.

### 32. How should browser tests run in CI?

Install the required dependencies and browsers, build or start the app in the chosen mode, wait for the local server, then run tests. Keep secrets in the CI secret store and preserve reports or traces for failures. Add browser projects according to risk and available runtime.

### 33. What is the difference between a smoke test and the full browser suite?

A smoke test checks that a small number of essential paths still start and work. A broader suite covers additional critical journeys, browser engines, and recovery behavior. Keep the fast path useful for frequent changes and schedule broader coverage as the project needs.

### 34. What is the key rule for a reliable browser test?

Make the test state explicit, perform the user action once, and assert the user-visible result. Use controlled boundaries where appropriate and investigate failure evidence instead of hiding it with sleeps or loose assertions.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="103-testing-forms-and-async-ui.md">103 - Testing Forms, Asynchronous UI, and API Boundaries: Questions and Answers</a></td><td><a href="../lessons/104-browser-level-and-end-to-end-tests.md">Lesson</a></td><td><a href="../examples/104-browser-level-and-end-to-end-tests.md">Example</a></td><td><a href="105-react-devtools-and-debugging.md">105 - React DevTools, Debugging, and Common Warnings: Questions and Answers</a></td></tr>
  </tbody>
</table>
