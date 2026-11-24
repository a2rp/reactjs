# 029. Text inputs, selects, checkboxes, and radio buttons

## Goal

React uses a small set of rules to control form elements, but each HTML control has its own value shape and behavior. This chapter applies the controlled pattern to common text inputs, single and multiple selects, checkbox groups, and radio groups.

Chapter 028 introduced the general controlled-value loop. Here the focus is choosing the right state shape and reading the right property for each kind of control.

## Learning outcomes

After this chapter, you should be able to:

- Choose useful input types and keep text-like values as strings.
- Control a single select with one string value and a multiple select with an array.
- Control an individual checkbox with a boolean.
- Represent a group of checkboxes as a set of selected values.
- Represent a radio group as one selected value.
- Give groups clear labels and understand how their names appear in form data.
- Apply common native input attributes without replacing accessible labels.

## Text inputs

Use the HTML `type` that describes the value and behavior you need. Common choices include `text`, `email`, `password`, `search`, `tel`, `url`, and `number`. A suitable type can give users a more relevant keyboard on mobile and enables browser behavior appropriate to that kind of field.

A controlled text input has a string `value` and updates that string from the input that fired the change:

~~~jsx
const [email, setEmail] = useState('');

<label htmlFor="account-email">Email address</label>
<input
  id="account-email"
  name="email"
  type="email"
  autoComplete="email"
  value={email}
  onChange={(event) => setEmail(event.currentTarget.value)}
/>
~~~

The `type` describes the field, `name` identifies it in form data, `id` connects it to its label, and `autoComplete` can help the browser fill known information. A placeholder can show an example, but it should not replace a visible label.

### Number inputs

A number input still exposes its current value as a string. Keep the editable value as a string so the field can represent an empty value while the user edits:

~~~jsx
const [quantity, setQuantity] = useState('1');

<input
  id="quantity"
  name="quantity"
  type="number"
  min="1"
  max="20"
  step="1"
  value={quantity}
  onChange={(event) => setQuantity(event.currentTarget.value)}
/>
~~~

Convert the string only when a calculation needs a number. Check for the empty string first, because `Number('')` evaluates to `0`:

~~~jsx
const quantityAsNumber = quantity === '' ? null : Number(quantity);
~~~

The `min`, `max`, and `step` attributes describe valid numeric ranges and increments to the browser. The next chapter covers form submission and browser validation in more detail.

## Single selects

For a single-choice select, state holds one string. Put the selected value on the parent `<select>`, and give each `<option>` a matching value:

~~~jsx
const [plan, setPlan] = useState('basic');

<label htmlFor="plan">Plan</label>
<select
  id="plan"
  name="plan"
  value={plan}
  onChange={(event) => setPlan(event.currentTarget.value)}
>
  <option value="basic">Basic</option>
  <option value="team">Team</option>
  <option value="studio">Studio</option>
</select>
~~~

Do not put `selected` on an `<option>`. React determines the selected option from the parent's `value` (or `defaultValue` for an uncontrolled select).

A required select can include a blank prompt option:

~~~jsx
<select name="department" value={department} onChange={handleDepartmentChange} required>
  <option value="">Choose a department</option>
  <option value="design">Design</option>
  <option value="engineering">Engineering</option>
</select>
~~~

The empty string is the prompt's value. If the user must make a choice, initialize the controlled state to `''` and retain the `required` attribute.

## Multiple selects

Set `multiple` when the user can choose more than one option. The controlled value is an array of strings, including when no options are selected:

~~~jsx
const [topics, setTopics] = useState([]);

function handleTopicsChange(event) {
  const selectedTopics = Array.from(
    event.currentTarget.selectedOptions,
    (option) => option.value,
  );

  setTopics(selectedTopics);
}

<label htmlFor="topics">Topics</label>
<select
  id="topics"
  name="topics"
  multiple
  size={4}
  value={topics}
  onChange={handleTopicsChange}
>
  <option value="components">Components</option>
  <option value="state">State</option>
  <option value="forms">Forms</option>
  <option value="effects">Effects</option>
</select>
~~~

Read the selected items from `selectedOptions`; `event.currentTarget.value` alone represents only one value and is not the full selection. When submitted as form data, a multiple select can produce several entries with the same name. Use `FormData.getAll('topics')` when you need every selected value.

## Individual checkboxes

A checkbox represents an independent true-or-false choice. Control it with a boolean `checked` prop, and read `event.currentTarget.checked` in its handler:

~~~jsx
const [sendUpdates, setSendUpdates] = useState(false);

<label>
  <input
    type="checkbox"
    name="sendUpdates"
    value="yes"
    checked={sendUpdates}
    onChange={(event) => setSendUpdates(event.currentTarget.checked)}
  />
  Send me updates
</label>
~~~

Here, `checked` controls whether the checkbox is selected. `value="yes"` is the string sent for that field when it is checked; it does not control the selection. An unchecked checkbox is omitted from form data.

## Checkbox groups

Use a checkbox group when users may choose any number of independent options. State can be an array of the selected option values:

~~~jsx
const [features, setFeatures] = useState([]);

function handleFeatureChange(event) {
  const { value, checked } = event.currentTarget;

  setFeatures((currentFeatures) => {
    if (checked) {
      return [...currentFeatures, value];
    }

    return currentFeatures.filter((feature) => feature !== value);
  });
}

<fieldset>
  <legend>Choose features</legend>

  <label>
    <input
      type="checkbox"
      name="features"
      value="sync"
      checked={features.includes('sync')}
      onChange={handleFeatureChange}
    />
    Sync across devices
  </label>

  <label>
    <input
      type="checkbox"
      name="features"
      value="reports"
      checked={features.includes('reports')}
      onChange={handleFeatureChange}
    />
    Usage reports
  </label>
</fieldset>
~~~

Each checkbox derives its own `checked` state by checking whether its value is in the array. The handler adds the changed option when it becomes checked and removes it when unchecked. A functional state update uses the latest array and avoids mutating it.

The shared `name` groups the options in form data. If you read the values from `FormData`, use `getAll('features')` to retrieve all checked values with that name.

## Radio groups

Use radio buttons when the user should choose one option from a group. Give every radio in the group the same `name`, give each a different `value`, and store the selected value as one string:

~~~jsx
const [delivery, setDelivery] = useState('standard');

<fieldset>
  <legend>Delivery speed</legend>

  <label>
    <input
      type="radio"
      name="delivery"
      value="standard"
      checked={delivery === 'standard'}
      onChange={(event) => setDelivery(event.currentTarget.value)}
    />
    Standard
  </label>

  <label>
    <input
      type="radio"
      name="delivery"
      value="express"
      checked={delivery === 'express'}
      onChange={(event) => setDelivery(event.currentTarget.value)}
    />
    Express
  </label>
</fieldset>
~~~

The selected radio has `checked={true}`; the other radio has `checked={false}`. The handler stores the selected radio's string `value`. A radio's `value` identifies the choice in form data, while `checked` represents whether that particular radio is selected.

Use `<fieldset>` and `<legend>` to give related controls a group label. A visible label for each option tells users what each choice means.

## Common attributes and accessibility

Use attributes that describe the field and its constraints:

| Attribute | Typical purpose |
| --- | --- |
| `id` | Connects a control to a label using `htmlFor`. |
| `name` | Identifies a field in form data and groups radio options. |
| `type` | Chooses the kind of input and its browser behavior. |
| `required` | Marks a value as required for native form validation. |
| `min`, `max`, `step` | Describe numeric limits and increments. |
| `minLength`, `maxLength` | Describe text length limits. |
| `autoComplete` | Gives the browser a hint about known information. |
| `disabled` | Makes a control unavailable for interaction and form submission. |
| `readOnly` | Prevents editing while leaving a text value focusable and included in form data. |

Always provide an accessible label. Placeholder text is only a hint and disappears as the user types. Use `fieldset` and `legend` for a related checkbox or radio group rather than relying on visual positioning alone.

Native constraints help browsers identify invalid values, but they do not replace clear instructions or server-side validation. Chapter 030 explains submission and browser validation.

## Choosing the right control

| User's choice | Suitable control | Typical React state |
| --- | --- | --- |
| Enter a short value | Text-like input | String |
| Enter a numeric value | Number input | Editable string; convert when needed |
| Choose one item from a list | Single select | String |
| Choose several items from a list | Multiple select | Array of strings |
| Toggle one independent option | Checkbox | Boolean |
| Choose one option from a small exclusive group | Radio group | One selected string |

Choose a select when a list is long or a compact control is useful. Choose radio buttons when seeing all mutually exclusive options helps the user compare them. Checkboxes are for choices that are independent rather than mutually exclusive.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Using `value` to determine whether a checkbox is checked | `value` is the submitted string, not the selection state. | Use boolean `checked`. |
| Reading `.value` to determine a checkbox's state | It does not tell you whether the box is selected. | Read `.checked`. |
| Giving radio options different `name` values | The browser no longer treats them as one exclusive group. | Use the same `name` and distinct `value` strings. |
| Giving every radio the same `value` | Form data cannot identify which option was selected. | Assign a distinct value to each choice. |
| Storing a radio group's state as several unrelated booleans | It permits contradictory states in application data. | Store one selected option string. |
| Mutating a checkbox array with `push` or `splice` | React may not see a new array reference, and old state is changed. | Return a new array with spread or `filter`. |
| Treating a multiple select as one string | A multiple select can have several selected options. | Store an array and read `selectedOptions`. |
| Putting `selected` on an `<option>` | React expects selection on the parent select. | Use `value` or `defaultValue` on `<select>`. |
| Converting an empty number field directly with `Number(value)` | The empty string becomes zero. | Preserve the string and check for `''` before converting. |
| Using a placeholder instead of a label | The hint disappears and does not reliably name the control. | Provide a visible associated label. |
| Assuming an unchecked checkbox submits an empty value | Unchecked checkboxes are omitted from form data. | Check the boolean state or account for the absent field. |

## Practice

1. Build a controlled email input with an associated label and `autoComplete`.
2. Add a number input whose state remains a string, including while it is empty.
3. Make a single select with a blank prompt and a required choice.
4. Make a multiple select and render all selected values.
5. Build a checkbox group where any number of options can be selected.
6. Build a radio group where exactly one delivery method is selected.
7. Inspect repeated checkbox and multiple-select fields using `FormData.getAll`.
8. Change a checkbox `value` while keeping `checked` the same and observe which part changes: selection or submitted data.

## Key takeaways

- Text-like and number input values are controlled with strings; convert numeric text only when needed.
- A single select uses one string; a multiple select uses an array of strings.
- A checkbox uses boolean `checked`; a checkbox's `value` is its submitted token.
- A checkbox group naturally maps to an array of selected values.
- A radio group uses the same `name`, distinct values, and one selected string in state.
- Labels, fieldsets, legends, and useful names make controls easier to use and understand.
- Browser input constraints are useful; form submission and validation are covered in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [028 - Controlled form elements](028-controlled-form-elements.md) | [Open Q&A](../qnas/029-inputs-selects-checkboxes-and-radio-buttons.md) | [Open Example](../examples/029-inputs-selects-checkboxes-and-radio-buttons.md) | [030 - Form submission and browser validation](030-form-submission-and-browser-validation.md) |
