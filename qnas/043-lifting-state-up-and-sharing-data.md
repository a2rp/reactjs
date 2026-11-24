# Lifting State Up and Sharing Data — Q&A

## Core ideas

### 1. What does “lifting state up” mean?

It means moving a piece of state from a component to one of its ancestors so that multiple components can use and coordinate the same source of truth. The usual destination is the closest common parent of the components that need the state.

### 2. Why would sibling components need shared state?

They may need to display the same changing value or coordinate behavior. For example, an editor and a preview need the same text, or several accordion panels need to ensure that only one is open at a time.

### 3. What is the closest common parent?

It is the nearest component in the component tree that contains all the components that need to coordinate. The shared state can live there and be passed down to those components.

### 4. Why should state usually go in the closest common parent instead of the app root?

The closest common parent can coordinate the consumers without tying unrelated parts of the application to that state. Keeping state closer to its users makes ownership and data flow easier to understand.

### 5. What does it mean for a parent to be the “source of truth”?

It means the parent owns the current value. Children render from the value the parent passes in, and ask for changes through callbacks. They do not maintain competing copies of that same value.

### 6. Does a child directly change its parent's state?

No. A child calls a function passed to it by the parent. The callback communicates the action or requested value; the parent decides how to update its own state.

### 7. What is the usual direction of data flow?

The current value flows down from parent to child through props. A user event in a child calls a callback prop to report a requested change. The parent updates state, then passes the new value down.

### 8. Why do children receive callback props?

A callback gives a child a way to report an interaction to its parent. The parent can then update the state it owns. Typical names include onChange, onSelect, onToggle, and onOpen.

### 9. What is a controlled component?

A component is commonly described as controlled when an important value is supplied by its parent through props and user interactions request changes through callbacks. The parent decides which value is rendered.

### 10. What is an uncontrolled component?

An uncontrolled component owns its own important state. It can manage that state without requiring a parent to coordinate it. This is a useful design when the component is independent.

### 11. Are “controlled” and “uncontrolled” mutually exclusive labels for an entire component?

No. A component may receive one value from its parent while owning other independent state locally. Decide ownership separately for each piece of state.

### 12. How can a child remain reusable when its state is controlled?

Give it a small API consisting of the values needed to render and callbacks for the interactions it supports. For example, a panel can receive isOpen and onToggle without knowing how its parent stores the active panel.

## State shape and coordination

### 13. How can state enforce that at most one accordion panel is open?

Store one active panel ID, or null when no panel is open. Derive each panel's isOpen value by comparing its ID to the active ID. This represents the exclusivity rule directly.

### 14. Why are separate booleans a poor fit for an exclusive selection?

With one boolean per option, multiple booleans can accidentally be true at once. That creates combinations the interface should not allow. One selected ID naturally represents one active option.

### 15. How should a toggle callback update the active panel?

The parent can use a functional state update so the decision is based on the latest value:

~~~jsx
setActivePanelId((currentId) =>
  currentId === panelId ? null : panelId
);
~~~

If the requested panel is already active, this closes it. Otherwise, it makes that panel active.

### 16. What if the interface always requires one panel to stay open?

Do not use null as a valid state. When the active panel is toggled, keep it active or select another allowed panel according to the interface rule. The state model should encode the actual product behavior.

### 17. Should a child pass its own ID to the callback or should the parent close over it?

Either can work. The parent can create a callback that closes over the panel ID, or pass a callback that accepts an ID. Choose the API that makes the component easiest to understand and reuse.

### 18. Why is a high-level callback often clearer than passing a state setter directly?

A callback such as onSelectItem communicates the child's supported action without exposing the parent's state implementation. Passing a setter can be fine in simple cases, but an intent-based callback gives the parent more control over how the request is handled.

### 19. What is a controlled text input?

It is an input whose displayed value comes from React props, typically backed by parent state. Its onChange handler reports the new text to the state owner, which passes the updated value back down.

### 20. How do an editor and preview stay synchronized?

Their closest common parent stores one text value and passes it to both. The editor reports changes through a callback. When the parent updates the value, both siblings render from the same new prop.

## Choosing where state belongs

### 21. When should state stay inside a child?

Keep state local when only that component needs it and no ancestor must coordinate it. For example, a tooltip's open state may belong to the tooltip if nothing else depends on it.

### 22. When should state move to a parent?

Move it when multiple components need the same changing value, or when a parent must enforce a rule across them. Put it in the closest common parent that can coordinate all of the consumers.

### 23. Should all state be lifted to the root?

No. Lifting everything high creates unnecessary coupling and makes unrelated state harder to follow. Start near the components that need the value and move it higher only when necessary.

### 24. Does lifting state mean every descendant needs to receive the state prop?

No. Only components that need the value or need to pass it to a consumer need to be involved. If many intermediary components must pass props through without using them, context may be worth considering.

### 25. How is lifting state different from Context?

Lifting state describes moving ownership to a common ancestor. Context is a mechanism for making a value available to descendants without explicitly passing it through every intermediate component. Context does not remove the need to choose who owns and updates the state.

### 26. When might Context be useful?

It may help when many distant descendants need the same value and passing props through several intermediary components becomes cumbersome. For nearby siblings, a common parent and props are usually simpler.

### 27. Does lifting state automatically mean using a global store?

No. A common parent can own shared state without making it global. Use the simplest ownership boundary that satisfies the coordination requirement.

## Avoiding synchronization problems

### 28. Why is copying a prop into useState often a mistake?

The value passed to useState is used only as the initial state. Later changes to that prop do not automatically update the local state. The child then has a separate value that can become stale.

### 29. How should a controlled child render its value?

Render directly from the prop that the parent supplies. When the child calls its callback and the parent updates state, React passes the new prop back to the child.

### 30. What is derived state?

Derived data is calculated from existing props or state rather than independently stored. For example, a panel's isOpen value can be derived from the active panel ID and its own ID.

### 31. Why avoid storing both a source value and a derived value?

They can disagree if one is updated and the other is not. Store the minimal source of truth and calculate values that can be derived from it during rendering.

### 32. Should sibling components synchronize their copies with an Effect?

Usually not. If siblings need the same state, let them read the same parent-owned value. Copying values between component state with Effects adds extra update steps and can cause temporarily inconsistent UI.

### 33. Does moving state higher affect rendering?

It can change which parts of the tree render in response to updates. First place state where ownership is clear and correct. If a real performance issue appears, profile it and optimize the relevant boundary.

### 34. Does the parent need to know every detail of its child?

No. A child can expose a focused interface through props and callbacks. For example, the parent can pass an active value and an onSelect callback without controlling the child's internal markup.

## Apply the concepts

### 35. Two sibling components show the selected product: one shows its details and one its price. Where should the selected product live?

In their closest common parent. That parent passes the selected product or its ID to both components so they render from the same selection.

### 36. A group of tabs allows one selected tab. What is a suitable state shape?

Store one active tab ID. If the interface allows all tabs to be deselected, allow null as well. Avoid separate booleans if multiple active tabs are invalid.

### 37. A search field is used by one component only. Should its text state move to the page component?

Not unless the page or another component needs to read or coordinate that text. Keeping it local avoids unnecessary ownership and prop passing.

### 38. A child receives value and onChange. Who decides the next value?

The child reports the user's input through onChange. The parent owns the value and decides what state update to perform before passing the updated value back.

### 39. A panel receives isOpen but also stores its own open boolean. What problem can occur?

The prop and local state can disagree. The panel should render from isOpen if the parent controls openness. It should keep local state only for a separate concern.

### 40. A child needs to tell the parent which item was selected. What can its callback do?

It can pass the selected item's ID or another useful value, for example onSelect(item.id). The parent receives the event information and updates its own state.

### 41. Two accordions on the same page should operate independently. Where should each active panel ID live?

Each Accordion component can own its own active panel ID because the two groups are independent. Lift that state further only if some outside component must coordinate both groups.

### 42. A parent stores an active item ID. How should each child know whether it is active?

The parent can compare the active ID with the child's ID and pass the resulting boolean as a prop. The child does not need to duplicate or reinterpret the parent's state.

## Quick review

### 43. What are the two directions involved in controlled interaction?

Values flow down through props. Interaction requests flow back up through callback props.

### 44. What is the main test for choosing a state owner?

Choose the closest component that can hold the value and coordinate every component that needs to read or change it.

### 45. What should you do when a state value can be derived from another value?

Keep the source value and calculate the derived value instead of storing both independently.

### 46. What is the simplest solution for sibling components that share one value?

Move that value to their closest common parent, pass it to the siblings as props, and pass callback props for requested updates.

## Key takeaway

Lifting state up gives coordinated components one clear source of truth. Keep independent state local; move shared state only to the closest common parent that needs to coordinate it.
