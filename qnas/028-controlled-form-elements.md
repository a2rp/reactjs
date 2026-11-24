# 028. Controlled form elements - Q&A

## Core ideas

### 1. What is a controlled form element?

It is a form element whose current value is supplied by React through a prop such as `value` or `checked`. React state holds that value, and an event handler updates the state when the user edits the element.

### 2. What is the controlled data flow?

React renders from state, the user changes the control, the `onChange` handler reads that edit and updates state, and React renders with the new state. The value flows back to the control on the next render.

### 3. Why is React state called the source of truth?

Because the value in state determines what the control displays. If the state changes for another reason, React can update the control to match it.

### 4. Does every form element need to be controlled?

No. Use a controlled element when the current value needs to drive React behavior, such as a live preview or conditional UI. An uncontrolled element can be simpler when the browser can hold the value until it is read or submitted.

### 5. What is the difference between controlled and uncontrolled?

A controlled element receives its current value from `value` or `checked` and updates through React state. An uncontrolled element keeps its current value in the DOM; React may provide an initial value through `defaultValue` or `defaultChecked`.

## Props and event values

### 6. Which prop controls a text input?

Use `value` with a string and an `onChange` handler that updates the state supplying that string.

### 7. Which prop controls a textarea?

Use `value` with a string and an `onChange` handler. In React, provide the text with the `value` or `defaultValue` prop, not as textarea children.

### 8. Which prop controls a select?

Use `value` on the parent `<select>` and update the state in `onChange`. The value must match an option's value.

### 9. How do you control a checkbox or radio button?

Use the boolean `checked` prop and update the corresponding state from `event.currentTarget.checked`.

### 10. Does a checkbox's `value` control whether it is checked?

No. `value` is the string submitted for that checkbox when selected. `checked` controls whether it is currently selected.

### 11. What should a change handler read from a text input?

Usually read `event.currentTarget.value`. React's `onChange` for a text input fires when its value changes, including while the user types.

### 12. What should a change handler read from a checkbox?

Read `event.currentTarget.checked`, which is a boolean. `event.currentTarget.value` is not the checkbox's selection state.

### 13. What value does a single select's change handler read?

For a single select, read `event.currentTarget.value`. It is the value of the selected option.

### 14. How do you get every selected value from a multiple select?

The controlled `value` should be an array of strings. Read the selected options with `Array.from(event.currentTarget.selectedOptions, option => option.value)` and store that array in state.

### 15. How are radio buttons commonly represented in state?

A group commonly stores the selected option's string value. Each radio's `checked` prop compares that state with its own `value`. A handler can read the selected radio's `event.currentTarget.value`.

## Updating state correctly

### 16. Why must `onChange` update the backing state?

React renders a controlled element from its current prop. If the handler does not update the state, React renders the old value again and the user's edit will not remain visible.

### 17. What does it mean to update the value synchronously?

Update the state in the change handler using the value from that event. Do not wait for an unrelated later action to copy the edit into state if the control is controlled.

### 18. What happens if I pass `value` but omit `onChange`?

The control is effectively read-only from the user's perspective because React keeps supplying the same value. If the field is intentionally not editable, use the appropriate read-only or disabled behavior; otherwise add a handler that updates its value.

### 19. Why should a controlled text value start as an empty string instead of `undefined`?

A text field with a string `value` is controlled. Starting with `undefined` and later supplying a string makes it switch from uncontrolled to controlled. Initialize it with `''` or use a string fallback such as `value={name ?? ''}`.

### 20. Can I change an element from controlled to uncontrolled later?

No. Keep it controlled or uncontrolled for its whole mounted lifetime. Switching ownership creates inconsistent behavior and React warns about it.

### 21. Can I pass both `value` and `defaultValue`?

Do not use both for the same control. `value` supplies the current controlled value; `defaultValue` supplies the initial value for an uncontrolled control. Choose one model.

### 22. What do `defaultValue` and `defaultChecked` do?

They set an uncontrolled input's initial value or an uncontrolled checkbox/radio's initial selection. They do not make React control the value after the user edits it.

### 23. If I change `defaultValue` after the control mounts, will it update the user's current edit?

No. It is an initial value. Use controlled state and `value` if later React updates must determine the current displayed value.

### 24. Why can transforming text in every `onChange` make the caret jump?

If the state value differs from the exact text the browser just produced, React may re-render a changed string and the selection position can move. Keep the controlled value in sync with the user's edit, and normalize at a deliberate point such as blur or submit when appropriate.

## Forms and practical decisions

### 25. Can controlled fields still be read with `FormData`?

Yes. A control with a `name` participates in form data when it is successful for that form. Controlled state determines its current displayed value; the `name` identifies it in the form data.

### 26. Why should a controlled field still have a `name`?

The `name` is the key used when the browser builds form data. It also makes the field's purpose clear in form markup. Controlling a value with React does not automatically provide a useful form-data key.

### 27. Should each form field have a label even when its value is controlled?

Yes. State management does not replace accessible form labeling. Associate a visible `<label>` with each field by nesting it or by matching `htmlFor` and `id`.

### 28. Can I store several controlled fields in one object?

Yes. Store related values in an object and update it immutably, often with a functional state update that spreads the previous object. Keep each field's controlled prop appropriate to its type, especially `checked` for checkboxes.

### 29. Does controlling a field mean I need to store every form value in state?

No. Control only values that React needs to use or update while the user edits. For a form that only needs its values at submission, an uncontrolled approach may be simpler.

### 30. Where should controlled form state live?

Keep it in the nearest component that needs to read or change it. If a large unrelated part of the page re-renders on every keystroke, consider moving the field state closer to the form.

### 31. Is a file input controlled with `value` like a text field?

No. Browsers restrict setting a file input's selected file through script, so file inputs are handled as uncontrolled elements. Read selected files from the input when needed.

### 32. What is the main rule for controlled form elements?

Choose a clear owner for the current value. If React owns it, pass a stable `value` or `checked` and synchronously update the backing state in `onChange`. If the browser owns it, use an initial-value prop when needed and do not treat that prop as ongoing control.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [027 - Event propagation and default actions](../lessons/027-event-propagation-and-default-actions.md) | [028 - Controlled form elements](../lessons/028-controlled-form-elements.md) | [Open Example](../examples/028-controlled-form-elements.md) | [029 - Text inputs, selects, checkboxes, and radio buttons](../lessons/029-inputs-selects-checkboxes-and-radio-buttons.md) |
