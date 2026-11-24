# Transitions with useTransition and startTransition

Lesson 063 introduced update priority. This lesson focuses on applying Transitions in components and async Actions: where to call the API, how pending work is reported, and what the API does and does not manage.

## Learning goals

By the end, you should be able to:

- choose between the Hook form, **useTransition**, and the standalone **startTransition** function;
- mark a non-urgent state update at the point where its setter is available;
- keep urgent input and feedback state responsive while an Action is pending;
- design a useful pending state without replacing the whole interface;
- use an async Action and wrap state updates after **await** when they should remain Transition updates;
- handle failures and request ordering deliberately;
- explain how Transitions interact with already revealed Suspense content.

## The Hook form

Call **useTransition** at the top level of a component or custom Hook. It takes no arguments and returns a pair:

~~~jsx
const [isPending, startTransition] = useTransition();
~~~

- **startTransition** marks the state updates scheduled by an Action as non-urgent Transition updates.
- **isPending** is true while the Transition started through this Hook has pending work.

Keep a normal state setter for the selected view. Call it from the Action:

~~~jsx
import { useState, useTransition } from "react";

function WorkspaceTabs() {
  const [tab, setTab] = useState("overview");
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <section aria-busy={isPending}>
      <button type="button" onClick={() => selectTab("overview")}>
        Overview
      </button>
      <button type="button" onClick={() => selectTab("reports")}>
        Reports
      </button>
      <p role="status" aria-live="polite">
        {isPending ? "Updating view…" : "View is ready."}
      </p>
      <WorkspacePanel tab={tab} />
    </section>
  );
}
~~~

The Action callback runs immediately. **startTransition** is not a timer. React assigns Transition priority to state updates scheduled in the callback and can interrupt the render work if a more urgent update arrives.

**isPending** is useful for local feedback, such as styling the selected tab or indicating that a panel is being prepared. It is a boolean, not a percentage or a measure of remaining time. Fast updates may finish before a pending message is painted.

## Pending state belongs near the affected UI

Keep useful existing content visible while a Transition is pending. Prefer a local status, a subtle visual treatment, or a pending style on the relevant control. Replacing the entire application with a spinner can hide the interface the user is trying to use.

The Hook must be called in a component. If a child owns the interaction, it can call **useTransition** and provide its **isPending** value to the relevant part of that component. If a parent needs to coordinate pending feedback for several interactions, place the Hook where those interactions are coordinated and pass the status down as a prop.

Do not disable every control just because **isPending** is true. Disable the specific action when repeating it would be unsafe or confusing. For example, an application may disable a one-time payment submission while it is saving, but leave navigation or unrelated controls usable.

## Standalone startTransition

Import **startTransition** directly from React when you have a state setter but cannot call a Hook where the update is initiated, such as in a non-component helper:

~~~jsx
import { startTransition } from "react";

function navigate(setPage, nextPage) {
  startTransition(() => {
    setPage(nextPage);
  });
}
~~~

The standalone function does not return **isPending** and is not tied to one component. Use **useTransition** when the component needs pending feedback. Either API still needs access to the setter for the state being updated. When the value comes from props and the setter is elsewhere, consider **useDeferredValue** at the slower consumer instead.

## Actions: synchronous or asynchronous work

React calls the callback passed to **startTransition** an Action. An Action may schedule state updates and may also perform async work. An async Action lets **isPending** remain true while its awaited work is pending:

~~~jsx
const [isPending, startTransition] = useTransition();

function saveDraft(draft) {
  startTransition(async () => {
    const savedDraft = await saveDraftToServer(draft);

    startTransition(() => {
      setSavedDraft(savedDraft);
    });
  });
}
~~~

The request begins when the Action runs. This API does not move the request to a background thread. While the request is waiting, the UI can respond to urgent state updates. The state update after **await** is wrapped in another **startTransition** because, in current React, updates after an async boundary are not automatically marked as Transition updates.

Keep immediate feedback separate from the non-urgent result. For example, a field should continue to reflect typing while a save is in progress. A result, chart, or selected view can be committed as a Transition after the request returns.

If an Action prop may be synchronous or asynchronous, await it inside the Transition:

~~~jsx
function SaveButton({ action, children }) {
  const [isPending, startTransition] = useTransition();

  return (
    <button
      type="button"
      disabled={isPending}
      onClick={() => {
        startTransition(async () => {
          await action();
        });
      }}
    >
      {isPending ? "Saving…" : children}
    </button>
  );
}
~~~

Awaiting the callback keeps the Transition pending for its async work and lets the callback support either a synchronous or an asynchronous implementation. Disable the button here only because this example treats repeated saves as inappropriate.

### Actions do not make synchronous work non-blocking

This is still synchronous work on the main thread:

~~~jsx
startTransition(() => {
  const result = runHugeSynchronousCalculation();
  setResult(result);
});
~~~

React can prioritize rendering caused by **setResult**, but it cannot interrupt **runHugeSynchronousCalculation** halfway through. Optimize or split an expensive computation, or choose a worker or other suitable approach when the calculation itself blocks the browser.

## Handle errors and request ordering

Transitions do not remove the need for error handling. An error thrown by an Action can be shown by an Error Boundary around the component. If the app wants an inline error message, catch the error in the Action and update error state deliberately:

~~~jsx
function handleSave(draft) {
  setError("");

  startTransition(async () => {
    try {
      const savedDraft = await saveDraftToServer(draft);
      startTransition(() => {
        setSavedDraft(savedDraft);
      });
    } catch {
      // Error feedback should appear immediately.
      setError("The draft could not be saved. Try again.");
    }
  });
}
~~~

The example keeps the error message urgent so the user sees a failure promptly. If error UI itself is expensive, choose its update priority intentionally.

An Action does not automatically make arbitrary overlapping network requests safe. If several requests can finish out of order, an older response may overwrite newer data unless the application handles ordering. Choose a request strategy that fits the task: disable duplicate submissions, cancel obsolete work, attach request IDs, or use a React abstraction that manages the action sequence.

For forms, server mutations, and common pending/error workflows, React provides higher-level options such as **useActionState** and form Actions. Use those when they match the problem instead of rebuilding their state and ordering behavior by hand.

## Transitions and Suspense

A Transition can keep already revealed content visible while React prepares an update that suspends. This avoids hiding a usable page behind its closest fallback every time the user navigates to a view that is not ready yet.

The Transition does not remove Suspense fallbacks. A nested Suspense boundary may still show its own fallback for content that has not been revealed. Put boundaries around meaningful loading regions and choose which content should remain visible during navigation.

Framework routers may mark navigation updates as Transitions for this reason. Application code should generally use the router’s navigation API instead of wrapping internal router state setters that the application does not own.

## Common mistakes

- Calling **useTransition** inside an event handler, condition, or loop instead of at the top level of a component or custom Hook.
- Wrapping a controlled text input’s value update in a Transition.
- Expecting the callback passed to **startTransition** to run later.
- Expecting one long synchronous function to be interruptible just because its setter is wrapped.
- Assuming **isPending** is a percentage or will always be visibly rendered.
- Removing all existing content while a non-urgent view is prepared.
- Wrapping state updates after **await** in nothing and assuming they keep Transition priority.
- Assuming Transition priority prevents older network responses from overwriting newer results.
- Expecting standalone **startTransition** to provide **isPending**.
- Disabling unrelated controls whenever any Transition is pending.

## Summary

- **useTransition** provides **[isPending, startTransition]** to a component; standalone **startTransition** marks updates without a pending flag.
- An Action callback runs immediately. State updates scheduled inside it get Transition priority.
- Async Actions can keep **isPending** true while awaited work is pending; wrap state updates after **await** in another **startTransition** when those updates should remain non-urgent.
- Keep input values and immediate error feedback responsive, and show pending status near the affected UI.
- Transitions do not interrupt synchronous calculations, guarantee request ordering, or replace error handling.
- Transitions work with Suspense to avoid hiding already revealed content, while nested boundaries can still show their own fallbacks.
