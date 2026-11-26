# Lifting State Up and Sharing Data

## Why this lesson matters

A component's state is local to the component that declares it. That is useful when a value belongs to one component, such as whether its own menu is open.

Sometimes sibling components need to coordinate. A text editor and preview must show the same text. Several accordion panels may follow a rule such as “only one panel can be open at a time.” If each child keeps a separate copy of the relevant state, those components cannot reliably stay in sync.

The usual solution is to move shared state to the closest component that contains all the components that need it. This is called **lifting state up**. The parent passes the current value down through props and passes event-handler props back down so children can request changes.

## Learning goals

After this lesson, you should be able to:

- recognize when sibling components need shared state;
- find the closest common parent that can coordinate them;
- move the state and its update logic into that parent;
- pass current values and event handlers to children as props;
- choose a state shape that represents a coordination rule;
- distinguish parent-controlled state from independent child state;
- avoid duplicated state and unnecessary lifting.

## Start with independent state

Imagine an accordion with two panels. If each panel owns its own isOpen state, it can open and close independently:

~~~jsx
import { useState } from "react";

function Panel({ title, children }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <section>
      <h2>{title}</h2>
      <button onClick={() => setIsOpen((open) => !open)}>
        {isOpen ? "Hide" : "Show"} {title}
      </button>
      {isOpen && <div>{children}</div>}
    </section>
  );
}
~~~

This design is appropriate when each panel is truly independent. It does not satisfy a different requirement: **at most one panel may be open at a time**. One panel cannot update another panel's private state.

The coordination rule tells us that the open-panel state belongs above both panels.

## Find the state owner

For sibling components, look upward in the component tree until you find the nearest component that contains them both. That component is their **closest common parent**.

Move the shared state there:

1. Put the state in the closest common parent.
2. Pass each child the value it needs as a prop.
3. Pass a callback prop so a child can report a user action.
4. Let the parent update its state in response.
5. Render the children again with the updated props.

The data flow is one way:

~~~text
parent state
    └── current value flows down through props
child event
    └── callback prop reports the requested change to the parent
parent updates state
    └── new props flow down
~~~

A child does not reach into a sibling or directly change its parent's state. It calls a function supplied by the parent.

## Controlled children and parent-owned state

A child is often called **controlled** when its important value comes from props and its interaction requests changes through a callback. The parent decides what value the child receives.

For the accordion, one parent state value can represent the rule:

~~~jsx
const [activePanelId, setActivePanelId] = useState("overview");
~~~

Here, activePanelId is either the ID of the open panel or null when no panel is open. Each panel receives a boolean derived from that value:

~~~jsx
const isOpen = activePanelId === panel.id;
~~~

When a panel is activated, the parent changes the active ID:

~~~jsx
function handlePanelToggle(panelId) {
  setActivePanelId((currentId) =>
    currentId === panelId ? null : panelId
  );
}
~~~

This state shape makes it impossible for two panels to be open at once. Two separate booleans would allow a state the product rule forbids:

~~~js
// This can represent both panels as open at the same time.
{ overviewIsOpen: true, detailsIsOpen: true }
~~~

Choose state that naturally expresses the valid choices. If there can be only one active item, a single ID (or null) is usually clearer than one boolean per item.

## Share a value between sibling components

Lifting state is also useful when siblings display or edit the same value. For example, an editor and preview should use one message state:

~~~jsx
function App() {
  const [message, setMessage] = useState("A shared message");

  return (
    <>
      <MessageEditor
        message={message}
        onMessageChange={setMessage}
      />
      <MessagePreview message={message} />
    </>
  );
}
~~~

The editor receives the current message and a callback. The preview receives that same current message. When the editor reports a new value, the parent updates state and both children receive the new prop.

~~~jsx
function MessageEditor({ message, onMessageChange }) {
  return (
    <label>
      Message
      <textarea
        value={message}
        onChange={(event) => onMessageChange(event.target.value)}
      />
    </label>
  );
}

function MessagePreview({ message }) {
  return <p>{message || "Your preview will appear here."}</p>;
}
~~~

This is a controlled input: React state is the source of the displayed value, and onChange requests an update. The same principle applies to selection controls, filters, tabs, and other shared UI state.

## Keep state at the nearest useful level

Lift state only as far as needed to coordinate its consumers.

- If one child uses a value and no sibling needs it, keep it in that child.
- If sibling components need the same value, their closest common parent is a good owner.
- If multiple descendants in separate branches need a value, identify their nearest common parent.
- If a value must be shared across distant parts of a large tree, prop passing may become cumbersome. Context or another state-management approach may be appropriate; those solve a broader distribution problem.

Putting every value at the application root creates unnecessary coupling and makes state ownership harder to identify. Start with the closest common parent and move state higher only when the actual consumers require that.

Moving state can also change which components render after an update. First choose a clear and correct owner. If profiling later shows that a particular update is expensive, optimize the relevant boundary rather than keeping confusing duplicated state.

## Design a clear child API

A parent should pass only what a child needs to render and report interactions. For example:

~~~jsx
<Panel
  title="Overview"
  isOpen={activePanelId === "overview"}
  onToggle={() => handlePanelToggle("overview")}
>
  Overview content
</Panel>
~~~

The panel can render isOpen and call onToggle. It does not need to know how the parent stores the active ID.

Naming callbacks after the event or request can make the API clear:

- onChange for a changed value;
- onSelect for a requested selection;
- onToggle for a toggle interaction;
- onOpen when the child only requests opening.

The callback can pass a value or identifier when the parent needs it:

~~~jsx
onSelect(item.id)
~~~

The child reports what happened; the parent decides how that affects state.

## Controlled and uncontrolled are design choices

An **uncontrolled** component owns its own important state. This is convenient when the component can operate independently.

A **controlled** component receives its important value from a parent and reports requested changes through callbacks. This is useful when a parent must coordinate siblings or enforce a rule.

These terms describe common designs, not rigid categories for every component. A component can receive one value as a prop while keeping unrelated presentation state locally. Choose ownership separately for each piece of state.

For example, a reusable search field may receive its text as value and report changes through onChange, while keeping a local flag for whether its help tooltip is expanded. The text is controlled; the tooltip can remain local.

## Avoid copying props into duplicate state

This pattern usually causes synchronization problems:

~~~jsx
function Panel({ isOpen }) {
  const [open, setOpen] = useState(isOpen);
  // ...
}
~~~

useState(isOpen) uses the prop as the initial value. It does not automatically keep open synchronized whenever the parent later passes a different isOpen. Now there are two competing values.

If the parent owns the open state, render directly from the prop:

~~~jsx
function Panel({ isOpen }) {
  return isOpen ? <div>Panel content</div> : null;
}
~~~

Keep local state only for a separate fact that the parent does not need to own.

## Avoid redundant derived state

If isOpen can be calculated from activePanelId and a panel's ID, do not store both values in state:

~~~js
const isOpen = activePanelId === panelId;
~~~

The boolean is derived from the source of truth. Storing both would require keeping them synchronized manually.

The same applies to a preview or character count that can be calculated from a shared string. Store the message; derive its length when rendering:

~~~jsx
<p>{message.length} characters</p>
~~~

## A practical decision checklist

Before adding state, ask:

1. What exact information changes?
2. Which components need to read it?
3. Which component handles the user action that requests a change?
4. What is the closest common parent of those components?
5. Can one state value encode the rule, including “none selected” if needed?
6. Can children render from props and request changes with callback props?
7. Is any state duplicated even though it can be derived?
8. Does any independent state need to be lifted, or can it stay local?

The state owner is usually the closest component that can both hold the value and coordinate all consumers. The parent does not need to own every detail of its children.

## Common mistakes

### Leaving coordinated state in separate siblings

When each sibling owns a private version of a shared value, one child cannot reliably update the other. Move the shared source of truth to their closest common parent.

### Lifting everything to the root

This makes unrelated state travel through more of the component tree and ties distant components together. Lift only to the nearest parent that needs to coordinate the consumers.

### Passing only a value

A child can display a value but cannot request a state change unless the parent provides an appropriate callback. Controlled APIs usually need both a value prop and an event callback.

### Passing a callback but rendering stale local state

Calling a callback does not help if the child continues rendering a separate copied value. Render from the controlled prop instead.

### Using multiple state variables for one exclusive choice

If only one tab or accordion panel can be active, separate booleans allow invalid combinations. Store one active ID, with null for no active item if that is allowed.

### Using effects to synchronize siblings

Sibling components that need the same value should usually share the state directly through their parent. Effects that copy state between components add extra steps and can briefly expose inconsistent values.

## Practice

1. Two sibling components show the selected product: one displays its details and one shows its price. Where should the selected product ID live?
2. A set of tabs allows exactly one active tab. What state shape represents that rule?
3. A form field is used by only one component. Does its state need to move to the page component?
4. A child receives value and onChange. Which component decides the next value?
5. Why can copying a prop into useState lead to stale UI?

Suggested answers:

1. In their closest common parent, which passes the selected product data or ID to both.
2. One active tab ID, optionally null if no tab may be active.
3. No. Keep it local unless another component must coordinate or consume the value.
4. The parent owns the value; the child requests a change through onChange.
5. The state initializer only sets the initial value. Later prop changes do not automatically update that separate state.

## Summary

- State declared in a component belongs to that component.
- When siblings must share or coordinate a value, lift that state to their closest common parent.
- Pass current values down as props and pass callbacks down for children to request changes.
- Let the parent remain the source of truth and derive child props from its state.
- Choose a state shape that represents valid choices and avoids impossible combinations.
- Keep independent state local, and avoid copied or redundant state.
- Context can help distribute values across distant descendants, but a common parent is the simplest starting point for sibling coordination.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="042-derived-values-and-duplicated-state.md">042 - Derived values and avoiding duplicated state</a></td><td><a href="../qnas/043-lifting-state-up-and-sharing-data.md">Q&amp;A</a></td><td><a href="../examples/043-lifting-state-up-and-sharing-data.md">Example</a></td><td><a href="044-single-source-of-truth-and-state-ownership.md">044 - Single Source of Truth and State Ownership</a></td></tr>
  </tbody>
</table>
