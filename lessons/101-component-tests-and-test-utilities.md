# Component Tests and React Test Utilities

## Learning goals

After this lesson, you should be able to:

- Render a React component into a test DOM with React Testing Library.
- Use screen and accessible queries to find rendered content.
- Choose between getBy, queryBy, and findBy query families.
- Supply context providers and other shared setup through a render wrapper.
- Understand rerender, unmount, cleanup, and renderHook, and when each is useful.
- Explain how React act relates to rendering and updates in tests.

## 1. What a component test renders

A component test renders a React element into a DOM supplied by a test environment. It examines the DOM the component produced, rather than a React component instance. This keeps the test focused on what a browser user or assistive technology can observe.

A typical component test has four parts:

1. Prepare a component and any required provider or props.
2. Render it into the test DOM.
3. Locate relevant output through accessible roles, names, labels, or visible text.
4. Assert that the output matches the component's contract.

For example, render a greeting card with a name and assert that a heading named “Hello, Ada” is present. Do not inspect the component's private state or React element tree to prove the greeting appeared.

React Testing Library's render helper creates a container and appends it to the document body. Its screen object contains queries bound to the document body, which is usually the clearest way to query the rendered page. Prefer screen queries over searching the render container directly.

## 2. Select the right query for the timing

Testing Library offers query families that express different expectations:

| Query family | What it means | Use it when |
| --- | --- | --- |
| getBy | The matching element should exist now | Checking initial content or a result after a completed synchronous action |
| queryBy | The matching element may be absent now | Confirming that something is not rendered |
| findBy | The matching element should appear asynchronously | Waiting for content that appears after a request or delayed update |

A getBy query throws if nothing matches, which gives a useful failure when content is expected. A queryBy query returns null if nothing matches, making it suitable for a not-present assertion. A findBy query returns a promise and waits for a matching element to appear before failing.

The query also expresses what the test cares about. Prefer a button role and accessible name for a button; prefer a label for a form field; use visible text for ordinary text content. A test identifier is a fallback for a target that has no useful semantic or user-facing query.

Do not use findBy as a general delay. If an element should already exist, getBy exposes a real failure immediately. If the behavior is genuinely asynchronous, findBy describes that expectation.

## 3. Render providers with a wrapper

Many components need context: a theme, router, localization setup, data client, or application store. The component should still be rendered normally, with the provider that production uses around it.

React Testing Library's render helper accepts a wrapper component. A small custom render function can provide common defaults and keep individual tests focused on their subject. Keep provider values easy to override when a test needs a specific case.

A custom render helper is application test infrastructure. It should not hide the behavior being tested or add every provider the application has. Include only providers that the component actually needs, and keep provider defaults explicit.

When a provider supplies a changing value, render the component with the relevant value. If you are verifying a full data-flow update from a parent, test that parent and child together instead of manually updating only the child props.

## 4. Understand the render result

The render helper returns useful controls for specific test cases:

- rerender renders a new React element into the existing test root. It can check how a component responds when its props change. Prefer testing the real parent when the parent's prop updates are part of the behavior.
- unmount removes the rendered React tree. It is useful when verifying cleanup for effects, subscriptions, or resources when a component leaves the page.
- container is the raw DOM container. It is available for special cases, but queries against it are usually less resilient than accessible screen queries.
- debug prints the DOM to help diagnose a failing query. Use it temporarily while investigating a failure; it is not an assertion.

Tests normally should not need to call rerender or unmount. Use them when the specific update or teardown is the subject of the test, not as routine setup for every case.

## 5. Clean up between renders

Each test should start with a clean DOM. React Testing Library can automatically call cleanup after a test when the test runner exposes a global afterEach hook. If the runner does not expose that hook globally, configure cleanup explicitly in a setup file.

Cleanup unmounts rendered React trees and removes their containers. This prevents content, effects, and DOM nodes from one test leaking into the next test. It also gives effects an opportunity to run their cleanup functions.

Automatic cleanup does not reset every kind of shared state. Reset mocks, timers, storage, module state, or other shared resources separately when a test uses them. Prefer creating fresh values within each test over relying on a particular test order.

## 6. React act and wrapped utilities

React may schedule state updates and effects after a render or user action. Before making an assertion, those updates need to be applied to the DOM.

React provides act for grouping UI work and flushing its related updates. React Testing Library wraps its normal rendering and event helpers so most component tests do not need to call act manually. Use the library helpers for ordinary renders and interactions.

When a test must trigger a React update outside the testing library's helpers, wrap that work in the async act function and wait for it to finish before asserting. For example, a test that directly resolves a deferred promise or advances a timer that updates a component may need explicit act around that operation.

For modern React, import act from react or use the wrapper exported by React Testing Library. Do not use act from react-dom/test-utils; that export is deprecated. Keep your React and test-library versions compatible with the application.

An act warning is a signal to examine the test's update path. Often the test has not waited for an async result, or it triggered a state change outside a supported helper. Do not hide the warning by globally suppressing console output.

## 7. Testing custom Hooks

Hooks must run inside React rendering, so they cannot be called as ordinary functions in a test. A renderHook utility provides a small test component and exposes the latest return value to a test.

Use renderHook selectively. It is useful for a reusable Hook library whose exported Hook is itself the product contract, or for a Hook behavior that is difficult to express through a consumer component. For ordinary application Hooks, prefer rendering a component that uses the Hook. The component test shows how the Hook affects the actual UI and avoids making the Hook's internal return shape the only contract.

If a Hook requires context, renderHook can receive a wrapper just like render. If its consumers rely on a visible effect, test that effect through a component as well.

## 8. Keep test utilities small

A useful utility removes repetitive setup without hiding the subject. Common examples include a custom render that supplies a router or theme, and a data factory that produces clear test inputs.

Avoid building a large testing framework around the test framework. If a helper makes the test harder to read, inline the setup. A test should make it clear which component was rendered, what context it received, and what result is expected.

Use test utilities as intended:

- Use render for components and use screen for page-level queries.
- Use a wrapper for required providers.
- Use getBy, queryBy, or findBy based on whether content should be present now, absent, or appear later.
- Use rerender only when prop replacement is the behavior under test.
- Use unmount when teardown is the behavior under test.
- Use cleanup to isolate tests when it is not already automatic.
- Use renderHook mainly for public reusable Hook contracts.
- Use act explicitly only for updates outside helpers that already handle React scheduling.

## Common mistakes

- Using raw container selectors for controls that have accessible roles or labels.
- Using getBy for content that is expected only after an asynchronous operation.
- Using findBy to add a delay when the content should be present immediately.
- Updating a child with rerender when the real contract belongs to a parent data flow.
- Forgetting explicit cleanup when the test runner does not provide a global afterEach.
- Assuming DOM cleanup also resets mocks, timers, or storage.
- Using renderHook for every application Hook instead of testing its effect in a component.
- Importing act from the deprecated react-dom/test-utils entry point.
- Wrapping ordinary Testing Library render and user-event calls in unnecessary act blocks.
- Suppressing act warnings without fixing the update or waiting behavior.
- Turning temporary debug output into an assertion or permanent workaround.

## Summary

Component tests render React into a DOM and query the result through the same semantic surface users rely on. Choose getBy, queryBy, and findBy according to whether content should exist now, be absent, or appear later. Use wrappers for needed providers, keep cleanup explicit when automatic cleanup is unavailable, and reach for rerender, unmount, renderHook, and act only when their specific behavior matters. Testing Library's wrapped helpers handle the ordinary React update path; clear test setup and scoped utilities make failures easier to understand.

## Practice checklist

1. Render one component with the providers it needs.
2. Find its heading, button, and field using semantic queries.
3. Use queryBy to check an intentionally absent message.
4. Use findBy only for content that appears asynchronously.
5. Write a small custom render helper if provider setup repeats.
6. Verify cleanup configuration for your chosen test runner.
7. Use rerender or unmount only to test prop changes or teardown.
8. Decide whether a Hook should be exercised through its consumer component.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="100-testing-strategy-and-component-design.md">100 - Testing Strategy and Testable Component Design</a></td><td><a href="../qnas/101-component-tests-and-test-utilities.md">Q&amp;A</a></td><td><a href="../examples/101-component-tests-and-test-utilities.md">Example</a></td><td><a href="102-testing-user-interactions.md">102 - Testing User Interactions and Complete Flows</a></td></tr>
  </tbody>
</table>
