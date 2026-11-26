# 030. Form submission and browser validation

## Goal

A form groups related controls and gives the browser a standard way to validate and submit their values. React can let a form submit normally, handle its submit event, or-starting with React 19-use a function as its `action`.

This chapter covers the browser and React basics: when submission happens, how native constraints affect it, and how to read named controls with `FormData`. Accessible custom error messages are the next chapter; asynchronous React form Actions are covered later in the course.

## Learning outcomes

After this chapter, you should be able to:

- Explain what happens when a form is submitted.
- Choose between normal browser submission and handling `onSubmit` in React.
- Prevent a page navigation when the client handles the submit event.
- Read named control values through `FormData`.
- Use common native constraints such as `required`, input types, and length or numeric limits.
- Explain how constraint validation changes the submit flow.
- Recognize the React 19 form `action` API and find the later Actions lessons.

## What happens when a form is submitted?

A user can submit a form by activating a submit button or, in many cases, pressing Enter in a field. The browser performs interactive constraint validation first when it is enabled. If a control is invalid, the browser reports the problem and the form's submit event does not run. If validation passes, the browser dispatches a submit event.

Without a client-side handler or a function action, the browser sends the form's named values to the destination in `action` and follows the normal navigation behavior. A form's default method is `GET`; a `POST` method sends the form data in the request body.

A basic semantic form looks like this:

~~~jsx
<form action="/contact" method="post">
  <label htmlFor="contact-email">Email</label>
  <input id="contact-email" name="email" type="email" required />

  <button type="submit">Send request</button>
</form>
~~~

Use a real `<form>` for a group of controls that submits together. The browser supports keyboard submission, native validation, and form data serialization.

## Handling submission with `onSubmit`

Attach `onSubmit` to the form, not just to a button. This handles submission by clicking a submit button and submission initiated from a field:

~~~jsx
function SearchForm() {
  function handleSubmit(event) {
    event.preventDefault();

    const formData = new FormData(event.currentTarget);
    const query = String(formData.get('query') ?? '');

    console.log(query);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="search-query">Search</label>
      <input id="search-query" name="query" type="search" required />
      <button type="submit">Search</button>
    </form>
  );
}
~~~

By default, a successful submit would make the browser navigate or reload as specified by the form. `event.preventDefault()` cancels that default action so the handler can process the data in the current page. It does not stop event propagation.

React's `onSubmit` event handler works across React versions. If an input is controlled by React state, you can read that state directly in the handler. If the values are in the DOM, `FormData` is a convenient way to collect named controls.

## Reading values with `FormData`

Construct `FormData` from the submitted form:

~~~jsx
function handleSubmit(event) {
  event.preventDefault();

  const form = event.currentTarget;
  const formData = new FormData(form);

  const email = formData.get('email');
  const topics = formData.getAll('topics');

  console.log({ email, topics });
}
~~~

A control needs a `name` to be included under a useful key. `FormData.get(name)` returns one entry; `FormData.getAll(name)` returns all entries for a repeated name, such as a multiple select or checked checkbox group.

Form data follows browser form rules:

- Unchecked checkboxes and radio buttons are omitted.
- Disabled controls are omitted.
- Controls without a `name` are omitted.
- A multiple select and same-name checked boxes can contribute repeated entries.
- Only the activated submit button contributes its own `name` and `value`.

Do not assume every returned value is a string. A file control can produce a `File` entry; file inputs and uploads are covered in chapter 034.

## Native constraint validation

HTML controls have built-in validation constraints. Use them where they accurately express what the user needs to enter:

| Constraint | Common use |
| --- | --- |
| `required` | The user must provide a value or make a selection. |
| `type="email"` or `type="url"` | Apply the browser's basic format check for that type. |
| `min` and `max` | Set numeric or date limits. |
| `step` | Set valid numeric increments. |
| `minLength` and `maxLength` | Set text length limits. |
| `pattern` | Require text to match a pattern. |

For example:

~~~jsx
<form onSubmit={handleSubmit}>
  <label htmlFor="signup-email">Email</label>
  <input
    id="signup-email"
    name="email"
    type="email"
    autoComplete="email"
    required
  />

  <label htmlFor="years">Years of experience</label>
  <input id="years" name="years" type="number" min="0" max="50" step="1" />

  <button type="submit">Continue</button>
</form>
~~~

With interactive validation enabled, the browser checks the constraints before it dispatches the form's submit event. An invalid form therefore does not reach this example's `handleSubmit`. The browser can focus the invalid control and display its native validation message.

The `type="email"` constraint checks that a value has an acceptable email-like format. It cannot prove that the address exists or belongs to the user.

### Turning off or checking native validation

The `noValidate` attribute on a form disables the browser's automatic interactive validation for that form. A submit button can use `formNoValidate` to bypass validation for that submission, such as a separate “Save draft” action.

The Constraint Validation API provides methods for code that needs to inspect validity:

- `form.checkValidity()` returns whether the associated controls pass their constraints.
- `form.reportValidity()` checks them and asks the browser to report invalid controls.
- `control.setCustomValidity(message)` marks a control invalid with a custom message; pass an empty string to clear that custom error.

Native constraints and these APIs are client-side checks. They improve input and feedback, but they do not establish that submitted data is trustworthy. Validate data again on the server.

Chapter 031 covers how to present validation errors and summaries accessibly. For cross-field rules and more involved business logic, use application validation in addition to the native constraints.

## Buttons inside forms

A button inside a form submits the form by default. Set its type explicitly so each action is clear:

~~~jsx
<button type="submit">Send</button>
<button type="button" onClick={clearDraft}>Clear draft</button>
<button type="reset">Reset fields</button>
~~~

Use `type="submit"` for the form's submit action, `type="button"` for an unrelated action, and `type="reset"` when the browser should reset uncontrolled controls to their initial values. A reset button does not automatically reset React state for controlled fields; update that state yourself when the form includes controlled values.

## React 19 form `action` overview

React 19 adds support for passing a function to a form's `action` prop. React calls that function with a `FormData` object and manages the action as a Transition. When the action succeeds, React resets uncontrolled fields. This is a newer submission model; this chapter's `onSubmit` examples show the manual event-handler approach that works across React versions.

The later chapters cover Actions, pending state, and action state in detail:

- [React form Actions and asynchronous submissions](078-react-form-actions.md)
- [Managing submissions with useActionState](079-useactionstate.md)
- [Pending form state with useFormStatus](080-useformstatus.md)

Do not pass a function to `action` and also call `preventDefault()` in `onSubmit` unless you intentionally want to cancel the form action. A form can use either model, but the APIs have different submission and reset behavior.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Adding `onClick` only to the submit button | Enter-key submission and other submit buttons can bypass that button-specific logic. | Handle the form's `onSubmit` event. |
| Forgetting `event.preventDefault()` in a client-side submit handler | The browser performs the normal navigation or reload after the handler. | Cancel the default when the client is taking over. |
| Calling `preventDefault()` before native validation can run | Preventing submission yourself can bypass the browser's normal submit flow. | Let native constraints run, or intentionally take responsibility for validation. |
| Leaving a button's type implicit | A button intended for another action may submit its form. | Set `type="button"` or `type="submit"` explicitly. |
| Expecting `FormData` to include unnamed controls | Form data is keyed by control names. | Give submitted controls meaningful `name` attributes. |
| Expecting an unchecked checkbox to produce false | Unchecked boxes are omitted from form data. | Treat absence as unchecked or use controlled boolean state. |
| Calling `get()` for a repeated field and expecting every choice | `get()` returns one entry for the name. | Use `getAll()` for repeated values. |
| Trusting browser validation as server security | Users can bypass or alter client-side checks. | Validate and authorize submitted data on the server. |
| Treating a valid-looking email as verified | A format check cannot prove deliverability or ownership. | Verify the address through an application flow when needed. |
| Using `type="reset"` without considering controlled state | The browser reset does not reset React's state value. | Reset both the relevant React state and any uncontrolled fields. |
| Combining a function `action` with an unconditional `preventDefault()` | The form action will not run if the submit event is canceled. | Choose the submission model and allow its intended flow. |

## Practice

1. Create a form with labeled name and email fields and a submit button.
2. Add `required` and `type="email"`, then try submitting empty and malformed values.
3. Add a number field with `min`, `max`, and `step`.
4. Handle `onSubmit` on the form, call `preventDefault()`, and inspect the submitted data with `FormData`.
5. Add two same-name checkboxes and read all checked values with `FormData.getAll()`.
6. Add a `type="button"` action and confirm it does not submit the form.
7. Compare normal browser navigation with a React handler that prevents the default.
8. Add a controlled field and implement a reset action that updates both its React state and any other values you want reset.

## Key takeaways

- A browser validates eligible controls before dispatching submit when interactive validation is enabled.
- Handle a React-managed submission on the form's `onSubmit` and call `preventDefault()` when replacing the browser navigation.
- `FormData` reads named controls; use `getAll()` for repeated values.
- Native validation is useful feedback, not a substitute for server-side validation.
- A submit button submits; set other button types explicitly.
- React 19 function form Actions are a separate model, covered in later chapters.
- Accessible validation messages and summaries are covered next.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="029-inputs-selects-checkboxes-and-radio-buttons.md">029 - Text inputs, selects, checkboxes, and radio buttons</a></td><td><a href="../qnas/030-form-submission-and-browser-validation.md">Q&amp;A</a></td><td><a href="../examples/030-form-submission-and-browser-validation.md">Example</a></td><td><a href="031-accessible-validation-messages.md">031 - Accessible validation messages and error summaries</a></td></tr>
  </tbody>
</table>
