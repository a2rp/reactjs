# Rules of Hooks — Q&A

## Core rules

### 1. What is a Hook in React?

A Hook is a React API that lets a function component or custom Hook use React features such as state, context, Effects, refs, or memoization. Built-in examples include useState and useEffect. A custom Hook is a function that uses one or more Hooks to package reusable logic.

### 2. Where can ordinary Hooks be called?

Call ordinary Hooks at the top level of a function component or at the top level of a custom Hook. They must be called while React renders that function.

### 3. What does “at the top level” mean?

It means the Hook call is part of the component or custom Hook's normal execution path, not inside a condition, loop, nested function, callback, or after a conditional early return. Its call order must stay the same on every render.

### 4. Why must ordinary Hooks be called in the same order on every render?

React needs to match each Hook call to the correct state or other Hook data for that component instance. If a render skips or adds a call, later calls can be associated with the wrong data. React can also report that fewer or more Hooks were rendered than expected.

### 5. Is the rule about Hook order just a formatting preference?

No. It is a correctness rule that lets React associate Hook data with the right component render. It also allows React's lint tooling and compiler to analyze component behavior.

### 6. Is calling a state setter from an event handler allowed?

Yes. Declare the Hook at the top level, then call its setter from an event handler to schedule a state update. It is the Hook call itself that must not happen in the event handler.

~~~jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount((current) => current + 1)}>
      Count: {count}
    </button>
  );
}
~~~

### 7. Can a Hook be called from a click handler?

No. The handler runs after rendering in response to an event. Declare Hooks at the component's top level and use the event to update their state or call an ordinary function.

### 8. Can a Hook be called in a regular utility function?

No. A regular helper is not a React component or custom Hook. If the function uses Hooks, give it a custom Hook name starting with use and call it at the top level of a component or another custom Hook.

### 9. Can a custom Hook call another custom Hook?

Yes. Reusing other Hooks is a central purpose of custom Hooks. The nested call must still be at the custom Hook's top level, not conditional or looped.

### 10. Can a class component call Hooks?

No. Hooks can be called from function components and custom Hooks. Class components use their supported class APIs or can be converted to function components when that is appropriate.

### 11. Can a Hook be called at module scope?

No. Module scope runs outside React rendering. Keep Hook calls inside a function component or custom Hook.

### 12. Can a component that uses Hooks be called like a normal function?

No. Render it through JSX, such as a component element. React needs to call the component so it can manage that component's identity and Hook behavior.

## Conditions, loops, and returns

### 13. Why is calling useState inside an if statement invalid?

The condition might be true in one render and false in another, changing the number and order of Hook calls. Call useState unconditionally, then use the condition to choose its initial value or determine what to render.

### 14. How can an Effect run only when a condition is true?

Call useEffect unconditionally at the top level and check the condition inside its callback:

~~~jsx
useEffect(() => {
  if (!isSignedIn) {
    return;
  }

  const subscription = subscribeToAccount(userId);
  return () => subscription.unsubscribe();
}, [isSignedIn, userId]);
~~~

### 15. Can a component return early before calling a Hook?

Not if that early return can skip Hook calls that happen later in the function. Move ordinary Hook calls above the conditional return so every render reaches them in the same order.

### 16. Is this valid if the Hook is below an early return?

~~~jsx
if (!user) {
  return <p>Sign in</p>;
}

const theme = useContext(ThemeContext);
~~~

No. When user is missing, the return skips useContext; when a user is present, it runs. Move the Hook before the return.

### 17. Why can’t a Hook be called once for every item inside map?

The number of Hook calls depends on the array length and can change between renders. Instead, return a child component for each item and let each child call its own Hooks.

### 18. Is this valid?

~~~jsx
items.map((item) => {
  const [selected, setSelected] = useState(false);
  return <li>{item.name}</li>;
});
~~~

No. This calls a Hook inside a loop callback. Render a component such as ItemRow for each item, give it a stable key, and place useState inside ItemRow.

### 19. Can a Hook be called inside a function passed to useMemo?

No. Call the Hook at the component's top level and use the result in the memo callback. The callback is not itself a component or custom Hook render.

### 20. Can a Hook be called inside a useEffect callback?

No. Call useEffect at the component's top level. Its callback may use ordinary JavaScript functions and captured values, but must not call another Hook.

### 21. Can Hooks be called in async functions?

Ordinary Hooks cannot be called inside async functions or asynchronous callbacks. Declare the Hook in the component or custom Hook, then use its setter after the asynchronous work completes.

### 22. Can ordinary Hooks be called inside try, catch, or finally?

No. Keep ordinary Hook calls outside those blocks. Handle errors through regular JavaScript control flow around operations that do not require Hook calls.

### 23. Can a custom Hook be called conditionally?

No. Custom Hooks follow the same ordering rules as built-in ordinary Hooks. If behavior should vary, pass the condition as an argument and handle it inside the custom Hook, or render a separate child component.

## Custom Hooks and state

### 24. How can a regular helper be distinguished from a custom Hook?

A custom Hook name begins with use followed by a capital letter and the function calls one or more Hooks. A helper that only calculates or transforms values should generally have an ordinary name, such as filterProducts.

### 25. Should every repeated line be extracted into a custom Hook?

No. Extract a custom Hook when it makes reusable stateful behavior or synchronization clearer. A tiny wrapper around one obvious operation may add indirection without helping.

### 26. Does a custom Hook share one state variable among all its callers?

No. Custom Hooks share logic, not state. Each call gets its own Hook state associated with the component that called it. To share state, lift ownership to a common parent or use an appropriate shared store.

### 27. Can a custom Hook decide not to start an Effect?

Yes. The useEffect call should remain unconditional, while the Effect callback can check a condition and return early. Its dependencies should include the reactive values it reads.

### 28. Is a function named getTheme allowed to call useContext?

No. A regular function should not call Hooks. Rename and design it as a custom Hook, such as useTheme, and call it at the top level of a component or another custom Hook.

## The special use API

### 29. Is React's use API an exception to the top-level rule?

Yes. React's use API can be called conditionally and inside loops. This exception applies specifically to use; it does not apply to useState, useEffect, useContext, or custom Hooks.

### 30. Where may React's use API be called?

It must still be called from a function component or a custom Hook. It cannot be called from a regular helper, event handler, or class component.

### 31. Can React's use API be placed in try/catch?

No. The use API must not be called inside try, catch, or finally. Its conditional and loop exception does not remove this restriction.

### 32. Does the use API exception mean custom Hooks can be conditional?

No. A custom Hook such as useOnlineStatus follows ordinary Hook ordering rules. Only the dedicated React use API has the documented conditional and loop exception.

## Tooling and debugging

### 33. Which tool helps find invalid Hook calls?

The React Hooks ESLint plugin checks many Rules of Hooks violations. Keep it enabled and resolve warnings instead of suppressing them without understanding the reason.

### 34. What does “Rendered fewer hooks than expected” usually suggest?

A render path skipped a Hook that a previous render called, often because a Hook is inside a condition or after an early return. Compare render paths and move ordinary Hook calls to the top level.

### 35. What does “Rendered more hooks than expected” usually suggest?

A render path added Hook calls that were skipped before, commonly because a condition or loop changed. Keep the ordinary Hook call sequence fixed.

### 36. Why should custom Hooks have clear names?

The use prefix signals that the function may call Hooks and must follow Hook call rules. It also helps lint tools identify custom Hook boundaries and helps readers find stateful behavior.

## Short recap

- Ordinary Hooks are called only while React renders a function component or custom Hook.
- Ordinary Hooks stay at the top level and in the same order on every render.
- Event handlers may call state setters, but may not call Hooks.
- Conditions go inside Hook logic; repeated item behavior belongs in child components.
- Custom Hooks reuse stateful logic, but each call has independent state.
- React's use API has a specific conditional and loop exception; other Hooks do not.
