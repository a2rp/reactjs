# React Example: A Keyboard-Friendly Native Confirmation Dialog

This example uses the browser's native dialog element for a destructive confirmation. React calls the browser dialog API through a ref. The example demonstrates initial focus, Escape, an explicit Cancel action, and a focus fallback when the invoking button disappears.

## File structure

    src/
    ├── DeleteAccountPage.tsx
    └── DeleteAccountPage.css

Render DeleteAccountPage from the application's entry point. This example uses React with TypeScript.

## 1. Create the React component

Create src/DeleteAccountPage.tsx:

    import { useEffect, useRef, useState } from 'react';
    import './DeleteAccountPage.css';

    export default function DeleteAccountPage() {
      const dialogRef = useRef<HTMLDialogElement>(null);
      const titleRef = useRef<HTMLHeadingElement>(null);
      const [deleted, setDeleted] = useState(false);

      function openConfirmation() {
        dialogRef.current?.showModal();
      }

      function cancelDeletion() {
        dialogRef.current?.close();
      }

      function confirmDeletion() {
        dialogRef.current?.close();
        setDeleted(true);
      }

      useEffect(() => {
        if (deleted) {
          // The dialog opener is removed after deletion, so give focus
          // a useful destination that remains in the document.
          titleRef.current?.focus();
        }
      }, [deleted]);

      return (
        <main className="page">
          <h1 className="page-title" ref={titleRef} tabIndex={-1}>
            Account settings
          </h1>

          <p>
            Review account actions and choose one only when you are ready.
          </p>

          {!deleted ? (
            <button
              className="danger-button"
              onClick={openConfirmation}
              type="button"
            >
              Delete account
            </button>
          ) : (
            <p className="success-message" role="status">
              Your account has been deleted.
            </p>
          )}

          <dialog
            aria-describedby="dialog-description"
            aria-labelledby="dialog-title"
            className="confirm-dialog"
            ref={dialogRef}
          >
            <h2 id="dialog-title">Delete this account?</h2>
            <p id="dialog-description">
              This action cannot be undone. Your account data will be removed.
            </p>

            <div className="dialog-actions">
              <button
                autoFocus
                className="cancel-button"
                onClick={cancelDeletion}
                type="button"
              >
                Keep account
              </button>

              <button
                className="danger-button"
                onClick={confirmDeletion}
                type="button"
              >
                Delete account
              </button>
            </div>
          </dialog>
        </main>
      );
    }

## 2. Style the page and keep focus visible

Create src/DeleteAccountPage.css:

    :root {
      color: #182230;
      background: #f4f6f8;
      font-family: system-ui, sans-serif;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
    }

    button:focus-visible {
      outline: 3px solid #164fc4;
      outline-offset: 3px;
    }

    .page {
      width: min(100% - 2rem, 42rem);
      margin: 4rem auto;
      padding: 2rem;
      border: 1px solid #cbd3df;
      border-radius: 0.8rem;
      background: #ffffff;
    }

    .page-title {
      margin-top: 0;
      font-size: clamp(1.8rem, 5vw, 2.4rem);
    }

    .page-title:focus {
      outline: 3px solid #164fc4;
      outline-offset: 4px;
    }

    .danger-button,
    .cancel-button {
      min-height: 2.8rem;
      padding: 0.65rem 1rem;
      border: 1px solid transparent;
      border-radius: 0.4rem;
      cursor: pointer;
      font: inherit;
      font-weight: 700;
    }

    .danger-button {
      background: #a51f2b;
      color: #ffffff;
    }

    .danger-button:hover {
      background: #821823;
    }

    .cancel-button {
      border-color: #68768a;
      background: #ffffff;
      color: #182230;
    }

    .cancel-button:hover {
      background: #eef1f5;
    }

    .confirm-dialog {
      width: min(100% - 2rem, 32rem);
      padding: 1.5rem;
      border: 1px solid #aeb9c8;
      border-radius: 0.8rem;
      color: #182230;
      box-shadow: 0 1rem 3rem rgb(0 0 0 / 24%);
    }

    .confirm-dialog::backdrop {
      background: rgb(13 22 35 / 64%);
    }

    .confirm-dialog h2 {
      margin-top: 0;
    }

    .confirm-dialog p {
      line-height: 1.6;
    }

    .dialog-actions {
      display: flex;
      flex-wrap: wrap;
      justify-content: flex-end;
      gap: 0.75rem;
      margin-top: 1.5rem;
    }

    .success-message {
      color: #17623b;
      font-weight: 700;
    }

## 3. Understand the focus lifecycle

1. The Delete account button calls showModal() on the dialog element stored in dialogRef.
2. The browser exposes a modal dialog, moves focus inside, and makes the rest of the page inert.
3. The Cancel control has autoFocus so the safe choice receives initial focus.
4. Tab and Shift+Tab remain inside the modal, Escape requests closure, and a visible button also closes it.
5. When Cancel or Escape closes the dialog, the browser normally returns focus to the still-present opener.
6. When deletion is confirmed, React removes the opener. The effect focuses the remaining page heading after the state update is committed.
7. The status message communicates the result without trying to move focus a second time.

The native dialog element opened with showModal() supplies modal semantics and behavior. This example therefore does not add role="dialog" or aria-modal. The visible heading names the dialog through aria-labelledby, and a short explanation is associated through aria-describedby.

## 4. Test the interaction

1. Tab to Delete account and press Enter or Space.
2. Confirm the dialog opens and focus starts on Keep account.
3. Tab forward and backward; focus should cycle among the dialog controls.
4. Press Escape. Confirm the dialog closes and focus returns to Delete account.
5. Open it again and activate Keep account. Confirm the same focus return.
6. Open it again and activate Delete account. Confirm the opener disappears, the result is announced, and focus moves to Account settings.
7. Repeat the flow with a screen reader and in the browsers supported by the application.

## 5. Important boundaries

- Keep the dialog mounted while it is closed so the ref points to a real DOM node when the opener runs.
- Do not also control the dialog with an open prop in this pattern; showModal() and close() are the chosen open-state mechanism.
- If application state must track Escape or other native closure, handle the dialog cancel and close events and keep that state synchronized deliberately.
- For a dialog with long or structured content, consider focusing a heading with tabIndex={-1} instead of its first button.
- If a custom dialog is unavoidable, implement and test focus containment, background inertness, Escape, naming, close controls, and focus restoration.

## Key lesson

React refs are appropriate for calling imperative browser APIs such as showModal(), close(), and focus(). Prefer the native dialog behavior, then manage focus explicitly when the normal return target no longer exists.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="097-semantic-html-and-accessible-components.md">097 - React Example: A Semantic and Accessible Contact Page</a></td><td><a href="../lessons/098-keyboard-focus-and-dialogs.md">Lesson</a></td><td><a href="../qnas/098-keyboard-focus-and-dialogs.md">Q&amp;A</a></td><td><a href="099-accessible-announcements-and-async-ui.md">099 - React Example: Accessible Async Search with Current-Request Protection</a></td></tr>
  </tbody>
</table>
