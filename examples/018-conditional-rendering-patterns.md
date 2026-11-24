# 018. Conditional rendering patterns - Example

## Goal

Build a small React page that demonstrates common conditional rendering decisions:

- Choose between two visible alternatives with a ternary.
- Include optional content with a boolean `&&` condition.
- Handle a zero count without accidentally rendering `0` as a message.
- Show one of several mutually exclusive request states.
- Return `null` when a component has no UI to display.

## Files

In an existing Vite React project, create or update these files:

- `src/main.jsx`
- `src/App.jsx`

No additional package or stylesheet is required.

## `src/main.jsx`

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const rootElement = document.getElementById('root');

if (rootElement === null) {
  throw new Error('Could not find the root element.');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## `src/App.jsx`

~~~jsx
function MembershipMessage({ isMember }) {
  return (
    <p>
      {isMember
        ? 'Your member account is active.'
        : 'Join to save your learning progress.'}
    </p>
  );
}

function VerifiedLabel({ isVerified }) {
  return (
    <p>
      Learner profile {isVerified && <strong>(verified)</strong>}
    </p>
  );
}

function UnreadMessage({ count }) {
  return (
    <p>
      {count > 0 && `You have ${count} unread messages.`}
      {count === 0 && 'You have no unread messages.'}
    </p>
  );
}

function RequestStatus({ status }) {
  if (status === 'loading') {
    return <p role="status">Loading your courses…</p>;
  }

  if (status === 'error') {
    return <p role="alert">Courses could not be loaded.</p>;
  }

  if (status === 'empty') {
    return <p>No courses match your search.</p>;
  }

  if (status === 'ready') {
    return <p>Course results are ready.</p>;
  }

  return <p>Waiting to start.</p>;
}

function MaintenanceNotice({ enabled }) {
  if (!enabled) {
    return null;
  }

  return <aside>Course registration is temporarily unavailable.</aside>;
}

export default function App() {
  return (
    <main>
      <h1>Conditional rendering patterns</h1>

      <section>
        <h2>Choose between two outcomes</h2>
        <MembershipMessage isMember />
        <MembershipMessage isMember={false} />
      </section>

      <section>
        <h2>Include optional content</h2>
        <VerifiedLabel isVerified />
        <VerifiedLabel isVerified={false} />
      </section>

      <section>
        <h2>Handle numeric conditions</h2>
        <UnreadMessage count={0} />
        <UnreadMessage count={3} />
      </section>

      <section>
        <h2>Show mutually exclusive request states</h2>
        <RequestStatus status="loading" />
        <RequestStatus status="error" />
        <RequestStatus status="empty" />
        <RequestStatus status="ready" />
      </section>

      <section>
        <h2>Return no UI when disabled</h2>
        <MaintenanceNotice enabled />
        <MaintenanceNotice enabled={false} />
      </section>
    </main>
  );
}
~~~

## What to notice

- `MembershipMessage` uses a ternary because both alternatives should be visible in their respective cases.
- `VerifiedLabel` uses `&&` for one optional label. `isVerified` is a boolean, so the false branch contributes no visible text.
- `UnreadMessage` compares `count` with zero. It does not put the number directly on the left side of `&&`, which would render `0` when the count is zero.
- `RequestStatus` uses early returns so each status selects one message. In an actual screen, the status value would normally represent the single current request state; the page displays several values here so you can compare the branches.
- `MaintenanceNotice` returns `null` when disabled. The second call contributes no element to the page.

## Practice

1. Add a `success` state to `RequestStatus` and show a success message.
2. Add a `count` prop to `VerifiedLabel` and display the number only when it is greater than zero.
3. Change `MembershipMessage` to use an early return, then compare which form is easier to scan.
4. Add a second optional notice and decide whether its condition should live in the parent or inside that component.
5. Rewrite the request-state branches with a `switch` and compare the readability.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [017 - Composition, containment, and reusable layouts](../lessons/017-composition-containment-and-layouts.md) | [018 - Conditional rendering patterns](../lessons/018-conditional-rendering-patterns.md) | [Open Q&A](../qnas/018-conditional-rendering-patterns.md) | [019 - Rendering collections and choosing stable keys](../lessons/019-rendering-collections-and-stable-keys.md) |