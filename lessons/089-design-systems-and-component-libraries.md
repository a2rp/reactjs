# Design Systems, Shared Components, and Component Libraries

## Learning goals

After this lesson, you should be able to:

- Distinguish a design system from a shared component folder or a published library.
- Decide whether a UI component should remain feature-owned or become reusable.
- Design component APIs that are understandable and accessible.
- Use design tokens to coordinate visual choices across components.
- Choose controlled state, local state, and composition deliberately.
- Document, validate, version, and distribute reusable components responsibly.

## 1. Shared components solve a specific problem

A component can be used in more than one place without being part of a complete design system. A shared button, date formatter, or layout component may simply prevent duplicated behavior in one application.

Before extracting a component, ask:

- Do these usages have the same behavior and meaning?
- Are the differences configurable without exposing implementation details?
- Would one change to this component reasonably serve all its consumers?
- Can a caller understand the component from its name, props, and examples?

If two components only look similar but have different responsibilities or behavior, forcing them into one generic component can make the API harder to use.

## 2. What is a design system?

A design system is a coordinated set of design decisions and implementation tools for building consistent products. It may include:

- Visual principles and design tokens.
- Typography, color, spacing, elevation, and motion choices.
- Accessible component patterns and interaction behavior.
- Reusable components and higher-level page patterns.
- Usage guidance, examples, and contribution processes.
- Versioning and ownership practices for changes.

A design system is broader than a folder called `shared/`. It describes how interfaces should fit together and how teams can use and evolve those conventions.

## 3. What is a component library?

A component library is a collection of reusable implementation units, often distributed as a package. It might implement part of a design system, but it is not automatically a complete design system.

A library commonly provides:

- Public component exports and a documented prop contract.
- Styling or a theming interface.
- Build output and package metadata.
- Compatibility expectations for React and other dependencies.
- Release notes and a process for breaking changes.

A private `shared/ui/` folder can be valuable without publishing a package. Publish only when separate applications or consumers actually need an independently versioned dependency.

## 4. Keep feature components and design-system components distinct

Feature components express product-specific behavior:

```jsx
<TaskDueDate task={task} onChange={updateDueDate} />
```

Design-system components express reusable interface behavior:

```jsx
<Button variant="primary">Save</Button>
<TextField label="Display name" />
```

A feature component can use design-system components. Keep a component feature-owned when it relies on feature concepts or rules. Move only the stable general interface into shared UI.

A practical structure can look like this:

```text
src/
  app/
    App.jsx
    tokens.css
  features/
    profile/
      ProfileSettings.jsx
      profile-settings.css
  shared/
    ui/
      Button.jsx
      TextField.jsx
      Card.jsx
```

The names and locations should fit the project. The important part is that feature-specific behavior and shared interface primitives have clear owners.

## 5. Start with design tokens

Design tokens give repeated visual decisions names. CSS custom properties are one practical implementation:

```css
:root {
  --color-text: #192235;
  --color-surface: #ffffff;
  --color-primary: #3448b5;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --radius-control: 0.5rem;
}
```

A component can use the token rather than hard-coding a different value at each use:

```css
.button {
  border-radius: var(--radius-control);
  padding: var(--space-2) var(--space-4);
  background: var(--color-primary);
  color: var(--color-surface);
}
```

Tokens make coordinated changes and theming easier. Avoid creating a token for every one-off value before a real system exists. Start with repeated decisions and add semantic names that express purpose, such as `--color-danger`, rather than names tied only to a particular screen.

## 6. Design component APIs around intent

A useful component API exposes choices the consumer needs and hides unnecessary implementation detail.

Prefer intent-based options:

```jsx
<Button variant="danger" size="small">Delete account</Button>
```

Avoid APIs that expose internal styling details or accept many contradictory flags:

```jsx
<Button red rounded compact shadowed isDelete />
```

Useful API questions include:

- What is the default behavior?
- Which props are required?
- Which values are allowed?
- Which native element does the component render?
- Which native attributes and event handlers does it support?
- Can the caller provide content or compose the component with children?
- What happens when it is disabled, invalid, loading, or empty?

Use a small, documented set of variants. Prefer native attributes and elements when they already express the desired behavior.

## 7. Make controlled and uncontrolled behavior clear

A controlled component receives its important value from props and reports changes through a callback:

```jsx
<TextField
  label="Display name"
  value={displayName}
  onChange={(event) => setDisplayName(event.target.value)}
/>
```

An uncontrolled component may keep its own internal state or leave value management to a native input. Both approaches can be useful, but a reusable component should make its behavior predictable.

Avoid keeping a private copy of a controlled prop and trying to synchronize it with Effects. Decide which part the caller controls and which details remain internal. If the component needs a default value without a controlled value, document that behavior explicitly.

## 8. Prefer composition over a giant configurable component

A reusable component does not need to handle every possible layout through props. Use `children` or smaller components when consumers need to arrange content:

```jsx
<Card title="Profile settings">
  <TextField label="Display name" />
  <Button>Save changes</Button>
</Card>
```

For more structured patterns, expose named subcomponents or render props only when they provide a real benefit. Avoid building elaborate compound APIs before consumers need them.

## 9. Accessibility belongs in the contract

A design-system component should help consumers produce accessible interfaces by default:

- Use the correct native element, such as `button` for an action and `a` for navigation.
- Preserve keyboard interaction and visible focus.
- Give form controls programmatic labels and descriptions.
- Expose disabled, selected, expanded, and error states accurately.
- Maintain sensible heading and landmark structure.
- Provide enough API support for accessible names and helper text.
- Check contrast and avoid communicating meaning with color alone.

A generic component should not invent ARIA roles or keyboard behavior without a defined pattern. When a component implements a complex widget, specify its keyboard and assistive-technology behavior and validate that pattern deliberately.

## 10. Keep styling predictable and themable

Choose a styling strategy the project can maintain. It may use CSS modules, regular CSS, a styling library, or another system. The important part is that consumers know how to customize the component without relying on hidden selector details.

A useful style contract can include:

- Shared tokens for color, spacing, type, and radius.
- Explicit component variants and sizes.
- A clear theme or CSS custom-property interface.
- Documented class-name or slot customization only when needed.

Avoid leaking every internal element as a public styling hook. Every public hook becomes something consumers may depend on and that you may need to preserve.

## 11. Document and validate reusable components

Documentation should answer practical questions:

- What does this component do?
- Which props are supported, and what are their types and defaults?
- Is it controlled, uncontrolled, or a mix?
- What HTML element does it render?
- How should it be used accessibly?
- What does it look like in common states?
- What should consumers avoid?

Useful validation can include:

- Unit or interaction tests for behavior and state.
- Accessibility checks for labels, focus, and keyboard interaction.
- Visual review of variants, themes, and responsive states.
- A demo or component catalog that exercises actual public APIs.
- Checks that examples match the shipped package.

A screenshot alone does not verify keyboard behavior or a prop contract.

## 12. If you publish a component package

A reusable in-repository UI folder becomes a published library only after you deliberately define a package boundary. Before publishing:

1. Choose the public entry points and exports.
2. Define supported React versions and dependency expectations.
3. Ensure React is not accidentally bundled as a second copy.
4. Include the built JavaScript and any required styles.
5. Document installation, imports, theme setup, and supported components.
6. Check the package contents and test it from a consumer project.
7. Use versioning and release notes that communicate breaking changes.

Build tools can provide library-oriented output, but configuration and package fields depend on the toolchain. Follow the current tool and package-manager conventions instead of copying a configuration from a different major version.

## 13. Adopt or build?

Before building a component library, consider whether an existing library satisfies the product's requirements. Compare:

- Accessibility and interaction behavior.
- Visual fit and customization options.
- Bundle and runtime impact.
- Framework and React-version compatibility.
- Maintenance activity and release practices.
- License compatibility with the project.
- The cost of upgrades and migration.

Building a small application-level design system can make sense when a product needs a consistent visual language or has requirements that existing tools do not meet. Rebuilding common widgets without a specific reason can create a large long-term maintenance burden.

## 14. Evolve the system with compatibility in mind

Once components have multiple consumers, changing a prop or default can affect many screens or applications. For each public component, distinguish:

- Internal implementation details.
- Stable documented props and behavior.
- Deprecated APIs and planned replacements.
- Changes that require consumer action.

For a package, follow a deliberate versioning and release process. For an internal system, communicate changes and update consumers together where possible.

Avoid making every implementation detail part of the contract. A small, intentional public API leaves room to improve internals.

## 15. Common mistakes

- Calling any shared folder a complete design system.
- Publishing a package before there are independent consumers.
- Combining unrelated behaviors into a generic component with many flags.
- Moving feature-specific rules into shared UI.
- Replacing semantic HTML with clickable generic containers.
- Treating accessibility as a consumer-only responsibility.
- Exposing internal class names or DOM structure as accidental API.
- Maintaining a second internal copy of controlled props.
- Creating tokens for isolated values with no coordinated meaning.
- Relying only on screenshots to validate interactive components.
- Copying package build configuration without checking the current toolchain.
- Changing public defaults without considering consumers.

## Key takeaways

- Shared components, a design system, and a published component library are related but different things.
- Start with reusable behavior that has a clear owner and real consumers.
- Use tokens and small intent-based APIs to keep visual and interaction choices consistent.
- Preserve native semantics, keyboard behavior, labels, and accessible states.
- Document and validate the public contract, not only the rendered appearance.
- Publish a package when independent consumers need an explicit, versioned boundary.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="088-organizing-react-applications.md">088 - Organizing Components, Features, and Application Code</a></td><td><a href="../qnas/089-design-systems-and-component-libraries.md">Q&amp;A</a></td><td><a href="../examples/089-design-systems-and-component-libraries.md">Example</a></td><td><a href="090-global-state-and-usesyncexternalstore.md">090 - Global State, `useSyncExternalStore`, and External Stores</a></td></tr>
  </tbody>
</table>
