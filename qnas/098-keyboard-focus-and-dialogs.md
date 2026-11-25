# Keyboard Support, Focus, and Dialogs: Questions and Answers

## Keyboard operation

### 1. What does keyboard accessibility require?

Users must be able to reach and operate all functionality without a pointer. The controls also need to provide clear focus feedback and behave in a way users can predict.

### 2. Which keys should I test on an ordinary page?

At minimum, test Tab, Shift+Tab, Enter, and Space. Test arrow keys for native or custom composite controls, and Escape for interactions that should be dismissible.

### 3. Should every element receive keyboard focus?

No. Interactive controls should be reachable. Static content is generally read in document order by assistive technology and does not need to be in the Tab sequence unless a specific workflow needs to move focus to it.

### 4. What makes a focus order logical?

The order lets users understand and operate the page, preserving meaningful relationships between the content and controls. It does not need to match every visual column exactly, but it should not jump unpredictably.

### 5. What is tabindex="0" for?

It puts an element that is not normally focusable into the regular Tab sequence. Use it sparingly, usually for a custom widget that implements the appropriate keyboard model.

### 6. What is tabindex="-1" for?

It allows an element to receive programmatic focus without adding it to the normal Tab sequence. It is useful for a heading or container that should receive focus after a navigation or state change.

### 7. Why should positive tabindex values be avoided?

Positive values create a custom focus sequence separate from DOM order. New controls can make that sequence confusing and difficult to maintain. Prefer a logical DOM order and ordinary tabindex behavior.

### 8. Is focus order controlled by CSS?

No. CSS changes visual presentation, while focus order normally follows the document order of focusable elements. A mismatch between the visual flow and focus order can confuse users.

### 9. Should I use a custom key handler on a native button?

Usually no. A native button already supports keyboard activation. Extra key handling can cause duplicate actions or conflict with browser behavior.

### 10. Does onClick on a native button support keyboard activation?

Yes. The browser activates a focused button through its expected keyboard behavior, and React's onClick receives the activation. A pointer-only handler is unnecessary.

### 11. What if an action is only available on hover?

Make it available through keyboard focus and other supported input methods too. A user who cannot hover must still be able to discover and operate the action.

### 12. What is the difference between focus and a screen reader's reading cursor?

Keyboard focus is the active target for keyboard input. A screen reader may also use a virtual or reading cursor to browse content without moving DOM focus. Both need sensible structure and interaction.

## Focus visibility and management

### 13. Why must focus be visible?

A sighted keyboard user needs to know which control will receive the next key. If focus is invisible, users can lose their place and operate the wrong control.

### 14. Is it okay to set outline: none?

Only if the design provides an equally clear and persistent focus indicator for keyboard use. Removing the browser outline without a replacement makes focus hard to locate.

### 15. What does :focus-visible do?

It lets styling target focus when the browser determines that a visible focus indicator is appropriate, commonly during keyboard navigation. It can be used to style a clear outline.

### 16. Should focus styles use color alone?

No. Use a visible shape or boundary such as an outline, and ensure it remains distinguishable against adjacent content and backgrounds. Check the result in high contrast modes too.

### 17. When should React move focus?

Move focus when a user action changes where they should continue, such as opening a modal, moving to a new page when needed, focusing an error summary, or recovering after the focused control is removed.

### 18. When should React leave focus alone?

For ordinary state updates that do not change the user's next interaction target. Unexpected focus movement can interrupt typing, reading, and exploration.

### 19. How can a React component focus a DOM node?

Create a ref with useRef, pass it to the rendered element, then call its focus method in response to the relevant interaction or after the element has been rendered.

### 20. Is direct DOM access always wrong in React?

No. React recommends declarative rendering for normal UI updates, but refs are appropriate for imperative browser operations such as focus, scrolling, measuring, or calling a platform API.

### 21. Should I focus a node during render?

No. Render should describe UI, not perform DOM side effects. Focus in an event handler or an effect after React commits the node.

### 22. Why might focus need to happen in an effect?

The element may not exist until a state update has caused React to render it. An effect runs after the commit, when the ref can point to the rendered node.

### 23. What should receive focus after a dialog's opener is removed?

Move focus to a logical remaining target, such as a page heading or the next relevant control. The browser cannot restore focus to an element that no longer exists.

## Dialog behavior

### 24. What makes a dialog modal?

A modal dialog prevents interaction with the rest of the page while it is open. It also needs a clear name, focus management, a way to close, and appropriate keyboard behavior.

### 25. What is the difference between showModal() and show()?

showModal() opens a modal dialog: the rest of the document is unavailable for interaction and focus stays within the dialog. show() opens a non-modal dialog, where users can continue to interact with the page.

### 26. Does adding the open attribute create a modal dialog?

No. The open attribute displays a dialog but does not by itself provide the same modal behavior as showModal().

### 27. What does the native dialog element handle for a modal?

When opened with showModal(), the browser provides modal behavior such as making the background inert, containing the tab sequence, allowing Escape to request closure, and managing focus return when the opener remains available.

### 28. How should a dialog get an accessible name?

Give it a visible title and reference that title with aria-labelledby. The name should make the dialog's purpose clear.

### 29. When should aria-describedby be used on a dialog?

Use it when a short piece of text provides a helpful summary or instruction. For a dialog with structured content such as several paragraphs or lists, one long flattened description may be less useful; let users navigate the content.

### 30. Should Escape be the only way to close a modal?

No. Provide a visible close, cancel, or confirm action. Escape is an expected additional method for many modal dialogs, but should not be the only exit.

### 31. Where should focus go when a dialog opens?

Focus should move to an appropriate element inside it. Choose based on the task: a cancel or safe action for a destructive confirmation, the first useful field for a short form, or a static heading for a long structured dialog.

### 32. Should every dialog focus its first button?

Not necessarily. The first button may be destructive or may not be the best starting point. Choose an initial target that helps users understand the content and avoid unintended actions.

### 33. Should focus stay inside a modal dialog?

Yes. Tab and Shift+Tab should cycle among focusable elements inside the modal while it is open. The user should close the modal before interacting with the page behind it.

### 34. What happens when a native modal dialog closes?

The browser normally returns focus to the element that opened it if that element is still present. If the action removes the opener, the application should move focus to a useful remaining target.

### 35. Does aria-modal="true" make a custom div modal?

No. It communicates a state to assistive technology; it does not trap focus, disable background controls, handle Escape, or restore focus. Those behaviors must be implemented separately.

### 36. Why is a custom modal hard to build correctly?

It must implement naming, initial focus, a contained tab sequence, Escape, a visible close action, background inertness, focus restoration, nested dialog handling, and assistive technology support. Missing any part can block or confuse users.

### 37. How does a React ref work with a native dialog?

Attach a ref to the dialog element, then call showModal() or close() on the current DOM node from an event handler. Keep one clear source of truth for whether it is open.

### 38. Should I use both the open prop and showModal()?

Avoid using a React open attribute and imperative showModal()/close() calls as competing controls for the same dialog. Choose a clear state strategy and synchronize close and cancel events if application state needs to reflect the dialog.

### 39. What does autoFocus do in a dialog?

It can specify an element that should receive focus when the dialog opens. Use it deliberately; the best initial focus depends on the dialog's content and purpose, and multiple autofocus elements are confusing.

### 40. How do multiple modal dialogs handle Escape?

Only the topmost open modal should close in response to Escape. Avoid stacking dialogs unless the workflow truly requires it, and test nested behavior carefully.

## Testing

### 41. What should I check with only a keyboard?

Check that controls are reachable in a sensible order, focus is always visible, actions work with the expected keys, and no interaction traps the user unexpectedly.

### 42. How should I test a dialog?

Open it by keyboard, confirm focus enters at a sensible place, cycle with Tab and Shift+Tab, close with its visible action and Escape, and confirm focus returns or moves to a clear fallback.

### 43. Is an automated scan enough for keyboard accessibility?

No. Automated tools can detect some structural issues, but a person must test whether the focus sequence is understandable, visible, and useful.

### 44. What should I test with a screen reader?

Check the dialog's name and description, how its controls are announced, whether the background is unavailable while modal, and whether the reading order remains understandable.

### 45. What is the main rule for focus management?

Do not move focus without a user-centered reason. When it must move, place it where the user can continue, make that location perceivable, and provide a route back when appropriate.

## Quick review

- Preserve a logical DOM and focus order.
- Keep focus visible.
- Prefer native keyboard behavior over custom handlers.
- Use refs only for clear imperative tasks.
- For modal dialogs, verify focus entry, containment, Escape, close controls, and focus return.
