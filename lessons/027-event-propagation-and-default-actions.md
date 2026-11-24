# 027. Event propagation, default actions, and cancellation

## Learning goals

By the end of this lesson, you should be able to:

- Predict how an event travels through nested React elements.
- Use capture and bubbling handlers intentionally.
- Stop an event from reaching ancestor handlers when the interaction should remain local.
- Prevent a browser default action when React is taking over that behavior.
- Explain why propagation and default cancellation are separate.
- Avoid unnecessary propagation tricks by using explicit callbacks and appropriate HTML.

## Events can travel through a component tree

A click starts on the element the user activates. The event can then be handled by ancestors as it travels through the tree. This upward travel is called bubbling or propagation.

~~~jsx
function Toolbar() {
  return (
    <div onClick={() => console.log('Toolbar handler')}>
      <button type="button" onClick={() => console.log('Button handler')}>
        Play
      </button>
    </div>
  );
}
~~~

Clicking Play runs the button's `onClick`, then the ancestor `<div>`'s `onClick`. Clicking elsewhere inside the `<div>` runs only its handler. A parent handler can therefore observe events from its descendants, which can be useful for delegated behavior or logging.

With nested elements, consider both the visual target and the React tree. An event can originate on a child element, run that child's handler, and then continue to handlers attached to its ancestors.

## Capture runs before the target handler

For many events, React supports capture-phase event props by appending `Capture` to the event name, such as `onClickCapture`. For a nested click, the simplified order is:

1. Capture handlers run from ancestors toward the target.
2. The target element's regular handler runs.
3. Bubble handlers run from the target back toward ancestors.

~~~jsx
<article
  onClickCapture={() => console.log('Parent capture')}
  onClick={() => console.log('Parent bubble')}
>
  <button type="button" onClick={() => console.log('Button')}>Open</button>
</article>
~~~

Clicking the button logs `Parent capture`, `Button`, then `Parent bubble`. A capture handler is useful when an ancestor needs to observe an interaction before descendants handle it, for example, for broad interaction logging. Most application behavior can be handled more directly at the component that owns the action.

Capture support and bubbling behavior can differ for particular event types. For example, React's `onScroll` does not bubble like `onClick`, and pointer/mouse enter and leave events have special behavior. Check the specific event rather than assuming every event follows the same path.

## Stop propagation when the ancestor should not handle the action

Call `event.stopPropagation()` to prevent the event from continuing to ancestor handlers:

~~~jsx
function DeleteButton({ onDelete }) {
  function handleClick(event) {
    event.stopPropagation();
    onDelete();
  }

  return (
    <button type="button" onClick={handleClick}>
      Delete
    </button>
  );
}
~~~

This can be appropriate when a child action is separate from its enclosing click behavior, such as a delete control inside a selectable row. The child handler still runs. Capture handlers that already ran are not undone, and stopping propagation does not cancel a browser default action.

Use `stopPropagation()` only when the event should not mean the same thing to an ancestor. If it is difficult to understand which handlers will run, consider passing an explicit callback instead of relying on bubbling to connect unrelated component behavior.

## Prevent the browser's default action when needed

Some interactions have a browser action associated with them. A link follows its `href`; a form normally submits and may navigate or reload the page. Call `event.preventDefault()` when the application intentionally handles that interaction itself:

~~~jsx
function SearchForm({ onSearch }) {
  function handleSubmit(event) {
    event.preventDefault();
    const formData = new FormData(event.currentTarget);
    onSearch(String(formData.get('query') ?? ''));
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Search
        <input name="query" />
      </label>
      <button type="submit">Search</button>
    </form>
  );
}
~~~

Without `preventDefault()`, the browser performs the form's normal submission behavior. With it, the handler can read the form and ask the application to display results without leaving the page.

Do not cancel a default action just because a handler exists. Let a link navigate when it is a real destination, and let a form submit normally when that is the intended behavior. Returning `false` from a React handler does not cancel the default; call `preventDefault()` explicitly.

## Keep the two methods separate

| Method | What it changes | What it does not do |
| --- | --- | --- |
| `event.stopPropagation()` | Stops this event from continuing to ancestor handlers. | Does not prevent a link, form, or other browser default action. |
| `event.preventDefault()` | Cancels the event's cancelable browser default action. | Does not stop ancestor handlers from running. |

Both can be called for one event if the interaction requires both behaviors, but do not combine them automatically. Identify the specific behavior you need to suppress.

## Prefer explicit intent over accidental bubbling

A clickable card with buttons inside can create ambiguous interactions. If clicking the card selects it, while one button opens details and another deletes it, define those behaviors deliberately. A button's event bubbles unless it is stopped, so clicking a child button may also trigger the card selection.

Possible approaches include:

- Keep the parent handler and stop propagation from a child action that must remain separate.
- Put the selection behavior on a dedicated button or link rather than the entire container.
- Pass an explicit callback to child controls so the action path is visible.
- Avoid nesting interactive controls inside other interactive controls. For example, do not place a button inside another button or a link inside another link.

Native semantics reduce surprises. Use links for navigation and buttons for actions; do not make a generic container clickable just to capture an event.

## React tree and portals

A portal places DOM nodes in a different physical location, but its children remain part of the same React tree. Events from a portal propagate according to the React tree, not solely according to the DOM nesting. A click inside a modal portal can therefore reach a React ancestor that appears unrelated in the browser's DOM inspector. Stop propagation in the portal or adjust the React tree when that behavior is not intended.

## A practical debugging sequence

When an unexpected ancestor handler runs:

1. Identify the element where the event originated.
2. Trace its React ancestors that have a matching event handler.
3. Check whether a capture handler ran before the target handler.
4. Decide whether the parent should observe this interaction.
5. Use `stopPropagation()` only if the event should not continue to that parent.
6. Separately check whether the browser default should happen; use `preventDefault()` only when replacing it.

Log the handler order while learning, but remove noisy debugging logs from production code when they are no longer useful.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming a child click only runs the child's handler | Bubbling can also run ancestor handlers. | Trace the event path and decide which ancestors should respond. |
| Calling `preventDefault()` to stop a parent handler | It cancels the browser default, not propagation. | Use `stopPropagation()` if the event must not reach ancestors. |
| Calling `stopPropagation()` to prevent navigation or submission | It does not cancel the browser default action. | Use `preventDefault()` when the app replaces that default. |
| Stopping every event by default | It hides useful parent behavior and makes event flow harder to follow. | Stop propagation only for a specific interaction boundary. |
| Relying on `return false` to cancel an event | React does not use the return value for cancellation. | Call `preventDefault()` explicitly. |
| Putting click handlers on nested interactive controls | Nested buttons or links have conflicting semantics and behavior. | Restructure the markup so interactive controls are siblings or use a single clear control. |
| Assuming a portal follows DOM bubbling | Portal events follow the React tree. | Consider React ancestry when debugging a portal interaction. |
| Assuming every event bubbles identically | Some event types have special propagation behavior. | Check the behavior of the specific React event prop. |

## Key takeaways

- Bubbling lets an event reach ancestor handlers; capture handlers run before the target handler.
- `stopPropagation()` affects the event path, while `preventDefault()` affects the browser's default action.
- Neither method substitutes for the other.
- Stop propagation only when a nested interaction should not be handled by its ancestor.
- Use explicit callbacks and semantic links/buttons to make interaction intent easier to follow.
- Portal events follow the React tree, and some event types have special propagation behavior.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [026 - Event objects and React's event system](026-event-objects-and-react-event-system.md) | [Open Q&A](../qnas/027-event-propagation-and-default-actions.md) | [Open Example](../examples/027-event-propagation-and-default-actions.md) | [028 - Controlled form elements](028-controlled-form-elements.md) |