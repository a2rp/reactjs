# 018. Conditional rendering patterns

## Learning goals

By the end of this lesson, you should be able to:

- Choose between `if`, early returns, the conditional operator, and `&&` for UI branches.
- Render nothing for a condition when that is the intended result.
- Use a variable or `switch` when several branches make JSX hard to read.
- Avoid rendering an unexpected `0` from a numeric `&&` condition.
- Represent mutually exclusive interface states clearly.

## Conditional rendering uses JavaScript

A component can show different UI when its props or other values change. React does not add a separate conditional syntax: write ordinary JavaScript control flow and return the JSX for the case you need.

~~~jsx
function Greeting({ isSignedIn, name }) {
  if (isSignedIn) {
    return <p>Welcome back, {name}.</p>;
  }

  return <p>Sign in to continue.</p>;
}
~~~

The component returns one branch. The `if` statement runs before JSX is returned, so it can choose an entire page, section, or component tree.

## Use `if` statements for clear branches

Use `if` when branches contain several elements, need additional logic, or are easier to understand as separate blocks. An early return can keep the main case from being nested inside several levels of JSX:

~~~jsx
function AccountPanel({ account }) {
  if (account === null) {
    return <p>No account is selected.</p>;
  }

  return (
    <section>
      <h2>{account.name}</h2>
      <p>Account details are ready.</p>
    </section>
  );
}
~~~

An early return is ordinary JavaScript. Once it runs, the component returns that branch and does not continue to the later return.

## Return `null` when a component has no UI to show

A component may return `null` when it should render no UI for its current props:

~~~jsx
function ModeratorTools({ canModerate }) {
  if (!canModerate) {
    return null;
  }

  return <button type="button">Review reports</button>;
}
~~~

The component can still be called by its parent; its output simply contributes no visible element. If the parent already knows whether the component belongs in the tree, it can conditionally include it there instead:

~~~jsx
function DiscussionPage({ canModerate }) {
  return (
    <main>
      <h1>Discussion</h1>
      {canModerate && <ModeratorTools canModerate={canModerate} />}
    </main>
  );
}
~~~

Use the approach that makes responsibility clear. The child can guard itself when its own props determine whether it has UI. The parent can omit it when the parent owns the condition.

## Use the conditional operator for two alternatives

The conditional operator, often called a ternary, is an expression with a condition and two results:

~~~jsx
condition ? valueWhenTrue : valueWhenFalse
~~~

Inside JSX braces it can select between two pieces of content:

~~~jsx
function SaveStatus({ isSaved }) {
  return (
    <p>
      {isSaved ? 'Your changes are saved.' : 'You have unsaved changes.'}
    </p>
  );
}
~~~

Both branches are explicit. This makes a ternary useful when either outcome should display something, such as a label, button, or different component:

~~~jsx
function EditControl({ canEdit }) {
  return canEdit
    ? <button type="button">Edit profile</button>
    : <p>You do not have permission to edit this profile.</p>;
}
~~~

For longer JSX branches, put each side on multiple lines and use parentheses. If a ternary becomes difficult to scan, use `if` statements before the return or assign the chosen JSX to a variable.

## Use `&&` for optional content

The logical AND operator is useful when the true case adds one piece of content and the false case should add nothing:

~~~jsx
function Profile({ isVerified }) {
  return (
    <section>
      <h2>Profile</h2>
      {isVerified && <span>Verified account</span>}
    </section>
  );
}
~~~

JavaScript evaluates the left side first. If it is truthy, the expression evaluates to its right side; otherwise, the expression evaluates to the left side. React does not display `false`, `null`, or `undefined` as text in JSX, so a boolean condition is a good fit for this pattern.

### Avoid the numeric `&&` pitfall

Do not use a number directly as the left side when zero should mean “show nothing”:

~~~jsx
function Inbox({ unreadCount }) {
  return (
    <section>
      <h2>Inbox</h2>
      {unreadCount && <p>You have unread messages.</p>}
    </section>
  );
}
~~~

When `unreadCount` is `0`, JavaScript returns `0` from the expression. React renders that number, so the page shows an unwanted `0`. Make the condition explicitly boolean:

~~~jsx
{unreadCount > 0 && <p>You have unread messages.</p>}
~~~

This also makes the rule easy to read: show the message when the count is greater than zero. Be careful with other non-boolean values too. If a condition's value might itself be meaningful text or a number, compare it explicitly or convert it to a boolean before using `&&`.

## Choose the pattern that matches the decision

| Situation | Useful pattern | Example shape |
| --- | --- | --- |
| Choose a whole component result or handle a longer branch | `if` and `return` | `if (loading) return <Spinner />;` |
| Choose between exactly two visible alternatives | Ternary | `{isOpen ? <Panel /> : <OpenButton />}` |
| Add one optional piece of content | Boolean `&&` | `{hasError && <ErrorMessage />}` |
| Hide the output of a component | Return `null` | `if (!visible) return null;` |
| Several branches or more involved decisions | Variable or `switch` before JSX | Set `content`, then render `{content}` |

These patterns all use JavaScript conditions. Pick the simplest form that makes the possible outcomes clear. Do not choose a compact expression just to fit all logic inside JSX.

## Use a variable for a complex JSX choice

When a component has several possible states, compute the content first and keep the returned JSX structure in one place:

~~~jsx
function RequestStatus({ status }) {
  let content;

  if (status === 'loading') {
    content = <p>Loading your request…</p>;
  } else if (status === 'error') {
    content = <p role="alert">The request could not be completed.</p>;
  } else if (status === 'empty') {
    content = <p>No results were found.</p>;
  } else if (status === 'ready') {
    content = <p>Your results are ready.</p>;
  } else {
    content = <p>Unknown request status.</p>;
  }

  return <section aria-live="polite">{content}</section>;
}
~~~

Only one branch assigns `content`. The shared `<section>` remains in one place, which makes its attributes and structure easier to maintain.

For a fixed set of distinct values, a `switch` can make the alternatives even more visible:

~~~jsx
function StatusMessage({ status }) {
  let message;

  switch (status) {
    case 'loading':
      message = 'Loading…';
      break;
    case 'error':
      message = 'Something went wrong.';
      break;
    case 'ready':
      message = 'Content is ready.';
      break;
    default:
      message = 'Waiting to start.';
  }

  return <p>{message}</p>;
}
~~~

Use `switch` or an `if`/`else if` chain outside the JSX when the state values are mutually exclusive. Several independent `&&` expressions may display more than one state at once if their conditions overlap.

## Use explicit conditions for fallbacks

JavaScript truthiness can make fallback expressions shorter, but the exact values matter. For example, `name || 'Guest'` uses `Guest` when `name` is an empty string, `0`, `false`, `null`, or `undefined`. If only `null` and `undefined` mean “missing,” use the nullish coalescing operator:

~~~jsx
function Welcome({ name }) {
  const displayName = name ?? 'Guest';
  return <p>Welcome, {displayName}.</p>;
}
~~~

This is a value fallback, not a way to include or omit JSX. Use `&&` or a conditional operator when the condition controls whether content is rendered.

## Keep conditional markup readable

Avoid deeply nested ternaries and long expressions with several logical operators:

~~~jsx
// Hard to scan: several choices are compressed into one expression.
const label = loading ? 'Loading' : error ? 'Error' : ready ? 'Done' : 'Waiting';
~~~

Use a variable, early returns, or a small component to name the decision instead. Clear branches are easier to change and make it less likely that two states appear together by mistake.

Also distinguish between omitting an element and merely hiding it with CSS. A conditional that does not include an element leaves it out of the rendered UI. CSS visibility rules keep the element in the document and affect its visual presentation instead. Choose based on whether the element should be present at all.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Writing an `if` statement inside JSX braces | JSX braces expect an expression, while `if` is a statement. | Put the `if` before `return`, or use a ternary expression. |
| Using `count && <Message />` when `count` can be zero | The expression evaluates to and renders `0`. | Use `count > 0 && <Message />`. |
| Using `&&` when the false case needs a message or control | The false case shows nothing. | Use a ternary with both outcomes. |
| Nesting several ternaries | The branches become difficult to understand. | Use `if`/`else if`, `switch`, or a named component. |
| Using separate conditions for states that exclude one another | Overlapping conditions can render multiple state messages. | Model exclusive states with one `if`/`else if` chain or `switch`. |
| Returning `null` without making the condition clear | A parent or reader may not understand why a child is absent. | Name the condition clearly and decide whether the parent should omit the child. |
| Using `||` when empty text is a valid value | An empty string is treated as missing. | Use `??` when only `null` and `undefined` should trigger the fallback. |

## Key takeaways

- Conditional rendering is JavaScript control flow that selects the JSX a component returns.
- Use `if` and early returns for clear branches or whole component outcomes.
- Use a ternary when both alternatives should be represented.
- Use `&&` for optional content only when its left side is a boolean condition.
- A numeric zero on the left of `&&` is rendered as `0`; compare the number explicitly.
- Use a variable or `switch` when several exclusive states make inline JSX hard to read.
- Use `null` when a component should render no UI, and keep the decision understandable.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [017 - Composition, containment, and reusable layouts](017-composition-containment-and-layouts.md) | [Open Q&A](../qnas/018-conditional-rendering-patterns.md) | [Open Example](../examples/018-conditional-rendering-patterns.md) | [019 - Rendering collections and choosing stable keys](019-rendering-collections-and-stable-keys.md) |