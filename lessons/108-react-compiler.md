# React Compiler and Automatic Optimization

## Learning goals

After this lesson, you should be able to:

- Explain what React Compiler optimizes and when the work happens.
- Distinguish compiler optimization from application correctness and runtime behavior.
- Add the compiler to a supported build pipeline without replacing existing configuration.
- Use compiler diagnostics, ESLint, and React DevTools to verify what was compiled.
- Adopt the compiler gradually and diagnose a component that is skipped or behaves differently.
- Understand when manual memoization and compiler directives are still relevant.

## 1. What React Compiler does

React Compiler is a build-time tool that analyzes React components and Hooks and emits optimized code. It can automatically preserve values, functions, and rendered subtrees when their inputs have not changed, reducing some unnecessary calculations and re-renders.

Without the compiler, developers may manually use `memo`, `useMemo`, and `useCallback` for measured performance cases. With the compiler enabled, many of these optimizations can be inferred from the code. The compiler does not change the APIs used to write ordinary components and Hooks.

The compiler is an optimization, not a new correctness model:

- Your UI must remain correct if a cache is discarded or an optimization is skipped.
- The compiler does not make a network request faster, shorten a long synchronous JavaScript operation, reduce an image's size, or automatically fix layout and paint work.
- Not every function is necessarily optimized. If the compiler cannot safely analyze a component or Hook, it can skip that code and keep it working normally.
- Correct React code and useful profiling still matter.

## 2. The Rules of React enable safe optimization

The compiler relies on components and Hooks following the Rules of React, including pure rendering, immutable props and state, and valid Hook usage. These rules already make React code predictable; the compiler uses them to determine when values can be reused safely.

If the compiler's analysis detects a pattern that it cannot safely optimize, it can report a diagnostic and leave that component or Hook uncompiled while compiling other eligible code. This lets a codebase adopt optimization incrementally instead of requiring every legacy pattern to be fixed at once.

Keep the React Hooks ESLint plugin configured. Its compiler diagnostics can identify violations and unsupported patterns before they become runtime problems. Address diagnostics by understanding the rule and the code path; do not silence a warning just to increase the number of compiled components.

## 3. Add the compiler to the build pipeline

Install the compiler as a development dependency:

```sh
npm install --save-dev babel-plugin-react-compiler
```

React 19 is the simplest target because its runtime includes the compiler support it needs. The compiler must be integrated into the build tool's Babel or React transform pipeline. Preserve the project's existing plugins and settings, and place the compiler before other Babel transformations that rewrite the original source.

A Babel configuration illustrates the ordering:

```js
// babel.config.js
module.exports = {
  plugins: [
    'babel-plugin-react-compiler',
    // Keep existing transforms after the React Compiler.
  ],
};
```

The exact integration depends on the framework and its version. For example, current Vite React plugin releases expose a compiler preset through their supported Babel integration; older releases use a different configuration shape. Follow the integration point supported by the version already in the project rather than replacing or downgrading the build tool blindly.

For React 17 or 18, configure the compiler target to match the React major version and install `react-compiler-runtime` as a regular dependency. React 19 normally uses the default target and does not need this extra runtime package. Do not confuse the compiler package, which is a build-time development dependency, with a runtime package needed by older React targets.

## 4. Verify compiler coverage

Use several signals rather than assuming installation means every component was compiled:

1. Run the normal development and production builds.
2. Review React Hooks ESLint diagnostics for Rules of React and compiler findings.
3. Open React DevTools Components and look for the compiler's memoization indicator on eligible components.
4. Profile a representative interaction before and after adoption.
5. Check the UI, forms, Effects, and state transitions for correct behavior.

Compiler coverage can be partial. A component without the indicator may be outside the configured files, not inferred by the current compilation mode, or skipped because its code is not safely analyzable. Treat the indicator as evidence about compilation, not proof that a user-visible performance issue has improved.

## 5. Adopt incrementally

For an existing application, introduce the compiler to a small, well-understood part of the application first:

1. Confirm the React and build-tool versions and establish a clean baseline build.
2. Enable the compiler for a pilot directory or feature using the build tool's supported configuration.
3. Turn on compiler-aware lint diagnostics and review the findings.
4. Exercise important user flows and compare profiles for the same interactions.
5. Fix correctness issues and unsupported patterns at their source.
6. Expand coverage to additional features in small steps.

Some Babel configurations support directory overrides for a gradual rollout. Keep each pilot observable and reversible through normal version control. Avoid adding a custom compilation mode or directive unless the default inference mode does not fit a real project requirement.

For a new React application, the compiler can be enabled earlier, provided the framework integration is supported and its behavior is covered by the project's normal development and production checks.

## 6. Compilation modes and directives

The default inference mode is intended to identify components and Hooks from React naming and usage conventions. In annotation mode, only functions marked with the `use memo` directive are compiled. A project should select such a mode intentionally through its compiler configuration; directives do not enable the compiler by themselves.

A directive can be useful when a project has chosen annotation mode:

```jsx
function SearchResults({ items, query }) {
  'use memo';

  const visibleItems = items.filter((item) =>
    item.label.toLowerCase().includes(query.toLowerCase()),
  );

  return <ResultList items={visibleItems} />;
}
```

`use no memo` opts a function out of compilation. It can help temporarily isolate a suspected compiler interaction or work around an incompatible integration while a fix is investigated. Prefer removing it after the underlying issue is corrected rather than leaving unexplained permanent exclusions.

## 7. Manual memoization with the compiler enabled

React Compiler can reduce the need for manual `memo`, `useMemo`, and `useCallback`, but it does not make existing uses invalid. Keep a manual memoization boundary when it is needed for a documented reason, such as a public API contract or an interoperability requirement.

Do not depend on referential stability for correctness. An Effect should synchronize with the values it actually uses, and event behavior should remain correct if a function or object identity changes. Memoization is a performance optimization; it is not a substitute for state, a ref, or an explicit lifecycle contract.

When reviewing manual memoization in a compiled app:

- Keep it if it protects a measured, understood boundary and the compiler preserves its behavior.
- Remove unnecessary layers when doing so makes the code easier to maintain and the result remains correct.
- Check lint diagnostics for cases where the compiler cannot preserve existing manual memoization safely.
- Re-profile after changes rather than assuming the compiler or manual cache guarantees a faster interaction.

## 8. Diagnose a component that was skipped

A skipped component is not necessarily a problem. The compiler's goal is to avoid work while preserving the output for the same inputs. If a component is not compiled, check:

- Is the file included in the compiler's build pipeline?
- Does the project use the expected compilation mode?
- Does the function follow component and Hook naming conventions?
- Did lint report a Rules of React violation or unsupported pattern?
- Is the React target and runtime configuration correct?
- Is the component provided by a dependency rather than compiled application source?

Fix the underlying convention or safety issue where practical. Avoid forcing compilation of code that depends on mutation, render side effects, or unstable semantic assumptions.

## 9. Debug behavior that changes after enabling the compiler

First confirm the UI behavior and reproduce it in a small path. Then check compiler and lint diagnostics for Rules of React violations, especially mutation during render, impure calculations, invalid Hook usage, and Effects that depend on accidental object identity.

If needed, temporarily add `use no memo` to the affected function as a diagnostic comparison. If the issue disappears, investigate whether the code relies on memoization for correctness or violates a React rule that the compiler could not detect. Fix the root cause, remove the temporary directive, and repeat the same user flow.

Do not immediately conclude that the compiler is at fault because behavior changed. Also do not dismiss a genuine compiler regression: create a minimal reproduction, record React and compiler versions, compare compiled and uncompiled behavior, and preserve the build or runtime diagnostic.

## 10. What the compiler does not replace

React Compiler does not replace:

- Profiling user-visible interactions.
- Correct state ownership and data-flow design.
- Correct Effect setup, cleanup, and dependencies.
- Stable keys for changing lists.
- Browser performance and network investigations.
- Code splitting, asset optimization, or server-side data strategy.
- Tests and review of behavior after changing the build pipeline.

It helps avoid some manual memoization work. It does not make every performance problem a component re-render problem.

## Common mistakes

- **Assuming package installation enables compilation:** verify that the project's build pipeline runs the compiler.
- **Replacing the whole Babel or Vite configuration:** add the supported integration while preserving existing transforms.
- **Putting the compiler after transforms that obscure source structure:** keep it early in the Babel pipeline.
- **Expecting every component to compile:** inspect diagnostics, file coverage, and compilation mode.
- **Suppressing lint diagnostics:** fix unsafe patterns so the compiler can safely analyze more code.
- **Relying on memoization for correctness:** make the UI correct when cached values are recomputed.
- **Adding `use memo` without annotation mode:** directives only have an effect according to the project's compiler configuration.
- **Leaving `use no memo` as an unexplained permanent fix:** use it to isolate a problem, then address the cause.
- **Claiming a performance win from a badge alone:** profile the interaction users actually experience.

## Summary

React Compiler analyzes components and Hooks during the build and can automatically memoize safe calculations, functions, and rendered subtrees. It relies on the Rules of React and may skip code it cannot safely compile. Integrate it through the framework's supported transform pipeline, use lint diagnostics and DevTools to verify coverage, and adopt it gradually. Treat caches as optimizations rather than correctness guarantees, and compare real interactions before and after enabling the compiler.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="107-react-performance-and-profiling.md">107 - React Performance Fundamentals and Profiling</a></td><td><a href="../qnas/108-react-compiler.md">Q&amp;A</a></td><td><a href="../examples/108-react-compiler.md">Example</a></td><td><a href="109-memo-and-optimization-decisions.md">109 - Memoization with memo and Optimization Decisions</a></td></tr>
  </tbody>
</table>
