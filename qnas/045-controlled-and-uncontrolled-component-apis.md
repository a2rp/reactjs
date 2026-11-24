# Controlled and Uncontrolled Component APIs — Q&A

## Core concepts

### 1. What makes a component controlled?

Its current value comes from a prop. A parent supplies the value and decides what it becomes after the child reports an interaction.

### 2. What makes a component uncontrolled?

The component or browser owns its current value. A parent may provide an initial value, but it does not continuously provide the current one.

### 3. What is the difference between value and defaultValue?

value supplies the current value for a controlled component. defaultValue supplies the initial value for an uncontrolled component. They express different ownership models.

### 4. Does defaultValue update the input whenever the prop changes?

No. It provides an initial value. Later user edits are not replaced just because defaultValue changes.

### 5. What does onChange do in a controlled component?

It reports a user's requested change. The parent updates its state and passes the new current value back through props.

### 6. Can an uncontrolled component have an onChange callback?

Yes. It can notify the parent about interactions while still owning the current value itself. Notification alone does not make it controlled.

### 7. Does a controlled component always mean the whole component is controlled?

No. A component may receive one value from its parent and keep unrelated interaction state locally. Ownership is considered separately for each value.

### 8. What problem is a controlled API useful for solving?

It lets a parent coordinate the value with other components, validate or transform changes, derive other UI, or reset the value from outside the component.

### 9. When can an uncontrolled API be simpler?

When the value is local to one component or form and the parent only needs to read it later, such as at submission time.

### 10. Does an uncontrolled component mean React cannot be informed about its value?

No. Callbacks or refs can expose information to a parent. The key distinction is who supplies and owns the current value.

## Designing a dual-mode API

### 11. How can a custom component support both controlled and uncontrolled use?

It can accept a current value prop for controlled mode and an initial defaultValue prop for uncontrolled mode. Internally it stores a value only in uncontrolled mode and reports user requests through a callback.

### 12. How can a component detect whether an optional value prop is controlled?

A common convention is to treat the value as controlled when it is not undefined:

~~~jsx
const isControlled = value !== undefined;
~~~

The component's API should document this convention.

### 13. Why should the component not use a truthiness check to detect controlled mode?

An empty string, false, and zero can all be meaningful controlled values. A truthiness check would mistake them for missing values.

### 14. Should a caller pass both value and defaultValue?

Usually no. The props describe different owners. A component should document a clear rule or warn about the combination; a caller should choose one mode.

### 15. Why should a component remain in one mode during its lifetime?

Switching modes changes who owns the current value. It can leave internal state stale or create ambiguous behavior. Choose the mode at mount and keep it stable.

### 16. In controlled mode, can the value temporarily be undefined?

A custom API that uses undefined to detect uncontrolled mode should keep supplying a defined controlled value. Use an explicit empty value or a separate loading state instead of switching ownership accidentally.

### 17. Should defaultValue changes reset an uncontrolled component?

Not automatically. defaultValue is initial configuration. If the parent needs to reset the current value, use an explicit reset API, native form reset, or an intentional remount strategy.

### 18. What should a dual-mode component do when the user requests a change?

In uncontrolled mode it updates its internal state. In controlled mode it calls the callback and renders the value supplied by the parent. It can notify the parent in either mode.

### 19. Should an onChange callback always receive a native event?

No. Native input onChange handlers receive a React event. A custom component may choose to pass the next value or a domain-specific payload. Document the callback signature.

### 20. Why might a named callback be preferable to exposing a state setter?

A callback such as onOpenChange describes the component's supported action without exposing the parent's state implementation. It gives the parent room to validate or transform the request.

## Native form controls

### 21. How is a text input controlled?

Pass its current string through value and update that state from event.target.value in onChange.

### 22. How is an uncontrolled text input initialized?

Use defaultValue. The browser then owns the current value after the user edits it.

### 23. How is a textarea controlled?

Pass its current text with value and update the state from event.target.value in onChange.

### 24. How is a checkbox controlled?

Use checked for its current boolean state and read event.target.checked in onChange.

### 25. What does defaultChecked do?

It sets an uncontrolled checkbox's initial checked state. It does not continuously force the checkbox to match later prop changes.

### 26. What is the difference between checked and value on a checkbox?

checked says whether the checkbox is selected. value is the form value submitted if that checkbox is selected.

### 27. How does a controlled radio group usually work?

The parent stores one selected option value. Each radio gets checked based on whether its own value matches the selected value, and reports changes to the parent.

### 28. How is a select element controlled?

Provide the selected option value through the select's value prop and update it from the select's onChange event.

### 29. How is an uncontrolled select initialized?

Use defaultValue on the select. The browser manages the current selection afterward.

### 30. What does a multiple select use for its selected values?

For controlled use, value can be an array of selected option values. For uncontrolled initial selection, defaultValue can also be an array.

### 31. Why is a file input normally uncontrolled?

The browser owns the local file selection and does not allow scripts to set an arbitrary local file path. Read selected files from the event or a ref.

### 32. Can a React app clear a file input?

It can clear the input, for example by setting its value to an empty string through a ref or remounting it. It cannot set it to an arbitrary file path.

## Rendering and updating

### 33. Why can a controlled input appear not to accept typing?

If its onChange handler does not update the value that the parent passes back, React keeps rendering the old value. The handler should update state promptly from the user's input.

### 34. What value should a controlled text field handler usually store?

Usually event.target.value. Keeping the state aligned with the input's current text avoids reverting the displayed value.

### 35. Why can transforming text on every keystroke affect the caret?

The transformed string may differ from the value the user just entered. The browser and React then reconcile a changed value, which can move the caret. Transform during editing only when that behavior is intended.

### 36. What does it mean if a controlled input has no onChange handler?

It is effectively read-only and React may warn. Add an update handler or mark it readOnly if the user is not supposed to edit it.

### 37. How should a parent reset a controlled input?

Set the parent's state to the reset value. The controlled value prop then updates the input.

### 38. How can an uncontrolled native form be read on submit?

Use the form's submit event and FormData, or use refs for specific fields. React does not need to store every field's current value in state.

### 39. What does a callback mean in controlled mode?

It reports the user's requested next value. The parent owns the decision and the value rendered afterward, so it can accept, reject, or transform that request.

### 40. What does a callback mean in uncontrolled mode?

It can notify the parent that the component's internally owned value changed. The callback does not transfer ownership by itself.

## Common confusions

### 41. Is a component controlled just because it has a callback?

No. It is controlled when its current value comes from the parent. A callback can also be a notification from an uncontrolled component.

### 42. Is a component uncontrolled just because it uses local state?

Not always. A component may use local state for unrelated details while a key value is controlled by props. Evaluate the ownership of each value.

### 43. Why should a controlled child not copy its prop to local state?

The local copy will not automatically follow later prop changes. Render from the controlled prop instead of maintaining a second version.

### 44. Can a custom component be controlled without having a callback?

It can receive a value and be read-only, but then the user cannot request changes through that API. If editing is expected, the component needs a callback for requests.

### 45. Should the component update its own internal value in controlled mode before the parent responds?

Usually no. The parent remains the source of truth. Optimistic UI is a separate design that needs clear reconciliation behavior.

### 46. Is a controlled form always better than an uncontrolled form?

No. Controlled forms are useful for live coordination and validation. Uncontrolled forms can be simpler when values are only needed at submission.

### 47. Can a form mix controlled and uncontrolled fields?

Yes. Each field can use the ownership model that fits its needs. For example, a live-preview text field can be controlled while a file input remains uncontrolled.

### 48. How can an uncontrolled field notify its parent without becoming controlled?

It can call a callback when the browser value changes. The parent observes the value but does not supply the current value back as a prop.

## Apply the concepts

### 49. A text input uses value={name} and onChange={setName}. Which component owns the current name?

The component whose state provides name owns the current value. The input renders the prop and reports updates.

### 50. A field uses defaultValue="React" and later the parent changes that prop to "Hooks". Should the current user-edited value be replaced?

No. defaultValue is the initial value for an uncontrolled field, not a continuing command to replace the current value.

### 51. A checkbox is controlled by a boolean. Which event property should update it?

Use event.target.checked.

### 52. A search box's value is only read when the user submits the form. Must it be controlled?

No. An uncontrolled input with a ref or FormData may be simpler.

### 53. A parent needs to disable a Save button while a field is empty. Which design is appropriate?

A controlled field is convenient because the parent already has the current value and can derive whether Save should be disabled. An uncontrolled design could still report changes, but then the parent needs another way to track the value.

### 54. A component receives isOpen={false}. Is it controlled?

Yes, if the API uses undefined to identify uncontrolled mode. False is a defined controlled value.

### 55. A component receives value="" with defaultValue="hello". Which mode should the caller intend?

Controlled mode with an empty current value. The caller should not also pass defaultValue; remove the conflicting prop.

### 56. A file input needs to upload the chosen file. How can the app read it?

Read event.target.files in the change handler or read the files from an input ref when needed. Do not try to assign a local file path through value.

## Quick review

### 57. What does value mean?

The current value supplied by the parent in a controlled API.

### 58. What does defaultValue mean?

The initial value for a component or input that owns its current value.

### 59. What is the most important controlled/uncontrolled design decision?

Choose who owns the current value, keep the mode stable, and make the value and callback props express that ownership clearly.

## Key takeaway

Controlled components render a value supplied by their parent. Uncontrolled components own their current value and may start from a default. Callbacks can report changes in either model; ownership is determined by who supplies and updates the current value.
