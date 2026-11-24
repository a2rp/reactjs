# Controlled and Uncontrolled Component APIs

## Why this distinction matters

A component can keep its own changing value, or it can receive that value from its parent. These are commonly called **uncontrolled** and **controlled** designs.

For a simple independent input, local state or the browser's form behavior may be enough. When a parent must validate the value, coordinate sibling components, reset it, or use it elsewhere, a controlled API is useful.

These are API and ownership choices. They do not mean that one approach is always better.

## Learning goals

After this lesson, you should be able to:

- explain how controlled and uncontrolled values differ;
- design APIs that use value and defaultValue intentionally;
- handle React inputs, checkboxes, selects, and file inputs correctly;
- understand the role of change callbacks in both designs;
- avoid switching a component between controlled and uncontrolled modes;
- choose a design based on who needs to own and coordinate the value.

## Controlled values

A component is controlled when its current value comes from a prop. The parent owns the value and passes it down. The child reports user input through a callback.

~~~jsx
function NameField({ value, onChange }) {
  return (
    <label>
      Name
      <input
        value={value}
        onChange={(event) => onChange(event.target.value)}
      />
    </label>
  );
}
~~~

The parent stores and updates the value:

~~~jsx
function ProfileForm() {
  const [name, setName] = useState("");

  return (
    <>
      <NameField value={name} onChange={setName} />
      <p>Preview: {name || "No name entered"}</p>
    </>
  );
}
~~~

The input renders the value it receives. When a user types, it reports the new value. The parent updates state and passes the new value back.

A controlled component is useful when the parent needs to:

- validate input as it changes;
- coordinate multiple components that depend on the same value;
- enable or disable other controls;
- display a preview, count, or derived message;
- reset the value from an external action;
- submit or transform the value using parent logic.

## Uncontrolled values

An uncontrolled component keeps its own current value. A parent may provide an initial value, often through a prop named defaultValue:

~~~jsx
function SearchBox() {
  return (
    <label>
      Search
      <input defaultValue="React" />
    </label>
  );
}
~~~

For a native uncontrolled input, the browser manages the current form value. React provides the initial value, but later changes to defaultValue do not update what the user has typed.

A custom uncontrolled component may store its current value in its own state, using defaultValue as the initial value. The key idea is that the parent does not continuously supply the current value.

Uncontrolled designs are useful when:

- the value is local to one component or form;
- the parent only needs the value at submit time;
- the browser's form behavior is sufficient;
- the component can manage its own interaction without sibling coordination.

Uncontrolled does not mean that a component cannot notify a parent. It can call onChange or onOpenChange to report user interaction while still owning the current value. A notification callback does not make the value controlled.

## value and defaultValue have different jobs

A clear component API commonly distinguishes:

- value: the current value supplied by the parent; this is controlled mode;
- defaultValue: the initial value for the component's own state; this is uncontrolled mode;
- onChange: a callback reporting a user's requested value change.

For a dual-mode component, use one mode at a time. Passing both value and defaultValue makes ownership unclear. Choose a documented rule or warn the caller; a simple API should usually avoid the combination.

An empty string, false, or zero can be a valid controlled value. Do not decide the mode by truthiness. For an optional value prop, check whether it is undefined:

~~~jsx
const isControlled = value !== undefined;
~~~

When controlled, continue providing a defined value for the component's lifetime. If a value is temporarily unavailable, choose an explicit representation such as an empty string or a loading state instead of switching to undefined.

## Controlled and uncontrolled React inputs

### Text inputs and textareas

A controlled text input uses value and onChange:

~~~jsx
<input
  value={name}
  onChange={(event) => setName(event.target.value)}
/>
~~~

A controlled textarea works the same way:

~~~jsx
<textarea
  value={message}
  onChange={(event) => setMessage(event.target.value)}
/>
~~~

An uncontrolled input uses defaultValue:

~~~jsx
<input defaultValue="Initial text" />
<textarea defaultValue="Initial message" />
~~~

React uses defaultValue as the initial value. It does not continuously control the input afterward.

For a controlled input, update the state from the event value promptly. If the prop keeps passing the old value, the input appears not to accept typing. Transforming the value on every keystroke can also affect caret position; only transform during editing when that behavior is intended.

### Checkboxes and radio buttons

The current checked state is controlled with checked and onChange:

~~~jsx
<input
  type="checkbox"
  checked={subscribed}
  onChange={(event) => setSubscribed(event.target.checked)}
/>
~~~

The uncontrolled initial checked state uses defaultChecked:

~~~jsx
<input type="checkbox" defaultChecked />
~~~

For checkboxes, read event.target.checked to obtain a boolean. For a radio group, the parent can own one selected value and pass checked to each radio based on that value.

### Select elements

A controlled select receives value and an onChange callback:

~~~jsx
<select
  value={selectedColor}
  onChange={(event) => setSelectedColor(event.target.value)}
>
  <option value="blue">Blue</option>
  <option value="green">Green</option>
</select>
~~~

An uncontrolled select can receive defaultValue. For a multiple select, value or defaultValue can be an array of selected option values.

### File inputs

File inputs are an important exception: browsers do not allow a script to set an arbitrary local file selection. Treat a file input as uncontrolled. Read the selected files from its change event or through a ref, and clear it with an appropriate DOM interaction if needed.

Do not try to control a file input by setting its value to a file path.

## A dual-mode custom component

A reusable component can support both ownership styles. For example, a disclosure can accept isOpen for controlled use or defaultOpen for uncontrolled use:

~~~jsx
function Disclosure({
  isOpen,
  defaultOpen = false,
  onOpenChange,
  title,
  children,
}) {
  const isControlled = isOpen !== undefined;
  const [internalIsOpen, setInternalIsOpen] = useState(defaultOpen);
  const open = isControlled ? isOpen : internalIsOpen;

  function requestOpenChange(nextOpen) {
    if (!isControlled) {
      setInternalIsOpen(nextOpen);
    }
    onOpenChange?.(nextOpen);
  }

  return (
    <section>
      <h2>
        <button
          type="button"
          aria-expanded={open}
          onClick={() => requestOpenChange(!open)}
        >
          {title}
        </button>
      </h2>
      {open && <div>{children}</div>}
    </section>
  );
}
~~~

In controlled mode, the parent passes isOpen and updates it in response to onOpenChange. The component reports the requested next value but does not replace the parent's value with internal state.

In uncontrolled mode, the component initializes its own state from defaultOpen and updates that state when the user interacts. It may also call onOpenChange to notify the parent.

The component uses the same callback to report the requested change in either mode, but only the owner updates the authoritative value.

## Do not switch modes during a component's lifetime

A component should not start uncontrolled and later become controlled, or the reverse. The ownership contract becomes unclear and internal state may no longer match the prop.

For a dual-mode API, decide the mode when the component is created and keep that mode stable. In controlled mode, always provide the value prop, including false, zero, or an empty string when those are valid values. Do not use undefined to mean an empty controlled value.

For a custom component library, it can be helpful to record the initial mode and warn during development if a later render changes it.

## Change callbacks report requests

A callback's name and arguments are part of the component's public API. For a custom component, a callback can report the next value:

~~~jsx
onOpenChange(nextOpen)
~~~

For a native input, React's onChange receives an event, so the handler usually reads event.target.value or event.target.checked.

An uncontrolled component can report a change even though it owns the current value. The parent is being informed, not asked to supply the next rendered value.

In a controlled component, the callback communicates the user's request. The parent decides whether to accept it and what value to pass on the next render. For example, the parent may reject a value that fails validation.

## Resetting values

A controlled component resets when its owner passes the reset value. For example, a parent can set an input's value to an empty string.

An uncontrolled native form can use the browser's reset behavior, or a parent can read its fields on submit with FormData. A custom uncontrolled component can offer an explicit reset action or be intentionally remounted when its identity changes.

Changing defaultValue after the component has mounted is not a reset mechanism. It supplies the initial value, not a command to replace the current value.

## Choose based on the owner

Ask these questions:

1. Does another component need the current value while the user is editing?
2. Must the parent validate or transform each change?
3. Do sibling components need to stay synchronized with it?
4. Does an external parent action need to reset or replace it?
5. Is reading the value only at form submission enough?

If the parent must own, coordinate, or replace the current value, use a controlled API. If the value is local and only needs to be read later, an uncontrolled design may be simpler.

A component may mix these approaches for different values. For example, a form can control a field that drives a live preview while leaving an independent native file input uncontrolled.

## Common mistakes

### Providing value without a change handler

A controlled input with no way to update its value will behave as read-only. Add an onChange handler, or explicitly mark the field readOnly when that is intended.

### Using truthiness to detect controlled mode

Values such as false, zero, and an empty string can be valid controlled values. Use a defined API contract and an undefined check for optional value props.

### Passing value and defaultValue together

These props express different owners. Choose one mode and document it. Do not leave callers guessing which value wins.

### Expecting defaultValue to keep updating

defaultValue provides an initial value. It does not track later prop changes. Use value for parent-controlled updates.

### Copying a controlled prop into local state

This creates two versions of one value. Render from the prop in controlled mode. Local state belongs in uncontrolled mode or when it represents a distinct concept such as an unsaved draft.

### Switching from uncontrolled to controlled

Keep the ownership mode stable. A component should not receive undefined on one render and a controlled value on another.

### Using value to set a file input

A browser does not allow arbitrary file paths to be assigned by script. Read files from the input event or a ref instead.

### Treating an uncontrolled callback as ownership

An onChange notification does not make the parent the source of truth. If the component owns its current value, it remains uncontrolled even when it reports changes.

## Practice

1. A parent supplies value and updates it from onChange. Is the input controlled or uncontrolled?
2. What does defaultValue do for an uncontrolled native input?
3. Why should a dual-mode component check value !== undefined instead of if (value)?
4. Can an uncontrolled component call onChange?
5. What should a controlled text input's onChange handler usually store?
6. Why should a component not switch from defaultValue mode to value mode after mount?
7. Which prop controls a checkbox: value or checked?
8. Why is a file input normally uncontrolled?
9. What should happen when a controlled input needs to reset?
10. A text field only needs to be read when a form is submitted. Is controlled state always required?

Suggested answers:

1. Controlled. The parent supplies the current value and owns updates.
2. It supplies the initial value. Later prop changes do not continuously replace the user's current value.
3. False, zero, and empty string can all be valid controlled values. Truthiness would misclassify them.
4. Yes. It can notify the parent while still owning and updating its own value.
5. Usually event.target.value, promptly, so the state passed back on render reflects the user's edit.
6. The component's owner and current source of truth become ambiguous, and internal state may disagree with the prop.
7. checked controls whether a checkbox is selected; value is the form value submitted when it is checked.
8. The browser controls the local file selection and does not permit arbitrary paths to be set programmatically.
9. The parent sets the controlled value to the desired reset value.
10. No. An uncontrolled form may be simpler if no live coordination or validation is needed.

## Summary

- Controlled values come from props; uncontrolled values are owned by the component or browser.
- value represents the current controlled value; defaultValue provides an initial uncontrolled value.
- onChange can report a request in controlled mode or notify a parent in uncontrolled mode.
- Keep a dual-mode component in the same ownership mode for its lifetime.
- Use checked/defaultChecked for checkbox state and treat file inputs as uncontrolled.
- Choose the mode based on who needs to own, coordinate, validate, reset, and read the current value.
