# Browser-Level Tests and End-to-End Behavior

## Learning goals

After this lesson, you should be able to:

- Explain what a browser-level end-to-end test proves and what it does not prove.
- Choose a small set of high-value user journeys for browser automation.
- Start a React application for Playwright tests and configure a base URL.
- Locate controls by their accessible role, name, or label.
- Wait for observable outcomes without fixed delays.
- Keep browser tests isolated and their data deterministic.
- Control network boundaries when a live service is not part of the test.
- Diagnose failures with traces, screenshots, and browser output.

## 1. What an end-to-end browser test covers

A browser-level test drives the application through a real browser engine. It can exercise the built client, browser DOM and events, CSS layout, routing, and the user-visible result of interactions. Depending on the test environment, it can also cross a real API and database boundary.

A useful end-to-end test follows an important user journey from its entry point to its outcome. For example, it can open a product page, add a product to a cart, submit checkout details, and verify the confirmation shown after the server accepts the order.

This scope is broader than a component test. A component test can check a form or widget quickly in a DOM emulator. It does not prove that the built app starts, routes load, styles work in a browser, or browser-specific behavior matches expectations. Conversely, an end-to-end test is slower and covers more moving pieces, so it is a poor way to exhaustively test every small branch or input combination.

Use a mix of test levels:

- Unit tests check small pure functions and isolated logic.
- Component tests check React behavior, events, and UI states efficiently.
- Integration tests check connected components and controlled service boundaries.
- Browser end-to-end tests protect a small number of critical user journeys and browser-specific behavior.

An end-to-end test does not automatically prove accessibility, visual correctness, backend correctness, or security. Add focused checks at the appropriate level. A successful click test cannot replace keyboard and accessibility checks; a mocked API response cannot prove that the production API works.

## 2. Select journeys by user and business risk

Start with the paths whose failure would block an important task. Typical candidates include:

- Signing in and reaching a protected area.
- Completing a primary form or purchase.
- Editing or deleting important data.
- Recovering from a validation or service error.
- Navigating directly to a route and refreshing it.

Keep each test focused on one outcome. Cover a successful journey and a small number of meaningful failure or recovery paths. Test permutations, detailed validation rules, and unusual edge cases in faster component or integration tests unless they depend on real browser behavior.

A clear test name states the user action and result. Prefer “customer receives an order confirmation after checkout” over “checkout works”.

## 3. Set up Playwright Test

Playwright Test provides a test runner, browser automation, fixtures, assertions, parallel execution, and failure artifacts. It can run the app in Chromium, Firefox, and WebKit. Install the runner in the project, then install the browser binaries used by the project:

```sh
npm install --save-dev @playwright/test
npx playwright install
```

A minimal configuration for a Vite app can build the production bundle, start its preview server, and use that server as the test base URL:

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: Boolean(process.env.CI),
  retries: process.env.CI ? 1 : 0,
  reporter: 'html',
  use: {
    baseURL: 'http://127.0.0.1:4173',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run preview -- --host 127.0.0.1 --port 4173 --strictPort',
    url: 'http://127.0.0.1:4173',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

This assumes the project has a Vite `preview` script and a current production build. If your project uses another framework or server, build it with that framework's command and replace the preview command with the project's local serving command. Keep the port and the server readiness check aligned. Do not point automated tests at a production site or a shared environment that contains real user data.

Run the suite with:

```sh
npx playwright test
```

To check more browser engines, add projects for Firefox and WebKit. Add browser coverage intentionally: a critical cross-browser path may deserve all supported engines, while an early smoke suite may use one engine and a smaller scheduled matrix.

## 4. Locate controls the way users identify them

Prefer locators based on the accessible interface:

```ts
page.getByRole('button', { name: 'Place order' });
page.getByLabel('Email');
page.getByRole('heading', { name: 'Order confirmed' });
```

Role and name locators encourage the application to expose meaningful semantics and are less coupled to markup structure. Label locators work well for form fields. Text locators can be useful for content. Use a test ID when an element has no suitable user-facing identifier and adding one is reasonable.

Avoid selectors tied to generated class names, DOM depth, or incidental implementation details. Avoid selecting the third button because it happens to be in position three. If a locator matches more than one element, make the query more meaningful or fix the ambiguous interface rather than silently choosing an arbitrary match.

A locator is a description of the element, not a saved DOM node. Playwright resolves it when an action or assertion runs. This helps when React re-renders the page between steps.

## 5. Use auto-waiting and web-first assertions

Playwright actions wait for the target to become actionable. For a click, it checks conditions such as uniqueness, visibility, stability, and whether the element can receive events. Assertions such as `toBeVisible` and `toHaveText` retry until the expected state appears or the assertion timeout expires.

Write the test in terms of the observed result:

```ts
await page.getByRole('button', { name: 'Save profile' }).click();
await expect(page.getByRole('status')).toHaveText('Profile saved.');
```

Do not add a fixed delay to guess when a render or request has finished. A sleep slows a fast run and still fails when the machine is slower than expected. Do not read a value once and immediately compare it if that value is expected to change asynchronously; use a retrying assertion for the expected state.

Keep each user action outside retry callbacks. A repeated assertion may be retried safely; a click or form submission inside a repeatedly executed callback could happen more than once.

## 6. Isolate each test

Playwright's page and browser-context fixtures give tests separate browser state. A fresh context isolates cookies, local storage, and session state. Keep tests independent so that order and parallel scheduling do not change the result.

For each test, make its data predictable:

- Create records with unique, known identifiers, or reset a dedicated test store.
- Do not rely on data left behind by another test.
- Do not share mutable accounts between tests that run in parallel.
- Keep credentials and test-only secrets out of source control.
- Use a dedicated test environment and accounts; never mutate production data.

If a test needs a signed-in session, create a controlled test account and use a saved storage state where appropriate. Treat the saved state as a credential: do not commit it, publish it, or reuse a real user's browser session.

## 7. Decide where the network belongs

Choose the network boundary to match the claim the test is meant to prove.

**Use a controlled response** when the test is about the React interface and needs a predictable success, rejection, or failure. Intercepting the API avoids dependency on a running service and makes the state repeatable. It does not prove that the real API or database works.

**Use a real test service** when the journey must verify how the browser, deployed client, API, and persistence work together. Point it to a disposable test environment, seed known data, and clean up safely. Keep it isolated from production and from unrelated developers' data.

Do not make every browser test hit a live third-party service. That adds rate limits, outages, changing content, and data ownership to a test whose purpose may be only to check the interface. Cover the integration with a smaller, deliberately managed set of tests.

## 8. Cover browser behavior that a DOM emulator cannot settle

Use an actual browser check when the requirement depends on browser behavior, such as:

- Native constraint validation and focus behavior.
- Route refreshes, history navigation, or deep links.
- Responsive layout, overflow, or an important visual state.
- Keyboard navigation and focus restoration after a dialog closes.
- Download, upload, clipboard, or other browser APIs.
- Hydration or an interaction that must work after the production bundle loads.

Browser tests can verify that an interaction is possible and that its visible outcome is correct. They do not replace dedicated accessibility review, assistive-technology testing, or visual regression checks when those are required.

## 9. Diagnose failures with evidence

When a test fails, inspect the step that failed and the browser state at that moment. Use the HTML report, trace, screenshot, console output, and network activity to distinguish among:

- An incorrect product expectation.
- A selector that no longer identifies the intended control.
- A real app error or failed asset/API request.
- Test data leaking between cases.
- A server that was not ready or used the wrong port.
- A genuine timing or browser-specific issue.

Reproduce the failure locally and fix its cause. Do not make the assertion looser or add a delay just to get a green run. Retries can collect useful diagnostics in CI, but a test that passes only on retry is still a flaky test to investigate.

## 10. Keep the suite useful in continuous integration

A practical CI setup installs the project dependencies and required browsers, builds the application, runs the suite against a local test server, and saves the HTML report or trace when a test fails. Keep secrets in the CI secret store and use a dedicated test configuration.

Run the critical smoke tests on changes that affect the application. Run a wider browser matrix when it provides useful coverage and the build capacity is available. Track repeated failures and remove sources of nondeterminism; do not normalize flaky tests as expected noise.

## 11. Common mistakes

- **Automating every UI detail end to end:** move exhaustive logic and validation cases to faster test levels.
- **Using CSS classes as the main contract:** query visible roles, names, and labels.
- **Waiting for a fixed number of milliseconds:** wait for the resulting state.
- **Sharing one mutable user across parallel tests:** isolate accounts and data.
- **Depending on a live external service in every test:** use controlled responses for UI behavior and a small test-service suite for integration.
- **Testing only that a button can be clicked:** assert the resulting user-visible behavior.
- **Treating a passing browser run as an accessibility audit:** add accessibility-specific checks and manual review.
- **Using retries to hide instability:** inspect traces and remove the underlying nondeterminism.

## Summary

Browser end-to-end tests protect a small number of important journeys in the real application and browser. Configure a local test server, use semantic locators, rely on auto-waiting assertions, and isolate browser state and data. Control external boundaries when testing the UI, and use a dedicated test service when the integration itself is the subject. Diagnose failures from traces and observable evidence instead of hiding them with sleeps or broad assertions.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="103-testing-forms-and-async-ui.md">103 - Testing Forms, Asynchronous UI, and API Boundaries</a></td><td><a href="../qnas/104-browser-level-and-end-to-end-tests.md">Q&amp;A</a></td><td><a href="../examples/104-browser-level-and-end-to-end-tests.md">Example</a></td><td><a href="105-react-devtools-and-debugging.md">105 - React DevTools, Debugging, and Common Warnings</a></td></tr>
  </tbody>
</table>
