# Testing User Interactions and Complete Flows: Questions and Answers

## Modeling user input

### 1. What is user-event?

It is a Testing Library companion that simulates common user interactions through a DOM. It can model a fuller interaction than dispatching a single low-level DOM event.

### 2. Why use user-event instead of fireEvent for a click?

A real click involves more than a click event. user-event performs the surrounding pointer behavior and checks whether a target can be interacted with. fireEvent dispatches a specific DOM event and is useful when the event itself is the low-level contract or user-event does not support that interaction.

### 3. Is fireEvent wrong?

No. It is a lower-level tool for dispatching a chosen event. It should not be the default replacement for a complete user interaction when user-event already supports that action.

### 4. Where should userEvent.setup() be called?

Inside the test, before rendering the component. Each test should get its own user session so keyboard and pointer state do not leak between tests.

### 5. Why should I await user-event methods?

The methods return promises. Awaiting them allows the simulated interaction and related updates to complete before the next action or assertion.

### 6. Can I create one user session in a global beforeEach hook?

It is better to create it in the test or a local setup helper used by that test. A session carries input device state, so sharing one can make tests depend on previous interactions.

### 7. How do I test typing into a field?

Find the field by its label or another suitable accessible query, then use the session's type method and await it. This exercises focus, keyboard input, and the resulting value more realistically than assigning the value directly.

### 8. When should I use keyboard() instead of type()?

Use type() to enter text into an input or textarea. Use keyboard() when the keys themselves are the behavior, such as pressing Enter, Escape, an arrow key, or a modifier combination.

### 9. How do I test tab navigation?

Use the session's tab method, then assert which element is focused. Use the shift option to move backward. Check that the focus order matches the expected user path.

### 10. How do I test a checkbox or radio interaction?

Find it by role and accessible name, perform a user click or another supported input action, and assert the checked state and any related visible result. Avoid assigning the checked property directly when the test is meant to verify interaction.

### 11. How do I test a select control?

Find the labeled select and use the appropriate user-event selection helper. Assert the selected value or the visible result that depends on the choice.

### 12. How do I test file selection?

Create a File value in the test, locate the labeled file input, and use user-event's upload method. Then assert the selected file or the user-visible result. A simulated upload does not test a real file chooser dialog.

### 13. Can user-event test hover or pointer behavior?

It provides hover and pointer APIs for many DOM interaction checks. In a jsdom environment there is no real page geometry, so these checks do not prove that a coordinate lands on the expected visual target in an actual browser.

## Structuring a complete flow

### 14. What does a complete interaction flow test verify?

It follows a meaningful user goal through the controls and visible states involved. For example: enter a task name, add it, mark it complete, and confirm the completed state is shown.

### 15. Should one test cover the entire application workflow?

No. Keep a flow focused on one coherent goal. A test spanning many unrelated screens can be slow and hard to diagnose. Use a browser-level test for a critical end-to-end journey and focused component tests for smaller contracts.

### 16. How many assertions should a flow test contain?

Enough to confirm meaningful checkpoints without asserting every implementation detail. After a create action, confirm the item appears; after a completion action, confirm its state changes. These checkpoints make a failing step easier to locate.

### 17. Should I assert that an event handler was called?

Only when the callback or invocation is part of a documented contract. Otherwise, verify the resulting visible behavior. A handler call alone does not prove that the interface updated correctly.

### 18. Should I inspect component state after each action?

Usually no. State is an implementation detail. Assert on the DOM, focus, form value, checked state, or a documented callback that a user or parent can observe.

### 19. How do I avoid brittle flow tests?

Use roles, accessible names, labels, and visible text. Assert on behavior rather than class names, internal markup order, or exact event sequences unless event order is itself specified behavior.

### 20. Should every possible interaction be in one test?

No. Separate alternate paths such as keyboard activation, validation failure, or removal into tests that express those behaviors clearly. One test should not become a hard-to-read inventory of every control.

## Keyboard and focus

### 21. Is a click test enough to prove keyboard support?

No. A click test proves the pointer path it exercised. If keyboard use matters, add a test that tabs to the control, activates it with the appropriate key, and checks focus and the resulting UI.

### 22. Should I test native button keyboard behavior manually in every component?

Not necessarily. Native buttons already support standard keyboard activation. Focus your component tests on application-specific outcomes and custom keyboard behavior, and manually check important workflows across target browsers and assistive technologies.

### 23. How should a dialog test check focus?

Assert that opening the dialog places focus at the intended control or dialog boundary, then exercise the relevant keyboard path. On close, assert focus returns to the appropriate trigger when that is the defined behavior.

### 24. Can jsdom prove that focus behavior works in all browsers?

No. It is a useful DOM simulation, not every browser's focus and rendering implementation. Test critical keyboard and dialog behavior in target browsers and assistive technology combinations.

## Async and retry helpers

### 25. How should a test wait for a result after an interaction?

Await a findBy query when a specific element should appear. Use waitFor when the condition is broader, such as a callback expectation. Do not add a fixed sleep when the test can wait for the condition itself.

### 26. What is waitFor for?

It retries an assertion callback until the callback stops throwing or the timeout is reached. It is useful for asynchronous conditions that do not fit one findBy query.

### 27. Should I put user.click inside waitFor?

No. waitFor may call its callback repeatedly. If a click is inside that callback, the test may perform it more than once. Perform the interaction once, then wait for the resulting condition.

### 28. Why does waitFor not retry when my callback returns false?

The callback must throw an error for waitFor to retry. An assertion that fails by throwing is the common pattern. Returning a false value alone does not signal a failed assertion to retry.

### 29. Should I use findBy or waitFor for an appearing heading?

Prefer findByRole or another findBy query because the expectation is that a particular element appears. Use waitFor for conditions such as a callback being invoked or several related assertions that need to become true.

### 30. Can a user-event call be made inside an async utility?

It is safer to keep the interaction outside the retry callback. Await the action once, then await the query or condition that represents its result.

## Browser boundaries and reliability

### 31. What can jsdom not prove about pointer interactions?

It does not perform real layout or coordinate hit-testing. It cannot prove that an element is visible at a location, that another element does not cover it, or that a real drag gesture works in the rendered layout.

### 32. When should I add a real-browser test?

When behavior depends on real layout, coordinate targeting, drag and drop, browser-native rendering, actual navigation, or a full production build. Keep fast component tests for semantics and state behavior, and use browser tests for the parts that require a browser.

### 33. Should a component interaction test call a live server?

Usually not. Control network results at a service boundary so the flow is repeatable. A dedicated end-to-end test may connect to a test service if verifying that connection is part of its purpose.

### 34. Why can tests be flaky even when the UI is correct?

They may use fixed sleeps, fail to await user-event, share input state, depend on test order, or assert before asynchronous output appears. Start from isolated state and wait for the visible condition.

### 35. What is the main rule for a user-flow test?

Start from a realistic rendered state, use the controls as a person would, await interactions, and assert on meaningful accessible output and focus. Use a browser or manual check when the behavior exceeds what a DOM simulator can represent.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="101-component-tests-and-test-utilities.md">101 - Component Tests and React Test Utilities: Questions and Answers</a></td><td><a href="../lessons/102-testing-user-interactions.md">Lesson</a></td><td><a href="../examples/102-testing-user-interactions.md">Example</a></td><td><a href="103-testing-forms-and-async-ui.md">103 - Testing Forms, Asynchronous UI, and API Boundaries: Questions and Answers</a></td></tr>
  </tbody>
</table>
