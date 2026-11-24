# 018. Conditional rendering patterns - Q&A

## Choosing a conditional pattern

### 1. What is conditional rendering?

Conditional rendering means returning or including different UI depending on a value such as a prop, a status, or a boolean condition. The component uses JavaScript to decide which JSX to return.

### 2. Is `if` a special React feature?

No. `if`, `else`, `switch`, the conditional operator, and `&&` are JavaScript. React renders the JSX value that the component returns or includes.

### 3. Why can’t I write an `if` statement directly inside JSX braces?

JSX braces contain a JavaScript expression. An `if` statement is a statement, so it cannot be used as the value inside braces. Put the `if` before the return, use a ternary expression, or calculate a value first:

~~~jsx
function Greeting({ isSignedIn }) {
  if (!isSignedIn) {
    return <p>Please sign in.</p>;
  }

  return <p>Welcome back.</p>;
}
~~~

### 4. What is an early return?

An early return exits a function before its usual final return. In a component, it can handle a special case near the top and keep the normal UI from being nested inside an `else` block.

~~~jsx
function AccountPanel({ account }) {
  if (account === null) {
    return <p>No account is selected.</p>;
  }

  return <h2>{account.name}</h2>;
}
~~~

### 5. Can a component return `null`?

Yes. Returning `null` means that component contributes no UI for that render. It is useful when the component's own props determine that it has nothing to display:

~~~jsx
function ModeratorTools({ canModerate }) {
  if (!canModerate) {
    return null;
  }

  return <button type="button">Review reports</button>;
}
~~~

### 6. Should a parent conditionally omit a component or should the component return `null`?

Either can work. Let the parent omit the child when the parent owns the decision. Let the child return `null` when its own props define whether it has UI. Choose the location that makes the responsibility clearest.

## Ternary and logical AND

### 7. What is the conditional or ternary operator?

It is a JavaScript expression with three parts: a condition, a value for the true case, and a value for the false case:

~~~jsx
isSaved ? 'Saved' : 'Not saved'
~~~

It is useful inside JSX when both outcomes should be visible alternatives.

### 8. When should I use a ternary in JSX?

Use one for a small choice between two alternatives, such as two labels or two components:

~~~jsx
{isOpen ? <DetailsPanel /> : <OpenButton />}
~~~

If either branch contains a lot of logic or nested markup, move the decision before the return or use a variable.

### 9. What does `condition && <Thing />` do?

JavaScript evaluates `condition` first. If it is truthy, the expression evaluates to `<Thing />`; otherwise, it evaluates to the condition's falsey value. React does not render boolean `false`, `null`, or `undefined` as visible text, so this pattern can add optional content when the condition is boolean.

### 10. Why can `count && <Message />` render an unwanted zero?

Logical AND returns an operand; it does not always return a boolean. If `count` is `0`, the expression evaluates to `0`, and React renders that number as text.

~~~jsx
{count && <p>New messages</p>}
~~~

When `count` is zero, the output includes `0`.

### 11. How do I safely conditionally render from a count?

Write a comparison that evaluates to a boolean:

~~~jsx
{count > 0 && <p>New messages</p>}
~~~

This makes the condition explicit and prevents zero from becoming the rendered child. The same idea applies to array lengths, such as `items.length > 0`.

### 12. When should I avoid `&&`?

Avoid it when the false case needs its own message or component, because the false case renders nothing. Use a ternary when both cases need output. Also avoid putting a number or another value that could display on the left side when it is falsey.

### 13. Does React display `false`, `null`, and `undefined` inside JSX?

They do not appear as visible text when used as children. A number such as `0` or a string such as `'0'` can appear. Check the value type and make the condition explicit when the value is not already a boolean.

## Several states and fallback values

### 14. How should I render several mutually exclusive states?

Use one `if`/`else if` chain or a `switch` before the return so one status selects one outcome:

~~~jsx
function RequestStatus({ status }) {
  if (status === 'loading') return <p>Loading…</p>;
  if (status === 'error') return <p role="alert">Request failed.</p>;
  if (status === 'empty') return <p>No results.</p>;
  if (status === 'ready') return <p>Results are ready.</p>;

  return <p>Unknown status.</p>;
}
~~~

Several independent `&&` conditions can show more than one branch if more than one condition is true. That may be appropriate for independent features, but it is usually not what you want for exclusive request states.

### 15. When is `switch` a good choice?

Use `switch` when one value, such as a status string, selects among several distinct cases. Each case should end with `break`, `return`, or another deliberate control-flow action so execution does not fall through by accident.

### 16. Can I assign JSX to a variable and render it later?

Yes. Assign a JSX element to a variable in a component, then include that variable inside the return:

~~~jsx
let content;

if (isLoading) {
  content = <p>Loading…</p>;
} else {
  content = <p>Content is ready.</p>;
}

return <section>{content}</section>;
~~~

This is useful when several branches share the same outer structure.

### 17. What is the difference between `||` and `??` in a fallback?

`value || fallback` uses the fallback for any falsey value, including `''`, `0`, and `false`. `value ?? fallback` uses the fallback only when `value` is `null` or `undefined`. Use the operator that matches what “missing” means for the data.

~~~jsx
const displayName = name ?? 'Guest';
~~~

These operators select values. They do not by themselves mean “conditionally show this component.”

### 18. Is `condition ? <Thing /> : null` valid?

Yes. It explicitly shows the true case and renders nothing for the false case. For a simple optional element, `condition && <Thing />` is shorter, as long as `condition` is boolean. Use whichever version makes the intended branches clearer.

## Readability and behavior

### 19. Are nested ternaries wrong?

They are valid JavaScript, but several nested ternaries can be hard to scan and update. Use early returns, a variable, a `switch`, or a small component when the branches become difficult to understand.

### 20. Should every conditional be written inside the returned JSX?

No. Conditions can be handled before the return with ordinary JavaScript. Keeping a complex decision outside JSX often makes the markup easier to read.

### 21. Is conditionally omitting an element the same as hiding it with CSS?

No. Conditional rendering leaves the element out of the rendered UI for that branch. CSS visibility or display rules affect presentation of an element that is still included in the markup. Choose based on whether the element should be present at all.

### 22. How do I choose among `if`, ternary, and `&&`?

- Use `if` for whole-component branches, longer logic, or multiple outcomes.
- Use a ternary for a concise choice between two explicit outcomes.
- Use `&&` for one optional piece of content when the condition is boolean.
- Use a variable or `switch` when several alternatives make inline JSX hard to follow.

The simplest readable form is usually best; no single syntax is required for every condition.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [017 - Composition, containment, and reusable layouts](../lessons/017-composition-containment-and-layouts.md) | [018 - Conditional rendering patterns](../lessons/018-conditional-rendering-patterns.md) | [Open Example](../examples/018-conditional-rendering-patterns.md) | [019 - Rendering collections and choosing stable keys](../lessons/019-rendering-collections-and-stable-keys.md) |