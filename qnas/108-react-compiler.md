# React Compiler - Q&A

## Core ideas

### 1. What is the React Compiler?

React Compiler is a build-time compiler that analyzes React code and adds safe memoization automatically. It can avoid repeating work when inputs have not changed. It does not run as a separate optimization engine in the browser.

### 2. Is the compiler the same as React?

No. React remains the UI library and runtime. The compiler is a build tool that transforms eligible components and Hooks before the application runs.

### 3. What does automatic memoization mean?

The compiler can preserve calculated values, functions, and component work between renders when their dependencies are unchanged. You write ordinary React code; the compiler determines where reuse is safe.

### 4. Does the compiler optimize every component?

No. It compiles code it can analyze safely. Code with unsupported patterns or violations of the Rules of React can be skipped, while other compatible components may still be compiled.

### 5. Does a skipped component make the app incorrect?

A skipped component should continue to work as normal React code. Compilation affects performance, not the intended meaning of a correct component.

### 6. Should I write code differently to help the compiler?

Write clear, idiomatic React and follow the Rules of React. Keep render logic pure, call Hooks at the top level of components or custom Hooks, and avoid mutating values that React expects to treat as immutable.

### 7. Does the compiler replace React's rendering model?

No. React still decides when to render and update the UI. The compiler can reduce avoidable repeated calculations and renders while preserving React's semantics.

## Compatibility and setup

### 8. Which React versions can I use?

React Compiler works best with React 19 and later. React 17 and 18 can be supported with the compiler runtime package and an appropriate compiler target. Follow the setup supported by the project's React and build-tool versions.

### 9. Is the compiler a runtime dependency?

The compiler integration is normally a development/build dependency. Older React versions that require the compiler runtime need that runtime package in the application dependencies because the generated code uses it.

### 10. Where should I configure the compiler?

Configure it in the build pipeline, usually through the Babel plugin or the integration recommended by the framework or bundler. Keep the configuration aligned with the installed versions.

### 11. Does every Vite project use the same setup?

No. The recommended integration can depend on the version of the Vite React plugin and the bundler. Check the setup for the exact versions in the project instead of copying a configuration from another version.

### 12. Where should the compiler Babel plugin appear?

When using Babel directly, the React Compiler plugin should run before other Babel plugins that transform React code. Follow the ordering required by the framework's integration.

### 13. Do I need to remove my existing Babel plugins?

Usually not. Add the compiler to the existing supported pipeline and keep other required plugins. Check ordering and compatibility if a plugin transforms component code.

### 14. Can I use the compiler in a framework project?

Often, yes, when the framework supports the compiler integration. Use the framework's documented configuration so its server, client, and build pipelines are configured consistently.

### 15. Does adding the compiler require rewriting components?

No. The intent is to optimize existing idiomatic React incrementally. Fix compiler diagnostics when they point to a real Rules of React issue, then confirm the affected code behaves correctly.

## Linting, modes, and directives

### 16. Why install or configure eslint-plugin-react-hooks?

Its compiler-aware diagnostics can report patterns that prevent safe compilation and violations of the Rules of React. The compiler can skip code it cannot safely optimize, so diagnostics help you understand what was skipped.

### 17. Must every compiler diagnostic be fixed before enabling it?

No. You can adopt the compiler gradually. Fix high-value issues when practical; compatible code can still be compiled while unsupported code remains ordinary React code.

### 18. What is the compiler's default compilation mode?

The default infer mode attempts to compile eligible components and Hooks automatically. A project can choose a different mode when it wants to opt in more selectively.

### 19. What is annotation mode?

Annotation mode compiles only components and Hooks that are explicitly marked for compilation. It is useful when introducing the compiler gradually or when a team wants explicit control over the initial rollout.

### 20. What does the use memo directive do?

In a configuration that supports directive-based compilation, it marks a component or Hook for compilation in annotation mode. It does not replace the Rules of React.

### 21. What does the use no memo directive do?

It temporarily opts a component or Hook out of compiler optimization. It is useful for debugging a suspected compiler interaction. Remove it after investigating unless there is a specific reason to keep the exclusion.

### 22. Should directives be added everywhere?

No. In infer mode, ordinary eligible code is compiled without annotations. Use directives only when the configured compilation mode or a focused debugging need calls for them.

## Existing memoization and verification

### 23. Should I delete every memo, useMemo, and useCallback call?

Not automatically. The compiler can reduce the need for manual memoization, but existing calls can remain valid. Remove them only when doing so preserves behavior and makes the code easier to understand.

### 24. Can manual memoization still be useful?

Yes. Public APIs, measured performance problems, or code that needs stable identity for a documented reason may still use manual memoization. Measure the result and avoid relying on memoization for correctness.

### 25. How can I tell if a component was compiled?

Use the compiler-aware ESLint diagnostics and React Developer Tools. The Components panel can show a compiler memoization indicator for compiled components. A production build and profiling can provide additional confirmation.

### 26. Does a DevTools indicator prove the app is faster?

No. It indicates that compiler optimization was applied, not that a user-visible performance problem was solved. Profile a representative interaction before and after when performance matters.

### 27. What should I test after enabling the compiler?

Check the same user-visible behavior as before: state changes, event handlers, forms, Effects, and rendered output. Pay particular attention to code that depends on object or function identity.

### 28. Can I enable the compiler only for part of the application?

Yes. A gradual rollout is a practical approach. Start with a route, package, or group of components, review diagnostics, and expand as the build and behavior are confirmed.

## Debugging and common pitfalls

### 29. A component behaves differently after enabling the compiler. What should I do?

First look for a Rules of React violation, such as mutation during render, impure rendering, or conditional Hook calls. Use compiler diagnostics to find the affected code. As a temporary isolation step, opt the component out with use no memo if the configured integration supports it, then compare behavior and fix the underlying issue.

### 30. Why can a Rules of React violation appear only after optimization?

Incorrect code can accidentally appear to work when values are recalculated often. Memoization may expose the assumption that a value is recreated on every render. The code should not depend on such accidental behavior; make render logic pure and treat inputs and state immutably.

### 31. Can I rely on a new object being created on every render?

Do not rely on that for correctness. The compiler may preserve values when safe. If identity itself is part of an API contract, make that requirement explicit and use the appropriate API rather than depending on incidental allocations.

### 32. Will the compiler fix an Effect that runs too often?

It may reduce work caused by unstable values in some cases, but it does not replace correct Effect design. Effects should synchronize with external systems and declare their actual reactive dependencies.

### 33. Does the compiler make impure render logic safe?

No. Rendering must remain pure. Do not mutate props, state, or shared objects during render, and do not perform external side effects from render logic.

### 34. Does the compiler fix slow network requests or large bundles?

No. It targets avoidable React computation and rendering work. Network latency, server response time, asset size, and expensive browser operations need their own diagnosis.

### 35. Can I force compilation of code the compiler cannot analyze?

A directive does not make unsafe code safe. Fix the unsupported pattern or leave that code uncompiled. Correctness takes priority over forcing optimization.

### 36. What if my library must support consumers who do not use the compiler?

Compile and test the library according to its supported toolchain and React versions. Do not assume every consumer uses the same compiler configuration; preserve normal React behavior and document runtime requirements.

### 37. Is compiler optimization guaranteed to improve every interaction?

No. It is intended to avoid unnecessary work, but real performance depends on the application and workload. Profile the interactions that matter and keep the implementation understandable.

### 38. What is a sensible rollout checklist?

1. Confirm the React, framework, bundler, and compiler integration versions.
2. Configure the compiler using the supported integration.
3. Enable compiler-aware Hook diagnostics.
4. Fix genuine Rules of React violations.
5. Start with compatible code or an explicitly selected area.
6. Check behavior and inspect compiler diagnostics.
7. Profile representative interactions before and after.
8. Expand the rollout while keeping an easy way to isolate regressions.

## Key takeaway

Treat the React Compiler as a build-time performance optimization. Keep components correct without memoization, follow the Rules of React, use diagnostics to guide adoption, and measure real interactions.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="107-react-performance-and-profiling.md">107 - React Performance Fundamentals and Profiling: Questions and Answers</a></td><td><a href="../lessons/108-react-compiler.md">Lesson</a></td><td><a href="../examples/108-react-compiler.md">Example</a></td><td><a href="109-memo-and-optimization-decisions.md">109 - Memoization and Optimization Decisions - Q&amp;A</a></td></tr>
  </tbody>
</table>
