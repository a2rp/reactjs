# 028. Controlled form elements

## Goal

A controlled form element gets its current value from React. When the user edits it, an event handler updates the state that supplies that value. This gives the component a current, predictable value that it can use to render other UI.

This chapter introduces the shared pattern for controlled inputs, textareas, selects, checkboxes, and radio buttons. The next chapter focuses on the individual control types and their details.

## Learning outcomes

After this chapter, you should be able to:

- Explain the difference between a controlled element and an uncontrolled element.
- Connect a form element's `value` or `checked` prop to React state.
- Update that state synchronously in `onChange`.
- Choose `defaultValue` or `defaultChecked` when React should set only an initial value.
- Keep a form element controlled for its entire lifetime.
- Decide when the form's current value needs to live in React state.

## The controlled data flow

For a controlled element, React state is the source of truth for the value currently shown in the interface:

1. React renders the element with a state value.
2. The user edits the element.
3. React's `onChange` handler reads the new value and updates state.
4. React renders again with the updated state.

The loop is:

**state → rendered value → user edit → `onChange` → state**

A text input controlled by state looks like this:

~~~jsx
import { useState } from 'react';

function NameField() {
  const [name, setName] = useState('');

  return (
    <label>
      Name
      <input
        name="name"
        value={name}
        onChange={(event) => setName(event.currentTarget.value)}
      />
    </label>
  );
}
~~~

The `value` prop tells React what the input should display. The handler immediately copies the user's edit into state, so the next render displays the new value.

`onChange` on a React text input fires as the value changes, including while the user types. Read the value from the element that fired the event, usually through `event.currentTarget.value`.

## Match the controlling prop to the element

Use the prop that represents the element's current state:

| Element | Controlled prop | Value type | Common change value |
| --- | --- | --- | --- |
| Text-like input | `value` | String | `event.currentTarget.value` |
| Textarea | `value` | String | `event.currentTarget.value` |
| Select | `value` | String, or an array of strings for a multiple select | Single select: `event.currentTarget.value`; multiple select: read `selectedOptions` |
| Checkbox | `checked` | Boolean | `event.currentTarget.checked` |
| Radio button | `checked` | Boolean | `event.currentTarget.checked`; in a group, often `event.currentTarget.value` |

For example, a controlled checkbox uses `checked`, because its current state is whether it is selected:

~~~jsx
const [isSubscribed, setIsSubscribed] = useState(false);

<input
  type="checkbox"
  checked={isSubscribed}
  onChange={(event) => setIsSubscribed(event.currentTarget.checked)}
/>
~~~

A checkbox's `value` is a separate concept: it is the value included in form data when that checkbox is checked. It does not control whether the checkbox is selected.

Radio buttons are also controlled with `checked`. In a group, state usually stores the one selected option; each radio's `checked` prop compares its option value with that state.

## Textareas and selects

A controlled textarea uses `value` and `onChange`, just like a text input. In React, put its text in the `value` prop; do not put initial text between the textarea tags.

~~~jsx
const [message, setMessage] = useState('');

<textarea
  name="message"
  value={message}
  onChange={(event) => setMessage(event.currentTarget.value)}
/>
~~~

A controlled select gets its selected option from the `value` on the parent `<select>`. Give each option a matching `value`:

~~~jsx
const [topic, setTopic] = useState('react');

<select
  name="topic"
  value={topic}
  onChange={(event) => setTopic(event.currentTarget.value)}
>
  <option value="react">React</option>
  <option value="accessibility">Accessibility</option>
</select>
~~~

Do not put a `selected` prop on an individual `<option>`. For a select with `multiple`, the controlled `value` is an array of strings. Read all selected values from `selectedOptions`:

~~~jsx
const [topics, setTopics] = useState([]);

<select
  multiple
  value={topics}
  onChange={(event) => {
    const selectedTopics = Array.from(
      event.currentTarget.selectedOptions,
      (option) => option.value,
    );
    setTopics(selectedTopics);
  }}
>
  <option value="react">React</option>
  <option value="accessibility">Accessibility</option>
</select>
~~~

For a radio group, the selected radio's `value` is often the value the application stores in state; `checked` determines whether each individual radio is selected.

## Controlled and uncontrolled are different choices

An uncontrolled element keeps its current value in the browser's DOM. React may provide its initial value, but React does not control later edits:

~~~jsx
<input name="nickname" defaultValue="Guest" />
~~~

`defaultValue` sets an initial value. `defaultChecked` sets an initial checkbox or radio state. Changing those props after the element mounts does not make them controlled.

A controlled element uses `value` or `checked` instead:

~~~jsx
<input
  name="nickname"
  value={nickname}
  onChange={(event) => setNickname(event.currentTarget.value)}
/>
~~~

Choose a controlled element when the current value should drive React behavior-for example, a live preview, conditional UI, or validation while typing. Choose an uncontrolled element when the browser can own the value until the form is read or submitted. The form submission chapter covers more ways to read form data.

Do not combine `value` with `defaultValue` or `checked` with `defaultChecked` for the same element. Decide who owns the current value.

## Keep the controlled value stable

An element must not switch between controlled and uncontrolled during its lifetime.

For text-like controls, provide a string from the first render. If data may be missing, use a fallback:

~~~jsx
<input
  value={profile.name ?? ''}
  onChange={handleNameChange}
/>
~~~

Do not start with `value={undefined}` and later pass a string. For a checkbox or radio, `checked` should consistently be a boolean.

A controlled field also needs an `onChange` handler that updates its backing value. If `value` is fixed and the handler does not update it, React renders that same value again and the user cannot keep their edit.

## Update from the user's edit

For ordinary controlled fields, store the value received from the element in the change handler:

~~~jsx
function handleChange(event) {
  setValue(event.currentTarget.value);
}
~~~

Update the state during that handler. If you delay the state update or store a different value on every keystroke, the displayed field may revert or the caret may move unexpectedly. If you need to normalize text, consider doing it when the user leaves the field or submits the form rather than changing every keystroke.

For a form with a few related fields, state can be an object. Use the field's `name` to update the matching property, while treating checkboxes as booleans:

~~~jsx
function handleChange(event) {
  const { name, type, value, checked } = event.currentTarget;

  setForm((currentForm) => ({
    ...currentForm,
    [name]: type === 'checkbox' ? checked : value,
  }));
}
~~~

Each field still has an explicit controlled prop: `checked` for a checkbox and `value` for a text field. The shared handler only avoids repeating the state update logic.

## Form data and accessibility still matter

Controlled values do not replace normal form structure:

- Give each submitted control a `name`.
- Associate each field with a visible `<label>` using nesting or matching `htmlFor` and `id`.
- Use the correct semantic input type and native constraints where they fit.
- Keep a `<form>` when the fields form a submission unit.

A named controlled field can still be included in `FormData`. React state determines the displayed value; `name` identifies the field in submitted form data.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Passing `value` without updating state in `onChange` | React keeps rendering the old value, so edits appear to be rejected. | Store the current value from the event in the handler. |
| Using `value` to control whether a checkbox is selected | A checkbox's `value` describes its form value, not its checked state. | Control selection with boolean `checked`. |
| Reading a checkbox's `.value` as its checked state | `.value` is not the boolean selection state. | Read `event.currentTarget.checked`. |
| Starting with `undefined` and later passing a string | The element changes from uncontrolled to controlled. | Initialize with `''` or use a string fallback. |
| Using `defaultValue` when the current value must drive React UI | `defaultValue` only provides the initial value. | Use state and `value`. |
| Setting `selected` on an `<option>` | React select state belongs on the parent `<select>`. | Use its `value` or `defaultValue`. |
| Transforming every keystroke into a different string | The browser's selection or caret can jump. | Keep the field value in sync; normalize at a deliberate point. |
| Omitting `name` because the field is controlled | The field may not be included under a useful key in form data. | Add a meaningful `name` to submitted controls. |

## Practice

1. Create a text field whose value is stored in state.
2. Render a preview that changes as the user edits the field.
3. Add a textarea using the same controlled pattern.
4. Add a select whose selected value is stored in state.
5. Add a checkbox and verify that its state is a boolean.
6. Temporarily remove the state update from `onChange`, observe the behavior, then restore it.
7. Start a field with `undefined`, then fix it by giving it a stable initial value.

## Key takeaways

- A controlled form element displays the value React supplies through `value` or `checked`.
- Its `onChange` handler updates the backing state synchronously from the user's edit.
- Text inputs, textareas, and selects use `value`; checkboxes and radios use `checked`.
- `defaultValue` and `defaultChecked` set initial values for uncontrolled elements.
- Keep a control consistently controlled or consistently uncontrolled.
- Keep labels, names, and semantic form markup even when React manages the value.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="027-event-propagation-and-default-actions.md">027 - Event propagation, default actions, and cancellation</a></td><td><a href="../qnas/028-controlled-form-elements.md">Q&amp;A</a></td><td><a href="../examples/028-controlled-form-elements.md">Example</a></td><td><a href="029-inputs-selects-checkboxes-and-radio-buttons.md">029 - Text inputs, selects, checkboxes, and radio buttons</a></td></tr>
  </tbody>
</table>
