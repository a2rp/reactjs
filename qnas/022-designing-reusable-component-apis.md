# 022. Designing reusable component APIs - Q&A

## Component API basics

### 1. What is a component API?

It is the contract callers use to configure and compose a component. In React, that usually includes its props, nested `children`, callbacks, and the rendered behavior callers can rely on.

### 2. Does reusable mean a component needs many props?

No. Reuse means the component supports the real situations it is meant for. A small, clear API is often easier to reuse than one with many speculative options.

### 3. How do I decide whether to add a prop?

Identify the caller's real decision. Add a prop when callers need to control that decision at this component boundary and the prop can be named and explained clearly. Otherwise, keep the choice internal or let callers compose a different child.

### 4. What makes a prop name clear?

It describes the value or intent. Names such as `title`, `disabled`, `isOpen`, and `onDismiss` tell the caller more than generic names such as `text`, `flag`, or `handler`.

### 5. What is the difference between a value prop and a callback prop?

A value prop describes data or the current condition, such as `isOpen`. A callback prop gives the component a function to call for an interaction or requested change, such as `onOpenChange`.

### 6. Why do callbacks commonly start with `on`?

It signals that the prop is a function associated with an event or action. Consistent names such as `onSave`, `onSelect`, and `onValueChange` help callers understand when it is used.

### 7. Should I call a callback while rendering JSX?

Usually no. Pass the function itself to the event prop, such as `onClick={onSave}`. Writing `onClick={onSave()}` calls it immediately during rendering instead of waiting for a click.

## Defaults and composition

### 8. What does a destructuring default value do?

It supplies a value when the prop is omitted or its value is `undefined`:

~~~jsx
function Notice({ tone = 'info' }) {
  return <aside className={`notice notice--${tone}`} />;
}
~~~

### 9. Does a destructuring default replace `null`, `false`, `0`, or an empty string?

No. Destructuring defaults apply to missing or `undefined` values. An explicit `null`, `false`, `0`, or `''` remains the value passed by the caller.

### 10. When should a prop be required?

When the component cannot do its intended job without that information. Make the requirement apparent in its call sites and documentation. Do not hide a missing essential value behind an arbitrary fallback.

### 11. When should I use `children`?

Use it for the component's primary nested content or when the caller should compose arbitrary JSX inside it. It can contain text, one element, or several elements.

### 12. When should I use a named JSX prop instead of `children`?

Use a named prop when the component has a distinct region with a specific purpose, such as `actions`, `footer`, or `emptyState`. The prop can still receive JSX.

### 13. Is a named JSX prop a special React feature?

No. JSX elements are values, so they can be passed through ordinary props. The name simply documents which region or role the supplied content fills.

### 14. Why is it confusing to pass the same content as both `title` and `children`?

The values can disagree, and callers may not know which one is displayed or authoritative. Give each prop a distinct purpose or remove the duplicate input.

### 15. Should a reusable component accept arbitrary children everywhere?

Only where that composition makes sense. Define the component's intended regions and structure so callers know where content belongs and the resulting HTML remains meaningful.

## Controlled and uncontrolled behavior

### 16. What is a controlled component?

Its parent supplies the current value and receives a callback when the user requests a change. The parent owns the source of truth; the component renders the value it was given.

### 17. What is an uncontrolled component?

It owns its current value internally. The caller may provide an initial value, often through a prop named with `default`, without controlling every later update.

### 18. Are “controlled” and “uncontrolled” special React APIs?

No. They are common terms for describing who owns a value. The behavior comes from the component's contract, not a special keyword.

### 19. What does `defaultValue` mean for an input?

It provides an initial value for an uncontrolled input. It does not mean the parent can keep changing the input's current value by updating `defaultValue` on later renders.

### 20. How should a custom component expose controlled state?

Expose a current-value prop and a callback describing requested changes. For example, `isOpen` can be paired with `onOpenChange`. The callback reports the interaction; the parent decides whether and how to update the value.

### 21. What is wrong with initializing internal state from a prop and then ignoring that prop?

The component appears controlled at its call site but can stop reflecting the parent's later values. The parent and child then act as competing sources of truth. Choose a clear contract, or document the initial-only meaning with a `default` prop.

### 22. Can one component support both controlled and uncontrolled modes?

Yes, if there is a real need and the API precisely defines how callers select the mode and which value is authoritative. Supporting both adds complexity, so a single predictable mode is often a better starting point.

### 23. Why is a callback paired with a controlled value?

The value tells the component what to display. The callback lets the component report that a user requested a different value. Without a callback or another documented path, the caller may have no way to respond to the interaction.

## API size, DOM behavior, and accessibility

### 24. Why can many boolean props become hard to maintain?

Independent flags create combinations that may be invalid, inconsistent, or unclear. If the caller is choosing one mode, use a named variant. If the structures are different, composition or separate components can express the distinction better.

### 25. Should I add a prop because a future caller might want it?

Usually not. Start from demonstrated use cases. Add a stable option when a real need appears instead of making the public contract larger based on speculation.

### 26. Is spreading every prop onto a DOM element a good default?

No. Component-only props may leak into the DOM, and attributes can land on an element they do not describe. Forward supported props deliberately to the appropriate native element.

### 27. When is forwarding remaining props useful?

It can be useful when a component is intentionally a thin wrapper around one native element and its contract says which standard attributes it accepts. Reserve the component's own props first, forward to the correct element, and avoid overriding behavior unexpectedly.

### 28. How should a component API support accessibility?

Render appropriate semantic HTML and expose the information or behavior callers need to provide, such as a visible label, an accessible name, a disabled state, or a controlled disclosure value. Don't rely on styling or generic containers to create native semantics.

### 29. Does a CSS variant such as `tone="warning"` replace semantic markup?

No. A visual variant changes presentation. The component still needs suitable HTML and, when needed, accessible text or attributes to communicate its purpose and state.

### 30. What is a quick way to review a component API?

Read a few realistic call sites. Check that the purpose, required values, optional defaults, content regions, and interaction callbacks are obvious, and that the values have one clear owner.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [021 - Pure components and predictable rendering](../lessons/021-pure-components-and-predictable-rendering.md) | [022 - Designing reusable component APIs](../lessons/022-designing-reusable-component-apis.md) | [Open Example](../examples/022-designing-reusable-component-apis.md) | [023 - Thinking in React: from a design to a component tree](../lessons/023-thinking-in-react-and-component-trees.md) |