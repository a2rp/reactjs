# React Native and the Shared React Mental Model

## Learning goals

After this lesson, you should be able to:

- Explain which React ideas carry over to React Native.
- Describe how React Native renders a user interface without the browser DOM.
- Choose basic native components for text, layout, input, touch, and lists.
- Recognize the differences between React Native styles and browser CSS.
- Share suitable logic across web and mobile while isolating platform-specific behavior.
- Explain the role of an app framework, native modules, navigation, and device testing.

## 1. What React Native is

React Native lets you build mobile applications with React components and JavaScript or TypeScript. React still describes the interface as a tree of components. A React Native renderer turns that tree into user interface elements for the target platform and connects interactions to platform capabilities.

React Native is not a browser page inside an app. A normal React Native screen does not render HTML elements into a browser DOM. It uses React Native components such as View, Text, Image, TextInput, and Pressable.

A web renderer can make React Native components available to web targets, but that is an additional platform setup. Do not assume that a regular React web component can be dropped into a native screen unchanged.

## 2. What carries over from React

The central React mental model remains useful:

- Components describe small, reusable parts of an interface.
- Props pass inputs from a parent to a child.
- State stores values that change over time.
- Rendering derives the visible interface from current props and state.
- Event handlers respond to user actions.
- Hooks such as useState, useEffect, useRef, useMemo, and useContext follow React's rules.
- Context can provide values to a subtree without passing every value through every intermediate component.
- Keys help React match list items when a collection changes.
- Effects synchronize with external systems and should clean up subscriptions or other ongoing work.

For example, a counter can use the same state update pattern on web and mobile. The component and its rendering primitives differ, but a functional state update still works the same way:

    setCount(currentCount => currentCount + 1);

React's state, props, and rendering model transfer well. Browser APIs and HTML elements do not transfer automatically.

## 3. Native components instead of HTML elements

Use the components supplied by React Native to build a screen:

| React Native component | Typical use |
| --- | --- |
| View | Layout container and grouping |
| Text | Display and nest text |
| Image | Display local or remote images |
| TextInput | Accept text from the platform keyboard |
| Pressable | Respond to press interactions |
| ScrollView | Scroll a small, bounded collection of content |
| FlatList | Render a long or changing list efficiently |
| SectionList | Render a long list divided into sections |

A View is a general layout container. Text displays text; place text inside a Text component rather than directly inside a View. Pressable is a useful basis for a custom touch target because it exposes press state and accessibility props.

Components represent intent, not necessarily an identical pixel-level control on every platform. Native controls, fonts, keyboard behavior, status bars, gestures, and system conventions can differ between iOS and Android.

## 4. JSX looks familiar, but the host environment is different

A React Native component uses familiar JSX and React event props:

    <Pressable onPress={handleSave}>
      <Text>Save</Text>
    </Pressable>

There is no browser click event in this example. The interaction is named onPress. TextInput uses onChangeText to deliver the new string directly:

    <TextInput value={name} onChangeText={setName} />

Do not use document.querySelector, window, localStorage, or DOM methods in code that is expected to run on a native device. A native app has its own APIs for storage, navigation, files, networking, permissions, and other device features. Some JavaScript APIs are available in multiple runtimes, but availability must be checked for the target runtime.

The React Native host config also means that browser-only components such as div, span, form, and button are not the native UI primitives. Translate the intent into appropriate React Native components instead of renaming HTML tags mechanically.

## 5. Styles are JavaScript objects, not browser CSS

React Native styles are usually JavaScript objects, often grouped with StyleSheet.create:

    const styles = StyleSheet.create({
      screen: {
        flex: 1,
        padding: 16,
        backgroundColor: '#f5f7fa',
      },
      title: {
        fontSize: 24,
        fontWeight: '700',
      },
    });

Properties use camelCase. Numeric dimensions are generally density-independent units; values such as 16 are not written as 16px. Color strings are still useful, and layout commonly uses Flexbox.

Important differences from web CSS include:

- Styles do not use browser selectors or a global cascade.
- There is no ordinary stylesheet pseudo-class such as :hover; use interaction state from Pressable when needed.
- flexDirection defaults to column rather than row.
- flexShrink defaults to 0 rather than 1.
- alignContent defaults to flex-start.
- The flex shorthand is not the full browser CSS shorthand; React Native flex commonly takes a single number.

These defaults can make a layout that appears familiar behave differently. Give containers explicit layout styles and inspect the screen on each target platform.

## 6. Scrolling and lists

ScrollView renders all of its children, then lets the user scroll through them. This is appropriate for short content with a manageable number of elements.

FlatList renders items as they are needed around the visible window, which is usually a better choice for a long or changing collection. Provide a stable key for each item:

    <FlatList
      data={tasks}
      keyExtractor={task => task.id}
      renderItem={({ item }) => <TaskRow task={item} />}
    />

Avoid putting a large map of rows inside a ScrollView when FlatList is the right fit. Also avoid generating new item identifiers during render; identifiers should remain stable while the item remains the same.

## 7. Sharing code with React for the web

Sharing code is a design choice, not a guarantee that all components can be reused.

Good candidates to share include:

- Pure calculations and validation.
- API client logic and data transformations.
- TypeScript types and domain models.
- State transition functions that do not depend on a platform API.
- Components intentionally built against a cross-platform component library.

Usually platform-specific pieces include:

- Screen layout and navigation.
- Touch, keyboard, and focus behavior.
- File access, notifications, camera, location, and other device APIs.
- Browser-only behavior such as DOM access.
- Platform conventions and system integration.

Share the domain logic first. Keep web and native screens separate when their interactions and conventions differ. If a codebase intentionally targets web and native from one component tree, choose a framework and compatible component libraries for that goal.

## 8. Handling platform differences

React Native provides Platform for small platform-dependent decisions. A component can also be split into files such as Card.ios.tsx and Card.android.tsx when the implementations differ more substantially.

Use the least complicated boundary that keeps the behavior understandable:

- Keep one component when only a small style or value differs.
- Use a platform-specific helper when a device API differs.
- Use platform-specific files when layout or behavior is meaningfully different.
- Keep shared business rules in platform-neutral modules.

Do not scatter checks like Platform.OS throughout the application. A small number of clear platform boundaries is easier to maintain and test.

## 9. Project setup and the app framework

A React Native app needs more than React components: it needs a native build, a JavaScript bundler, development tooling, device integration, and often routing and native libraries.

The React Native team recommends using a framework for a new app in most cases. Expo is a production-grade React Native framework and includes tooling and a set of supported libraries. A framework is not mandatory; a project can use the React Native Community CLI when its requirements call for a lower-level setup.

A typical Expo starter command is:

    npx create-expo-app@latest my-mobile-app

Use the current project documentation when creating a real app because setup steps and supported platforms can change. Android and iOS also have different local build requirements. Developing or building for one platform does not prove that the other platform works.

Navigation is not just a component that React automatically provides. Choose a navigation solution that fits the selected app framework, then test back behavior, deep links, and screen transitions on devices.

## 10. Device APIs and native modules

React Native JavaScript can use React Native APIs and libraries that bridge to native platform capabilities. Examples include notifications, camera access, secure storage, location, and platform permissions.

Before adding a native library, check:

- Whether it supports the framework and React Native version in the project.
- Whether it supports the target platforms.
- Whether native configuration or a development build is required.
- How permissions are requested and denied.
- Whether the API behaves differently on simulators and physical devices.

A browser package that accesses the DOM is not automatically a valid native dependency. Prefer libraries designed for the actual runtime.

## 11. Accessibility and verification

Mobile accessibility includes screen readers such as VoiceOver and TalkBack, larger text, touch target size, contrast, and platform navigation behavior. Provide meaningful labels and roles for custom interactive controls. Native defaults help, but do not replace testing.

Verify the user experience on both iOS and Android when both are targets. Check:

- Text wrapping and font scaling.
- Keyboard appearance, dismissal, and input behavior.
- Safe display areas, status bars, and device rotation if supported.
- Back navigation and touch feedback.
- VoiceOver and TalkBack announcements.
- Empty, loading, error, and long-list states.
- Platform-specific permissions and denied-permission flows.

A browser preview or a single emulator is useful during development, but it does not cover every device behavior.

## 12. Common mistakes

- Treating React Native as HTML rendered inside a browser.
- Using div, button, or a DOM event where a native component and event are required.
- Assuming browser CSS, CSS selectors, or the cascade work unchanged.
- Placing raw text directly inside a View instead of inside Text.
- Using ScrollView for a very large collection that should use FlatList.
- Sharing a screen merely because the React syntax looks similar.
- Assuming an API or native package supports both Android and iOS.
- Testing only on one platform while claiming cross-platform behavior.
- Forgetting labels, roles, keyboard behavior, or screen-reader checks.
- Assuming a framework is mandatory or, at the other extreme, ignoring the native build and tooling that an app needs.

## Summary

React Native reuses React's component, props, state, hooks, and rendering mental model. It uses a different host environment: native components, touch-oriented events, JavaScript-based styles, and platform APIs replace browser HTML, CSS, and DOM assumptions. Reuse platform-neutral logic where it makes sense, isolate native differences, and verify the app on every platform you support.

## Practice checklist

1. Explain why a React Native View is not an HTML div.
2. Build a screen from View, Text, TextInput, Pressable, and FlatList.
3. Describe at least three differences between React Native Flexbox defaults and browser CSS.
4. Identify which parts of a small web app can be shared with its mobile app.
5. Name two behaviors that must be checked on both iOS and Android.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="095-frameworks-and-rendering-strategies.md">095 - Frameworks, Client Rendering, SSR, and Static Generation in React</a></td><td><a href="../qnas/096-react-native-and-shared-react-concepts.md">Q&amp;A</a></td><td><a href="../examples/096-react-native-and-shared-react-concepts.md">Example</a></td><td><a href="097-semantic-html-and-accessible-components.md">097 - Semantic HTML and Accessible React Components</a></td></tr>
  </tbody>
</table>
