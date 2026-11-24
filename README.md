# ReactJS: Practical Notes

These are my personal learning notes for React. This repository is planned as a practical, beginner-friendly guide that grows into a detailed reference for building accessible, reliable React web applications.

React is a JavaScript library for building user interfaces from components. The roadmap starts with the React mental model and core APIs, then covers state, effects, modern React features, application architecture, accessibility, testing, performance, and delivery.

## Quick guides

- [ReactJS summary](reactjs-summary.md)
- [ReactJS keywords and concepts](reactjs-keywords.md)

## Learning outcomes

By following the roadmap, you will learn to:

- Explain how React renders a component tree and updates the user interface.
- Build and compose components with JSX, props, events, and accessible HTML.
- Model local and shared state without unnecessary duplication or effects.
- Use built-in Hooks and create reusable custom Hooks.
- Handle forms, asynchronous work, loading states, and errors.
- Understand Suspense, hydration, server rendering, Server Components, and Actions.
- Choose appropriate routing, styling, state, and framework approaches.
- Test user-facing behavior and diagnose problems with React tools.
- Improve performance, protect user data, and prepare an application for production.

## Before you begin

Basic JavaScript is useful: functions, arrays, objects, modules, promises, and async/await. Familiarity with HTML, CSS, a terminal, and Git will make the examples easier to follow. The early chapters introduce the React-specific ideas from the ground up.

## How to use this repository

1. Read the lessons in order and try each idea in a small project.
2. Use the matching Q&A file to review the concepts.
3. Follow the matching example to see the idea applied and understand its expected behavior.
4. Build the practical features yourself before looking back at the example.
5. Use the summary and keywords files for quick review.

The roadmap links use one shared chapter number and filename across the lessons, Q&A, and examples folders. These companion files will be added as the course is written. A roadmap link identifies a planned file; it does not mean that the file is already present or complete.

## React version and project setup

This roadmap starts from stable React 19.3 as of November 24, 2026. Lessons will identify version-sensitive APIs and distinguish stable features from framework-specific behavior. React 19.2 features such as Activity and useEffectEvent, and React 19.3 features such as View Transitions and Fragment refs, are included in the relevant chapters.

The setup chapters will use a current build-tool workflow to teach React fundamentals and explain when a full-stack React framework is appropriate for an application. They will not start new projects with Create React App.

## Course roadmap

| No. | Module | Lesson | Q&A | Example |
| ---: | --- | --- | --- | --- |
| 001 | Foundations | [Introduction to React and declarative UI](lessons/001-introduction-to-react-and-declarative-ui.md) | [Open Q&A](qnas/001-introduction-to-react-and-declarative-ui.md) | [Open Example](examples/001-introduction-to-react-and-declarative-ui.md) |
| 002 | Foundations | [How React fits into the web platform](lessons/002-how-react-fits-into-the-web-platform.md) | [Open Q&A](qnas/002-how-react-fits-into-the-web-platform.md) | [Open Example](examples/002-how-react-fits-into-the-web-platform.md) |
| 003 | Foundations | [React, React DOM, and the framework ecosystem](lessons/003-react-react-dom-and-frameworks.md) | [Open Q&A](qnas/003-react-react-dom-and-frameworks.md) | [Open Example](examples/003-react-react-dom-and-frameworks.md) |
| 004 | Foundations | [JavaScript and browser prerequisites](lessons/004-javascript-and-browser-prerequisites.md) | [Open Q&A](qnas/004-javascript-and-browser-prerequisites.md) | [Open Example](examples/004-javascript-and-browser-prerequisites.md) |
| 005 | Foundations | [Choose a React project setup](lessons/005-choose-a-react-project-setup.md) | [Open Q&A](qnas/005-choose-a-react-project-setup.md) | [Open Example](examples/005-choose-a-react-project-setup.md) |
| 006 | Foundations | [Project files, scripts, and the development server](lessons/006-project-files-scripts-and-dev-server.md) | [Open Q&A](qnas/006-project-files-scripts-and-dev-server.md) | [Open Example](examples/006-project-files-scripts-and-dev-server.md) |
| 007 | Foundations | [JSX syntax and markup](lessons/007-jsx-syntax-and-markup.md) | [Open Q&A](qnas/007-jsx-syntax-and-markup.md) | [Open Example](examples/007-jsx-syntax-and-markup.md) |
| 008 | Foundations | [JavaScript expressions, attributes, and escaping in JSX](lessons/008-jsx-expressions-attributes-and-escaping.md) | [Open Q&A](qnas/008-jsx-expressions-attributes-and-escaping.md) | [Open Example](examples/008-jsx-expressions-attributes-and-escaping.md) |
| 009 | Foundations | [JSX styles, event props, and DOM properties](lessons/009-jsx-styles-events-and-dom-properties.md) | [Open Q&A](qnas/009-jsx-styles-events-and-dom-properties.md) | [Open Example](examples/009-jsx-styles-events-and-dom-properties.md) |
| 010 | Foundations | [Fragments and returning multiple elements](lessons/010-fragments-and-multiple-elements.md) | [Open Q&A](qnas/010-fragments-and-multiple-elements.md) | [Open Example](examples/010-fragments-and-multiple-elements.md) |
| 011 | Foundations | [Create a root and render a React application](lessons/011-create-a-root-and-render-an-application.md) | [Open Q&A](qnas/011-create-a-root-and-render-an-application.md) | [Open Example](examples/011-create-a-root-and-render-an-application.md) |
| 012 | Foundations | [Strict Mode and development checks](lessons/012-strict-mode-and-development-checks.md) | [Open Q&A](qnas/012-strict-mode-and-development-checks.md) | [Open Example](examples/012-strict-mode-and-development-checks.md) |
| 013 | Components and JSX | [Function components and component naming](lessons/013-function-components-and-naming.md) | [Open Q&A](qnas/013-function-components-and-naming.md) | [Open Example](examples/013-function-components-and-naming.md) |
| 014 | Components and JSX | [Props and passing data](lessons/014-props-and-passing-data.md) | [Open Q&A](qnas/014-props-and-passing-data.md) | [Open Example](examples/014-props-and-passing-data.md) |
| 015 | Components and JSX | [Destructuring props and default values](lessons/015-destructuring-props-and-default-values.md) | [Open Q&A](qnas/015-destructuring-props-and-default-values.md) | [Open Example](examples/015-destructuring-props-and-default-values.md) |
| 016 | Components and JSX | [The children prop and nested content](lessons/016-children-prop-and-nested-content.md) | [Open Q&A](qnas/016-children-prop-and-nested-content.md) | [Open Example](examples/016-children-prop-and-nested-content.md) |
| 017 | Components and JSX | [Composition, containment, and reusable layouts](lessons/017-composition-containment-and-layouts.md) | [Open Q&A](qnas/017-composition-containment-and-layouts.md) | [Open Example](examples/017-composition-containment-and-layouts.md) |
| 018 | Components and JSX | [Conditional rendering patterns](lessons/018-conditional-rendering-patterns.md) | [Open Q&A](qnas/018-conditional-rendering-patterns.md) | [Open Example](examples/018-conditional-rendering-patterns.md) |
| 019 | Components and JSX | [Rendering collections and choosing stable keys](lessons/019-rendering-collections-and-stable-keys.md) | [Open Q&A](qnas/019-rendering-collections-and-stable-keys.md) | [Open Example](examples/019-rendering-collections-and-stable-keys.md) |
| 020 | Components and JSX | [Component boundaries and responsibility](lessons/020-component-boundaries-and-responsibility.md) | [Open Q&A](qnas/020-component-boundaries-and-responsibility.md) | [Open Example](examples/020-component-boundaries-and-responsibility.md) |
| 021 | Components and JSX | [Pure components and predictable rendering](lessons/021-pure-components-and-predictable-rendering.md) | [Open Q&A](qnas/021-pure-components-and-predictable-rendering.md) | [Open Example](examples/021-pure-components-and-predictable-rendering.md) |
| 022 | Components and JSX | [Designing reusable component APIs](lessons/022-designing-reusable-component-apis.md) | [Open Q&A](qnas/022-designing-reusable-component-apis.md) | [Open Example](examples/022-designing-reusable-component-apis.md) |
| 023 | Components and JSX | [Thinking in React: from a design to a component tree](lessons/023-thinking-in-react-and-component-trees.md) | [Open Q&A](qnas/023-thinking-in-react-and-component-trees.md) | [Open Example](examples/023-thinking-in-react-and-component-trees.md) |
| 024 | Components and JSX | [Component identity, position, and keys](lessons/024-component-identity-position-and-keys.md) | [Open Q&A](qnas/024-component-identity-position-and-keys.md) | [Open Example](examples/024-component-identity-position-and-keys.md) |
| 025 | Events and forms | [Event handlers and user interactions](lessons/025-event-handlers-and-user-interactions.md) | [Open Q&A](qnas/025-event-handlers-and-user-interactions.md) | [Open Example](examples/025-event-handlers-and-user-interactions.md) |
| 026 | Events and forms | [Event objects and React's event system](lessons/026-event-objects-and-react-event-system.md) | [Open Q&A](qnas/026-event-objects-and-react-event-system.md) | [Open Example](examples/026-event-objects-and-react-event-system.md) |
| 027 | Events and forms | [Event propagation, default actions, and cancellation](lessons/027-event-propagation-and-default-actions.md) | [Open Q&A](qnas/027-event-propagation-and-default-actions.md) | [Open Example](examples/027-event-propagation-and-default-actions.md) |
| 028 | Events and forms | [Controlled form elements](lessons/028-controlled-form-elements.md) | [Open Q&A](qnas/028-controlled-form-elements.md) | [Open Example](examples/028-controlled-form-elements.md) |
| 029 | Events and forms | [Text inputs, selects, checkboxes, and radio buttons](lessons/029-inputs-selects-checkboxes-and-radio-buttons.md) | [Open Q&A](qnas/029-inputs-selects-checkboxes-and-radio-buttons.md) | [Open Example](examples/029-inputs-selects-checkboxes-and-radio-buttons.md) |
| 030 | Events and forms | [Form submission and browser validation](lessons/030-form-submission-and-browser-validation.md) | [Open Q&A](qnas/030-form-submission-and-browser-validation.md) | [Open Example](examples/030-form-submission-and-browser-validation.md) |
| 031 | Events and forms | [Accessible validation messages and error summaries](lessons/031-accessible-validation-messages.md) | [Open Q&A](qnas/031-accessible-validation-messages.md) | [Open Example](examples/031-accessible-validation-messages.md) |
| 032 | Events and forms | [Managing state for multi-field forms](lessons/032-multi-field-form-state.md) | [Open Q&A](qnas/032-multi-field-form-state.md) | [Open Example](examples/032-multi-field-form-state.md) |
| 033 | Events and forms | [Uncontrolled forms, refs, and default values](lessons/033-uncontrolled-forms-and-refs.md) | [Open Q&A](qnas/033-uncontrolled-forms-and-refs.md) | [Open Example](examples/033-uncontrolled-forms-and-refs.md) |
| 034 | Events and forms | [File inputs and upload flows](lessons/034-file-inputs-and-upload-flows.md) | [Open Q&A](qnas/034-file-inputs-and-upload-flows.md) | [Open Example](examples/034-file-inputs-and-upload-flows.md) |
| 035 | Events and forms | [Working with browser APIs and external event listeners](lessons/035-browser-apis-and-event-listeners.md) | [Open Q&A](qnas/035-browser-apis-and-event-listeners.md) | [Open Example](examples/035-browser-apis-and-event-listeners.md) |
| 036 | Events and forms | [Build an accessible interactive form feature](lessons/036-accessible-interactive-form-project.md) | [Open Q&A](qnas/036-accessible-interactive-form-project.md) | [Open Example](examples/036-accessible-interactive-form-project.md) |
| 037 | State and data flow | [State with useState](lessons/037-state-with-usestate.md) | [Open Q&A](qnas/037-state-with-usestate.md) | [Open Example](examples/037-state-with-usestate.md) |
| 038 | State and data flow | [State as a render snapshot](lessons/038-state-as-a-render-snapshot.md) | [Open Q&A](qnas/038-state-as-a-render-snapshot.md) | [Open Example](examples/038-state-as-a-render-snapshot.md) |
| 039 | State and data flow | [Queued updates, batching, and functional updates](lessons/039-queued-batched-and-functional-updates.md) | [Open Q&A](qnas/039-queued-batched-and-functional-updates.md) | [Open Example](examples/039-queued-batched-and-functional-updates.md) |
| 040 | State and data flow | [Updating object state immutably](lessons/040-updating-object-state-immutably.md) | [Open Q&A](qnas/040-updating-object-state-immutably.md) | [Open Example](examples/040-updating-object-state-immutably.md) |
| 041 | State and data flow | [Updating array state immutably](lessons/041-updating-array-state-immutably.md) | [Open Q&A](qnas/041-updating-array-state-immutably.md) | [Open Example](examples/041-updating-array-state-immutably.md) |
| 042 | State and data flow | [Derived values and avoiding duplicated state](lessons/042-derived-values-and-duplicated-state.md) | [Open Q&A](qnas/042-derived-values-and-duplicated-state.md) | [Open Example](examples/042-derived-values-and-duplicated-state.md) |
| 043 | State and data flow | [Lifting state up and sharing data](lessons/043-lifting-state-up-and-sharing-data.md) | [Open Q&A](qnas/043-lifting-state-up-and-sharing-data.md) | [Open Example](examples/043-lifting-state-up-and-sharing-data.md) |
| 044 | State and data flow | [Single source of truth and state ownership](lessons/044-single-source-of-truth-and-state-ownership.md) | [Open Q&A](qnas/044-single-source-of-truth-and-state-ownership.md) | [Open Example](examples/044-single-source-of-truth-and-state-ownership.md) |
| 045 | State and data flow | [Controlled and uncontrolled component APIs](lessons/045-controlled-and-uncontrolled-component-apis.md) | [Open Q&A](qnas/045-controlled-and-uncontrolled-component-apis.md) | [Open Example](examples/045-controlled-and-uncontrolled-component-apis.md) |
| 046 | State and data flow | [Complex state with useReducer](lessons/046-complex-state-with-usereducer.md) | [Open Q&A](qnas/046-complex-state-with-usereducer.md) | [Open Example](examples/046-complex-state-with-usereducer.md) |
| 047 | State and data flow | [Reducer design, transitions, and state machines](lessons/047-reducer-design-and-state-machines.md) | [Open Q&A](qnas/047-reducer-design-and-state-machines.md) | [Open Example](examples/047-reducer-design-and-state-machines.md) |
| 048 | State and data flow | [Preserving and resetting state with tree position and keys](lessons/048-preserving-and-resetting-state.md) | [Open Q&A](qnas/048-preserving-and-resetting-state.md) | [Open Example](examples/048-preserving-and-resetting-state.md) |
| 049 | Hooks and effects | [Rules of Hooks and why they matter](lessons/049-rules-of-hooks.md) | [Open Q&A](qnas/049-rules-of-hooks.md) | [Open Example](examples/049-rules-of-hooks.md) |
| 050 | Hooks and effects | [Context and useContext](lessons/050-context-and-usecontext.md) | [Open Q&A](qnas/050-context-and-usecontext.md) | [Open Example](examples/050-context-and-usecontext.md) |
| 051 | Hooks and effects | [Designing context providers and avoiding overuse](lessons/051-context-provider-design.md) | [Open Q&A](qnas/051-context-provider-design.md) | [Open Example](examples/051-context-provider-design.md) |
| 052 | Hooks and effects | [Effects with useEffect, useLayoutEffect, and useInsertionEffect](lessons/052-effect-hooks-and-synchronization.md) | [Open Q&A](qnas/052-effect-hooks-and-synchronization.md) | [Open Example](examples/052-effect-hooks-and-synchronization.md) |
| 053 | Hooks and effects | [Effect dependencies, closures, and synchronization](lessons/053-effect-dependencies-and-closures.md) | [Open Q&A](qnas/053-effect-dependencies-and-closures.md) | [Open Example](examples/053-effect-dependencies-and-closures.md) |
| 054 | Hooks and effects | [Effect cleanup and resource lifetimes](lessons/054-effect-cleanup-and-resource-lifetimes.md) | [Open Q&A](qnas/054-effect-cleanup-and-resource-lifetimes.md) | [Open Example](examples/054-effect-cleanup-and-resource-lifetimes.md) |
| 055 | Hooks and effects | [When an effect is unnecessary](lessons/055-when-an-effect-is-unnecessary.md) | [Open Q&A](qnas/055-when-an-effect-is-unnecessary.md) | [Open Example](examples/055-when-an-effect-is-unnecessary.md) |
| 056 | Hooks and effects | [useEffectEvent and non-reactive event logic](lessons/056-useeffectevent.md) | [Open Q&A](qnas/056-useeffectevent.md) | [Open Example](examples/056-useeffectevent.md) |
| 057 | Hooks and effects | [Custom Hooks and useDebugValue](lessons/057-custom-hooks-and-usedebugvalue.md) | [Open Q&A](qnas/057-custom-hooks-and-usedebugvalue.md) | [Open Example](examples/057-custom-hooks-and-usedebugvalue.md) |
| 058 | Hooks and effects | [useRef for DOM nodes and mutable values](lessons/058-useref.md) | [Open Q&A](qnas/058-useref.md) | [Open Example](examples/058-useref.md) |
| 059 | Hooks and effects | [React 19 refs, forwardRef, and imperative handles](lessons/059-refs-and-imperative-handles.md) | [Open Q&A](qnas/059-refs-and-imperative-handles.md) | [Open Example](examples/059-refs-and-imperative-handles.md) |
| 060 | Hooks and effects | [Memoization, useId, useMemo, and useCallback](lessons/060-memoization-useid-usememo-usecallback.md) | [Open Q&A](qnas/060-memoization-useid-usememo-usecallback.md) | [Open Example](examples/060-memoization-useid-usememo-usecallback.md) |
| 061 | Rendering and React DOM | [Rendering, committing, and updating the UI](lessons/061-rendering-and-committing.md) | [Open Q&A](qnas/061-rendering-and-committing.md) | [Open Example](examples/061-rendering-and-committing.md) |
| 062 | Rendering and React DOM | [Reconciliation, component identity, and keys](lessons/062-reconciliation-and-component-identity.md) | [Open Q&A](qnas/062-reconciliation-and-component-identity.md) | [Open Example](examples/062-reconciliation-and-component-identity.md) |
| 063 | Rendering and React DOM | [Concurrent rendering and update priorities](lessons/063-concurrent-rendering-and-priorities.md) | [Open Q&A](qnas/063-concurrent-rendering-and-priorities.md) | [Open Example](examples/063-concurrent-rendering-and-priorities.md) |
| 064 | Rendering and React DOM | [Transitions with useTransition and startTransition](lessons/064-transitions-and-usetransition.md) | [Open Q&A](qnas/064-transitions-and-usetransition.md) | [Open Example](examples/064-transitions-and-usetransition.md) |
| 065 | Rendering and React DOM | [useDeferredValue for interruptible updates](lessons/065-usedeferredvalue.md) | [Open Q&A](qnas/065-usedeferredvalue.md) | [Open Example](examples/065-usedeferredvalue.md) |
| 066 | Rendering and React DOM | [Suspense boundaries and fallbacks](lessons/066-suspense-boundaries-and-fallbacks.md) | [Open Q&A](qnas/066-suspense-boundaries-and-fallbacks.md) | [Open Example](examples/066-suspense-boundaries-and-fallbacks.md) |
| 067 | Rendering and React DOM | [Lazy loading components with lazy](lessons/067-lazy-loading-components.md) | [Open Q&A](qnas/067-lazy-loading-components.md) | [Open Example](examples/067-lazy-loading-components.md) |
| 068 | Rendering and React DOM | [Error boundaries and recovery UI](lessons/068-error-boundaries-and-recovery-ui.md) | [Open Q&A](qnas/068-error-boundaries-and-recovery-ui.md) | [Open Example](examples/068-error-boundaries-and-recovery-ui.md) |
| 069 | Rendering and React DOM | [Portals and rendering outside the parent DOM node](lessons/069-portals.md) | [Open Q&A](qnas/069-portals.md) | [Open Example](examples/069-portals.md) |
| 070 | Rendering and React DOM | [React DOM roots, hydration, and client APIs](lessons/070-react-dom-roots-and-client-apis.md) | [Open Q&A](qnas/070-react-dom-roots-and-client-apis.md) | [Open Example](examples/070-react-dom-roots-and-client-apis.md) |
| 071 | Rendering and React DOM | [Synchronous DOM updates with flushSync](lessons/071-flushsync.md) | [Open Q&A](qnas/071-flushsync.md) | [Open Example](examples/071-flushsync.md) |
| 072 | Rendering and React DOM | [React DOM resource hints and browser APIs](lessons/072-react-dom-resource-hints-and-browser-apis.md) | [Open Q&A](qnas/072-react-dom-resource-hints-and-browser-apis.md) | [Open Example](examples/072-react-dom-resource-hints-and-browser-apis.md) |
| 073 | Async data and server features | [Client-side data fetching, the use API, and async values](lessons/073-client-side-data-fetching-and-use-api.md) | [Open Q&A](qnas/073-client-side-data-fetching-and-use-api.md) | [Open Example](examples/073-client-side-data-fetching-and-use-api.md) |
| 074 | Async data and server features | [Request cancellation, stale responses, and race conditions](lessons/074-request-cancellation-and-race-conditions.md) | [Open Q&A](qnas/074-request-cancellation-and-race-conditions.md) | [Open Example](examples/074-request-cancellation-and-race-conditions.md) |
| 075 | Async data and server features | [Loading, empty, success, and error states](lessons/075-loading-empty-success-and-error-states.md) | [Open Q&A](qnas/075-loading-empty-success-and-error-states.md) | [Open Example](examples/075-loading-empty-success-and-error-states.md) |
| 076 | Async data and server features | [Caching, deduplication, invalidation, and revalidation](lessons/076-caching-and-revalidation.md) | [Open Q&A](qnas/076-caching-and-revalidation.md) | [Open Example](examples/076-caching-and-revalidation.md) |
| 077 | Async data and server features | [Async UI, Suspense, and framework data loading](lessons/077-async-ui-and-framework-data-loading.md) | [Open Q&A](qnas/077-async-ui-and-framework-data-loading.md) | [Open Example](examples/077-async-ui-and-framework-data-loading.md) |
| 078 | Async data and server features | [React form Actions and asynchronous submissions](lessons/078-react-form-actions.md) | [Open Q&A](qnas/078-react-form-actions.md) | [Open Example](examples/078-react-form-actions.md) |
| 079 | Async data and server features | [Managing submissions with useActionState](lessons/079-useactionstate.md) | [Open Q&A](qnas/079-useactionstate.md) | [Open Example](examples/079-useactionstate.md) |
| 080 | Async data and server features | [Pending form state with useFormStatus](lessons/080-useformstatus.md) | [Open Q&A](qnas/080-useformstatus.md) | [Open Example](examples/080-useformstatus.md) |
| 081 | Async data and server features | [Optimistic UI with useOptimistic](lessons/081-useoptimistic.md) | [Open Q&A](qnas/081-useoptimistic.md) | [Open Example](examples/081-useoptimistic.md) |
| 082 | Async data and server features | [Server Functions and form actions](lessons/082-server-functions-and-form-actions.md) | [Open Q&A](qnas/082-server-functions-and-form-actions.md) | [Open Example](examples/082-server-functions-and-form-actions.md) |
| 083 | Async data and server features | [React Server Components and server rendering](lessons/083-react-server-components.md) | [Open Q&A](qnas/083-react-server-components.md) | [Open Example](examples/083-react-server-components.md) |
| 084 | Async data and server features | [Client and server boundaries, serialization, and security](lessons/084-client-server-boundaries-and-serialization.md) | [Open Q&A](qnas/084-client-server-boundaries-and-serialization.md) | [Open Example](examples/084-client-server-boundaries-and-serialization.md) |
| 085 | Application architecture | [Routing concepts and choosing a router](lessons/085-routing-concepts-and-router-choice.md) | [Open Q&A](qnas/085-routing-concepts-and-router-choice.md) | [Open Example](examples/085-routing-concepts-and-router-choice.md) |
| 086 | Application architecture | [Nested routes, layouts, and route errors](lessons/086-nested-routes-layouts-and-errors.md) | [Open Q&A](qnas/086-nested-routes-layouts-and-errors.md) | [Open Example](examples/086-nested-routes-layouts-and-errors.md) |
| 087 | Application architecture | [URL state, search parameters, and navigation](lessons/087-url-state-and-navigation.md) | [Open Q&A](qnas/087-url-state-and-navigation.md) | [Open Example](examples/087-url-state-and-navigation.md) |
| 088 | Application architecture | [Organizing components, features, and application code](lessons/088-organizing-react-applications.md) | [Open Q&A](qnas/088-organizing-react-applications.md) | [Open Example](examples/088-organizing-react-applications.md) |
| 089 | Application architecture | [Design systems, shared components, and component libraries](lessons/089-design-systems-and-component-libraries.md) | [Open Q&A](qnas/089-design-systems-and-component-libraries.md) | [Open Example](examples/089-design-systems-and-component-libraries.md) |
| 090 | Application architecture | [Global state, useSyncExternalStore, and external stores](lessons/090-global-state-and-usesyncexternalstore.md) | [Open Q&A](qnas/090-global-state-and-usesyncexternalstore.md) | [Open Example](examples/090-global-state-and-usesyncexternalstore.md) |
| 091 | Application architecture | [TypeScript with React components and Hooks](lessons/091-typescript-with-react.md) | [Open Q&A](qnas/091-typescript-with-react.md) | [Open Example](examples/091-typescript-with-react.md) |
| 092 | Application architecture | [Styling strategies, CSS Modules, and theming](lessons/092-styling-and-theming.md) | [Open Q&A](qnas/092-styling-and-theming.md) | [Open Example](examples/092-styling-and-theming.md) |
| 093 | Application architecture | [Animation, motion preferences, and View Transitions](lessons/093-animation-and-motion-preferences.md) | [Open Q&A](qnas/093-animation-and-motion-preferences.md) | [Open Example](examples/093-animation-and-motion-preferences.md) |
| 094 | Application architecture | [Internationalization, localization, and text direction](lessons/094-internationalization-and-localization.md) | [Open Q&A](qnas/094-internationalization-and-localization.md) | [Open Example](examples/094-internationalization-and-localization.md) |
| 095 | Application architecture | [Frameworks, client rendering, SSR, and static generation](lessons/095-frameworks-and-rendering-strategies.md) | [Open Q&A](qnas/095-frameworks-and-rendering-strategies.md) | [Open Example](examples/095-frameworks-and-rendering-strategies.md) |
| 096 | Application architecture | [React Native and the shared React mental model](lessons/096-react-native-and-shared-react-concepts.md) | [Open Q&A](qnas/096-react-native-and-shared-react-concepts.md) | [Open Example](examples/096-react-native-and-shared-react-concepts.md) |
| 097 | Accessibility, testing, and debugging | [Semantic HTML and accessible React components](lessons/097-semantic-html-and-accessible-components.md) | [Open Q&A](qnas/097-semantic-html-and-accessible-components.md) | [Open Example](examples/097-semantic-html-and-accessible-components.md) |
| 098 | Accessibility, testing, and debugging | [Keyboard support, focus, and dialogs](lessons/098-keyboard-focus-and-dialogs.md) | [Open Q&A](qnas/098-keyboard-focus-and-dialogs.md) | [Open Example](examples/098-keyboard-focus-and-dialogs.md) |
| 099 | Accessibility, testing, and debugging | [Accessible announcements and asynchronous interfaces](lessons/099-accessible-announcements-and-async-ui.md) | [Open Q&A](qnas/099-accessible-announcements-and-async-ui.md) | [Open Example](examples/099-accessible-announcements-and-async-ui.md) |
| 100 | Accessibility, testing, and debugging | [Testing strategy and testable component design](lessons/100-testing-strategy-and-component-design.md) | [Open Q&A](qnas/100-testing-strategy-and-component-design.md) | [Open Example](examples/100-testing-strategy-and-component-design.md) |
| 101 | Accessibility, testing, and debugging | [Component tests and React test utilities](lessons/101-component-tests-and-test-utilities.md) | [Open Q&A](qnas/101-component-tests-and-test-utilities.md) | [Open Example](examples/101-component-tests-and-test-utilities.md) |
| 102 | Accessibility, testing, and debugging | [Testing user interactions and complete flows](lessons/102-testing-user-interactions.md) | [Open Q&A](qnas/102-testing-user-interactions.md) | [Open Example](examples/102-testing-user-interactions.md) |
| 103 | Accessibility, testing, and debugging | [Testing forms, asynchronous UI, and API boundaries](lessons/103-testing-forms-and-async-ui.md) | [Open Q&A](qnas/103-testing-forms-and-async-ui.md) | [Open Example](examples/103-testing-forms-and-async-ui.md) |
| 104 | Accessibility, testing, and debugging | [Browser-level tests and end-to-end behavior](lessons/104-browser-level-and-end-to-end-tests.md) | [Open Q&A](qnas/104-browser-level-and-end-to-end-tests.md) | [Open Example](examples/104-browser-level-and-end-to-end-tests.md) |
| 105 | Accessibility, testing, and debugging | [React DevTools, debugging, and common warnings](lessons/105-react-devtools-and-debugging.md) | [Open Q&A](qnas/105-react-devtools-and-debugging.md) | [Open Example](examples/105-react-devtools-and-debugging.md) |
| 106 | Accessibility, testing, and debugging | [Profiling component behavior during development](lessons/106-profiling-during-development.md) | [Open Q&A](qnas/106-profiling-during-development.md) | [Open Example](examples/106-profiling-during-development.md) |
| 107 | Performance, security, and delivery | [React performance fundamentals and profiling](lessons/107-react-performance-and-profiling.md) | [Open Q&A](qnas/107-react-performance-and-profiling.md) | [Open Example](examples/107-react-performance-and-profiling.md) |
| 108 | Performance, security, and delivery | [React Compiler and automatic optimization](lessons/108-react-compiler.md) | [Open Q&A](qnas/108-react-compiler.md) | [Open Example](examples/108-react-compiler.md) |
| 109 | Performance, security, and delivery | [Memoization with memo and optimization decisions](lessons/109-memo-and-optimization-decisions.md) | [Open Q&A](qnas/109-memo-and-optimization-decisions.md) | [Open Example](examples/109-memo-and-optimization-decisions.md) |
| 110 | Performance, security, and delivery | [Large lists, virtualization, and responsive rendering](lessons/110-large-lists-and-virtualization.md) | [Open Q&A](qnas/110-large-lists-and-virtualization.md) | [Open Example](examples/110-large-lists-and-virtualization.md) |
| 111 | Performance, security, and delivery | [Code splitting, asset loading, and bundle analysis](lessons/111-code-splitting-and-asset-loading.md) | [Open Q&A](qnas/111-code-splitting-and-asset-loading.md) | [Open Example](examples/111-code-splitting-and-asset-loading.md) |
| 112 | Performance, security, and delivery | [Security basics: escaping, URLs, and untrusted data](lessons/112-security-and-untrusted-data.md) | [Open Q&A](qnas/112-security-and-untrusted-data.md) | [Open Example](examples/112-security-and-untrusted-data.md) |
| 113 | Performance, security, and delivery | [HTML injection, sanitization, and Trusted Types](lessons/113-html-injection-and-trusted-types.md) | [Open Q&A](qnas/113-html-injection-and-trusted-types.md) | [Open Example](examples/113-html-injection-and-trusted-types.md) |
| 114 | Performance, security, and delivery | [Production builds, configuration, and deployment](lessons/114-production-builds-and-deployment.md) | [Open Q&A](qnas/114-production-builds-and-deployment.md) | [Open Example](examples/114-production-builds-and-deployment.md) |
| 115 | Performance, security, and delivery | [Production errors, monitoring, and maintenance](lessons/115-production-monitoring-and-maintenance.md) | [Open Q&A](qnas/115-production-monitoring-and-maintenance.md) | [Open Example](examples/115-production-monitoring-and-maintenance.md) |
| 116 | Performance, security, and delivery | [Build and ship a complete accessible React application](lessons/116-react-capstone-project.md) | [Open Q&A](qnas/116-react-capstone-project.md) | [Open Example](examples/116-react-capstone-project.md) |
| 117 | Performance, security, and delivery | [Upgrading React and handling version changes](lessons/117-react-upgrades-and-version-changes.md) | [Open Q&A](qnas/117-react-upgrades-and-version-changes.md) | [Open Example](examples/117-react-upgrades-and-version-changes.md) |
| 118 | Performance, security, and delivery | [React 19.2: Activity and performance tooling](lessons/118-react-19-2-features.md) | [Open Q&A](qnas/118-react-19-2-features.md) | [Open Example](examples/118-react-19-2-features.md) |
| 119 | Performance, security, and delivery | [React 19.3: View Transitions, Fragment refs, and browser APIs](lessons/119-react-19-3-features.md) | [Open Q&A](qnas/119-react-19-3-features.md) | [Open Example](examples/119-react-19-3-features.md) |
| 120 | Performance, security, and delivery | [React quick reference and glossary](lessons/120-react-quick-reference-and-glossary.md) | [Open Q&A](qnas/120-react-quick-reference-and-glossary.md) | [Open Example](examples/120-react-quick-reference-and-glossary.md) |

The roadmap is the planned course sequence. The lesson, Q&A, and example files will be created and completed in order as the notes are developed.
