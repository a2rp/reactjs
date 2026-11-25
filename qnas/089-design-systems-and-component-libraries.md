# Design Systems, Shared Components, and Component Libraries — Q&A

## Foundations

### 1. What is a design system?

A design system is a coordinated set of visual decisions, interaction patterns, reusable implementations, guidance, and maintenance practices. It helps products feel and behave consistently while giving teams a shared way to build and evolve interfaces.

### 2. Is a design system the same thing as a component library?

No. A component library is a collection of implementation components, often distributed as a package. A design system is broader: it can include tokens, principles, accessibility guidance, patterns, documentation, and ownership processes. A library may implement part of a design system.

### 3. Is a `shared/ui/` folder automatically a design system?

No. It may be a useful location for reusable UI, but a complete design system also needs deliberate conventions, interaction behavior, documentation, and an approach to change.

### 4. Is a shared component always worth extracting?

No. Extract it when the behavior and meaning are genuinely shared and the interface improves reuse. If two usages only look similar but have different responsibilities, separate components may be clearer.

### 5. When should I publish a component package?

When separate projects or consumers need the components as an independently versioned dependency. A package requires public exports, documented setup, compatibility expectations, release practices, and a build that consumers can use. Internal reuse alone does not require publishing.

### 6. Should I build my own design system or adopt an existing library?

Compare the existing options with product needs: accessibility, visual fit, customization, compatibility, maintenance, license, and upgrade costs. Build custom components when requirements justify owning their ongoing implementation and maintenance.

## Component API design

### 7. What makes a component API good?

It exposes the decisions consumers need, uses clear names, has predictable defaults, and hides implementation details that callers should not depend on. Native HTML attributes and elements are useful when they match the component's purpose.

### 8. Why use intent-based props such as `variant="danger"`?

They describe the meaning or supported appearance of the component. They are clearer and more consistent than combinations of low-level styling flags that expose how the component happens to be implemented.

### 9. How many variants should a component have?

Enough to cover real, repeated use cases, but not every theoretical possibility. A small documented set is easier to understand and maintain. Add a variant when it has clear semantics and consumers need it.

### 10. Should a component allow a custom `className`?

Sometimes. It can be a useful escape hatch, but it also encourages consumer coupling to styles. Define a supported theming or customization interface instead of exposing every internal selector by accident.

### 11. Should every component support an `as` prop to change its HTML element?

No. Polymorphic APIs can be flexible but complicate types, semantics, and accessibility. Use them only when consumers have a concrete need and the component can preserve the correct behavior for each supported element.

### 12. How can a component accept flexible content?

Use `children` for content that belongs inside the component's structure. For structured content, use focused subcomponents or explicit props when that makes the relationship easier to understand. Avoid a complex composition API without real use cases.

### 13. Should the component use native HTML or custom ARIA roles?

Use native semantic elements whenever they express the intended behavior. Add ARIA only when necessary to convey a valid role, name, state, or relationship. A complex custom widget also requires deliberate keyboard behavior.

### 14. What should a button component support?

At minimum, it should render a real `button`, preserve standard button attributes and event handlers, support a meaningful `type`, reflect the native disabled state, and keep keyboard focus visible. Navigation should generally use a link rather than a button.

### 15. Why should a design-system text field accept a label?

A visible, programmatically associated label gives the control an accessible name and helps users understand what to enter. A placeholder is not a reliable replacement for a label.

## State and styling

### 16. What is a controlled component?

A controlled component receives its important value from props and reports proposed changes with a callback. Its parent owns the source of truth, which allows sibling components to coordinate around the same value.

### 17. What is an uncontrolled component?

An uncontrolled component keeps some important state internally or allows the native element to own its value. This can make a component simpler to use when consumers do not need to coordinate that state.

### 18. Can a component mix controlled and uncontrolled behavior?

It can, but make the contract explicit. Decide what the caller controls and what stays internal. Avoid silently switching modes or keeping a second internal copy of a controlled prop that must be synchronized.

### 19. Do custom Hooks share state between separate components?

No. A custom Hook shares reusable logic. Each component calling it has its own Hook state. To share one state value, give it a common owner and expose it through props, context, or a store.

### 20. What are design tokens?

Named values for repeated design decisions, such as semantic colors, spacing, typography, and radii. They make coordinated visual changes and themes easier. Use tokens for repeated meaningful decisions rather than creating a token for every isolated value.

### 21. Should every component use CSS variables?

Not necessarily, but tokens are useful when multiple components need to coordinate or support themes. Components may also use local styles or project-specific styling tools. Keep the customization approach understandable and consistent.

### 22. How should themes work?

Define a deliberate set of semantic tokens and assign their values for each theme. Components consume the tokens rather than embedding theme-specific values. Ensure that all supported states and colors remain legible and usable.

### 23. Where should shared component styles live?

Near their component or in the library's styling entry point, depending on the project's conventions. Feature-specific layout belongs near the feature; shared design tokens and global reset styles may belong at the application or package level.

## Accessibility and quality

### 24. Who is responsible for accessibility: the library or the consumer?

Both. The library should provide semantic defaults and a usable component contract. Consumers must provide context-specific labels, content, and correct usage. The library cannot infer every page's meaning, but it should not make accessible use unnecessarily difficult.

### 25. What should documentation say about accessibility?

Describe the rendered semantic element, required labels or names, keyboard behavior, focus behavior, state announcements, and any usage constraints. For complex widgets, specify the supported interaction pattern rather than relying only on a demo.

### 26. Is a visual screenshot enough to validate a component?

No. It can show appearance but not all interaction or accessibility behavior. Also verify keyboard usage, labels, focus, disabled and error states, and the component's prop contract.

### 27. How do I test a reusable component?

Test observable behavior through its public interface: rendered semantics, user interactions, callback behavior, and important states. Add accessibility checks and visual review where they provide value. Do not test private implementation details unless there is a strong reason.

### 28. Should a component library include a demo or catalog?

A demo or catalog is useful when it helps consumers discover components, see supported states, and try the public API. Ensure that demos exercise the shipped interface and do not depend on inaccessible internal shortcuts.

## Packaging and maintenance

### 29. Why is React commonly declared as a peer dependency in a React library?

The consuming application should provide the compatible React runtime. Bundling a separate React copy can cause duplicate-runtime problems. The library's package metadata and build configuration should both reflect the supported dependency relationship.

### 30. What does it mean to externalize React during a library build?

It means the build leaves React as an import for the consuming application rather than embedding React's runtime in the library bundle. Exact configuration depends on the build tool and output formats.

### 31. What should a published package document?

Installation, supported React versions, import paths, style or theme setup, public exports, component props and defaults, accessibility requirements, and any breaking changes or migrations.

### 32. How should component changes be versioned?

Treat documented props and behavior as a public contract. Communicate breaking changes, use release notes, and apply a versioning policy appropriate to the package or internal consumer setup. Avoid making internal implementation details part of the contract.

### 33. What is a breaking change in a component library?

A change that requires consumers to update their code or changes documented behavior incompatibly. Removing or renaming a prop, changing defaults unexpectedly, or removing a public export can be breaking. Internal refactoring with the same public behavior usually is not.

### 34. Should all components be exported from the package root?

Export components that are part of the intended public API. Keep helpers and internal implementation private unless consumers have a clear need for them. A smaller public API is easier to support.

### 35. What should I check before adopting a third-party component library?

Review accessibility behavior, visual and theme fit, React compatibility, bundle and runtime impact, maintenance and upgrade patterns, license terms, and the effort required to adapt or replace it later.

## Quick review

- A design system is broader than a component package or a shared folder.
- Extract components for real shared behavior, not only similar appearance.
- Use a small intent-based API with semantic HTML and accessible defaults.
- Make controlled and uncontrolled state behavior explicit.
- Use tokens for repeated design decisions.
- Document and validate the public contract, including keyboard behavior.
- Publish and version a package only when consumers need that boundary.
