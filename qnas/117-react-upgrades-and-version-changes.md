# Q&A: Upgrading React and Handling Version Changes

## 1. Why is a React upgrade more than changing a package number?

The application also depends on React DOM, a framework or bundler, type definitions, test tools, component libraries, and packages with React peer dependencies. The code may use APIs or behaviors that changed between versions. An install succeeding does not prove the whole application works.

## 2. What does semantic versioning mean for React's stable releases?

A version is written as `major.minor.patch`. A patch release changes the final number, a minor release changes the middle number, and a major release changes the first number. React's stable versioning policy describes patch releases for critical fixes, minor releases for new features or non-critical fixes, and majors for breaking changes.

## 3. Can a minor React update add a feature?

Yes. React's stable policy allows new features in minor releases. A minor release number does not mean “no new behavior”; review release information and dependency compatibility.

## 4. Should a production app use an experimental React release channel?

Only if the project deliberately accepts the stability and compatibility expectations of that channel. Experimental builds are intended for early feedback and should not be treated as interchangeable with stable releases.

## 5. What should you inspect before an upgrade?

Record installed React and React DOM versions, read the package manifest and lockfile, identify the package manager and framework, check peer dependency ranges, and note the build, test, type-check, and browser support setup. Start from a clean, committed working tree.

## 6. Why check a framework's supported React versions?

Frameworks can control rendering, hydration, routing, server components, and build configuration. A React version that installs may still be unsupported by the selected framework version.

## 7. What is a peer dependency conflict?

It is a dependency stating that it expects a particular range of a package such as React. A conflict may mean that a library has not declared or implemented compatibility with the target React version. Investigate it instead of forcing installation without review.

## 8. Why upgrade `react` and `react-dom` together?

React DOM is the browser and server renderer that works with the React package. Keeping their versions compatible avoids an unsupported combination and makes the dependency graph easier to reason about.

## 9. Should TypeScript applications update React's type packages?

Yes, update `@types/react` and `@types/react-dom` to the appropriate target major and then resolve the type changes. Type definitions describe the API to TypeScript; they do not replace updating the runtime packages.

## 10. Why did React publish React 18.3 before React 19?

React 18.3 is based on 18.2 and adds warnings for deprecated APIs and changes needed for the React 19 migration. Upgrading to it first can expose work while the app remains on the previous major version.

## 11. What is the modern JSX transform, and why did React 19 require it?

It is the automatic JSX runtime supported by modern build tools. React 19's JSX improvements require it. Many current toolchains enable it already, but older custom Babel or TypeScript configurations should be checked.

## 12. Does every app need to edit its JSX configuration manually?

No. Frameworks and bundlers often manage it. Check which tool owns the configuration before changing Babel or TypeScript settings.

## 13. What replaced `ReactDOM.render`?

For a client-rendered application, create a root from `react-dom/client` and render through that root:

```jsx
import { createRoot } from "react-dom/client";

createRoot(container).render(<App />);
```

Server-rendered markup should use the supported hydration API such as `hydrateRoot`, rather than the removed legacy `hydrate` call.

## 14. What happened to `unmountComponentAtNode`?

It was removed in React 19. Keep the root returned by `createRoot` or `hydrateRoot` and call `root.unmount()` when the application must be unmounted.

## 15. What should replace `findDOMNode`?

Use a ref attached directly to the DOM node you need. This keeps the component-to-DOM relationship explicit and avoids relying on the removed legacy API.

## 16. What replaces function-component `defaultProps`?

Use JavaScript default parameter values:

```jsx
function Greeting({ name = "friend" }) {
  return <p>Hello, {name}.</p>;
}
```

React 19 removed the old `defaultProps` behavior for function components. Class component support remains.

## 17. Does React 19 still run function-component `propTypes` checks?

No. React 19 removed React's `propTypes` checks for function components; existing code should not assume those checks still run. Use TypeScript or another appropriate validation method.

## 18. What should replace string refs?

Use callback refs or `useRef` in function components. String refs are a removed legacy pattern.

## 19. What should replace legacy context?

Use `createContext` with a provider and modern context consumers such as `useContext` or a class component's `contextType`. The legacy `contextTypes` and `getChildContext` APIs were removed.

## 20. What happened to `React.createFactory`?

React 19 removed it. Use JSX to create elements.

## 21. Where should `act` be imported from after upgrading?

React moved `act` to the `react` package. Review test imports from `react-dom/test-utils`; other removed test utilities need supported alternatives.

## 22. Should a project keep using `react-test-renderer`?

React has deprecated it and recommends modern testing approaches such as React Testing Library for many application tests. Plan a migration instead of assuming the deprecated renderer is a stable long-term choice.

## 23. What should happen to custom error reporting during the React 19 upgrade?

Review the actual reporting integration. React 19 changed the handling and reporting paths for caught and uncaught render errors. Verify root-level handlers and Error Boundary reporting with a deliberate test error in a safe environment; do not infer behavior only from development console messages.

## 24. What does a codemod do?

A codemod applies known source transformations, such as replacing a deprecated API pattern. It saves repetitive editing but cannot decide whether every result is correct for the application's behavior or integrations.

## 25. Does running the React 19 codemod mean the migration is finished?

No. Codemods only find patterns they support. Review the diff, search for removed APIs, update dependencies and types, test behavior, and check libraries that a source transformation cannot change.

## 26. Why should you commit before running a codemod?

A clean commit separates automated migration changes from unrelated work. It makes the diff easier to inspect and allows you to revert or compare the transformation without losing other changes.

## 27. What does the React 19 migration recipe command do?

The React guide documents `npx codemod@latest react/19/migration-recipe` for the migration recipe. It applies listed code transforms. Review the command's effects and resulting diff; the recipe does not cover every TypeScript change or every project-specific migration.

## 28. Why is `@latest` worth reviewing in an automation command?

It resolves a moving version of the tool. That may be convenient, but it makes a future rerun less reproducible. For a controlled migration, review the tool and consider pinning its version.

## 29. Why should you avoid changing package managers mid-upgrade?

Different package managers may create different lockfiles and dependency resolutions. That adds unrelated dependency changes and makes it harder to see what React migration caused a problem.

## 30. What does a lockfile contribute?

It records a specific resolved dependency tree for repeatable installs. Review its changes, but do not mistake a lockfile for proof that the dependency set is compatible or secure.

## 31. Should all dependency updates be bundled with a React major upgrade?

Usually not. Keep changes reviewable. Separate unrelated upgrades when possible so a regression can be traced to a smaller set of changes and rolled back more safely.

## 32. Which checks should follow the package installation?

Review peer dependency warnings and installed versions; run the project's type, lint, and test checks; create a production build; test important rendering, form, ref, hydration, and error-reporting behavior; and smoke-test the deployed build.

## 33. Why test Strict Mode and development behavior?

Strict Mode can reveal side effects and lifecycle assumptions during development. An upgrade may expose code that depends on behaviors that should not be relied upon. Check that Effects, refs, and cleanup logic remain correct.

## 34. Why is a successful development server not enough?

The development server can differ from the optimized production build in transforms, assets, environment variables, and error reporting. Build and check the production application too.

## 35. How should a high-impact upgrade be released?

Use a staging environment and, when useful, a canary or staged rollout. Have a release owner, key user journeys, monitoring signals, and a known-good rollback commit and lockfile.

## 36. When should a team roll back?

Roll back when a serious regression is affecting users and reverting is the safest fast recovery. Preserve release and dependency information, then investigate and correct the upgrade before trying again.

## 37. As of the lesson update, which React version did the official React versions page list as latest stable?

As of 2026-11-25, it listed React 19.3. Version numbers change, so check the current official version information before copying a version-specific command.

## 38. What is the central rule for a safe React upgrade?

Make one understandable change at a time, follow the migration guide for the target version, inspect compatibility across the dependency graph, verify real application behavior, and keep a rollback path.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="116-react-capstone-project.md">116 - Q&amp;A: Build and Ship a Complete Accessible React Application</a></td><td><a href="../lessons/117-react-upgrades-and-version-changes.md">Lesson</a></td><td><a href="../examples/117-react-upgrades-and-version-changes.md">Example</a></td><td><a href="118-react-19-2-features.md">118 - Q&amp;A: React 19.2 Activity and Performance Tooling</a></td></tr>
  </tbody>
</table>
