# Keyboard Support, Focus, and Dialogs in React

## Learning goals

After this lesson, you should be able to:

- Operate and review an interface using only a keyboard.
- Preserve a logical focus order and a visible focus indicator.
- Use native controls rather than rebuilding their keyboard behavior.
- Move focus with a React ref when an interaction requires it.
- Build a modal dialog that supports initial focus, contained focus, Escape, and focus return.
- Explain why a custom dialog has more responsibilities than a native dialog.

## 1. Keyboard access is part of every interaction

Some people use a keyboard instead of a mouse, and others use keyboards, switches, speech input, or assistive technology that relies on keyboard behavior. Every action available to a pointer user should also be reachable and usable from a keyboard.

For the web, common keyboard behavior includes:

- Tab moves to the next sequentially focusable control.
- Shift+Tab moves to the previous control.
- Enter activates links and commonly activates focused buttons.
- Space activates a focused button and toggles many native controls.
- Arrow keys operate many native composite controls such as radio groups and select menus.
- Escape dismisses many browser and widget interactions, including native modal dialogs.

Let native elements provide their expected keyboard interaction. A real button already supports keyboard activation; adding custom key handlers to it can cause duplicate behavior.

## 2. Focus order should preserve meaning

Keyboard focus normally follows the order of interactive elements in the document. Keep the DOM order aligned with the reading order and the way the page is visually presented.

Avoid positive tabindex values such as tabindex="5". They create a second, fragile focus order that can change when controls are inserted. Use ordinary document order. Use tabindex="0" only when a non-native custom widget must join the tab sequence, and tabindex="-1" for an element that should be focusable by code but skipped by ordinary Tab navigation.

CSS can rearrange where elements appear visually without changing their DOM order. If focus then jumps around in a way that contradicts what users see, keyboard operation becomes confusing. Prefer markup and layout that keep reading, visual, and focus sequences aligned.

Do not make static content tabbable merely to compensate for poor structure. Give focus to a heading or container only when a workflow has a clear reason, such as moving focus after a route change, closing a dialog whose opener disappeared, or directing attention to an error summary.

## 3. Keep focus visible

People who use a keyboard need to know which control currently has focus. Do not remove the browser outline without providing an equally clear replacement.

    button:focus-visible,
    a:focus-visible,
    input:focus-visible,
    textarea:focus-visible,
    select:focus-visible {
      outline: 3px solid #174dbb;
      outline-offset: 3px;
    }

The indicator should contrast with nearby colors and remain visible in high contrast modes. Test it against the actual backgrounds and across focusable controls. Do not show focus only on hover: keyboard users do not hover.

The focus-visible selector lets a design emphasize focus when the browser determines it is useful, commonly for keyboard navigation. A clear focus indicator can also be appropriate when pointer users benefit from seeing focus. Never leave keyboard focus invisible.

## 4. Use native controls and their built-in behavior

Use anchors for navigation, buttons for actions, inputs for data entry, and native radio buttons and checkboxes for their corresponding choices. These elements already participate in keyboard navigation and expose familiar roles, states, and names.

A clickable div requires extra work: it must become focusable, respond to the correct keys, expose the right role and state, and preserve disabled behavior. Prefer a button instead.

For a group of radio options, native radio inputs provide the expected Tab and arrow-key behavior. For a custom menu or tablist, follow the keyboard model for that widget rather than adding arbitrary key handlers. Most applications should first ask whether a native element already does the job.

Use onClick on a native button for an action. React and the browser deliver its activation for pointer and keyboard use. Do not implement a parallel click-only or mouse-only path.

## 5. Focus management in React

Most React components should let the browser manage focus. When a user action needs focus to move somewhere specific, a ref provides access to the rendered DOM node.

    import { useRef } from 'react';

    function SearchBox() {
      const inputRef = useRef<HTMLInputElement>(null);

      function focusSearch() {
        inputRef.current?.focus();
      }

      return (
        <div>
          <label htmlFor="site-search">Search</label>
          <input id="site-search" ref={inputRef} type="search" />
          <button type="button" onClick={focusSearch}>
            Focus search
          </button>
        </div>
      );
    }

Use refs for imperative interactions such as focus, scrolling, or calling a native browser API. Do not read or write DOM nodes during render. Do not focus an element on every render; that can interrupt typing and move users unexpectedly.

When focus must follow a React state update, wait until the target has been rendered. An effect can focus it after the commit:

    useEffect(() => {
      if (showResults) {
        resultsHeadingRef.current?.focus();
      }
    }, [showResults]);

The target should have a reason to receive programmatic focus and should usually use tabindex="-1" if it is not normally interactive.

## 6. Move focus when the interface changes

Some changes affect where a user can continue:

- After opening a modal dialog, move focus into it.
- After closing a dialog, return focus to its opener when that opener still exists.
- If an action removes the opener, move focus to a sensible remaining control or heading.
- After navigation, place focus where the new page content starts if the navigation model does not already do so.
- After a failed form submission, help users reach the first invalid field or an error summary that explains the problem.

Do not move focus for every content update. Unexpected focus changes can make typing and reading difficult. Prefer status announcements for updates that do not require a new focus location; dedicated lessons cover those patterns.

## 7. Prefer the native dialog element for modal dialogs

A modal dialog temporarily makes the rest of the page unavailable for interaction. The native dialog element opened with showModal() provides important browser behavior:

- Focus moves into the modal.
- Tab and Shift+Tab remain within the modal.
- Content outside the modal is made inert while it is open.
- Escape requests that it close.
- Focus normally returns to the element that opened it when it closes.

A dialog should have an accessible name, usually provided by a visible heading referenced with aria-labelledby. A concise description may be connected with aria-describedby when it helps explain the dialog. Include a visible button for closing or cancelling; do not make Escape the only way out.

Choose initial focus for the task. For a short confirmation, focus the safest or most likely next action, often Cancel for a destructive operation. For a long dialog with structured content, focus a static heading at the start so users can read from there instead of landing deep in the dialog.

Use showModal() for a modal dialog. Merely adding the open attribute or calling show() does not create the same modal behavior. Use a non-modal dialog only when users should be able to continue working with the page behind it.

## 8. Connecting a native dialog to a React component

The dialog's open state is managed by its browser API in this pattern. A React ref calls showModal() or close() in response to user actions:

    const dialogRef = useRef<HTMLDialogElement>(null);

    function openDialog() {
      dialogRef.current?.showModal();
    }

    function closeDialog() {
      dialogRef.current?.close();
    }

    return (
      <>
        <button type="button" onClick={openDialog}>
          Delete account
        </button>

        <dialog
          aria-labelledby="delete-title"
          aria-describedby="delete-description"
          ref={dialogRef}
        >
          <h2 id="delete-title">Delete this account?</h2>
          <p id="delete-description">
            This action cannot be undone.
          </p>
          <button autoFocus type="button" onClick={closeDialog}>
            Cancel
          </button>
          <button type="button" onClick={confirmDelete}>
            Delete account
          </button>
        </dialog>
      </>
    );

The initial-focus choice above puts focus on Cancel. In a real app, the confirm handler should perform the action and then close the dialog. If the triggering control is removed as part of that action, explicitly direct focus to a logical remaining element after React commits the update.

Be deliberate with React and native dialog state. Do not simultaneously toggle the open attribute and call showModal() as competing ways to control the same dialog. Handle the browser's cancel and close behavior if application state must stay synchronized with the dialog.

## 9. Why custom modal dialogs are difficult

A custom div with role="dialog" does not automatically become modal. A complete custom implementation must handle all of the following:

- An accessible name and any helpful description.
- Initial focus placement.
- A contained Tab and Shift+Tab sequence.
- Escape and a visible close action.
- Making background controls unavailable while the modal is open.
- Restoring focus after it closes.
- Preventing background scrolling if required by the design.
- Nested dialogs and cleanup.
- Screen reader announcements and browser differences.

Using aria-modal="true" alone does not implement those behaviors. If a native dialog cannot meet a real requirement, use a well-maintained dialog primitive that has been tested with the target browsers and assistive technologies rather than building a partial modal from scratch.

## 10. A keyboard review checklist

Test the complete interaction without a mouse:

1. Tab through all links and controls; confirm the order makes sense.
2. Use Shift+Tab to move backwards.
3. Confirm each focused control has a clear visible indicator.
4. Activate links and buttons with their expected keys.
5. Check radio groups, menus, and other composite controls with arrow keys.
6. Open the dialog and confirm focus enters it at an appropriate place.
7. Press Tab and Shift+Tab repeatedly; focus must not escape a modal dialog.
8. Press Escape and confirm the dialog closes.
9. Confirm focus returns to the opener or moves to an appropriate fallback.
10. Use a screen reader to confirm the dialog's name, description, and controls make sense.
11. Test the same flow in the browsers and assistive technologies the application supports.

Automated checks can detect some issues, but they cannot prove the focus order feels logical or that the chosen focus target helps users complete the task.

## Common mistakes

- Removing outlines without adding a visible replacement.
- Using positive tabindex values to force a custom order.
- Making an entire card focusable when only its link and button need focus.
- Rebuilding a button or radio group with generic elements and incomplete key handling.
- Moving focus on every state change.
- Opening a modal without moving focus inside or naming it.
- Letting Tab leave a modal while the rest of the page appears blocked.
- Closing a modal without a visible close action.
- Leaving focus on a control that React removed.
- Assuming aria-modal alone makes a custom dialog behave modally.
- Testing with a mouse only.

## Summary

Keyboard support depends on real, operable controls, a sensible focus order, and a visible focus indicator. Let native HTML provide its built-in behavior. Use React refs when focus must move for a clear reason. For modal dialogs, prefer the native dialog API and verify entry, containment, Escape, and focus return.

## Practice checklist

1. Complete a page's main workflow using only Tab, Shift+Tab, Enter, Space, and arrow keys where appropriate.
2. Add a visible focus indicator to every interactive control.
3. Open a native modal dialog and test its focus lifecycle.
4. Remove a dialog's opener after confirmation and move focus to an appropriate remaining element.
5. Explain why aria-modal does not create a working modal by itself.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="097-semantic-html-and-accessible-components.md">097 - Semantic HTML and Accessible React Components</a></td><td><a href="../qnas/098-keyboard-focus-and-dialogs.md">Q&amp;A</a></td><td><a href="../examples/098-keyboard-focus-and-dialogs.md">Example</a></td><td><a href="099-accessible-announcements-and-async-ui.md">099 - Accessible Announcements and Asynchronous Interfaces in React</a></td></tr>
  </tbody>
</table>
