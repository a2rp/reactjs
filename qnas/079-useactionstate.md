# Managing Submissions with useActionState - Q&A

## API and Action arguments

### 1. What does useActionState do?

It stores the latest value returned by an Action and gives the component a wrapped dispatcher plus a pending flag.

### 2. What does it return?

A three-item array: current state, a wrapped Action dispatcher, and isPending.

### 3. What is the initial state used for?

It supplies the state before the Action first runs. A later change to the initialState argument does not reset an already-used Hook.

### 4. What arguments does a form Action receive when wrapped?

The previous state first, then the submitted FormData.

### 5. What is the signature without useActionState?

A normal React form Action receives FormData as its first argument. Wrapping it changes the argument order.

### 6. Why does the wrapped Action need to return a value?

The returned value becomes the next state shown by the component.

### 7. Should the Action mutate previousState?

No. Return a new state value so React can render the new result predictably.

### 8. Can the Action be asynchronous?

Yes. It can await network or other asynchronous work. isPending stays true while the wrapped Action is running.

### 9. Where should useActionState be called?

At the top level of a React component or custom Hook, following its Hook rules.

### 10. What kinds of values can state contain?

Any appropriate React state value. When using Server Functions, keep values serializable according to the framework's requirements.

## Forms, validation, and state

### 11. How do I connect the dispatcher to a form?

Pass the returned dispatcher as the form's action prop.

### 12. Does the form submission need a manual startTransition?

No. React wraps a dispatcher passed to a form Action prop in a Transition.

### 13. How can I keep a validation error visible?

Return a state object with an error message or field errors, then render those values beside the form.

### 14. Why return submitted values too?

The UI can use them as updated defaultValue values so the user can correct their input instead of retyping it.

### 15. Does React reset uncontrolled inputs after a returned validation error?

The Action resolved normally, so it is considered successful. Include submitted values in returned state and update the fields' defaults so they are preserved during the reset.

### 16. How can successful submission clear the fields?

Return a state where submitted values are empty or null and render those as the new defaults. React's automatic reset then clears the uncontrolled fields.

### 17. What happens to controlled inputs after submission?

Their displayed values are driven by React state. Update that state deliberately; DOM form reset does not replace the controlled value.

### 18. Can native constraints replace Action validation?

No. required and type constraints help users, but the Action or trusted server must validate submitted values again.

### 19. How should file input values be validated?

FormData may return a File. Check the type, size, contents, and whether the current user is allowed to upload it.

### 20. What belongs in initialState?

A complete initial result shape that lets the UI render before the first submission, such as status, message, errors, and submitted values.

## Pending and errors

### 21. What does isPending indicate?

It is true while this useActionState Action is pending. It is useful for disabling a submit button and displaying progress.

### 22. How is that different from useFormStatus?

useActionState returns pending to the component that owns the Action. useFormStatus reads a parent form's status from a child nested inside that form.

### 23. Should I use both pending APIs?

Usually one is enough. Use the API that fits component ownership; useFormStatus is useful for a reusable nested control.

### 24. Should every field be disabled while pending?

Not always. Disable only controls whose changes or duplicate submissions would make the operation confusing or unsafe.

### 25. Should expected validation failures throw?

Usually no. Return them as normal Action state so the form can display them and remain usable.

### 26. What should happen to an unexpected exception?

It can be thrown to the nearest Error Boundary. Ensure the user gets a recovery path and no sensitive implementation detail.

### 27. What happens to queued calls if an Action throws?

A thrown Action error can cancel subsequent queued dispatches and be rethrown to an Error Boundary. Return recoverable expected errors as state instead.

### 28. What is the benefit of queueing calls?

Each Action receives the previous result, so state-dependent submissions are processed in order.

### 29. What is the tradeoff of queueing?

Independent operations cannot finish in parallel through that queue; several slow submissions may take longer. Use a different state/concurrency design when parallelism is required.

### 30. Does aborting a request undo a server mutation?

No. Canceling a browser request does not necessarily undo work already performed on the server. Design idempotency and mutation recovery separately.

## Dispatch and reset behavior

### 31. Can the dispatcher be called outside a form?

Yes, but call it inside a Transition, such as startTransition, so pending state is tracked correctly.

### 32. What payload does an outside-form dispatch pass?

The supplied argument becomes the Action's second argument after previousState. For a form it is FormData.

### 33. Does useActionState provide a reset function?

No built-in reset function is provided. Design a reset Action, dispatch a reset payload, or remount the component with a new key.

### 34. Is a reset the same as clearing a form?

Not necessarily. Clearing fields, clearing the last result, and clearing errors may be separate decisions.

### 35. What is the optional permalink argument for?

In a framework supporting Server Functions, it can identify the destination for a submission before hydration on pages with dynamic content. The destination must render the matching form/action/permalink.

### 36. Does permalink make a Vite app server-rendered?

No. It does not add framework or server support. It only participates in a supported progressive-enhancement setup.

## Choosing the right tool

### 37. How is useActionState different from useReducer?

useReducer is commonly used for pure UI-state transitions. useActionState is designed to store results from Actions and can perform asynchronous side effects.

### 38. How is it different from useState?

useState stores arbitrary local values. useActionState additionally coordinates an Action's previous result, dispatcher, and pending state.

### 39. When should I avoid useActionState?

Avoid it when submissions should run independently in parallel, when the operation does not need a returned state, or when a query/data library already owns the same state.

### 40. What should I check before shipping it?

Check the argument order, returned state shape, validation and field preservation, pending feedback, thrown-error behavior, queue semantics, reset behavior, and server authorization.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="078-react-form-actions.md">078 - React Form Actions and Asynchronous Submissions - Q&amp;A</a></td><td><a href="../lessons/079-useactionstate.md">Lesson</a></td><td><a href="../examples/079-useactionstate.md">Example</a></td><td><a href="080-useformstatus.md">080 - Q&amp;A: Pending form state with useFormStatus</a></td></tr>
  </tbody>
</table>
