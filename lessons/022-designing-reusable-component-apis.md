# 022. Designing reusable component APIs

## Learning goals

By the end of this lesson, you should be able to:

- Describe a component API as the contract between a component and its callers.
- Choose clear, minimal props and sensible defaults.
- Use `children` and named JSX props to support composition.
- Design controlled and uncontrolled component behavior deliberately.
- Name callback props so callers understand which interaction they handle.
- Avoid overloaded props and accidental DOM prop forwarding.

## A component API is a contract

A component API is the public way another part of an application asks a component to render or behave. In React, that contract is usually made up of props and nested JSX (`children`). It communicates which values the caller supplies, which interactions the component reports, and which parts of the UI the caller can customize.

For example, a call site should make a component's purpose apparent:

~~~jsx
<Notice tone="warning" title="Unsaved changes">
  Save your edits before leaving this page.
</Notice>
~~~

A reader can tell what kind of notice this is, what its heading says, and what content appears inside it. Compare that with a component that takes a dozen vaguely named flags and makes the caller guess which combinations are valid. Reuse is valuable when the API remains understandable; accepting every possible option is not the goal.

Before adding a prop, ask what decision it gives the caller and whether that decision belongs at this component boundary. Keep the API as small as the real use cases allow.

## Name props for their meaning

Choose names that describe the value or intent, not its implementation detail. `title`, `disabled`, `isOpen`, and `onOpenChange` are more informative than `text`, `flag`, `state`, or `callback`.

A useful API distinguishes values from actions:

~~~jsx
function SaveButton({ disabled = false, onSave, children = 'Save' }) {
  return (
    <button type="button" disabled={disabled} onClick={onSave}>
      {children}
    </button>
  );
}
~~~

`disabled` describes a condition; `onSave` describes an action. The component renders a native button, so it also preserves the browser's built-in button behavior. The caller decides what saving means:

~~~jsx
<SaveButton onSave={saveDraft}>Save draft</SaveButton>
~~~

Pass a function to an event prop. Do not call it while rendering: `onClick={onSave}` registers the handler, while `onClick={onSave()}` invokes it immediately.

Use names consistently within a codebase. Callback names often start with `on`, such as `onSelect`, `onDismiss`, or `onValueChange`. A callback should communicate the event or requested change, rather than expose how the parent stores data.

## Separate required values from optional defaults

Make essential information explicit and use defaults for optional choices with a clear fallback. Destructuring defaults keep those choices visible in the function signature:

~~~jsx
function Notice({ tone = 'info', title, children }) {
  return (
    <aside className={`notice notice--${tone}`}>
      {title && <h2>{title}</h2>}
      <div>{children}</div>
    </aside>
  );
}
~~~

Here, `tone` has a useful default, while `title` and `children` are optional in the implementation. If a title is required by the design, make that requirement clear at the call site and in the API's documentation rather than silently inventing one.

A destructuring default is used when a prop is omitted or has the value `undefined`. It does not replace an explicit `null`, `false`, `0`, or empty string. Choose defaults that match the meaning of the prop; do not use truthiness as a substitute for checking whether a value was supplied.

Avoid sending a value both as a prop and as duplicated child content unless the two have different, clearly named roles. Duplicate inputs can disagree and make it unclear which one controls the display.

## Use children for natural composition

Use `children` when callers should provide the main content nested between a component's tags. It can hold text, one element, or a larger JSX tree:

~~~jsx
<Notice title="Profile updated">
  <p>Your changes are saved.</p>
  <a href="/profile">View profile</a>
</Notice>
~~~

When a component has multiple distinct regions, named props can make the call site easier to scan. Those props can receive JSX just like `children`:

~~~jsx
function Panel({ heading, actions, children }) {
  return (
    <section className="panel">
      <header className="panel__header">
        <h2>{heading}</h2>
        <div className="panel__actions">{actions}</div>
      </header>
      <div className="panel__body">{children}</div>
    </section>
  );
}
~~~

~~~jsx
<Panel
  heading="Account"
  actions={<button type="button">Edit</button>}
>
  <p>Manage your profile details.</p>
</Panel>
~~~

The `actions` prop is a named slot: it tells the caller which region it fills. Prefer a small number of meaningful named regions over a long list of booleans that turn markup on and off. Choose semantic HTML elements that fit the content and interaction; CSS class names do not provide semantics by themselves.

## Decide who owns changing data

A reusable component needs a clear answer to the question: who owns its changing value?

### Controlled behavior

A controlled component receives its current value from its parent and reports user requests through a callback. The parent remains the source of truth. A disclosure might expose `isOpen` and `onOpenChange`:

~~~jsx
function Disclosure({ title, isOpen, onOpenChange, children }) {
  return (
    <section>
      <h2>{title}</h2>
      <button
        type="button"
        aria-expanded={isOpen}
        onClick={() => onOpenChange(!isOpen)}
      >
        {isOpen ? 'Hide details' : 'Show details'}
      </button>
      {isOpen && <div>{children}</div>}
    </section>
  );
}
~~~

The component doesn't decide whether the value is stored in local state, shared with another component, or restored from a URL. Its caller provides the value and decides what to do when the user asks to change it. The button's `aria-expanded` value reflects the actual state.

Controlled behavior is useful when a parent must coordinate multiple components, enforce rules, or keep the value in a larger form. For native form inputs, React's `value` or `checked` prop similarly makes the displayed value controlled; provide an `onChange` handler that updates the supplied value in response to user input.

### Uncontrolled behavior

An uncontrolled component owns its current value internally and may allow an initial value such as `defaultOpen`. This can make a self-contained interaction easy to use when callers do not need to coordinate every update. For example, a native input's `defaultValue` provides an initial value without making the parent responsible for each keystroke.

The word “default” matters: an initial value is not the same contract as a value that is expected to stay synchronized with changing parent props. Do not initialize local state from a prop and then ignore later prop changes while claiming the component is controlled.

Controlled and uncontrolled are common design terms, not special React APIs. Pick one clear contract for a component. If an API intentionally supports both modes, define the rule for selecting the mode and avoid letting both the value prop and internal state compete. For a simple reusable component, offering one predictable mode is often easier to maintain.

## Keep component variations understandable

A few meaningful variations can make a component useful. Too many independent boolean props can create combinations no one intended:

~~~jsx
<Dialog isModal isCompact isDangerous isCentered hasFooter />
~~~

Ask whether these are truly independent choices. If a set of values represents one choice, use a named prop such as `variant="danger"` or `size="compact"`. If two modes have substantially different structure or behavior, composition or separate components can be clearer than a growing matrix of flags.

Do not add a prop just because it might be useful someday. Start with demonstrated needs, then expand the API when a real use case appears. An API should support its intended use cases without exposing internal details that callers should not depend on.

## Forward props with care

A wrapper may need to expose selected native element props such as `id`, `disabled`, `className`, or accessibility attributes. Decide which element each prop belongs to and forward it deliberately. Blindly spreading every prop onto a `<div>` can put attributes on the wrong element, leak component-only props into the DOM, or override behavior the wrapper relies on.

When a component is intentionally a thin wrapper around one native element, forwarding remaining props can be convenient, but reserve and handle the wrapper's own props first. Keep the supported contract documented and preserve native semantics. For example, a button wrapper should render a `<button>` and should not turn a decorative `<div>` into a pretend button when native button behavior is needed.

## Review an API from the call site

Read the JSX as another developer would. Check that:

1. The component's purpose is clear from its name.
2. Required information is obvious, and optional values have sensible defaults.
3. Values and callbacks have specific, consistent names.
4. Nested content and named regions have an understandable structure.
5. A changing value has one clear owner.
6. The rendered HTML remains semantic and accessible.
7. Unsupported or contradictory prop combinations are hard to express.

Small call-site examples are often the quickest way to spot an API that is difficult to use. If callers need comments to explain ordinary combinations, simplify the contract.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Adding a prop for every possible styling tweak | The public contract grows faster than real needs. | Start with demonstrated use cases and a small set of meaningful variants. |
| Naming several unrelated callbacks `callback` or `handler` | Callers cannot tell which action will run. | Name callbacks for the event or requested change, such as `onDismiss`. |
| Passing the same content through both `title` and `children` | The values can conflict and the source of display becomes unclear. | Give each input one role or remove the duplicate. |
| Mixing a controlled value with private state that overrides it | The parent and child disagree about the current value. | Choose a single owner and pair controlled values with change callbacks. |
| Treating `defaultValue` as a value that stays synchronized | “Default” describes initialization, not ongoing control. | Use a controlled value when the parent must keep it synchronized. |
| Adding many interacting boolean props | Invalid or confusing combinations become easy to create. | Use a named variant, composition, or focused components. |
| Spreading all props onto an arbitrary DOM node | Component-only props or behavior can reach the wrong element. | Forward only supported props to the element they describe. |

## Key takeaways

- A component API is a contract made from props, nested content, callbacks, and rendered behavior.
- Use specific names, clear required inputs, and defaults that match the value's meaning.
- Use `children` for primary nested content and named JSX props for distinct regions.
- A controlled value comes from the parent and is paired with a callback; an uncontrolled component owns its current value.
- Avoid duplicated sources of truth, speculative props, and combinations of flags that callers cannot reason about.
- Forward props intentionally and preserve the semantics and accessibility of the underlying HTML.
- Review the API at its call site before expanding it.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [021 - Pure components and predictable rendering](021-pure-components-and-predictable-rendering.md) | [Open Q&A](../qnas/022-designing-reusable-component-apis.md) | [Open Example](../examples/022-designing-reusable-component-apis.md) | [023 - Thinking in React: from a design to a component tree](023-thinking-in-react-and-component-trees.md) |