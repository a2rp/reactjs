# 033. Uncontrolled forms and refs - Q&A

## Controlled and uncontrolled fields

### 1. What is an uncontrolled form field?

It is a form control whose current value is maintained by the browser DOM rather than being supplied on every render from React state. React can still provide its initial value.

### 2. What is a controlled form field?

It is a control whose current value comes from React, usually through state, and whose event handler updates that state as the person edits. The rendered control reflects the React value.

### 3. Is one approach always better?

No. Use controlled fields when the current value needs to affect the UI while editing or must be changed by application logic. Uncontrolled fields can be simpler when values are needed only at submission.

### 4. Can one form mix controlled and uncontrolled fields?

Yes, if the distinction is intentional and each control has one clear value owner. Avoid accidentally mixing the two models on the same control.

### 5. What does defaultValue do?

It supplies the initial value for an uncontrolled text-like input, select, or textarea. It does not continuously control the current value after the person edits it.

### 6. What does defaultChecked do?

It supplies the initial checked state for an uncontrolled checkbox or radio button. Use checked with React state when the control should be controlled.

### 7. What happens if a defaultValue prop changes after the field has mounted?

It does not act as a command to replace the field's current edited value. If later application values must control the field, use a controlled value prop and state.

### 8. Can I pass value and defaultValue to the same control?

Do not do that. They describe different ownership models. Choose controlled value or an uncontrolled initial default for each field.

### 9. What is the equivalent mistake for checkboxes?

Passing both checked and defaultChecked to the same checkbox or radio combines controlled and uncontrolled instructions. Choose one model.

### 10. Does an uncontrolled field cause React to render on every keystroke?

No. Editing the DOM value alone does not update React state and does not cause a component render. A submit handler can read the value later.

## FormData and names

### 11. How do I read several uncontrolled fields at submit time?

Handle the form's submit event, create a FormData object from the form element, and read each field by its name. This collects the current successful controls as a group.

### 12. Why does a field need a name?

FormData uses the name as the key for the field's value. A label gives the control an accessible and visible name, but does not replace the form-data name.

### 13. Should the submit handler go on the form or on a button?

Put submission logic on the form's onSubmit event. That covers activation of different submit buttons and supported keyboard submission.

### 14. Why might event.preventDefault be called in a client-side submit handler?

It cancels the browser's normal navigation or reload so the handler can process the values on the current page. It does not disable validation that already runs before submission.

### 15. What type of value does FormData.get usually return?

For ordinary text controls it returns a string. For a file control it can return a File object. If there is no successful entry with that name, it returns null.

### 16. What is the difference between get and getAll?

get returns one value for a name. getAll returns every value with that name, which is useful for checkbox groups or other repeated controls.

### 17. How does an unchecked checkbox appear in FormData?

It is omitted. It does not submit the boolean false. For one optional checkbox, FormData.has(name) can be used to determine whether the checked value was submitted.

### 18. How do checked checkbox groups work?

If several checked checkboxes share one name, their values appear as repeated entries. Use getAll to read all selected values. If the application needs a live selection display, controlled state may be simpler.

### 19. How do radio buttons appear in FormData?

A radio group shares one name, and the selected radio contributes its value. If no radio in the group is selected, the name is omitted.

### 20. Are disabled controls submitted?

No. Disabled controls are omitted from FormData. Controls without a name are also not submitted under a field key.

### 21. Does native constraint validation still run before onSubmit?

Yes, when interactive validation is enabled. The browser normally blocks submission and does not dispatch the submit event if a control is invalid. A custom validation flow can disable the browser's automatic UI and render its own feedback.

### 22. Can FormData be used with controlled controls too?

Yes. FormData reads successful controls in the DOM regardless of whether React controls their values. For a controlled interface, reading directly from React state may be more natural if that state is already the source of truth.

## Refs and DOM access

### 23. What is a ref?

A ref is a stable object with a current property that can hold a value between renders. A common use is to let React place a DOM node into that property.

### 24. How do I attach a DOM ref?

Call useRef with an initial value such as null at the top level of a component, then pass the returned ref object to a built-in DOM element's ref prop.

### 25. When does a DOM ref become available?

React sets the ref's current property to the DOM node after it mounts. It is null before the node is attached and after React removes that node.

### 26. What can I do with an input DOM ref?

You can call browser methods such as focus or select in response to a user action. A ref can also be useful when a browser API requires the DOM element.

### 27. Does changing ref.current cause a render?

No. React does not track mutations to ref.current. If a changed value should update the rendered interface, store it in state instead.

### 28. Should I create a ref for every field to submit a form?

Usually not. A single FormData created from the form can read all named controls. Use per-field refs for a specific DOM task, not just to avoid using FormData.

### 29. Should I read ref.current during render?

No, aside from initializing a ref in a rare supported pattern. During render the DOM node may not be available, and render logic should remain predictable. Read refs in event handlers or effects.

### 30. Should I directly change or remove DOM nodes through a ref?

Avoid destructive edits to DOM that React manages. React expects its rendered tree to match the DOM and may overwrite or conflict with those edits. Prefer state and props for interface changes.

### 31. Can changing a ref store a value that the UI displays?

It can store it, but React will not know to render again when the ref changes. Use React state for values that determine displayed output.

### 32. Can a ref point to a form element rather than an input?

Yes. A ref to a form element can be used for browser APIs such as reset. It does not have to point only to inputs.

## Resetting and choosing a model

### 33. What does a native reset button do to uncontrolled controls?

A button with type reset restores the form controls to their initial DOM values, including values supplied through defaultValue and defaultChecked.

### 34. Does form.reset update React state?

No. It resets DOM controls. If those controls are controlled, React state still supplies their values and must be updated separately. Unrelated state such as a receipt also needs its own update if it should be cleared.

### 35. Do I need a ref just to use a reset button?

No. A button with type reset resets its associated form natively. A form ref is useful if a handler needs to call reset as part of a larger action or coordinate other state updates.

### 36. When is controlled state a better fit than FormData?

Use controlled state when the page needs a live preview, conditional interface, immediate feedback, programmatic field changes, or other UI updates based on current values.

### 37. When is an uncontrolled form a good fit?

It can fit a form whose values are only needed at submission and whose fields can remain under browser control until then. FormData makes it straightforward to read the named fields together.

### 38. Should a failed submission clear the fields?

Usually no. Keep the entered values so the person can fix a problem or retry. After a successful operation, clearing is a product decision; reset the fields and any related React state that should also disappear.

### 39. Does reading values with FormData validate them?

No. FormData collects values; it does not establish that they are valid or safe. Use browser constraints or application validation for feedback and validate again on the server when data is submitted.

### 40. What are the main rules to remember?

Use defaultValue and defaultChecked for initial uncontrolled values, give submitted fields names, read the form with FormData, and use refs for focused DOM actions. Use React state when current values need to drive rendering.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="032-multi-field-form-state.md">032 - Managing state for multi-field forms - Q&amp;A</a></td><td><a href="../lessons/033-uncontrolled-forms-and-refs.md">Lesson</a></td><td><a href="../examples/033-uncontrolled-forms-and-refs.md">Example</a></td><td><a href="034-file-inputs-and-upload-flows.md">034 - File inputs and upload flows - Q&amp;A</a></td></tr>
  </tbody>
</table>
