# 012. Strict Mode and development checks

## Learning goals

By the end of this lesson, you should be able to:

- Enable Strict Mode around a React component tree.
- Explain why some render logic runs an extra time in development.
- Recognize the extra Effect and ref-callback setup/cleanup checks.
- Treat Strict Mode output as a signal to investigate code, not as a production behavior.
- Fix the underlying impurity or missing cleanup instead of hiding the diagnostic.

## What Strict Mode does

Strict Mode is a development aid. It enables extra checks that help expose mistakes such as a component changing data while rendering or an Effect that starts work without cleaning it up.

To enable it for the whole app, import StrictMode from React and wrap the root component when rendering:

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';

const container = document.getElementById('root');

if (container === null) {
  throw new Error('The root container was not found in index.html.');
}

const root = createRoot(container);

root.render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

StrictMode is a React component, not a DOM element. It does not add a wrapper to the browser page. It applies its development checks to the components inside it and their descendants. It accepts no props.

React recommends enabling Strict Mode for a new app. A framework may configure it for you; follow the framework's instructions instead of adding a competing root setup.

## What checks does it enable?

### Extra render calls help reveal impure code

React expects rendering to be pure: given the same inputs, component rendering should calculate the same UI without changing outside data or causing actions. In development, Strict Mode calls functions that should be pure an extra time to help reveal code that breaks this rule.

For example, do not append directly to an array received from a prop:

~~~jsx
function TopicList({ topics }) {
  topics.push({ id: 'create', label: 'Create a new topic' });

  return (
    <ul>
      {topics.map((topic) => (
        <li key={topic.id}>{topic.label}</li>
      ))}
    </ul>
  );
}
~~~

This changes data owned by another part of the app. Each render may append another item. Create new data instead of mutating the input:

~~~jsx
function TopicList({ topics }) {
  const displayedTopics = [...topics, { id: 'create', label: 'Create a new topic' }];

  return (
    <ul>
      {displayedTopics.map((topic) => (
        <li key={topic.id}>{topic.label}</li>
      ))}
    </ul>
  );
}
~~~

An extra render can make a mutation show up twice. Fix the mutation; do not try to make rendering perform an action only once.

React may also call certain initializer or updater functions more than once in development when checking that they are pure. Event handlers are not called extra just because Strict Mode is enabled. A click-triggered action still belongs in the handler for that action.

### Effects get an extra setup and cleanup check

An Effect synchronizes a component with something outside React. When Strict Mode is enabled at the root, React performs an additional setup-and-cleanup cycle for Effects in development when a component first mounts. A typical sequence is:

~~~text
setup -> cleanup -> setup
~~~

This checks that cleanup correctly undoes the work from setup. For example, an Effect that subscribes to an event or connects to a service should return cleanup that unsubscribes or disconnects.

~~~jsx
useEffect(() => {
  const connection = createConnection(roomId);
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId]);
~~~

The development check may make a connection appear to start, stop, then start again. With correct cleanup, there should not be two active connections. Effects and their dependency rules are covered in later chapters; the key point here is that setup needs matching cleanup when it creates something that must be undone.

Do not use an Effect to perform a one-time user action such as submitting an order just because Strict Mode makes an Effect run again during development. A user-triggered action belongs in its event handler. Use Effects for synchronization caused by the component being present or its inputs changing.

### Ref callbacks get an extra setup and cleanup check

Callback refs can also receive an extra setup and cleanup cycle in development. If a callback ref stores a node in a collection, it must remove that node when the callback receives null or its cleanup runs. Otherwise, the collection can contain stale entries.

~~~jsx
function ListItem({ registerNode }) {
  return <li ref={registerNode}>Topic</li>;
}
~~~

The ref callback details come later. For now, if a node seems to be registered twice in development, check that the callback has a matching cleanup.

### Deprecated API usage may be reported

Strict Mode also checks for usage of certain deprecated APIs and reports diagnostics in development. Read the warning and update the code or dependency that triggers it. Strict Mode does not automatically rewrite old APIs for you.

## Development behavior versus production

Strict Mode's additional render, Effect, ref-callback, and deprecated-API checks are development-only. They do not mean the production app will intentionally render each component twice or run every Effect twice on every update.

The checks do not guarantee that every bug will be found, and they do not replace tests or careful review. They make some common mistakes easier to notice while developing.

When debugging, distinguish a diagnostic from a real duplicated action:

- Repeated console output can reveal that render logic ran extra times.
- Repeated setup may be paired with cleanup as part of the development check.
- A correct Effect should leave one active subscription or connection after setup and should release it during cleanup.
- Removing Strict Mode may hide the symptom while leaving the underlying impurity or cleanup bug in place.

## Scope Strict Mode to part of an app

You can wrap a subtree if you are adding Strict Mode gradually:

~~~jsx
function App() {
  return (
    <>
      <SiteHeader />
      <StrictMode>
        <CourseDashboard />
      </StrictMode>
      <SiteFooter />
    </>
  );
}
~~~

Import StrictMode from React in that file. Components inside the boundary receive the checks; siblings outside do not.

For new applications, wrapping the root is the clearest default. When Strict Mode is enabled only below the root, React avoids development behavior that could not happen in production, such as re-running only child Effects on the initial mount without also re-running their parents' Effects.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming an extra render means React is broken | Development checks intentionally repeat some pure logic to reveal impurity. | Make rendering pure and avoid mutating inputs. |
| Appending to a prop array during render | Every render changes shared data again. | Create a new array or object for derived values. |
| Removing Strict Mode to hide a duplicate | The underlying mutation or missing cleanup remains. | Find and correct the cause of the repeated behavior. |
| Treating the Effect setup check as two production connections | The development sequence includes cleanup between setups. | Ensure cleanup fully undoes the setup work. |
| Sending a purchase request from an Effect | Development checks can expose a side effect tied to rendering instead of a user action. | Send the request from the event handler that represents the purchase. |
| Registering a callback ref without removing its node | Extra setup can leave duplicate or stale entries in a collection. | Implement matching ref cleanup. |
| Expecting Strict Mode to find every bug | It runs a set of focused development checks, not a complete correctness proof. | Keep using tests, debugging, and code review. |

## Key takeaways

- Strict Mode is a development-only diagnostic wrapper; it adds no DOM element.
- It helps find impure render logic, missing Effect cleanup, missing ref cleanup, and deprecated API use.
- Extra calls are intentional checks, not production behavior to reproduce or suppress.
- Rendering must not mutate inputs or perform user actions.
- Give external work started by Effects or refs a matching cleanup.
- Enable Strict Mode for the full app when possible, and fix the issues its checks reveal.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [011 - Create a root and render a React application](011-create-a-root-and-render-an-application.md) | [Open Q&A](../qnas/012-strict-mode-and-development-checks.md) | [Open Example](../examples/012-strict-mode-and-development-checks.md) | [013 - Function components and component naming](013-function-components-and-naming.md) |
