# Example: Upgrade a React Application Deliberately

This runbook demonstrates a staged migration for a fictional Vite application moving from React 18.2 to React 19. It shows how to make the change reviewable, handle representative API migrations, inspect dependency compatibility, and prepare to release.

The version numbers in the commands are dated examples. **As of 2026-11-25, React's official version page lists React 19.3 as the latest stable version.** Before copying the commands in the future, check the current official version and migration guide. This document does not modify a real application's dependencies.

## Scenario and target

Assume the project currently has:

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

For this walkthrough, the target is React and React DOM 19.3.0. Keep the two runtime packages aligned. A TypeScript project should update its React type packages to the matching major version as well.

## 1. Start from a known-good state

Create a branch or commit the existing application. Confirm there are no unrelated edits and that the project has the expected lockfile.

```sh
git status --short --branch
npm ci
npm ls react react-dom
npm run build
```

Record existing warnings and failures. They are the baseline for this migration, not necessarily problems introduced by the upgrade.

If the project has existing checks, run its type checker, linter, unit/component tests, and browser tests too. Do not skip a check because the dependency installation succeeds.

## 2. Inspect packages that depend on React

Review the framework, router, design system, test packages, and internal component libraries. Check their peer dependency ranges and release notes for target-version compatibility. Find out whether any package uses undocumented React internals.

Use the package manager to understand why React is installed and whether multiple copies are present:

```sh
npm ls react react-dom
npm explain react
```

Resolve compatibility issues deliberately. Do not silence a peer conflict with a force option without understanding the affected packages.

## 3. Use React 18.3 to expose migration warnings

For an application moving from React 18 to React 19, first move to React 18.3. This intermediate version adds warnings that can expose APIs and patterns that should be updated before the major upgrade.

```sh
npm install --save-exact react@18.3.1 react-dom@18.3.1
npm ls react react-dom
npm run build
```

Run the application in development and review the console. Fix deprecation warnings in small changes while the application still uses the React 18 runtime. Review the framework and package compatibility again after each relevant update.

## 4. Migrate old application-root calls

Before, an older application might mount like this:

```jsx
import { render } from "react-dom";
import App from "./App.jsx";

render(<App />, document.getElementById("root"));
```

Use the client root API instead:

```jsx
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

const container = document.getElementById("root");

if (!container) {
  throw new Error("The root element was not found.");
}

createRoot(container).render(<App />);
```

For a server-rendered application, use its framework's supported hydration integration. Do not replace server hydration with a client render; that changes the rendering model.

## 5. Migrate function component defaults

Before:

```jsx
function PageTitle({ title }) {
  return <h1>{title}</h1>;
}

PageTitle.defaultProps = {
  title: "Untitled",
};
```

After:

```jsx
function PageTitle({ title = "Untitled" }) {
  return <h1>{title}</h1>;
}
```

React 19 no longer applies `defaultProps` to function components. Class component `defaultProps` support remains.

## 6. Replace removed or outdated patterns

Search the repository and its tests for APIs that the target migration removes or deprecates. Representative changes include:

- replace `ReactDOM.render` with `createRoot`;
- replace legacy `hydrate` with `hydrateRoot` where the framework does not manage hydration;
- use `root.unmount()` instead of `unmountComponentAtNode`;
- replace `findDOMNode` with a direct DOM ref;
- replace string refs with callback refs or `useRef`;
- replace legacy `contextTypes` and `getChildContext` with modern Context;
- replace function component `defaultProps` with parameter defaults;
- stop relying on React's removed function `propTypes` checks;
- import `act` from `react` rather than `react-dom/test-utils`; and
- migrate away from removed test utility paths.

For a TypeScript application, review the React 19 type migration separately. For example, a ref should usually have an explicit initial value:

```tsx
const panelRef = useRef<HTMLDivElement | null>(null);
```

The precise type fix depends on the code. Do not globally rewrite a type without checking how the ref is used.

## 7. Run the codemod as a reviewable change

After committing the React 18.3 fixes, you can run the React 19 migration recipe described in React's upgrade guide:

```sh
npx codemod@latest react/19/migration-recipe
```

The tool version behind `@latest` can change over time. For a controlled migration, review the package and consider pinning its version. The codemod only changes patterns it knows how to transform.

Inspect the entire diff:

```sh
git diff --stat
git diff
```

Review each refactor for behavior, especially changed refs, root setup, type changes, and test helpers. Search again for removed APIs that the codemod may not have recognized. Keep only changes that make sense for this project.

## 8. Install the target versions together

After addressing React 18.3 warnings and dependency compatibility, install the target pair:

```sh
npm install --save-exact react@19.3.0 react-dom@19.3.0
```

For a TypeScript project, update the type packages to the React 19 major version and review the exact versions the package manager resolved:

```sh
npm install --save-dev @types/react@19 @types/react-dom@19
```

Then inspect the manifest and lockfile. The target version is a historical example current as of this lesson's update date; choose a version supported by the framework and package set in the actual project.

## 9. Verify the JSX transform

React 19 requires the modern automatic JSX transform. Current Vite React projects commonly already enable it. Older or custom setups should check their tooling.

For a TypeScript configuration, one common setting is:

```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

For a Babel setup, check the React preset's runtime configuration. Do not add another transform configuration if the framework already owns it. Build the project and resolve any outdated-transform warning.

## 10. Review error reporting

React 19 changed how caught and uncaught render errors are reported. If the application has custom monitoring, confirm what happens in both cases:

1. Trigger a render error caught by an Error Boundary in a safe environment.
2. Trigger an uncaught render error in a disposable test route or development build.
3. Confirm each error reaches the intended reporting path once.
4. Check that the report has the expected release and component context.
5. Confirm the visitor sees the correct fallback and no diagnostic secrets.

React 19 root APIs support error callbacks for custom reporting. Only add root options after reviewing the current API for the target version and the framework's integration. Do not collect production error details by printing secrets or raw user input to the browser console.

## 11. Review the dependency and code changes

Before release, review:

```sh
git status --short
git diff -- package.json
git diff -- package-lock.json
npm ls react react-dom
npm run build
```

Also run the project's type checker, linter, component tests, and browser tests. Inspect warnings rather than treating them as harmless because the build exits successfully.

Test the application flows that exercise:

- initial root rendering and unmounting;
- server rendering and hydration, if used;
- forms and submission states;
- refs and focus management;
- Error Boundaries and root error handlers;
- routing, navigation, and lazy-loaded screens;
- third-party component libraries; and
- the browsers and devices supported by the project.

## 12. Prepare rollout and rollback

Before deploying, record the last known-good commit and dependency lockfile. Confirm that the old application build can still be restored with compatible backend behavior. Assign someone to watch errors and key user journeys.

For a staged rollout:

1. Deploy to a staging environment.
2. Smoke-test the production build with production-like configuration.
3. Release to a small percentage of users if the platform supports it.
4. Compare error, hydration, and performance signals with the baseline.
5. Expand rollout if the results are healthy.
6. Roll back promptly if a serious regression affects users.

Keep the React upgrade separate from unrelated feature work where possible. Smaller changes are easier to review, diagnose, and reverse.

## Review checklist

- [ ] Start from a clean commit and record the old versions.
- [ ] Confirm framework and library peer dependency support.
- [ ] Upgrade React 18.2 to 18.3 first when moving to React 19.
- [ ] Review and resolve deprecation warnings.
- [ ] Update React and React DOM as a pair.
- [ ] Update React type packages for TypeScript.
- [ ] Verify the modern JSX transform.
- [ ] Inspect codemod and lockfile diffs.
- [ ] Remove old APIs and check test integrations.
- [ ] Run type, lint, test, build, and browser checks.
- [ ] Verify error reporting and production hydration if used.
- [ ] Prepare a release owner, monitoring signals, and rollback path.

## Expected result

The application now uses the selected compatible React and React DOM versions, no longer depends on removed migration APIs, and has a lockfile that records the resolved dependency tree. Its existing behavior has been checked with the project's real framework and integrations.

This runbook does not certify every package or application for React 19.3. The compatibility decision and final version selection must be based on the specific project's dependency graph and current React release guidance.
