# 033. Uncontrolled forms and refs

## Goal

In an uncontrolled form, the browser keeps the current value of each control in the DOM. React can provide an initial value and read the fields when the form is submitted, without storing every keystroke in React state.

A ref gives a component access to a DOM node for a focused task, such as focusing the first field or resetting a form. This chapter covers uncontrolled controls, reading values with `FormData`, and practical refs. File uploads are covered in Chapter 034.

## Learning outcomes

After this chapter, you should be able to:

- Explain the difference between controlled and uncontrolled form controls.
- Set an initial uncontrolled value with `defaultValue` or `defaultChecked`.
- Read named form controls with `FormData` in a submit handler.
- Explain how checkboxes and repeated field names appear in `FormData`.
- Create a DOM ref with `useRef` and access its `current` value in an event handler.
- Use a ref for focus or form reset without using it as a substitute for ordinary React state.
- Avoid switching a control between controlled and uncontrolled modes.

## Controlled and uncontrolled values

A controlled input receives its current value from React state. An uncontrolled input keeps the current value in the browser-managed DOM; React may provide its initial value, but does not update the value on every edit.

~~~jsx
// Uncontrolled: React sets the initial value.
<input name="nickname" defaultValue="Guest" />

// Controlled: React supplies the current value and handles each edit.
<input name="nickname" value={nickname} onChange={handleNicknameChange} />
~~~

Use a controlled input when the interface needs the current value during editing, such as for a live preview, conditional fields, or immediate validation. An uncontrolled input is useful when the value is only needed at a specific point, such as when the user submits the form.

These are design choices, not separate categories for every form. A form can contain both kinds of control, but choose intentionally and do not pass both a current-value prop and an initial-value prop to the same control.

## Set initial values

For uncontrolled text-like inputs, use `defaultValue`. For a checkbox or radio control, use `defaultChecked`:

~~~jsx
<input name="fullName" defaultValue="" />
<input name="email" type="email" defaultValue="" />

<label>
  <input
    name="receiveUpdates"
    type="checkbox"
    value="yes"
    defaultChecked={false}
  />
  Send me product updates
</label>
~~~

For an uncontrolled select, set `defaultValue` on the select. For a textarea, set its initial text through `defaultValue`:

~~~jsx
<select name="topic" defaultValue="">
  <option value="">Choose a topic</option>
  <option value="account">Account</option>
  <option value="billing">Billing</option>
</select>

<textarea name="message" defaultValue="" rows="4" />
~~~

An initial value only seeds the DOM control. If a parent later changes the `defaultValue` prop, React does not treat it as a command to replace the value the person has edited. If the interface needs the value to track new props or application changes, use a controlled value instead.

## Read values with FormData

Each field that should be submitted needs a meaningful `name`. In a submit handler, construct `FormData` from the form and read the named values:

~~~jsx
function handleSubmit(event) {
  event.preventDefault();

  const formData = new FormData(event.currentTarget);
  const fullName = String(formData.get('fullName') ?? '');
  const email = String(formData.get('email') ?? '');

  console.log({ fullName, email });
}
~~~

Attach submission logic to the form's `onSubmit` event:

~~~jsx
<form onSubmit={handleSubmit}>
  <label htmlFor="fullName">Full name</label>
  <input id="fullName" name="fullName" defaultValue="" />

  <label htmlFor="email">Email address</label>
  <input id="email" name="email" type="email" defaultValue="" />

  <button type="submit">Continue</button>
</form>
~~~

The form submit event covers submission through the submit button and supported keyboard actions. `event.preventDefault()` prevents the browser's normal navigation, allowing the handler to use the values in the current page. Native interactive validation, if enabled, runs before the submit handler.

A control with no `name` is not included under a useful key in form data. A visible label does not replace a name; labels and names have different jobs.

## FormData values

For ordinary text controls, `FormData.get(name)` returns the value for that field. If several controls use the same name, use `getAll(name)` to retrieve all values:

~~~jsx
const formData = new FormData(form);
const selectedTopics = formData.getAll('topics');
~~~

A checked checkbox contributes its value. An unchecked checkbox is omitted, rather than included as false. For a single optional checkbox, check whether the form data has that name:

~~~jsx
const wantsUpdates = formData.has('receiveUpdates');
~~~

A group of same-name checkboxes can contribute multiple values. Radio buttons with a shared name contribute the selected value. File controls contribute `File` objects, not ordinary text; file-specific handling is covered in Chapter 034.

When a value may be missing, handle that explicitly. `get()` returns `null` if there is no successful control with that name. A disabled control is not submitted, and a control without a name is not included.

## Store submitted results when the interface needs them

Uncontrolled input values do not make the component re-render on each keystroke. If the interface should show a receipt after submission, read the DOM values with `FormData` and then store the submitted snapshot in state:

~~~jsx
function ContactForm() {
  const [receipt, setReceipt] = useState(null);

  function handleSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);

    setReceipt({
      fullName: String(formData.get('fullName') ?? ''),
      email: String(formData.get('email') ?? ''),
      topic: String(formData.get('topic') ?? ''),
    });
  }

  // Render the uncontrolled form and, when present, the receipt.
}
~~~

This state is a submitted snapshot used for rendering. It does not control the fields themselves. If a live preview is required while the person types, make the relevant fields controlled or deliberately listen to input events; do not expect a ref or `FormData` to trigger a render.

## What is a ref?

A ref is a stable object that can hold a value between renders. When a ref is attached to a built-in DOM element, React sets its `current` property to that DOM node after it is mounted:

~~~jsx
import { useRef } from 'react';

function SearchBox() {
  const searchRef = useRef(null);

  function focusSearch() {
    searchRef.current?.focus();
  }

  return (
    <>
      <label htmlFor="site-search">Search</label>
      <input id="site-search" ref={searchRef} name="query" />
      <button type="button" onClick={focusSearch}>
        Focus search
      </button>
    </>
  );
}
~~~

Before the node is available, and after React removes it, `searchRef.current` is `null`. Use a null check when the node might not be mounted. Read or update the ref in an event handler or an effect, not as part of rendering.

Changing `ref.current` does not trigger a React render. This is why refs are appropriate for DOM access or values that do not determine the rendered interface. If a value must update what React displays, use state.

## Use refs for focused DOM actions

Useful form-related ref tasks include:

- Focus an input after a user activates a button.
- Scroll to a particular control after a deliberate interaction.
- Call the browser's reset method on an uncontrolled form.
- Integrate a DOM API that React does not provide as a prop.

Refs are an escape hatch. Prefer rendering from props and state for ordinary UI changes. Avoid manually inserting, deleting, or rearranging DOM nodes that React owns; React may later render a different tree and conflict with those edits.

A ref is not required just to read all fields during submit. `new FormData(event.currentTarget)` is usually simpler than making one ref for every control.

## Reset an uncontrolled form

A ref can access the form node and call its native `reset()` method. This restores the controls to their initial DOM values:

~~~jsx
const formRef = useRef(null);

function handleReset() {
  formRef.current?.reset();
}

return (
  <form ref={formRef}>
    <input name="fullName" defaultValue="" />
    <button type="reset">Reset fields</button>
  </form>
);
~~~

A native button with `type="reset"` already resets its form. Use a handler and ref only when the interface also needs to clear related React state, close a message, or perform another action.

Resetting uncontrolled fields does not reset unrelated React state such as a submitted receipt. Update that state separately when the product behavior calls for it. For controlled inputs, resetting the DOM alone is not enough: update the React state that supplies their values.

## Choosing controlled or uncontrolled

Choose controlled fields when the current value affects rendering while the person edits, when a value must be changed by application logic, or when field behavior depends on other React state.

Choose uncontrolled fields when the browser can own edits until submission and the application only needs to read values at a clear boundary. Read all the named controls together with `FormData`. Use a ref only for an imperative DOM operation such as focus or reset.

Whichever approach you choose:

- Keep labels associated with their controls.
- Give submitted controls meaningful names.
- Do not combine `value` with `defaultValue` or `checked` with `defaultChecked` on the same control.
- Do not switch a control from controlled to uncontrolled or back during its lifetime.
- Validate and handle submitted values on the server when sending real data.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Expecting `defaultValue` to update a field after it has mounted | It sets an initial value, not a continuously controlled value. | Use `value` and state when the application must control later changes. |
| Combining `value` and `defaultValue` on one input | The control is being given both controlled and uncontrolled instructions. | Choose one value model for that control. |
| Reading a field from FormData without a name | Form data uses names as keys. | Give each submitted control a meaningful `name`. |
| Expecting an unchecked checkbox to submit false | Unchecked checkboxes are omitted from form data. | Use `formData.has(name)` for a single checkbox or interpret the returned entries. |
| Using `get()` for a checkbox group | It returns only one matching entry. | Use `getAll()` for repeated names. |
| Using a ref for every field when only submitting the form | It creates extra wiring for a task FormData already handles. | Construct FormData from the form in its submit handler. |
| Expecting a ref update to re-render the component | React does not subscribe to changes to ref.current. | Use state for information that changes the displayed UI. |
| Reading ref.current during rendering | The DOM node may not be attached yet, and render should stay predictable. | Read the ref in an event handler or effect. |
| Manually removing or inserting React-managed nodes | React's next render can conflict with direct DOM changes. | Change props or state and let React update its DOM. |
| Resetting a controlled input only with form.reset() | Its displayed value comes from React state. | Reset the state that supplies the controlled value. |
| Assuming browser validation replaces server validation | Client checks can be bypassed. | Validate submitted values on the server as well. |

## Practice

1. Create an uncontrolled form with a name input, email input, select, textarea, and checkbox.
2. Set initial values with `defaultValue` and `defaultChecked`.
3. Add a submit handler that constructs `FormData` from the form and reads named values.
4. Give two checkboxes the same name and compare `get()` with `getAll()`.
5. Leave a checkbox unchecked and confirm it is absent; use `has()` to produce a boolean.
6. Add a ref to the form and reset its fields after a deliberate button action.
7. Add a ref to the name input and focus it from a button with `type="button"`.
8. Store a receipt in React state after submit while leaving the input controls uncontrolled.
9. Explain which parts of the screen update while typing and which update only after submission.
10. Convert one field to a controlled input and make sure you do not also pass `defaultValue`.

## Key takeaways

- An uncontrolled field's live value is kept by the DOM; `defaultValue` and `defaultChecked` only provide initial values.
- Use meaningful `name` attributes and `FormData` to read a group of fields at submission.
- Use `getAll()` for repeated names and `has()` to check an optional checkbox.
- A ref's `current` property can point to a DOM node; use it for focused actions such as focus or reset.
- Changing `ref.current` does not trigger a render; use state for values that affect rendered UI.
- Do not manipulate React-owned DOM directly or mix controlled and uncontrolled props on a control.
- Client-side form handling does not replace server-side validation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [032 - Managing state for multi-field forms](032-multi-field-form-state.md) | [Open Q&A](../qnas/033-uncontrolled-forms-and-refs.md) | [Open Example](../examples/033-uncontrolled-forms-and-refs.md) | [034 - File inputs and upload flows](034-file-inputs-and-upload-flows.md) |
