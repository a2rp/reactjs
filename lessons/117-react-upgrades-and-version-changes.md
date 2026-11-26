# Upgrading React and Handling Version Changes

React upgrades can include new features, changed behavior, removed deprecated APIs, and updated type definitions. A reliable upgrade is a reviewed migration of the whole application dependency graph, not a version-number edit followed by hope.

This lesson explains a repeatable process for both routine updates and major migrations. Its React 19 examples describe the published migration from React 18 to React 19. **As of 2026-11-25, the official React versions page lists React 19.3 as the latest stable version.** Check the current official release information before applying version-specific commands later; package versions and upgrade guidance change over time.

## Learning goals

After this lesson, you should be able to:

- distinguish routine patch/minor updates from major-version migrations;
- inspect the installed dependency tree and package constraints;
- plan a staged React and React DOM upgrade;
- recognize important React 19 migration points;
- use codemods as reviewable assistance rather than automatic proof;
- test application code, types, build tooling, and third-party integrations; and
- deploy an upgrade with monitoring and a rollback plan.

## 1. Why upgrades need a plan

A React application depends on more than the `react` package. Its runtime, renderer, framework, build tool, type definitions, test environment, component libraries, and internal packages may all have compatibility requirements.

A successful package installation only proves that the package manager found a dependency solution. It does not prove that:

- every dependency supports the new React version;
- the application uses only supported APIs;
- type checks still describe the runtime correctly;
- the production bundler uses the required JSX transform;
- rendering, hydration, forms, refs, and error reporting still behave as expected; or
- the deployed application works in the browsers the product supports.

Treat a React upgrade as a code and dependency change with a reviewable diff.

## 2. Understand stable version numbers

Stable React releases follow semantic versioning. A version has the form `major.minor.patch`, often written `x.y.z`.

- A **patch** release changes the last number and is intended for critical fixes.
- A **minor** release changes the middle number and can add features or non-critical fixes.
- A **major** release changes the first number and can include breaking changes.

React describes its stable release policy in its versioning documentation. Minor releases can still contain meaningful features. Patch updates should still be reviewed along with the rest of the dependency graph and lockfile.

Pre-release and experimental channels exist for early feedback. Do not use them in a production application unless the product deliberately accepts their stability and compatibility expectations.

## 3. Inspect the project before changing versions

Start from a clean, committed working tree. Record the current state:

```sh
npm ls react react-dom
npm outdated
```

Also review:

- the `dependencies` and `devDependencies` in `package.json`;
- the package manager and lockfile used by the project;
- the Node.js version and framework version;
- peer dependency ranges for React-related libraries;
- TypeScript React type packages, if used;
- JSX transform and build configuration;
- unit, component, and browser test tooling; and
- any libraries that access React internals.

Use equivalent commands for your package manager when appropriate. Do not create a second lockfile by switching package managers halfway through an upgrade.

Commit or branch from a known-good state. That gives you a clean comparison and a direct rollback point.

## 4. Check compatibility across the dependency graph

Before updating React:

1. Check the framework's supported React versions.
2. Check peer dependency ranges for routers, design systems, test libraries, renderers, and internal packages.
3. Review open issues or release notes for dependencies that integrate with rendering, refs, hydration, or React internals.
4. Upgrade incompatible dependencies in separate, understandable changes where possible.
5. Avoid multiple installed copies of React in the same rendered application. Duplicate copies can cause invalid Hook call problems.

A peer dependency warning is information about a compatibility expectation. Do not dismiss it automatically, and do not force-install a package graph without understanding the conflict.

## 5. Stage a major migration

A cautious major upgrade has clear checkpoints:

1. **Establish a baseline.** Build the current version and run the checks the project already relies on. Record existing warnings and failures so they are not confused with upgrade regressions.
2. **Read the migration guide.** Review breaking changes, deprecations, type changes, and required tooling updates for the target release.
3. **Move through warning releases when recommended.** For React 18 to React 19, React published React 18.3 to surface deprecation warnings before the major upgrade.
4. **Resolve warnings and old APIs.** Fix the code while it still runs on the old major when practical.
5. **Update React and React DOM together.** Keep their versions compatible and update the type packages for TypeScript applications.
6. **Update the build tool.** Confirm that the modern JSX transform is enabled when the target version requires it.
7. **Apply codemods selectively.** Review every change and preserve a clean diff.
8. **Run checks in layers.** Type check, lint, component or unit tests, production build, then browser smoke checks.
9. **Test integrated behavior.** Include the app's actual framework, router, server rendering, forms, error reporting, and supported browsers.
10. **Release gradually when possible.** Monitor the new version and retain a rollback path.

The exact sequence depends on the framework and packages in the project. Do not blindly copy a migration for another application's setup.

## 6. React 18 to React 19: changes to review

React's React 19 upgrade guide documents multiple removals and behavior changes. The following are important review areas; this list is not a substitute for reading the complete guide for the project.

### Upgrade to React 18.3 first when migrating from React 18

React 18.3 is based on React 18.2 and adds warnings for deprecated APIs and other changes needed for React 19. The React team recommends upgrading to 18.3 first so that the application can expose issues before the major version change.

Fix these warnings, then update the major versions.

### The modern JSX transform is required

React 19 requires the modern JSX transform. Many current toolchains already enable it, but older custom Babel, TypeScript, or bundler configurations may not.

For TypeScript, check the JSX compiler setting. A common automatic-runtime value is:

```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

For Babel, check the React preset's runtime setting. Confirm that the framework or bundler owns this configuration before changing it manually.

### Update old root APIs

The legacy `ReactDOM.render` API was removed. Use a root from `react-dom/client`:

```jsx
import { createRoot } from "react-dom/client";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The root element was not found.");
}

createRoot(container).render(<App />);
```

Server-rendered applications should use the appropriate hydration API, such as `hydrateRoot`, rather than the removed legacy `hydrate` call. Also review code that uses `unmountComponentAtNode` or `findDOMNode`; these legacy APIs were removed. Modern root management and DOM refs provide replacement patterns.

### Replace removed function-component defaults and PropTypes checks

Function components should use JavaScript default parameters for default values:

```jsx
function Greeting({ name = "friend" }) {
  return <p>Hello, {name}.</p>;
}
```

React 19 no longer provides the old `defaultProps` behavior for function components. It also removes React's old `propTypes` checking from the React package; existing checks should not be assumed to run. Use TypeScript or another maintained validation approach appropriate for the project. Class components retain their `defaultProps` support.

### Replace other removed legacy patterns

Review for:

- **string refs**, replacing them with callback refs or `useRef`;
- **legacy context** based on `contextTypes` and `getChildContext`, replacing it with `createContext` and modern context APIs;
- **`React.createFactory`**, replacing it with JSX;
- **`react-dom/test-utils` imports**, using `act` from `react` where applicable and supported testing-library APIs for other test behavior;
- **shallow-renderer imports from `react-test-renderer/shallow`**, migrating away from the removed path; and
- **libraries that rely on undocumented React internals**, updating or replacing them.

A migration guide may also include TypeScript-specific changes and newly deprecated APIs. Review those against the codebase rather than assuming JavaScript codemods cover them.

### Review error reporting

React 19 changed how render errors are reported. Errors caught by an Error Boundary and uncaught errors may reach different reporting paths than in earlier versions. If application monitoring relied on errors being re-thrown or on a specific console behavior, verify it with an intentional test error in a safe environment and update the root error callbacks or monitoring integration as required.

Do not infer production behavior from development console output alone.

## 7. Use codemods carefully

A codemod can automate repetitive syntax changes. It cannot decide whether a migration is semantically correct for your application or its dependencies.

A safe codemod workflow is:

1. Read which files and patterns the codemod changes.
2. Commit or stash unrelated work.
3. Run it on the intended project directory.
4. Inspect the full diff.
5. Review changed types, callbacks, refs, and control flow manually.
6. Run the project's checks after accepting appropriate changes.
7. Keep or revert each change based on evidence.

React's published React 19 migration recipe is:

```sh
npx codemod@latest react/19/migration-recipe
```

Treat a command using `@latest` as a moving tool version. Review the package and output, pin a tool version when reproducibility is important, and do not run it over uncommitted or unrelated edits.

Codemods only cover the transformations they were designed to find. They do not guarantee that every removed API or incompatible dependency has been found.

## 8. Align React, React DOM, and type packages

React and React DOM work together and should be upgraded as a compatible pair. For an application that uses React 19.3.0 and npm, an exact-version install can make the intended version explicit:

```sh
npm install --save-exact react@19.3.0 react-dom@19.3.0
```

For a TypeScript application, update React's type packages to the corresponding major version too:

```sh
npm install --save-dev @types/react@19 @types/react-dom@19
```

Then inspect `package.json`, the lockfile diff, and the resulting dependency tree. This command is an example for the version listed by React as latest stable when this lesson was written; check the official current version page before applying it later.

Do not mix an unreviewed range change with a React migration. Decide whether the project uses ranges or exact versions based on its dependency policy, and let the lockfile record the exact resolved tree.

## 9. Verify behavior, not just installation

An upgrade review should include:

- dependency-tree inspection and peer dependency warnings;
- type checking and linting;
- component tests for important interactions;
- form submission and action behavior;
- root mounting and unmounting;
- server rendering and hydration if used;
- Error Boundary and root error reporting;
- ref callbacks and imperative DOM integrations;
- Strict Mode behavior during development;
- production build and asset loading;
- all supported browsers and devices; and
- a deployed smoke test.

A successful `npm install` is only the first checkpoint. The production build and user journeys are what reveal application-level compatibility issues.

## 10. Roll out and maintain the upgrade

For a low-risk application, a normal release may be enough. For a widely used or sensitive product, consider a staging environment, canary, or feature-controlled rollout.

Before deployment:

- capture the current known-good commit and package lockfile;
- confirm a rollback can restore a compatible dependency set;
- prepare a short list of key user journeys;
- assign someone to review error and performance signals; and
- document the release version.

After deployment, watch errors, failed requests, rendering or hydration warnings, performance, and reports grouped by release. If a serious regression appears, roll back first when that is the safest recovery, then investigate using the captured release and dependency data.

## Common mistakes

- **Updating only `react` but not `react-dom`:** treat them as a compatible pair.
- **Ignoring peer dependency conflicts:** a package may not support the target version yet.
- **Jumping directly from 18.2 to 19 without reviewing 18.3 warnings:** the intermediate release can expose deprecated APIs.
- **Assuming a codemod completed the migration:** inspect changes and search the codebase for removed APIs.
- **Changing the JSX transform without checking framework ownership:** the framework may manage that configuration.
- **Keeping old function `defaultProps` or PropTypes assumptions:** the old behavior may no longer apply in React 19.
- **Testing only the development server:** build and verify the production deployment too.
- **Mixing package managers or rewriting the lockfile unnecessarily:** this obscures the dependency changes.
- **Ignoring error monitoring differences:** validate the real reporting path after the React upgrade.
- **Upgrading several unrelated libraries in one change:** smaller migrations are easier to review and roll back.

## Upgrade checklist

- [ ] Working tree is clean and the current version is recorded.
- [ ] Target React version and framework compatibility are confirmed.
- [ ] React, React DOM, type packages, and peer dependencies are reviewed.
- [ ] Migration guide and required JSX transform changes are reviewed.
- [ ] Deprecated API warnings are resolved before the major migration when possible.
- [ ] Codemod changes are inspected manually.
- [ ] Dependency tree and lockfile changes are understood.
- [ ] Type, lint, test, and production build checks pass.
- [ ] Browser, hydration, error-reporting, and key user flows are reviewed.
- [ ] Rollout owner and rollback path are known.

## Key takeaways

- Treat upgrades as deliberate dependency and application changes.
- Read the official version-specific migration guide and verify framework support.
- Upgrade React and React DOM together; align type packages where needed.
- Use warnings and codemods to find work, then validate their changes.
- Check integrations and production behavior, not only package installation.
- Deploy with monitoring and a rollback plan appropriate to the application's impact.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="116-react-capstone-project.md">116 - Build and Ship a Complete Accessible React Application</a></td><td><a href="../qnas/117-react-upgrades-and-version-changes.md">Q&amp;A</a></td><td><a href="../examples/117-react-upgrades-and-version-changes.md">Example</a></td><td><a href="118-react-19-2-features.md">118 - React 19.2: Activity and Performance Tooling</a></td></tr>
  </tbody>
</table>
