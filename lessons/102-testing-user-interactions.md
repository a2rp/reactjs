# Testing User Interactions and Complete Flows

## Learning goals

After this lesson, you should be able to:

- Simulate clicks, typing, keyboard navigation, and form actions in a rendered React component.
- Build tests around a coherent user goal instead of internal event handlers.
- Use one user-event session per test and await its interactions.
- Verify the visible outcome and focus behavior along a flow.
- Choose between user-event and lower-level event dispatch.
- Recognize when a DOM-based interaction test needs a real-browser companion.

## 1. Test a goal, not a list of handlers

A person experiences an interface as a sequence: find a control, enter information, activate an action, and observe a result. A useful interaction test follows a meaningful slice of that sequence.

For example, a task-list test can add a task, mark it complete, and remove it. The test verifies that the visible list responds to the actions. It does not need to call the component's submit, toggle, or remove functions directly.

Keep the flow focused. Test the normal path and the important boundary or alternate path. A test that tries to cover every unrelated screen in one scenario becomes difficult to diagnose. A collection of tests that each asserts one private implementation detail also misses the experience as a whole.

## 2. Start a user session for each test

Create a user-event instance inside the test, before rendering the component:

    const user = userEvent.setup();
    render(<TaskBoard />);

The instance represents a user's input device for that test. Use its methods to interact with the rendered controls, and await each method:

    await user.click(addButton);
    await user.type(taskField, 'Review the lesson');
    await user.keyboard('{Enter}');
    await user.tab();

A user-event operation is asynchronous. Awaiting it lets the simulated interaction and related React updates finish before the next assertion or action. Do not create a shared user instance in a global before hook; a fresh test session keeps keyboard and pointer state isolated.

## 3. Prefer user-event for ordinary interactions

A click is more than one low-level event. Typing involves focus, key input, text insertion, selection, and input events. user-event simulates a fuller interaction and checks whether the target can be interacted with, such as rejecting a click on a disabled control.

Use user-event for common actions such as:

- Click a button, link, checkbox, radio, or select option.
- Type or clear text in an input or textarea.
- Press a key or move focus with Tab and Shift+Tab.
- Hover or unhover a target when hover behavior is part of the UI contract.
- Upload a file into a file input when the upload interaction itself is under test.

Use the lowest-level event helper only when the specific event you need is not modeled by user-event or when the test deliberately checks a low-level event contract. A direct dispatch is not a substitute for a realistic user path.

## 4. Assert the result people can observe

After the action, check the page as it now appears. Depending on the interaction, useful assertions may check:

- New or updated visible text.
- A checkbox's checked state or a button's disabled state.
- A result, confirmation, or validation message.
- Focus staying on a useful control or moving to the expected next control.
- An item appearing, changing, or disappearing from a named list.
- A documented callback receiving meaningful values.

Use semantic queries to find controls. When a screen has repeated content, scope a query to the relevant named region with a helper such as within. This reduces ambiguity without depending on a CSS selector.

Do not assert that a handler ran a certain number of times unless that call count is itself part of the public contract. A user normally cares that the desired state appeared, not which internal function ran.

## 5. Include keyboard paths

A control should work for people who use a keyboard. Test a keyboard path explicitly when keyboard access is important to the flow:

1. Render the interface.
2. Move focus with Tab or Shift+Tab.
3. Assert which control is focused.
4. Activate the control with the appropriate key.
5. Assert the visible result and the expected focus location.

Native buttons and form controls already support standard keyboard behavior. Prefer those elements when they fit. If an application defines a custom keyboard interaction, test the documented key behavior and the focus changes it requires.

One click test does not prove that a complete workflow is keyboard accessible. Conversely, testing every browser keyboard detail in a DOM emulator does not replace manual keyboard checks or browser tests for behavior that depends on layout and native browser behavior.

## 6. Check focused checkpoints in a flow

A complete flow can contain several actions, but assertions should mark meaningful checkpoints. For example, after adding a task, assert that it appears in the task list. After marking it complete, assert the completion state. After removal, assert it is absent.

These checkpoints help locate failures. If a single final assertion fails after ten actions, it can be unclear which part broke. Do not assert every transient DOM detail; select checkpoints that represent meaningful changes in the user's goal.

When an action starts asynchronous work, wait for a user-visible condition. Use a findBy query for content expected to appear, or waitFor for an assertion that cannot be expressed as one query. Always await the asynchronous helper.

Keep actions outside a waitFor callback. waitFor may run its callback repeatedly until it stops throwing, so placing a click or typing action inside it can repeat the action and create unpredictable tests. Put the action first, then wait only for the expected result.

## 7. Handle focus as part of the interaction

Focus is visible to keyboard users and can determine whether a multi-step flow remains usable. When focus behavior matters, assert the active control after the interaction. user-event can move focus through normal tab order, and a document-level active element assertion can confirm the focused node.

A dialog or menu may intentionally move focus when it opens and restore it when it closes. Those requirements should be tested at the user-facing boundary. Do not assert an internal focus-management function call.

Some DOM emulators cannot reproduce focus and layout exactly as every browser does. For important dialogs or browser-specific controls, combine automated tests with browser and manual checks.

## 8. Avoid overfitting the simulated browser

DOM-based tests are fast and useful, but they do not render a page with real geometry. They cannot reliably prove that a control is visible at a particular coordinate, that elements overlap correctly, or that a drag gesture works with the real browser layout.

Use a real-browser test when the behavior depends on layout, actual navigation, browser-native rendering, or a complete application build. Keep DOM interaction tests for clear component behavior and fast feedback. Use manual checks when assistive technology or device-specific behavior is important.

## 9. Keep interaction tests deterministic

Use known component inputs and controlled external responses. Do not depend on a live service, machine speed, or arbitrary sleep. Await each user-event operation. When the UI has an async response, wait for the response's visible state rather than waiting a fixed amount of time.

Reset state between tests and avoid relying on the order in which tests run. If fake timers are necessary, configure user-event with the test runner's timer advancement function; do not disable its typing delays by setting a null delay.

## Common mistakes

- Calling component event handlers instead of using the rendered controls.
- Using low-level fireEvent for every click and keystroke when user-event covers the interaction.
- Forgetting to await user-event methods.
- Sharing one user session across tests.
- Placing clicks or typing inside a waitFor callback that can run repeatedly.
- Checking only that a handler ran instead of checking the user-visible outcome.
- Testing a long workflow with no intermediate checkpoints.
- Testing only mouse clicks when keyboard use is part of the requirement.
- Assuming jsdom proves layout, coordinate hit-testing, drag behavior, or browser-specific behavior.
- Adding arbitrary sleeps instead of waiting for a visible condition.

## Summary

Model interactions through the controls people use. Create a fresh user-event session inside each test, await its actions, and assert on meaningful visible checkpoints through semantic queries. Include keyboard paths and focus when the workflow requires them. Keep actions out of retrying waitFor callbacks, and use real-browser or manual checks when layout, browser behavior, or assistive technology is central to the requirement.

## Practice checklist

1. Choose one complete user goal and write its starting state and expected result.
2. Create a user-event session before rendering.
3. Find controls by role, accessible name, or label.
4. Perform each action through the rendered UI and await it.
5. Assert visible checkpoints as the flow progresses.
6. Add a keyboard path when keyboard use matters.
7. Keep side effects and clicks outside waitFor callbacks.
8. Decide whether layout or native browser behavior requires a browser-level check.
