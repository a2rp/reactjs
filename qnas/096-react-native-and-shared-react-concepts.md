# React Native and the Shared React Mental Model: Questions and Answers

## Core concepts

### 1. What is React Native?

React Native is a framework for building native applications with React components and JavaScript or TypeScript. React describes the component tree; a React Native renderer turns it into UI for the target platform and connects interactions to platform capabilities.

### 2. Is React Native a web page inside a mobile browser?

No. A standard React Native screen does not render HTML into a browser DOM. It uses React Native components and platform rendering. A web target requires an additional web renderer or framework setup.

### 3. Which React ideas are shared with React Native?

Components, props, state, rendering from current data, event handlers, keys, context, and the Rules of Hooks all carry over. The host components and platform APIs are different.

### 4. Can I use useState and useEffect in React Native?

Yes. These are React Hooks and follow the same rules. Use effects to synchronize with external systems and clean up subscriptions, timers, or listeners when necessary.

### 5. Does setState work differently on mobile?

No. React state update semantics are the same. Functional updates such as setCount(current => current + 1) are useful when the next value depends on the previous value.

### 6. Can a React web component be imported directly into a native screen?

Only if it is built for a compatible cross-platform renderer. A component that depends on HTML elements, DOM events, browser CSS, or browser APIs needs a native alternative or an explicit cross-platform implementation.

### 7. Does React Native use the browser DOM?

No. DOM APIs such as document.querySelector are not the standard way to find or update React Native UI. React should update the interface from state and props.

### 8. Can I use window or localStorage?

Do not assume those browser globals exist in a native runtime. Use a library or platform API designed for the app's storage, navigation, files, and device capabilities. Verify runtime support for every API you use.

## Native components and interactions

### 9. What is View used for?

View is the general-purpose container for grouping and laying out other components. It also supports styles, touch handling, and accessibility properties.

### 10. Why must displayed text usually be inside Text?

Text is React Native's text-rendering component. Put text content inside Text rather than writing a bare string directly under View.

### 11. What replaces HTML input elements?

TextInput is used for editable text. It receives a value and an onChangeText handler, along with props that control placeholder text, keyboard behavior, and accessibility.

### 12. What replaces an HTML button?

Pressable is commonly used for custom touchable controls. It exposes press callbacks and pressed state, and it supports accessibility properties. Other components may be appropriate for a standard platform control.

### 13. Is onPress the same as onClick?

They serve a similar interaction purpose but belong to different host environments. React Native components commonly use onPress; do not assume browser click events exist on native components.

### 14. How does TextInput deliver new text?

Its onChangeText callback receives the new string directly. For example, onChangeText={setName} can update a string state value.

### 15. What component displays an image?

Image displays images in React Native. Its source format and loading behavior are not identical to every browser img use case, so check the source requirements for bundled and remote images.

### 16. When should I use ScrollView?

Use ScrollView for a reasonably small amount of content that should scroll as a whole. It renders all of its children, so it is not the best choice for an extremely long list.

### 17. When should I use FlatList?

Use FlatList for a larger or changing collection. It renders items around the visible area instead of mounting every row at once. Supply stable item keys.

### 18. Why do list items need stable keys?

Keys help React match items across inserts, removals, and reordering. A stable identifier from the data is better than an array index when the collection can change.

## Styles and layout

### 19. Are React Native styles browser CSS?

No. They are JavaScript style objects with a supported set of properties. They do not use browser selectors or the usual global CSS cascade.

### 20. Why are style property names camelCase?

Styles are JavaScript objects, so properties use JavaScript naming such as backgroundColor, marginTop, and flexDirection.

### 21. Do numeric dimensions include pixels?

Numeric layout dimensions are generally density-independent units. Write padding: 16 rather than padding: '16px'.

### 22. Is Flexbox the same as browser Flexbox?

It is similar, but React Native defaults and supported shorthand differ. In particular, flexDirection defaults to column, flexShrink defaults to 0, alignContent defaults to flex-start, and flex commonly takes a single number.

### 23. Why does a row layout stack vertically?

React Native's default flexDirection is column. Set flexDirection: 'row' when children should be arranged horizontally.

### 24. Does React Native support CSS selectors and a global cascade?

Not in the same way as a browser. Apply style objects to components and share style values or functions deliberately rather than relying on selectors to target elements elsewhere in the tree.

### 25. How do I style a pressed control?

Pressable can receive a style callback with pressed state. Use that state to select an appropriate style, while also providing an accessible name and a sufficiently large touch target.

### 26. Can I use CSS pseudo-classes such as :hover?

Do not assume browser pseudo-classes exist on native components. Use interaction props and state such as Pressable's pressed state for native touch feedback.

## Sharing and platform differences

### 27. What code is usually easiest to share between web and native?

Pure calculations, validation, data transformation, API logic that uses supported APIs, TypeScript types, and platform-neutral domain rules are often good sharing candidates.

### 28. What code is commonly platform-specific?

Screen layout, navigation, touch and focus behavior, browser DOM access, native device APIs, and conventions tied to iOS or Android commonly need separate implementations.

### 29. Should an entire app share every screen component?

Not necessarily. Share components when their rendering and interaction model is genuinely compatible. Separate screens can be clearer when web and mobile conventions differ substantially.

### 30. What does Platform provide?

Platform exposes information about the current operating system and helpers such as Platform.select. It is useful for a small number of platform-dependent choices.

### 31. What are platform-specific file extensions?

Files such as Button.ios.tsx and Button.android.tsx allow the bundler to select an implementation for a platform while the importing code uses a shared module path.

### 32. When should I split a component by platform?

Use a small Platform condition for a small difference. Use a platform-specific helper or file when behavior or layout differs enough that one component becomes difficult to understand.

### 33. Can I share business logic while keeping separate screens?

Yes. Put data models, validation, and platform-neutral rules in shared modules. Let each screen adapt that logic to the platform's navigation and interaction patterns.

## Frameworks, APIs, and project setup

### 34. Is a React Native framework mandatory?

No. A project can use React Native without a framework, but the React Native team recommends starting with a framework for most new apps because it supplies common tooling and app capabilities.

### 35. What is Expo?

Expo is a React Native framework and toolset. It provides a supported way to build apps and access common app features. A project can still use native code when its needs require it.

### 36. What is Metro?

Metro is the JavaScript bundler commonly used by React Native projects. It transforms and serves the JavaScript bundle used by the app during development.

### 37. Is navigation automatically included in React?

No. React's component and state APIs do not define an app's navigation system. Choose a navigation solution that supports the selected framework and implement platform behaviors such as back navigation and deep links.

### 38. How do native modules expose device features?

A native module or library connects JavaScript code to platform functionality such as a camera, notifications, location, or secure storage. It may require native configuration, permissions, or a development build.

### 39. What should I check before adding a native library?

Check its support for the project's React Native version and framework, target platforms, native setup requirements, permission behavior, and whether it works in the intended development or release build.

### 40. Does a browser package work in React Native?

Not automatically. A package that depends on the DOM, browser-only APIs, or web styling may not work in a native runtime. Check the package's platform support before depending on it.

## Accessibility and testing

### 41. What accessibility features should custom controls provide?

Provide a meaningful label, role, and state where appropriate. Check that the control is discoverable and understandable with VoiceOver and TalkBack, and make the touch target and contrast usable.

### 42. Are accessibility behaviors identical on iOS and Android?

No. Both platforms support assistive technologies, but their APIs and screen-reader behavior differ. Test with the actual platform screen readers rather than assuming one result covers both.

### 43. Is one emulator enough to verify a cross-platform app?

No. An emulator is valuable, but test each target platform. Physical devices can reveal differences in keyboard behavior, permissions, performance, fonts, and system integration.

### 44. What UI states should a mobile screen test?

Test initial, loading, empty, success, error, long-content, keyboard-open, and permission-denied states when relevant. Include rotation or larger text if the app supports them.

### 45. Does a web preview prove that the native app works?

No. A web preview can help inspect shared code, but it does not prove native builds, platform APIs, permissions, navigation, or accessibility behavior work on iOS and Android.

### 46. What is the most important mental model to remember?

React's component, props, state, and rendering model is shared. The host environment is not: React Native uses native components and platform APIs instead of browser HTML, CSS, and the DOM.

## Quick review

- Share React concepts and platform-neutral logic.
- Use View, Text, TextInput, Pressable, and list components for native UI.
- Treat styles as JavaScript objects, not browser CSS.
- Prefer FlatList for long collections.
- Isolate differences at clear platform boundaries.
- Use a suitable framework for app setup and verify every supported platform.
