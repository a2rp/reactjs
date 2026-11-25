# Testing Forms, Asynchronous UI, and API Boundaries: Questions and Answers

## Forms and submitted values

### 1. Why should a form test submit through the rendered form?

Submitting through a labeled field and submit control exercises the real form event path, field values, and button wiring. Calling a handler directly skips those parts of the user experience.

### 2. How do I submit a form in a component test?

Create a fresh user-event session, fill the labeled fields, and activate the submit button. If Enter submission is part of the requirement, test it from the relevant form field or submit control as a separate path.

### 3. Why does FormData not contain my input value?

A form control generally needs a name attribute to be included in FormData. Check that the component renders the correct name and that the test enters a value through the actual labeled control.

### 4. Should a test call preventDefault itself?

No. The component's submit behavior should handle the event. The test submits through the form and asserts the resulting UI and service call.

### 5. Do I need to test required fields?

Test required or validation behavior when it is part of the form contract. Native constraint validation may prevent a submit event before React receives it, and DOM emulators do not reproduce every browser detail. Add a real-browser check for important native validation behavior.

### 6. Should I test FormData values directly?

Test the values at the service boundary when the submitted data is important. Prefer a form test that enters realistic field values, submits the form, and asserts that the service receives the normalized payload.

## API boundaries and promises

### 7. Should a component test call the real API?

Usually not. Inject a service function or control it at a network boundary so the test is repeatable and does not need a live service. A separate integration or end-to-end test can verify the real connection when that is the goal.

### 8. What is a deferred promise?

It is a promise whose resolve or reject function the test controls. It lets the test submit the form, inspect the pending state while the request is still unresolved, and then decide when to complete the response.

### 9. Why use a deferred promise instead of Promise.resolve in every test?

An immediately resolved promise may finish before the test can reliably observe meaningful pending UI. A deferred promise makes the pending interval explicit and controllable.

### 10. What should I assert while the request is pending?

Assert on visible status text, a pending button label, disabled controls, or another part of the documented UI. Do not inspect a private pending boolean.

### 11. Should the submit button be disabled during a request?

That is a product decision. If the form is supposed to prevent duplicate submission, test the disabled state while the request is pending. If repeated submissions are supported, test the intended concurrency behavior instead.

### 12. How should I test the success response?

Resolve the controlled service with a successful result, then await the visible success message or result. Also verify that pending UI ends and that controls return to the expected state.

### 13. How should I test a normal business rejection?

Resolve the service with a valid response indicating rejection, such as a username already taken. Assert that the form displays a correction message and keeps or updates the values according to its contract.

### 14. How should I test a network or service failure?

Reject the controlled promise or make the service throw. Assert the recovery message users should see, such as an explanation that checking failed and a useful next step. Avoid asserting on a raw implementation-specific exception string.

### 15. Are business rejection and network failure the same case?

No. A business rejection is an expected result that often tells the user what to change. A network failure is an inability to complete the operation and usually needs a retry or recovery message. Test them separately when both are possible.

### 16. Should I check how many times the API mock was called?

Check the call count if duplicate submission behavior matters. Also assert the submitted values and user-visible result. A single mock call by itself does not prove the form worked.

### 17. How do I avoid a live network in a component test?

Pass a controlled service function into the component or replace the network boundary with a request mock. Keep the form component and its rendering logic real so the test still exercises the behavior it is meant to prove.

## Waiting for asynchronous UI

### 18. How do I wait for a success message to appear?

Use an awaited findBy query for that message or its accessible role. It retries until the matching element appears or the query times out.

### 19. When should I use waitFor?

Use it for an asynchronous expectation that cannot be expressed conveniently as one appearing or disappearing element, such as waiting for an API mock to be called with a particular payload. Its callback must throw for the retry to happen.

### 20. Why should the interaction stay outside waitFor?

waitFor can run its callback multiple times. A click or request inside it might happen more than once. Perform the user action once, then wait for the result.

### 21. Why are fixed sleeps a poor way to wait?

A fixed delay guesses when work will finish. It can waste time on a fast machine and fail on a slower one. Wait for the UI condition the user cares about instead.

### 22. How should I test that pending UI disappears?

If the pending element is removed from the DOM, use an element-removal helper or query for its absence after the response. If the element stays mounted with changed content, await the final visible state instead.

### 23. Do asynchronous Testing Library helpers need to be awaited?

Yes. findBy, waitFor, and removal helpers return promises. Awaiting them ensures the test does not finish before the expected result is observed.

### 24. What if my test ends with a pending request?

Resolve or reject every deferred promise before the test finishes. Otherwise the component may update after teardown or leak state into later work. If the test intentionally unmounts during a pending request, test the component's cleanup behavior explicitly.

## React form submission patterns

### 25. Does the traditional onSubmit pattern still work?

Yes. It works across React versions. The handler can prevent the browser's default navigation, read FormData, and invoke a service function.

### 26. What is different about a function passed to form action in React 19?

React calls the action with FormData and runs the submission in a Transition. Pending status and concurrency are integrated with React's form Action model. A successful function action also resets uncontrolled fields by default.

### 27. Which pattern should my tests use?

Test the pattern the application actually uses. Do not rewrite a form solely for the test. For React 19 function actions, submit the real form and assert pending and final UI. For an onSubmit form, exercise its submit event and assert its behavior.

### 28. What does useFormStatus return?

It provides status for the nearest parent form, including pending information and submitted data. It must be called from a component rendered inside that form.

### 29. Why is useFormStatus.pending always false?

The component using the Hook may not be inside the form, or it may be calling the Hook in the same component that renders the form. Put the Hook in a descendant component inside the form it needs to track.

### 30. What does useActionState add?

It stores the result of an Action along with a dispatcher and pending state. It can represent known validation results as returned state and gives the UI a way to display that result.

### 31. How should expected validation errors be represented in a React 19 Action?

Return a state value that describes the validation result and render a useful message. An expected condition such as “username is unavailable” does not need to be thrown as an unexpected exception.

### 32. What about unexpected exceptions from an Action?

If the design allows them to propagate, React reports them to the nearest Error Boundary. Test the visible fallback at the boundary that owns that behavior. If the form catches the failure and displays a recovery message itself, assert that message instead.

### 33. Do Action forms reset fields after failure?

A function passed to form action resets uncontrolled fields after it succeeds. For validation outcomes or failures, preserve values deliberately when needed and test the intended behavior. Do not assume every Action form clears or keeps fields in the same way.

### 34. Can action and onSubmit both run?

Yes, depending on how the form is wired. React documents that a function action can run after onSubmit unless onSubmit prevents the default submission. Avoid accidentally invoking two submission paths and test the one the application intends.

### 35. What is the key rule for testing forms and API boundaries?

Use the real form controls and submit path, keep the service boundary controlled, and assert pending, success, rejection, and failure outcomes that matter to users. Await the result instead of sleeping, and account for the exact React form pattern in the application.
