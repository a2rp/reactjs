# ReactJS Keywords and Concepts

A practical glossary for the React learning roadmap. It defines React terms, built-in APIs, rendering concepts, and common ecosystem vocabulary so you can quickly look up unfamiliar names while studying.

This file is a companion reference, not a substitute for the lessons. Roadmap: [README](README.md).

## Version scope

This glossary describes the stable React 19 API available on November 24, 2026. A label such as **React 19.2** or **React 19.3** identifies the release in which a feature became stable. Framework features and third-party tools are marked separately because their behavior and release schedules are not controlled by React itself. Experimental and Canary APIs are not presented as generally available.

## Core mental model

| Term | Meaning |
| --- | --- |
| React | A JavaScript library for describing and rendering user interfaces from components. |
| Declarative UI | Describing the interface that should appear for current inputs and state, instead of manually issuing DOM changes for each update. |
| Component | A reusable unit of UI, commonly a JavaScript function that returns React nodes. |
| Component tree | The nested structure formed when components render other components. |
| Element | A lightweight description of what React should render. JSX usually compiles into calls that create elements. |
| Node | Anything React can render as a child, including elements, text, numbers, empty values, arrays of nodes, and portals. |
| JSX | JavaScript syntax extension for writing element descriptions with markup-like syntax. JSX is transformed by the build tool; browsers do not execute it directly. |
| Render | React calls components to calculate the next UI description from props, state, and context. Rendering should be pure. |
| Commit | The phase when React applies the completed render to the host environment, such as updating browser DOM nodes and running relevant effects. |
| Re-render | Another render of a component, usually caused by changed state, changed context, or a parent rendering. A render does not always produce a DOM change. |
| Reconciliation | React's process for comparing the previous and next UI descriptions and deciding what work is needed. |
| Identity | React's way of matching a component instance between renders. Type, tree position, and keys affect whether state is preserved or reset. |
| Purity | A component or calculation gives the same result for the same inputs and does not cause observable side effects while rendering. |
| Side effect | Work that interacts with something outside the render calculation, such as subscribing to an event source, changing a title, or starting a timer. |
| Source of truth | The place that owns the authoritative value. Keeping one source of truth avoids conflicting copies of the same state. |
| One-way data flow | Data generally moves from parent components to children through props; children communicate requested changes through callbacks or shared state mechanisms. |
| Host environment | The platform React renders into. React DOM targets the browser DOM; other renderers can target other platforms. |

## Components, props, and JSX

| Term | Meaning |
| --- | --- |
| Function component | A JavaScript function whose inputs are props and whose result describes UI. Component names begin with a capital letter in JSX. |
| Root component | The component at the top of an application or a particular rendered subtree. |
| Props | Read-only inputs passed to a component by its parent. A component should not mutate its props. |
| children | The conventional prop containing nested content placed between a component's opening and closing JSX tags. It may be any renderable React node. |
| Composition | Building larger interfaces by combining smaller components and passing content or behavior through their props. |
| Component API | The props and documented behavior a component exposes to its callers. |
| Conditional rendering | Choosing which nodes to return based on a condition, using normal JavaScript expressions and control flow. |
| List rendering | Producing multiple nodes from an array, commonly with methods such as `map`. |
| Key | A stable identifier on a list child that helps React match that child across insertions, removals, and reordering. Keys are local to siblings and are not passed as props. |
| Fragment | A grouping of children that does not add a wrapper DOM element. Short syntax is `<></>`; use `<Fragment key={...}>` when a keyed fragment is needed. |
| Fragment ref / FragmentInstance | **React 19.3.** A ref on an explicit `<Fragment>` provides group operations for its first-level DOM children, such as focus, event handling, observation, and measurement, without adding a wrapper element. |
| JSX expression | JavaScript expression embedded in JSX with braces, such as `{name}` or `{items.map(...) }`. |
| JSX attribute | A prop-like value written on a JSX element. Most DOM attributes use React's DOM property conventions, for example `className` and `htmlFor`. |
| Spread props | Passing object properties as props with syntax such as `<Card {...options} />`. Use deliberately so unintended values are not forwarded. |
| Event prop | A callback prop such as `onClick` or `onChange` that React invokes for a matching interaction. |
| Boolean prop | A prop whose presence or value communicates a true/false option. DOM boolean attributes have specific rendering rules. |
| Default prop value | A fallback value supplied by JavaScript parameter defaults or component logic when a prop is missing or undefined. |
| Controlled component | An input component whose important value is driven by props or React state. The parent controls the displayed value. |
| Uncontrolled component | A form element that keeps its current value in the DOM; React may set its initial value and read it later through a ref or form submission. |
| Lift state up | Move state to the closest common parent that needs to coordinate or share it with multiple descendants. |
| Inversion of control | Let a caller provide a component, callback, or child content so a reusable component can customize behavior without knowing every detail. |
| Render prop | A function passed as a prop or child and called to produce UI. It is a composition technique, not a special React API. |

## State and data flow

| Term | Meaning |
| --- | --- |
| State | Data owned by a component or another state system that can change over time and affect rendered UI. |
| Local state | State used by one component or a small subtree, often held with `useState` or `useReducer`. |
| Shared state | State needed by multiple components, owned by a common ancestor, context provider, or an external state store. |
| State snapshot | The state value visible to a particular render. Event handlers created by that render observe that render's values. |
| State setter | The function returned by `useState` that requests a future render with a new state value. Calling it does not mutate the current render's snapshot. |
| Functional updater | A function passed to a state setter, such as `setCount(c => c + 1)`, so an update is calculated from the queued previous value. |
| Update queue | React's queued state updates, processed to calculate the next rendered state. |
| Batching | Grouping multiple state updates so React can render them together, reducing unnecessary work while preserving update ordering. |
| Immutable update | Creating a replacement object or array rather than changing existing state in place. This makes changes easier for React and developers to detect and reason about. |
| Derived value | A value that can be calculated from existing props or state during render. Avoid storing a duplicate unless there is a clear reason. |
| Reducer | A pure function that receives current state and an action and returns the next state. Commonly used with `useReducer`. |
| Action (reducer) | An object or value describing an event that a reducer uses to calculate a state update. Do not confuse it with form Actions. |
| Context | A way to make a value available to descendants without passing it through every intermediate component. |
| Context provider | The component that supplies a context value to descendants. React 19 also allows rendering the context object itself as a provider; `.Provider` remains supported. |
| Prop drilling | Passing a value through several components that do not use it, only so a deeper component can receive it. Composition or context may be alternatives. |
| State colocation | Keeping state near the components that use it, which can reduce unnecessary sharing and simplify ownership. |
| State normalization | Storing shared entities by stable identifiers instead of duplicating nested copies that can become inconsistent. |
| State machine | A model that describes a finite set of states and the allowed events or transitions between them. It can help represent complex workflows. |

## Hooks and related APIs

Hooks are called at the top level of function components or custom Hooks, following the Rules of Hooks. A Hook name starts with `use`; the name alone does not make an ordinary function a React Hook.

| Term or API | Meaning |
| --- | --- |
| Rules of Hooks | Call Hooks unconditionally and in the same order on every render; call them only from function components or custom Hooks. |
| `useState` | Adds a state value and setter to a function component. |
| `useReducer` | Adds reducer-managed state when updates are naturally expressed as actions or involve coordinated transitions. |
| `useContext` | Reads the nearest matching context value above the calling component. |
| `useRef` | Holds a mutable `.current` value between renders without causing a render when it changes. Often used to access a DOM node or retain non-visual data. |
| `useEffect` | Synchronizes a component with an external system after a commit, such as a subscription, timer, or imperative browser API. It is not a general tool for deriving render data. |
| Effect setup | The function in an Effect that starts synchronization or connects to an external system. |
| Effect cleanup | The function returned by an Effect that undoes or stops its setup, such as removing a listener or disconnecting. |
| Effect dependency list | The reactive values read by an Effect. React compares them between commits; include all reactive values used by the Effect. |
| `useLayoutEffect` | Runs layout-related synchronization after DOM mutations and before the browser paints. Use sparingly because it can delay painting. |
| `useInsertionEffect` | A specialized Hook for CSS-in-JS libraries to insert styles before layout Effects. Most application code should not need it. |
| `useEffectEvent` | **React 19.2.** Defines an Effect Event for non-reactive event logic called from an Effect. It sees current props and state, but it must not be used to hide real Effect dependencies or passed as a general callback. |
| `useMemo` | Caches a calculation between renders when dependencies are unchanged. It is a performance optimization, not a correctness or state mechanism. |
| `useCallback` | Caches a function definition between renders when dependencies are unchanged. It is mainly useful when function identity matters to an optimization or API. |
| `useImperativeHandle` | Customizes the value exposed through a component ref, usually to provide a small imperative API. |
| `useDebugValue` | Adds a label for a custom Hook in React Developer Tools. |
| `useId` | Generates IDs suitable for connecting accessibility relationships in server and client rendering. Do not use it for list keys. |
| `useSyncExternalStore` | Subscribes to a store outside React with a snapshot API that supports consistent rendering, including server rendering when configured. |
| `useTransition` | Marks state updates as non-urgent transitions and exposes whether the transition is pending. |
| `startTransition` | Marks state updates inside a callback as non-urgent. It can be called without a component Hook. |
| Transition | A non-urgent update that React may keep in the background so urgent interactions can stay responsive. |
| `useDeferredValue` | Lets a part of the UI use a deferred version of a value so urgent updates can render first. It does not debounce network requests. |
| `use` API | Reads a Promise or context during render. Reading a pending Promise suspends to the nearest Suspense boundary; unlike regular Hooks, `use` may be used in conditions and loops where documented. |
| Custom Hook | A reusable function that composes Hooks to share stateful logic. Each call has independent Hook state unless it connects to a shared store or context. |
| Dependency array | An array of reactive values supplied to certain Hooks. Values are compared with `Object.is`; it is not a manual scheduling switch. |
| Stale closure | A callback retaining values from an older render. Use correct dependencies, functional updates, or an appropriate event pattern rather than assuming captured values update. |
| Hook state position | React associates Hook state with call order in a component. Conditional Hook calls break that association. |

## Events, forms, and Actions

| Term or API | Meaning |
| --- | --- |
| Event handler | A function passed to an event prop and called in response to an interaction, such as a click or submit. |
| React event | The event object passed to a React event handler. It exposes event information through React's event system and browser event behavior. |
| Event propagation | The movement of an event through capture and bubble phases in the DOM tree. |
| Default action | A browser behavior associated with an event, such as submitting a form or following a link. `preventDefault()` cancels it when appropriate. |
| Controlled input | An input whose displayed value is set by React props, with an event handler that updates the controlling state. |
| Uncontrolled input | An input whose value is maintained by the browser DOM, optionally initialized with `defaultValue` or `defaultChecked`. |
| Form action | A function supplied to a form or submit control to process submission. React Actions can integrate with pending state and form reset behavior. |
| Action | In modern React, an async function used as a transition-oriented update or form operation. The same word can also mean a reducer event; context determines the meaning. |
| `useActionState` | Combines a form or Action with previous result state and a pending flag, returning an Action dispatcher. |
| `useFormStatus` | Reads the status of a parent form submission from a descendant component, including whether it is pending and submitted form data. |
| `useOptimistic` | Shows a temporary optimistic value while an Action is in progress, then reconciles with the final result. |
| Optimistic UI | UI that immediately reflects an expected result while the authoritative operation is still pending, with a path to reconcile or recover on failure. |
| Pending state | A state indicating that an Action, transition, or other asynchronous operation has not finished. |
| FormData | A browser object representing a form's submitted name/value pairs, including supported file fields. |
| Native form validation | Browser-provided validation using HTML constraints such as `required`, input types, and length or range attributes. |
| Accessible error summary | A prominent list of validation problems that helps users identify and navigate to fields that need attention. |

## Rendering, effects, and async UI

| Term or API | Meaning |
| --- | --- |
| Client rendering | Rendering a React tree in a browser after JavaScript runs. This can begin with an empty page or hydrate server-generated HTML. |
| Server rendering | Producing HTML for a React tree on a server, so content can be delivered before client JavaScript runs. |
| Static rendering | Producing HTML ahead of a request, commonly at build time. The hosting or framework layer controls when this is possible. |
| Hydration | Attaching React's client behavior to HTML generated on the server. Initial client output must match the server output. |
| Hydration mismatch | A difference between the initial server HTML and the first client render. It can cause warnings and incorrect or replaced content. |
| Streaming rendering | Sending server-rendered HTML in chunks as parts of the UI become available, often coordinated with Suspense. |
| Suspense | A boundary that displays a fallback while a descendant is not ready, for example because a supported resource is pending or a lazy component has not loaded. |
| Fallback | Temporary UI supplied to a Suspense boundary while its content is waiting. |
| `lazy` | Loads a component module on demand and works with Suspense to show a fallback while the module loads. |
| Error Boundary | A class component boundary that catches rendering errors in descendant components and displays recovery UI. It does not catch every error source, such as ordinary event handler errors. |
| Concurrent rendering | React's ability to prepare, pause, resume, or abandon some rendering work so urgent updates can be handled promptly. It does not mean application code runs in parallel on multiple JavaScript threads. |
| Interruptible render | Background render work that React may pause or restart when a more urgent update arrives. |
| Urgent update | An update that should be reflected promptly, such as typing into an input. |
| `createRoot` | Creates a client React DOM root in a browser container. |
| `hydrateRoot` | Creates a React DOM root attached to existing server-rendered HTML. |
| `createPortal` | Renders children into a different DOM node while retaining their place in the React tree for context and React event propagation. |
| `flushSync` | Forces React DOM updates to flush synchronously. It is an uncommon escape hatch and can hurt performance or interact poorly with Suspense. |
| Strict Mode | A development-only wrapper that helps reveal unsafe patterns by running additional checks. It does not change production behavior. |
| Effect lifecycle | An Effect's setup and cleanup cycles as dependencies change or a component leaves the tree. It is distinct from a component's render cycle. |
| External system | A system React does not own, such as a browser API, network connection, timer, or third-party widget; Effects can synchronize with it. |
| Resource | A value such as a Promise, stylesheet, image, or font whose availability can affect rendering. React APIs and frameworks may coordinate resources with Suspense. |
| `Activity` | **React 19.2.** A component for hiding or showing a subtree while preserving its state; its mode controls whether the subtree is visible or hidden. |
| `<ViewTransition>` | **React 19.3.** A DOM component that coordinates enter, exit, update, or shared-element animations with browser View Transitions for eligible React transitions. |
| `addTransitionType` | **React 19.3.** Adds a label to a transition so a ViewTransition can select animation behavior based on why the update occurred. |
| `browser()` | **React 19.3.** A browser-rendering context API that can suspend work until it is rendering in the browser. Use only for the documented client-only rendering scenarios. |

## DOM, accessibility, and security

| Term | Meaning |
| --- | --- |
| React DOM | The official renderer that lets React render into browser DOM and provides client and server rendering APIs. |
| DOM node | A browser object representing an element, text node, or another part of a document tree. |
| Ref | A mutable reference object or callback used to access a DOM node or expose an imperative value. Changing a ref does not itself render the component. |
| Callback ref | A function React calls when assigning or clearing a ref. Useful when assignment needs to trigger logic or state. |
| Imperative API | A small set of commands exposed for a task that cannot be expressed cleanly through props, such as focus or scroll. Prefer declarative props where possible. |
| Portal | A React subtree rendered into a DOM container outside its parent DOM location, commonly used for dialogs and overlays. |
| Semantic HTML | HTML elements chosen for their meaning and built-in behavior, such as `<button>`, `<nav>`, and `<main>`. |
| Accessibility (a11y) | Designing interfaces that people with different abilities and assistive technologies can use. |
| Accessible name | The name announced for a control by assistive technology, usually provided by visible text or a correctly associated label. |
| Label association | Connecting a form control to its label, for example with matching `htmlFor` and `id`, so the control has an accessible name. |
| ARIA | Attributes that communicate roles, states, and properties when native HTML alone cannot express the needed semantics. Native semantic elements are usually preferable. |
| Focus management | Deliberately moving or restoring keyboard focus, especially when dialogs open or close or validation fails. |
| Trusted Types | A browser security API for restricting values passed to dangerous DOM injection sinks. **React 19.3** passes supported Trusted Types values through to the browser without coercing them to strings. |
| Cross-site scripting (XSS) | A class of vulnerabilities where untrusted input is interpreted as executable content. Escape or sanitize untrusted content and avoid unsafe HTML injection. |
| `dangerouslySetInnerHTML` | An escape hatch for inserting HTML markup into a DOM node. Only use with trusted or correctly sanitized content. |
| Content Security Policy (CSP) | A browser-enforced policy that limits resource loading and script execution to reduce the impact of content injection attacks. |
| Input validation | Checking user-provided values for expected shape, length, and meaning. Client-side checks improve usability but do not replace server-side validation. |

## Server Components and framework architecture

| Term | Meaning |
| --- | --- |
| React Server Components (RSC) | A React architecture for rendering some components on the server and sending their output to a client application. Actual support and integration are provided by a compatible framework or bundler. |
| Server Component | A component in an RSC-enabled environment that runs on the server and can access server-side data. It cannot use client-only state or Effects. |
| Client Component | A component marked for client execution in an RSC framework, commonly with a `'use client'` directive at a module boundary. It can use browser interaction, state, and Effects. |
| Server/client boundary | The module and data boundary separating server-executed code from browser-executed code. Values crossing it must be supported by the framework's serialization rules. |
| `'use client'` | A directive understood by RSC-compatible tools that marks a module and its imported client subtree for client execution. It is not a directive needed in every ordinary client-rendered React app. |
| `'use server'` | A directive used by RSC-compatible tooling to identify a Server Function. It does not create a secure server endpoint by itself. |
| Server Function | An async function that a compatible framework can call across the client/server boundary. Validate authorization and input on the server. |
| Server Action | A Server Function used as an Action, often for form submission or mutation. The framework defines transport, security, and deployment behavior. |
| Framework | A toolset that combines React with routing, data loading, rendering, bundling, and deployment conventions. These capabilities are not all part of React core. |
| Router | A library or framework subsystem that maps URLs and navigation to application UI. React does not prescribe one built-in router. |
| Route | A URL pattern and the page or UI associated with it. |
| Server-side rendering (SSR) | Rendering HTML for a request on the server, commonly followed by hydration in the browser. |
| Static site generation (SSG) | Producing pages ahead of requests, often during a build. |
| Client-side rendering (CSR) | Rendering application UI in the browser with JavaScript. |
| Data loader | A framework or router mechanism for obtaining data associated with a route, often before or during navigation. |
| Serialization | Converting data to a supported representation for transport between server and client. Functions and many runtime objects cannot be sent as ordinary JSON. |
| Cache invalidation | Marking stored data as stale or refreshing it after a mutation so later reads do not rely on outdated results. |

## Tooling, testing, and performance

| Term | Meaning |
| --- | --- |
| React Compiler | A build-time compiler that can automatically optimize supported React code by memoizing components and values. It does not remove the need for pure render logic or correct data flow. |
| Compiler directive | A special string such as `"use memo"` or `"use no memo"` that a compatible compiler can interpret to control optimization in supported configurations. |
| Bundler | A build tool that processes modules and assets and produces browser-ready files. |
| Transpiler | A tool that transforms source syntax, such as JSX or newer JavaScript, into syntax understood by the target environment. |
| Development server | A local server that serves the app during development and may provide fast refresh or hot module replacement. |
| Fast Refresh | A development feature that updates edited modules while attempting to preserve component state. |
| Linter | A static analysis tool that reports likely errors and style issues. The Hooks lint rules help enforce correct Hook use and dependencies. |
| React Developer Tools | Browser tools for inspecting a React component tree, props, state, and profiling information. |
| Profiler | A React Developer Tools feature and React API for measuring rendering work and identifying expensive updates. |
| Memoization | Reusing a prior calculation or rendered result when inputs have not changed. It is a performance technique with memory and complexity tradeoffs. |
| `memo` | A component wrapper that can skip some re-renders when props compare equal. It is an optimization, not a guarantee that a component never renders. |
| React Testing Library | A third-party testing library that encourages tests based on accessible, user-visible behavior. It is not part of React core. |
| Unit test | A test of a small function or component behavior in isolation. |
| Integration test | A test that checks how multiple parts work together, such as a form, validation, and submission flow. |
| End-to-end test | A test that drives an application through a browser-like environment to verify complete user journeys. |
| Mock | A controlled replacement for a dependency used to isolate a test. Excessive mocking can hide integration problems. |
| Regression | A previously working behavior that breaks after a change. A regression test protects an important behavior from returning. |
| Performance budget | A project target for measures such as JavaScript size, loading time, or interaction responsiveness. |
| Code splitting | Dividing output JavaScript into smaller chunks that can be loaded when needed, often with dynamic imports and `lazy`. |
| Lazy loading | Delaying the loading of code or resources until they are needed. |
| Core Web Vitals | A set of browser experience metrics focused on loading, interactivity, and visual stability. They are web-platform metrics, not React APIs. |
| Memory leak | Resources retained after they should be released, often due to missing Effect cleanup or an unremoved external subscription. |
| TypeScript | A typed superset of JavaScript commonly used to describe props, state, events, and component contracts. React itself can be used with or without TypeScript. |

## Common distinctions

| These terms | How they differ |
| --- | --- |
| Render vs commit | Render calculates a candidate UI; commit applies an accepted result to the host environment. |
| State vs ref | State changes schedule rendering and affect UI; changing a ref's `.current` value does not schedule rendering. |
| Props vs state | Props come from a parent; state is owned and updated by a component or its state system. Both are inputs to render. |
| Effect vs event handler | An event handler responds to a particular interaction; an Effect keeps an external system synchronized with rendered state. |
| `useMemo` vs `useState` | Memo stores a recalculable performance cache; state stores application data across renders. |
| `useCallback` vs calling a function | `useCallback` caches function identity for a reason; it does not make the function execute less often by itself. |
| Key vs prop | A key helps React match siblings and is not delivered as a component prop. Pass an identifier separately when the component needs it. |
| React core vs framework | React provides UI rendering primitives; a framework adds conventions and systems such as routing, data loading, and deployment. |
| Server rendering vs Server Components | Server rendering emits HTML; Server Components define where components execute in an RSC architecture. They can be used together but are distinct ideas. |
| Suspense vs Error Boundary | Suspense handles supported pending work with a fallback; an Error Boundary handles render errors in its descendants. |
| Controlled vs uncontrolled input | A controlled input reads its current value from React props/state; an uncontrolled input keeps its current value in the DOM. |
| React Action vs reducer action | A React Action coordinates async or form work; a reducer action is an input value describing a state transition. |

## Abbreviations

| Abbreviation | Expansion |
| --- | --- |
| API | Application Programming Interface |
| ARIA | Accessible Rich Internet Applications |
| CSR | Client-Side Rendering |
| CSS | Cascading Style Sheets |
| CSP | Content Security Policy |
| DOM | Document Object Model |
| HTML | HyperText Markup Language |
| JSX | JavaScript Syntax Extension, commonly used as the name for React's markup-like JavaScript syntax |
| RSC | React Server Components |
| SSR | Server-Side Rendering |
| SSG | Static Site Generation |
| UI | User Interface |
| URL | Uniform Resource Locator |
| XSS | Cross-Site Scripting |
