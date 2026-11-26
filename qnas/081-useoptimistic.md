# Optimistic UI with useOptimistic - Q&A

## API and core behavior

### 1. What does useOptimistic do?

It lets a component display a temporary, likely result while an Action is pending. It does not perform the request or save data.

### 2. Where is useOptimistic imported from?

Import it from react:

    import { useOptimistic } from "react";

### 3. What does useOptimistic return?

A pair: the current optimistic state and a setter that requests a temporary value during an Action.

### 4. What arguments can useOptimistic receive?

It receives a base value and an optional reducer. The base value is displayed when no Action is pending. The reducer calculates optimistic state from the current state and an action value.

### 5. What is the base value?

It is the authoritative value outside pending optimistic work. It usually comes from props, useState, or another data owner.

### 6. Is optimisticState always different from the base?

No. When there is no relevant pending Action, it is the base value. During an Action, it reflects the temporary update.

### 7. When should I call the optimistic setter?

Inside an Action. A function passed to a form action prop already runs as an Action. For a normal event handler, start a Transition with startTransition.

### 8. What happens if I call the setter during an ordinary click handler without an Action?

React warns because the optimistic update is not part of an Action lifecycle. Wrap the asynchronous work in startTransition or use an Action prop.

### 9. Does the setter return a value?

No. It schedules an optimistic state update for the Action.

### 10. Is the setter the same as a useState setter?

No. A useState setter changes the persistent state value. The useOptimistic setter changes a temporary view that lasts for pending Action work.

## Success, failure, and reconciliation

### 11. What should I do after the request succeeds?

Update the authoritative base value with the confirmed response, or refresh the owner of that data. Then the final UI reflects confirmed data when the Action ends.

### 12. What if the server confirms a different value than I predicted?

Use the server's returned value as the new base. For example, show a server-corrected total rather than keeping the client's estimate.

### 13. What happens if the Action fails and the base was not changed?

When the Action ends, the optimistic state goes away and the unchanged base value appears again. This is the normal rollback path.

### 14. Do I have to manually reverse the optimistic update on failure?

Usually not if the base value remained unchanged. The temporary display automatically ends with the Action. Show a useful error message instead.

### 15. Does rollback show the user why the operation failed?

No. Rollback only restores the base display. Add error state, a returned Action result, or another user-facing explanation.

### 16. What if I update the base optimistically too?

Then there may be nothing left to roll back to. Keep speculative data in useOptimistic and only update the base after confirmation.

### 17. If I catch a failed request, will the optimistic value still disappear?

Yes. When the Action completes, optimistic state stops being active. Catching the error lets the component display a recovery message; it does not turn the failed result into confirmed data.

### 18. What if the request succeeds but I forget to update the base?

When the Action ends, the UI returns to the old base value. Update or refetch authoritative data on success.

### 19. Is returning an error object from useActionState the same as throwing?

No. A returned validation result is normal Action state. The optimistic view still ends when the Action finishes. Render the returned error separately and keep the base value consistent with what the server accepted.

### 20. What should happen if an Error is thrown?

The Action ends and optimistic state returns to the current base. Handle unexpected errors with an Error Boundary or catch and display a safe recovery message.

## Reducers and collections

### 21. Why would I pass a reducer?

Use a reducer when the optimistic change depends on the current state, such as appending an item or applying an operation to a list.

### 22. What arguments does the optimistic reducer receive?

It receives the current state and the optimistic action value passed to the setter. It returns the next optimistic view.

### 23. What makes a reducer pure?

It only calculates and returns a value. It does not mutate its inputs, make network requests, write to storage, or modify outside state.

### 24. Why might a reducer run again?

React can recalculate optimistic state when the base value changes while work is pending. A reducer should safely derive a view from the latest base plus the optimistic intent.

### 25. How should I represent a new optimistic list item?

Give it an identity that can be correlated with the confirmed item, and mark it as pending if that helps the interface. On success, replace or reconcile it with the server's canonical item rather than showing duplicates.

### 26. Should an optimistic item use a temporary ID?

It can. Use a stable client ID to match the optimistic item to the confirmed response, then replace that identity as needed. Ensure list keys remain stable through reconciliation.

### 27. Can the reducer send the request?

No. Reducers can be re-run and must remain pure. Start the request in the Action; pass the intended change to the optimistic setter.

### 28. How should I remove an item optimistically?

Use a reducer that returns a list without that item while the delete Action is pending. If the delete fails, leave the base unchanged so the item reappears, then show an error or retry option.

## Transitions and related Hooks

### 29. How do I use it from a click handler?

Start an Action with startTransition, set the optimistic value inside it, await the operation, then update the base from the confirmed result.

### 30. Why wrap a state update after await in another startTransition?

React's current Transition behavior requires state updates scheduled after an await to be wrapped in another startTransition. Keep controlled text input updates out of Transitions.

### 31. Does a form Action need startTransition around the optimistic setter?

No. A function passed to a form action prop already runs as an Action, so the setter can be called directly inside that Action.

### 32. How is useOptimistic different from useTransition?

useOptimistic provides temporary display state. useTransition starts or tracks pending Transition work with an isPending flag. They can be used together: one for the speculative value, one for pending feedback.

### 33. How is useOptimistic different from useActionState?

useOptimistic manages a temporary value while an Action is pending. useActionState stores values returned by an Action, such as validation errors or a confirmed result. They solve different parts of the interaction.

### 34. Can I combine useOptimistic and useActionState?

Yes. For example, useOptimistic can display a comment immediately while useActionState returns a saved comment or a validation message. Keep the confirmed source of truth clear.

### 35. Does useOptimistic replace useState?

No. The base value should still be owned by state, props, or a data store. useOptimistic only supplies a temporary view during an Action.

## UX, concurrency, and safety

### 36. Which operations are good candidates?

Operations with predictable results and clear recovery, such as a like toggle, adding a comment, or changing a simple preference.

### 37. Which operations need extra caution?

Operations involving money, permissions, inventory, concurrent edits, irreversible side effects, or a result that depends heavily on server rules.

### 38. What if multiple operations are pending at once?

Define how they compose and how responses are ordered. Use stable IDs, idempotent writes, and reducers that can recalculate from updated base data. Disable repeat submissions if overlapping operations would confuse users.

### 39. Does disabling a button guarantee the operation only happens once?

No. Client UI can be bypassed and requests can be retried. Enforce uniqueness, authorization, and other important rules in the server or data layer.

### 40. Can an optimistic update work offline?

It can show a local prediction, but useOptimistic does not persist a queue or retry when connectivity returns. Offline storage, retries, conflict resolution, and synchronization require an explicit data strategy.

### 41. Should pending optimistic content be visually distinct?

Often yes. A subtle “Sending…” or “Pending” label helps users distinguish a prediction from a confirmed result. Avoid relying only on color or animation.

### 42. What is the main mental model?

The Action carries a temporary optimistic view. The base value remains the truth after the Action ends. Update that base from confirmed results, and explain failures when the prediction rolls back.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="080-useformstatus.md">080 - Q&amp;A: Pending form state with useFormStatus</a></td><td><a href="../lessons/081-useoptimistic.md">Lesson</a></td><td><a href="../examples/081-useoptimistic.md">Example</a></td><td><a href="082-server-functions-and-form-actions.md">082 - Server Functions and form Actions - Q&amp;A</a></td></tr>
  </tbody>
</table>
