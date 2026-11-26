# Optimistic UI with useOptimistic

## Learning goals

After this lesson, you should be able to:

- Explain why an interface may update before a server confirms a change.
- Use useOptimistic to show temporary state during an Action.
- Choose between a direct optimistic value and a reducer.
- Reconcile the optimistic view with confirmed application state.
- Let the UI roll back when an operation fails.
- Recognize where optimistic UI can create confusing or risky behavior.

## Why optimistic UI exists

A user may click Like, send a message, or add an item, then wait for a network request. If the interface waits for the server before showing any change, the app can feel slow even when it is working correctly.

An optimistic update immediately shows the likely result while the real operation continues. The interface assumes the operation will succeed, then settles on the confirmed result when it does. If it fails, the temporary view goes away and the user should receive a clear explanation.

Optimistic UI is a presentation technique. It does not save data, make a request, or prove that a server accepted the change.

## The useOptimistic API

Import useOptimistic from react and call it at the top level of a component:

    import { useOptimistic } from "react";

    const [optimisticState, setOptimisticState] =
      useOptimistic(baseState);

The first argument, baseState, is the value to show when no Action is pending. The Hook returns:

1. optimisticState: the value currently rendered.
2. setOptimisticState: a function that requests a temporary optimistic value.

The temporary value is shown while an Action is in progress. When that Action ends, React returns to the base value unless the real state has been updated to the confirmed result.

## The setter belongs inside an Action

Call the optimistic setter inside an Action. A function passed to a React form action prop already runs as an Action. For a button or event handler, start a Transition with startTransition:

    import { startTransition, useOptimistic } from "react";

    function LikeButton({ likes, saveLike }) {
      const [optimisticLikes, setOptimisticLikes] =
        useOptimistic(likes);

      function handleLike() {
        startTransition(async () => {
          setOptimisticLikes(likes + 1);
          const confirmedLikes = await saveLike(likes + 1);
          startTransition(() => {
            // Update the real state from the confirmed result here.
          });
        });
      }

      return <button onClick={handleLike}>Likes: {optimisticLikes}</button>;
    }

The real state update is intentionally left as a comment because the component needs an owner for confirmed state. In a complete app, that may be local state, a parent callback, a form Action result, or a data cache update.

Calling the optimistic setter directly during an ordinary event handler, outside an Action or Transition, is not the intended usage and produces a warning.

## Base state is the source of truth

Treat the first argument as the authoritative value outside the pending Action. It commonly comes from props or regular React state:

    const [likes, setLikes] = useState(12);
    const [optimisticLikes, setOptimisticLikes] =
      useOptimistic(likes);

The Action first asks React to display a likely value:

    setOptimisticLikes(likes + 1);

Then it performs the real operation and updates likes from the confirmed response:

    const confirmedLikes = await saveLike(likes + 1);

    startTransition(() => {
      setLikes(confirmedLikes);
    });

The optimistic value and confirmed value should converge. If the server returns a different value, such as a corrected count, show that returned value. Do not assume the server will always accept the exact change the client predicted.

When updating state after an await inside a Transition, wrap the state update in another startTransition. Keep controlled text input updates outside Transitions.

## Use a reducer for optimistic collections

A reducer is useful when an optimistic operation depends on the current value, such as appending a pending item to a list:

    function addTodo(currentTodos, optimisticTodo) {
      return [...currentTodos, optimisticTodo];
    }

    const [optimisticTodos, addOptimisticTodo] =
      useOptimistic(todos, addTodo);

The reducer receives the current state and an optimistic action, then returns the next display state. Keep it pure: it should calculate and return a value without making requests, mutating arrays, or changing external state.

For more complex operations, the action can describe the intent:

    function updateTodos(currentTodos, action) {
      if (action.type === "add") {
        return [...currentTodos, action.todo];
      }

      if (action.type === "remove") {
        return currentTodos.filter((todo) => todo.id !== action.id);
      }

      return currentTodos;
    }

    const [optimisticTodos, dispatchOptimistic] =
      useOptimistic(todos, updateTodos);

    dispatchOptimistic({ type: "add", todo: newTodo });

When the confirmed base value changes while work is pending, React can apply the reducer to the latest base value. Design reducers so an operation can be recalculated from current data; avoid assuming the base never changes.

## What happens on success

After success, update the authoritative state from the response or refresh the data source that owns it. The optimistic display is temporary. Once the Action completes, the base state determines what remains on screen.

For example, after a server confirms that a comment was created, add the returned comment, including its server ID and normalized fields, to the real comments state. The optimistic comment may use a temporary client ID while pending, but the confirmed record should use the stable identity expected by the application.

## What happens on failure

If an Action ends without updating the base value, the temporary optimistic display goes away and the original base value appears again. This is the rollback; you usually do not need a second setter that manually reverses the optimistic change.

The user still needs feedback. Catch expected failures and show an error message, or return structured error state through useActionState. An optimistic rollback without an explanation can look like the click was ignored.

If you catch an error, do not update authoritative state as if the operation succeeded. If a server reports a conflict or a corrected value, reconcile with that explicit response.

## Relationship to related Hooks

- useOptimistic manages a temporary view of a value during an Action.
- useState or a parent data owner holds the confirmed value.
- useTransition starts and tracks an Action initiated from an event handler.
- useActionState stores values returned by an Action and exposes an isPending flag.
- useFormStatus lets a child component inspect the pending status of its parent form.

These Hooks can be combined, but they have separate jobs. For example, useOptimistic can display a pending comment, while useActionState stores a validation message and confirmed result.

## Good use cases and cautions

Optimistic updates work well when the expected result is easy to predict and the interface can recover cleanly:

- Liking or unliking a post.
- Adding a comment or task to a visible list.
- Updating a simple preference.
- Reordering an item when the server normally accepts the same order.

Use extra care when:

- The result depends on server rules, inventory, permissions, or concurrent edits.
- Reversing the change would be confusing or costly.
- The action moves money, grants access, or has another important consequence.
- Multiple requests can overlap or return out of order.
- The optimistic display could be mistaken for a completed or verified operation.

Do not use optimistic UI as authorization, validation, or a substitute for persistence. The server must verify and save the operation. Consider disabling repeated actions, assigning stable IDs, making writes idempotent, and deciding how conflicts should be explained.

## Common mistakes

### Updating the base state immediately with the guess

If the guessed value is saved as confirmed state before the request succeeds, the rollback behavior is lost. Keep speculative state in useOptimistic and update the base from confirmed data.

### Calling the optimistic setter outside an Action

Wrap the work in startTransition, or call it from a form Action. This gives React a pending Action lifecycle for the temporary state.

### Forgetting to update confirmed state on success

The temporary value disappears when the Action ends. If the base value is unchanged, the UI returns to the old value even though the server succeeded. Update or refresh the real data.

### Reversing the update manually on failure

If the base value was left unchanged, React already returns to it after the Action ends. Show an error; do not apply a second inverse update that could corrupt newer data.

### Putting side effects in the optimistic reducer

Reducers may be re-run to calculate a display from current state. They must stay pure and must not send requests or mutate data.

### Assuming a successful request always matches the prediction

The server can normalize, reject, merge, or recalculate a value. Use the response as the new base state.

## Key takeaways

- useOptimistic gives immediate temporary feedback while an Action is pending.
- Its base value remains the source of truth after the Action ends.
- Call its setter inside an Action or Transition.
- Use a pure reducer for optimistic list or object updates.
- Update the base state from confirmed results on success.
- Leave the base unchanged on failure so the optimistic value can roll back.
- Explain failures to users and enforce important rules on the server.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="080-useformstatus.md">080 - Pending form state with useFormStatus</a></td><td><a href="../qnas/081-useoptimistic.md">Q&amp;A</a></td><td><a href="../examples/081-useoptimistic.md">Example</a></td><td><a href="082-server-functions-and-form-actions.md">082 - Server Functions and form Actions</a></td></tr>
  </tbody>
</table>
