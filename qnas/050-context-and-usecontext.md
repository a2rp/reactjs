# Context and useContext - Q&A

## Foundations

### 1. What problem does context solve?

Context lets a component make a value available to descendants without forwarding it through every intermediate component. It can reduce repetitive prop passing when many components in a subtree need the same value.

### 2. Does context make props obsolete?

No. Props are usually clearer for local parent-to-child data flow. Context is useful when a value is needed by many descendants or when intermediate components would only forward it.

### 3. What does createContext return?

It returns a context object. The object identifies a particular context channel and is used both by a provider and by useContext.

### 4. Where should a context object usually be created?

At module scope, outside components. Creating it during rendering would create a different context object on each render, so providers and consumers might no longer refer to the same context.

### 5. What is a context provider?

A provider is a component in the tree that supplies a value for its context to descendants. In React 19 and later you can render the context object itself as the provider; earlier versions use its Provider property.

### 6. How does a component read context?

Call useContext with the context object at the top level of a function component or custom Hook. React returns the value from the closest matching provider above that consumer.

### 7. Does context pass values upward or sideways?

No. A provider's value is available to components rendered below it in that branch of the tree. It does not pass to a parent, sibling, or component outside the provider.

### 8. What does “closest provider” mean?

Starting at the component that calls useContext, React searches upward through its ancestors and uses the first provider for the same context object. A nested provider overrides an outer provider for its own descendants.

### 9. Can two different contexts override each other?

No. Each context object is separate. A ThemeContext provider changes only ThemeContext consumers; it has no effect on LocaleContext consumers.

## Defaults and provider placement

### 10. When is the createContext default value used?

Only when there is no matching provider above the component calling useContext. The default is a static fallback and does not change over time.

### 11. Does a provider with value={undefined} cause useContext to use the default?

No. The provider exists, so its value is undefined. React uses the default only when no matching provider exists in the ancestor tree.

### 12. Can a component read a provider that it returns itself?

No. useContext only searches above the component currently rendering. A provider in that component's returned JSX is below it and affects its descendants, not the component itself.

### 13. How can one component own a provider and also display its value?

Put the provider above a child component, and let that child call useContext. The child is rendered inside the provider.

### 14. What happens when no provider exists and the default is null?

The consumer receives null. A custom Hook can check for null and throw a clear error that identifies the missing provider.

### 15. Why must the provider and consumer use the exact same context object?

React matches context objects by identity. If a build setup or duplicate module creates separate context objects, one object can provide while the other reads, and the consumer will see its default instead of that provider's value.

## Updating and reading context

### 16. Does the context object itself store changing state?

No. Context is a way to provide and read a value. Put changing data in component state or a reducer, then pass the current value through a provider.

### 17. How can a context value change over time?

Store the value in state near the provider and pass that state as the provider value. When the state changes, the provider supplies a new value to its consumers.

### 18. Can a provider pass functions as part of its context value?

Yes. A provider can pass any JavaScript value, including objects, functions, state, and update actions. Consumers can read and call the provided action.

### 19. How does React detect a changed context value?

React compares the previous and next provider values using Object.is. A different value causes components that read that context below the provider to receive the update.

### 20. Why can an object context value cause extra consumer updates?

Creating an object literal during each provider render creates a new object identity. Object.is sees the new reference as different even if its fields contain equivalent values, so context consumers can update.

### 21. Should every context object be wrapped in useMemo?

No. Keep the provider simple unless the new value identity causes a measurable or meaningful update cost. Memoization can help when a provider re-renders often and its context data has not changed.

### 22. Does memo prevent a component from updating when context it reads changes?

No. A context consumer receives updates when its context value changes even if it is wrapped in memo. Memo can still skip some prop-based updates, but it does not block the context subscription.

### 23. Does every descendant re-render when a context value changes?

Components that read that context below the changed provider receive the new value. Components that do not read that context are not subscribed to it, though they may render for other reasons such as their parent rendering with new props or children.

## Context design

### 24. When should a value stay as a prop instead of moving into context?

Keep it as a prop when only a nearby child or small part of the tree needs it, when explicit dependencies improve reuse, or when prop forwarding is still easy to understand.

### 25. What are good context use cases?

Common examples include a theme, current account, locale, routing information, and a shared state interface used by many distant descendants.

### 26. Is context a global variable?

No. Context is scoped to a provider and its descendant branch. Separate parts of the tree can have different values, and nested providers can override values locally.

### 27. Is context a state-management solution by itself?

No. Context distributes values. State or a reducer owns updates. Combining a reducer with context can expose state and actions to distant components, but the reducer still defines the transitions.

### 28. Why might separate contexts be better than one large context object?

Values may have different owners or update frequencies. Separating them can reduce unrelated consumer updates and clarify which parts of the tree depend on which data.

### 29. Can component composition be an alternative to context?

Yes. Passing already-created JSX through children or another composition prop can let a component render content without receiving every piece of data itself. This can avoid a long prop chain while keeping dependencies explicit.

### 30. Does putting a user or permission value in context secure an application?

No. Context only controls UI data flow in the client. It is not a security boundary. The trusted server or service must still enforce authentication and authorization.

## Code and troubleshooting

### 31. What does useContext return in this example?

~~~jsx
const LanguageContext = createContext("en");

function Greeting() {
  const language = useContext(LanguageContext);
  return <p>Language: {language}</p>;
}
~~~

With no matching provider above Greeting, it returns the default string en. If a provider above it supplies another value, it returns that value instead.

### 32. What does the consumer see here?

~~~jsx
<ThemeContext value="dark">
  <Toolbar />
  <ThemeContext value="light">
    <PrintPreview />
  </ThemeContext>
</ThemeContext>
~~~

Toolbar reads dark. PrintPreview reads light because its nearest ThemeContext provider is the nested one.

### 33. Why does this component not read dark?

~~~jsx
function Page() {
  const theme = useContext(ThemeContext);

  return (
    <ThemeContext value="dark">
      <Toolbar />
    </ThemeContext>
  );
}
~~~

Page reads the closest provider above Page, or the default. The provider it returns is below Page and therefore only affects Toolbar.

### 34. What is a good default when a provider is mandatory?

Use a sentinel that cannot be confused with a real value, such as null, then expose a custom Hook that checks for the sentinel and reports a clear missing-provider error.

### 35. What should be checked if a consumer sees the default despite an apparent provider?

Check that the provider is actually above the consumer in the rendered tree, that it is for the same context object, and that the consumer is inside its subtree. Also check that the context value is not explicitly undefined.

### 36. Why might a custom Hook be used to read context?

It can provide a short, consistent API and centralize validation. For example, useAccount can call useContext(AccountContext) and throw a descriptive error when its required provider is absent.

## Short recap

- Context provides a value to descendants without forwarding it through every intermediate component.
- useContext reads the nearest provider above the calling component.
- The default applies only when there is no matching provider; undefined from a provider is still undefined.
- Context distributes a value; state or a reducer changes it.
- React compares old and new values with Object.is.
- Use props for local relationships and context for shared subtree values.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="049-rules-of-hooks.md">049 - Rules of Hooks - Q&amp;A</a></td><td><a href="../lessons/050-context-and-usecontext.md">Lesson</a></td><td><a href="../examples/050-context-and-usecontext.md">Example</a></td><td><a href="051-context-provider-design.md">051 - Designing Context Providers and Avoiding Overuse - Q&amp;A</a></td></tr>
  </tbody>
</table>
