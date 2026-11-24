# 032. Managing state for multi-field forms

## Goal

A form with several inputs usually has related values that should update together. React can keep those values in one state object and render each control from that object. A shared change handler can update the field identified by the control's `name`.

This chapter focuses on structuring and updating form values. Accessible validation feedback is covered in Chapter 031. Uncontrolled form fields and refs are covered next in Chapter 033.

## Learning outcomes

After this chapter, you should be able to:

- Choose between separate state variables and one object for related form values.
- Keep controlled text inputs, selects, and textareas synchronized with React state.
- Update one property without discarding the other properties.
- Use a shared change handler with a control's `name` and `value`.
- Handle checkbox state through `checked`.
- Compute display values from current state instead of storing duplicate values.
- Explain why form values should be initialized with stable types.

## A form's values are related state

A single-field form can use one state variable. When a form has several related fields, storing those values together can make it easier to pass them to a preview or submit handler:

~~~jsx
import { useState } from 'react';

const initialForm = {
  fullName: '',
  email: '',
  topic: '',
  message: '',
};

function ContactForm() {
  const [form, setForm] = useState(initialForm);

  // Render the controls and submit handler here.
}
~~~

The object describes one form submission. Each property is a field value, and the initial values match the value types expected by their controls.

An object is not mandatory. Separate state variables can be clearer when fields have independent behavior or are used by different parts of a component. Group values when they belong to one cohesive form and are commonly read or updated together.

## Controlled controls

A controlled input gets its current value from React and reports edits through an event handler. The UI then renders from the updated state:

~~~jsx
function NameField({ value, onChange }) {
  return (
    <label>
      Full name
      <input
        name="fullName"
        value={value}
        onChange={onChange}
      />
    </label>
  );
}
~~~

In a complete form, a string field can read and update its value from the state object:

~~~jsx
const [form, setForm] = useState({
  fullName: '',
  email: '',
  topic: '',
  message: '',
});

function handleChange(event) {
  const { name, value } = event.currentTarget;

  setForm((currentForm) => ({
    ...currentForm,
    [name]: value,
  }));
}
~~~

The input's `name` selects the property to update. The spread copies the other existing fields into a new object; the computed property name then replaces just the edited value.

The functional updater receives the latest state. This is a reliable way to build a new object from the previous object, especially when several updates may be queued before React renders again.

## Do not mutate the state object

State objects should be treated as read-only. This changes the existing object and does not tell React to render a new state:

~~~jsx
function handleChange(event) {
  form[event.currentTarget.name] = event.currentTarget.value;
  setForm(form);
}
~~~

Instead, return a new object:

~~~jsx
function handleChange(event) {
  const { name, value } = event.currentTarget;

  setForm((currentForm) => ({
    ...currentForm,
    [name]: value,
  }));
}
~~~

The spread is shallow, which is enough when the form fields are top-level properties. If an object contains a nested object, copying the outer object alone does not copy or update the nested object immutably. Prefer a simple, flat form shape when it fits the data.

## Use the shared handler in a form

The same handler can serve text inputs, a select, and a textarea as long as each control has the matching `name` and the state object has that property:

~~~jsx
<form>
  <label htmlFor="fullName">Full name</label>
  <input
    id="fullName"
    name="fullName"
    value={form.fullName}
    onChange={handleChange}
  />

  <label htmlFor="email">Email address</label>
  <input
    id="email"
    name="email"
    type="email"
    value={form.email}
    onChange={handleChange}
  />

  <label htmlFor="topic">Topic</label>
  <select
    id="topic"
    name="topic"
    value={form.topic}
    onChange={handleChange}
  >
    <option value="">Choose a topic</option>
    <option value="account">Account</option>
    <option value="billing">Billing</option>
  </select>

  <label htmlFor="message">Message</label>
  <textarea
    id="message"
    name="message"
    value={form.message}
    onChange={handleChange}
    rows="4"
  />
</form>
~~~

The label connects to the control through `htmlFor` and `id`. The control's `name` is used by the shared handler, while its `value` comes from React state.

A controlled text input, select, or textarea should receive a string value throughout its lifetime. Initialize empty fields with `''` rather than `undefined` or `null`. Update the state synchronously from the control's current value in `onChange`. Otherwise the control can appear to reject typing or React can warn when a control switches between controlled and uncontrolled modes.

## Checkboxes use boolean state

A checkbox's selected state is represented by `checked`, not by the text input's `value`:

~~~jsx
const [form, setForm] = useState({
  fullName: '',
  email: '',
  receiveUpdates: false,
});

function handleChange(event) {
  const { name, type, value, checked } = event.currentTarget;
  const nextValue = type === 'checkbox' ? checked : value;

  setForm((currentForm) => ({
    ...currentForm,
    [name]: nextValue,
  }));
}
~~~

Render the checkbox with the boolean property:

~~~jsx
<label>
  <input
    type="checkbox"
    name="receiveUpdates"
    checked={form.receiveUpdates}
    onChange={handleChange}
  />
  Send me product updates
</label>
~~~

For a radio group, each radio has the same field name and a distinct value. In that case, a shared handler should store the selected radio's value. Checkboxes represent independent booleans unless the form intentionally models a group of selected values.

## Derive display values during render

Do not store a second state value when it can be calculated from the form values. For example, a preview can be derived directly:

~~~jsx
const summary = [
  form.fullName || 'Your name',
  form.email || 'Your email',
].join(' — ');
~~~

When either input changes, React renders again and calculates the latest summary. Keeping a separate summary state would require every change handler to update both the field and the summary, creating a chance that they disagree.

The same rule applies to simple counts, whether a required field is empty, and other values that can be calculated cheaply from current state. Keep state for information that can change independently, not for a duplicate of information already available.

## Clear or replace form values

To clear the form, replace its values with the initial shape:

~~~jsx
function handleReset() {
  setForm({
    fullName: '',
    email: '',
    topic: '',
    message: '',
  });
}
~~~

A reset button can call this function:

~~~jsx
<button type="button" onClick={handleReset}>
  Clear form
</button>
~~~

For an actual form submission, read the current state in the submit handler. If the operation succeeds and the intended behavior is to clear the controls, set the state back to its initial values. If it fails, preserve the values so the person can correct or retry them.

This chapter keeps submission behavior simple. Browser submission and validation behavior are covered in Chapter 030, and accessible validation feedback is covered in Chapter 031.

## Choosing a state shape

Use a structure that makes valid updates straightforward:

- **Group values that belong together.** A form object is useful when its fields are read or submitted together.
- **Keep independent behavior independent.** Separate state variables can be easier to understand when parts update on unrelated schedules.
- **Avoid duplicated values.** Derive previews, labels, counts, and simple flags from the current form state.
- **Keep updates shallow where practical.** Flat objects are easier to copy and update than deeply nested structures.
- **Use stable value types.** Text-like controls start with strings; checkboxes start with booleans.
- **Make every update immutable.** Return a new object rather than assigning into the existing state object.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Updating the state object in place | React may not see a new object identity, and other properties can be affected unexpectedly. | Return a copied object with the changed property. |
| Replacing the whole object with one field | The other form values are discarded. | Copy the previous object, then overwrite the one field. |
| Forgetting a control's `name` | A generic handler cannot identify which property should change. | Match every control name to a property in the state object. |
| Using `value` to control a checkbox | Checkbox selection is represented by a boolean checked state. | Use `checked` and store a boolean. |
| Passing `undefined` for an initially empty text field | The control can switch between uncontrolled and controlled. | Initialize it with an empty string. |
| Storing a preview as another state variable | It can disagree with the current input values. | Calculate it from the form state during rendering. |
| Parsing every number input value immediately | Empty and partially entered values can be lost or converted unexpectedly. | Keep the input representation as a string and parse when the application needs a number. |
| Storing every field in one deeply nested object | Updates become harder to read and copy correctly. | Prefer a shallow form object or separate related state. |

## Practice

1. Create state for a form with name, email, topic, message, and a checkbox.
2. Add controlled text, select, textarea, and checkbox controls.
3. Use a shared handler for text-like fields and read their `name` and `value`.
4. Update the checkbox using `checked` and keep its state boolean.
5. Type into each field and confirm editing one field preserves the others.
6. Derive a short preview from the name and topic without adding another state variable.
7. Add a clear button with `type="button"` that restores the initial values.
8. Temporarily remove the spread from the object updater and observe how other fields disappear; then restore it.
9. Try initializing one text field with `undefined`, observe the controlled/uncontrolled warning, and restore the empty string.

## Key takeaways

- A multi-field form can store related values in one state object.
- Controlled values come from state, and `onChange` updates that state synchronously.
- Use each control's `name` to update only the matching property.
- Copy objects and return new state; do not mutate the existing object.
- Use `checked` with boolean state for checkboxes.
- Keep text-like values as strings and compute simple derived information during render.
- Choose a form state shape that is easy to understand and hard to update incorrectly.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [031 - Accessible validation messages and error summaries](031-accessible-validation-messages.md) | [Open Q&A](../qnas/032-multi-field-form-state.md) | [Open Example](../examples/032-multi-field-form-state.md) | [033 - Uncontrolled forms and refs](033-uncontrolled-forms-and-refs.md) |
