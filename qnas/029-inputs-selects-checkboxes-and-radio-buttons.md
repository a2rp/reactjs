# 029. Text inputs, selects, checkboxes, and radio buttons - Q&A

## Text inputs

### 1. How should I choose an input `type`?

Choose the type that describes the information: for example, `email` for an email address, `password` for a password, `search` for a search field, and `number` for a numeric quantity. The type can affect browser behavior, validation, and the keyboard offered on mobile devices.

### 2. What value type should I keep for a controlled text input?

Keep it as a string and pass it through `value`. In the change handler, store `event.currentTarget.value` so React can render exactly what the user is editing.

### 3. Is the value of a controlled number input a JavaScript number?

No. An input of type `number` still exposes its editable value as a string. Keep the string in state, including the empty string, and convert it when a calculation needs a number.

### 4. Why should I check for an empty string before calling `Number(value)`?

Because `Number('')` is `0`. If an empty field means “not entered,” check for `value === ''` before converting so it is not mistaken for zero.

### 5. Should I use `type="number"` for a telephone number?

Usually no. Telephone numbers may contain a plus sign, spaces, or leading zeroes and are identifiers rather than quantities. Use a text-like telephone field such as `type="tel"` when that better represents the data.

### 6. Can a placeholder replace the field's label?

No. Placeholder text disappears when the user types and is not a reliable replacement for a visible label. Associate a visible `<label>` with the control.

### 7. What do `name` and `autoComplete` do?

`name` identifies the field in form data. `autoComplete` gives the browser a hint about the kind of saved information that can fill the field. Neither one replaces a label or controls the value.

## Single and multiple selects

### 8. What state shape does a single select use?

Usually one string containing the selected option's value. Pass that string as the parent select's `value` and update it from `event.currentTarget.value`.

### 9. Where should I put the controlled selection value?

On the parent `<select>`. Give its `<option>` elements values that match the value stored in state.

### 10. Can I control the selected option by adding `selected` to an `<option>`?

No. React does not support `selected` on an option. Use `value` on a controlled parent select or `defaultValue` for an uncontrolled select.

### 11. How do I make a required select prompt?

Add a prompt option with `value=""`, initialize the controlled state to an empty string, and mark the select `required`. The browser can then treat the empty choice as missing when the form is validated.

### 12. What state shape does a multiple select use?

An array of strings, one for each selected option. It remains an array when no options are selected.

### 13. How do I read every selected option from a multiple select?

Use `Array.from(event.currentTarget.selectedOptions, option => option.value)`. `event.currentTarget.value` alone does not represent the entire selection.

### 14. How can I retrieve all same-name values from form data?

Use `FormData.getAll(name)`. A multiple select and a group of checked checkboxes can produce several form-data entries with the same name. Converting entries with `Object.fromEntries` keeps only one value for a repeated key.

## Checkboxes

### 15. How do I control one checkbox?

Use a boolean `checked` prop, and update it from `event.currentTarget.checked` in `onChange`.

### 16. What is the difference between a checkbox's `checked` and `value`?

`checked` determines whether it is selected. `value` is the string submitted for the checkbox when it is checked. Changing the value does not toggle the checkbox.

### 17. What happens to an unchecked checkbox in form data?

It is omitted from the form data. It does not submit an empty string or a false value automatically.

### 18. How should I represent a group of independent checkboxes in state?

An array of selected option values is a useful choice. For each checkbox, derive `checked` by asking whether its value is in the array.

### 19. How do I add or remove an item from the checkbox array?

Read `checked` and `value` from the changed checkbox. Return a new array: add the value when checked and filter it out when unchecked. Do not mutate the existing array with `push` or `splice`.

### 20. Can checkboxes in one group share a `name`?

Yes. Give the choices the same meaningful name and distinct values. To get all checked values with that name from `FormData`, call `getAll`.

## Radio buttons

### 21. When should I use radio buttons instead of checkboxes?

Use radio buttons when the user should choose one option from a mutually exclusive group. Use checkboxes when each option is independent and several can be selected.

### 22. What must radio buttons in one group have in common?

They should share the same `name` and have distinct `value` strings. The shared name groups them; the distinct values identify the selected choice.

### 23. How do I control a radio group with React state?

Store one selected value as a string. Each radio receives `checked={selectedValue === optionValue}`, and its change handler stores `event.currentTarget.value`.

### 24. Does a radio's `value` tell me whether that radio is selected?

No. The `value` identifies the option. The boolean `checked` prop represents whether that particular radio is selected.

### 25. Why should I use `fieldset` and `legend` around radio or checkbox groups?

They give related controls a semantic group and a shared accessible name. Also provide a label for each individual option so users can identify its meaning.

## Attributes and behavior

### 26. What do `min`, `max`, and `step` do on a number input?

They describe the allowed numeric range and increment to the browser. They do not change the fact that a controlled number field's editable value is a string.

### 27. What do `minLength`, `maxLength`, and `required` do?

They provide constraints that the browser can apply during form validation. They do not replace clear instructions or validation by the server. Chapter 030 covers how submission and browser validation work together.

### 28. What is the difference between `disabled` and `readOnly`?

A disabled control is unavailable for interaction and is omitted from form data. Read-only applies to supported text controls: the value cannot be edited, but the control can remain focusable and included in form data. Checkboxes and selects do not use `readOnly` as an equivalent of disabled.

### 29. Does every input need to be controlled?

No. Control a value when React needs to use or change it as the user edits. An uncontrolled field can be suitable when the browser should own its value until a later read or submission.

### 30. What is a useful checklist for choosing a control?

Ask what the user is choosing and how many choices are allowed:

- Free-form text or a numeric quantity: use the appropriate input type.
- One choice from a long list: use a single select.
- Several choices from a list: use a multiple select.
- One independent on/off choice: use a checkbox.
- Several independent choices: use a checkbox group.
- Exactly one choice from a small visible set: use a radio group.

Then give each control a label and name, and choose state that matches the control's value shape.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="028-controlled-form-elements.md">028 - Controlled form elements - Q&amp;A</a></td><td><a href="../lessons/029-inputs-selects-checkboxes-and-radio-buttons.md">Lesson</a></td><td><a href="../examples/029-inputs-selects-checkboxes-and-radio-buttons.md">Example</a></td><td><a href="030-form-submission-and-browser-validation.md">030 - Form submission and browser validation - Q&amp;A</a></td></tr>
  </tbody>
</table>
