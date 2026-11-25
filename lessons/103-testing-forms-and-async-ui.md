# Testing Forms, Asynchronous UI, and API Boundaries

## Learning goals

After this lesson, you should be able to:

- Test form submission through its visible controls and submit event.
- Assert loading, success, validation, and service-failure states.
- Control an API boundary without making a live network request.
- Hold a request open to verify pending UI before resolving it.
- Wait for asynchronous results without arbitrary sleeps or repeated actions.
- Distinguish portable onSubmit handling from React 19 form Actions.

## 1. Treat a form as a user-facing contract

A form combines labels, controls, browser validation, a submit path, and feedback. Test it through the visible field and submit button, not by calling the form handler directly. This exercises how the user supplies data and how the application responds.

For each important form, identify the expected behavior for:

- A valid submission.
- Invalid or incomplete values.
- A request that is still pending.
- A successful response.
- A known business rejection, such as a name already being taken.
- An unexpected service failure.

Not every form needs all these states. Choose cases that match its contract and risk. Make errors and next steps visible, and expose dynamic status in an accessible way.

## 2. Use the real submit path

Create a user-event session, fill a labeled field, and activate the submit button. When the user presses Enter in a form field, test that path if keyboard submission is part of the requirement.

This lets the browser-like test environment dispatch the submit event and lets React run the component's actual handling path. A direct call to a submit function would skip labels, field values, submit-button wiring, and the form event.

For an onSubmit form, the handler commonly prevents a page reload and reads the submitted values from FormData. Inputs need a name attribute to be included in FormData. The test should enter data through the labeled control and assert that the API boundary receives the expected normalized value.

Native HTML validation can block a submit event before React receives it. A DOM emulator does not perfectly reproduce every browser validation detail. For important constraint-validation behavior, add a browser check as well as component-level tests.

## 3. Control the API boundary

A component test should not depend on a live service. Give the form a service function or adapter boundary, then pass a controlled implementation from the test. That lets the test decide whether the request succeeds, rejects for a business reason, or throws unexpectedly.

A controlled service can record the input and return a promise. In a pending-state test, leave that promise unresolved until the test has asserted the loading state. Then resolve or reject it and assert the resulting message.

This approach keeps the real form, React state, submit event, and rendering logic intact. It replaces only the part outside the component's responsibility: the service response.

Do not make the mock resolve immediately in every test if pending UI matters. A promise resolved at once may never give the test a chance to observe a useful pending state. Use a deferred promise to control the exact moment the response completes.

## 4. Test loading state before the request finishes

After submitting, assert the pending indicator or status and any temporary disabled state while the service promise is still waiting. Then complete the promise and assert that the pending state ends and the final result appears.

A test should verify the actual behavior the user sees: the button can say “Checking…”, controls may be disabled to prevent duplicate submissions, and the page can expose a concise status message. Do not assert only that an internal pending boolean became true.

If repeated submissions are allowed, define their behavior and test it. The interface might disable the submit button, allow the new request to replace the old one, or queue requests. Avoid assuming one policy without a product requirement.

## 5. Wait for the result, not a duration

Use an awaited findBy query when a particular message or result should appear after a request. Use waitFor for a broader assertion, such as verifying that a service mock was called or that several related conditions have settled.

Wait helpers return promises, so await them. A waitFor callback is retried when its assertion throws. Keep the user action outside that callback: if a click or API invocation is inside a callback that retries, it may happen more than once.

Do not use a fixed sleep to guess when the request has completed. A duration can be too long on a fast machine and too short on a slow one. Wait for the visible state the user cares about.

For disappearing loading UI, use a removal helper when the element should leave the DOM, or assert that its accessible state changes when it remains mounted. Choose the query to match the implementation's user-visible contract.

## 6. Distinguish response types

A returned validation result is an expected outcome, not necessarily an exceptional failure. For example, “That username is already taken” is a normal service response that the form should display as a useful correction path.

A rejected promise or thrown error may represent an unavailable network or an unexpected failure. Catch it at the boundary where the interface can show a recovery message, or allow it to reach the error boundary when that is the chosen architecture. The test should match the intended error-handling contract.

Keep these cases distinct in tests:

- Success: the expected result appears and pending UI ends.
- Business rejection: the field or form explains what the user can change.
- Service failure: the form explains that the operation failed and what to try next.

Avoid asserting on raw exception strings from a network library. Users need a helpful message, not an implementation-specific error object.

## 7. React form Actions in React 19

React 19 supports passing a function to a form's action prop. React calls it with FormData in a Transition. The action pattern can expose pending state with useFormStatus, and useActionState can hold a returned result and pending flag. A child component must call useFormStatus from inside the form it tracks.

When testing an Action form, test the form submission through its submit button and assert the pending UI and final state. The action can call the same injected service adapter used in a portable form test. For known validation outcomes, assert the returned validation state. If an unexpected error is intended to reach an Error Boundary, test that boundary's visible fallback.

A successful function action resets uncontrolled fields by default. If keeping or restoring submitted values is part of the design, assert that behavior. Do not accidentally assume that a React 19 Action form and a manual onSubmit form reset fields in the same way.

The onSubmit event-handler pattern works across React versions. A function passed to action is the preferred form submission API starting in React 19. Pick the pattern supported by the application and test its actual behavior. If both action and onSubmit are present, understand that both can run unless onSubmit prevents the default submission.

## 8. Keep async tests isolated

Each test should start with fresh form state and its own controlled service response. Resolve or reject all deferred promises before the test ends. Reset mock call history between tests. Do not let a previous test's pending request update the next test's DOM.

If a form has an effect or request that can outlive its component, include cleanup or cancellation behavior in the implementation. A test can unmount the form while a request is pending and verify that cleanup rules are respected when this is part of the component's contract.

## Common mistakes

- Calling the submit handler directly instead of submitting through the rendered form.
- Forgetting a name attribute when the handler reads a field from FormData.
- Depending on a live API in a component test.
- Resolving every promise immediately and never checking the pending state.
- Adding a timeout or sleep instead of waiting for the result.
- Placing a click or request inside a retrying waitFor callback.
- Treating a business rejection and a service outage as the same outcome.
- Asserting on raw service exceptions instead of user-facing recovery text.
- Forgetting that React 19 function actions reset uncontrolled form fields after success.
- Calling useFormStatus in the component that renders the tracked form instead of a child inside it.
- Leaving a deferred promise unresolved after the test completes.
- Assuming a DOM emulator fully implements native browser validation.

## Summary

Test forms through their labeled fields and real submit path. Control only the service boundary, hold promises open when checking pending UI, and assert the success, validation, or failure state users see. Await a result query or assertion rather than sleeping, and keep interactions outside retry callbacks. For React 19, account for function actions, useFormStatus, useActionState, Error Boundaries, and automatic uncontrolled-field reset; use the form pattern the application actually ships.

## Practice checklist

1. Give every form control a useful label and name.
2. Submit using the real button or keyboard path.
3. Inject a controlled service instead of calling a live API.
4. Assert the submitted normalized values at the service boundary.
5. Hold a promise open and check the visible pending state.
6. Resolve success and assert the final result and re-enabled controls.
7. Cover a business rejection and a service failure separately.
8. Await the visible condition and avoid arbitrary sleeps.
9. Verify form reset behavior for the chosen React submission pattern.
10. Use a browser-level check for important native validation behavior.
