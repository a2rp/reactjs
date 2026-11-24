# Suspense Boundaries and Fallbacks

A **Suspense** boundary defines which part of the interface React may replace with a fallback while supported content is not ready. Good boundaries keep the loading state near the content that needs it and preserve useful surrounding UI.

## Learning goals

After this lesson, you should be able to:

- wrap a region with **Suspense** and provide an appropriate fallback;
- explain which React resources can activate a boundary and which cannot;
- choose between one boundary for a group and nested boundaries for progressive reveal;
- keep unrelated page content visible while a child is suspended;
- understand how already revealed content behaves when an update suspends again;
- recognize that state from a tree that suspends before its first mount is not preserved;
- separate loading fallbacks from error handling.

## Basic boundary

**Suspense** takes two props:

- **children**: the content React should try to render;
- **fallback**: the placeholder shown by this boundary if its children suspend.

~~~jsx
import { Suspense } from "react";

function ProfilePage({ user }) {
  return (
    <>
      <h1>{user.name}</h1>
      <p>Profile details stay visible.</p>

      <Suspense fallback={<p role="status">Loading activity…</p>}>
        <ActivityFeed userId={user.id} />
      </Suspense>
    </>
  );
}
~~~

While **ActivityFeed** is suspended, React can show the fallback in its place. The heading and profile description are outside that boundary, so this boundary does not replace them.

The closest Suspense boundary above the content that suspends handles that suspension. If there is no matching boundary, rendering may fail rather than showing an in-place loading placeholder. A fallback should be lightweight and ready to render; if the fallback itself suspends, the next parent boundary may handle it.

## Suspense only works with supported sources

Suspense activates when rendering encounters a source that integrates with it. Common cases include:

- a component declared with **lazy** whose code has not loaded yet;
- reading a cached Promise with React’s **use** API;
- a Suspense-enabled framework or data source;
- selected React DOM resources such as supported stylesheet loading.

Suspense does not detect data fetched inside **useEffect**, a click handler, or an ordinary Promise callback. Rendering a normal loading boolean after starting a request in an Effect is a different pattern; wrapping that component in **Suspense** does not automatically connect the request to the boundary.

When reading a Promise with **use**, reuse the same cached Promise across renders. Do not create a fresh Promise during each render, or React may repeatedly suspend on a new resource.

This lesson uses **use** only to demonstrate a supported Promise resource. The next lesson covers **lazy** in more detail.

## One boundary reveals a group together

If several children are under the same boundary, the boundary waits for the suspended content in that group before revealing the group:

~~~jsx
<Suspense fallback={<CourseSkeleton />}>
  <CourseSummary summaryPromise={summaryPromise} />
  <CourseLessons lessonsPromise={lessonsPromise} />
</Suspense>
~~~

This is useful when the parts need to appear together or when showing only half the content would be confusing. The fallback should represent the whole region that disappears while the boundary waits.

Do not wrap the entire application in one boundary by default. A broad boundary can replace too much of the page for one delayed child. Place boundaries around meaningful loading regions.

## Nested boundaries reveal content progressively

Nested boundaries let ready outer content appear while a slower inner section still shows its own fallback:

~~~jsx
<Suspense fallback={<PageSkeleton />}>
  <CoursePage>
    <CourseSummary summaryPromise={summaryPromise} />
    <Suspense fallback={<LessonsSkeleton />}>
      <CourseLessons lessonsPromise={lessonsPromise} />
    </Suspense>
  </CoursePage>
</Suspense>
~~~

The outer boundary waits for the page and summary. Once those can render, the nested boundary can show **LessonsSkeleton** while lessons remain unavailable. Users can read the summary before the lesson list is ready.

Choose reveal groups based on the user’s task. If two pieces are meaningful only together, keep them within one boundary. If an early section is useful on its own, give the slower section a nested boundary.

## What happens when content suspends again?

If a boundary has not revealed its children yet, it shows its fallback while waiting for the first successful render.

After content has been revealed, a later update can cause it to suspend again. By default, the boundary may show its fallback again, replacing the revealed content inside it. A Transition or **useDeferredValue** update can let React keep already revealed content visible while preparing the new result.

The boundary’s placement determines how much content can be replaced. Content outside the boundary remains outside that fallback. Use Transitions when the update is a non-urgent navigation or view change; use a deferred value when a slower consumer should catch up to a changing value.

## State and Effects around a suspension

If a component suspends before it mounts for the first time, React does not preserve state from that uncommitted attempt. Once the resource becomes available, React retries the tree from scratch. Keep important state above a boundary if it must survive that initial suspended render.

If already visible content suspends again and React hides it, React cleans up its layout Effects while that content is hidden. When the content is revealed again, the layout Effects run again. Do not assume a hidden tree can keep measuring visible DOM layout.

## Fallback and error handling are different

A Suspense fallback represents content that is pending. It does not display a rejected Promise error.

Use an Error Boundary to handle errors thrown while rendering, including a rejected Promise read through **use** or a failed lazy import. The fallback should give the user an understandable loading state; the Error Boundary should give the user a recovery or failure state.

A common structure places the Error Boundary outside Suspense:

~~~jsx
<ErrorBoundary fallback={<p>Could not load this section.</p>}>
  <Suspense fallback={<p role="status">Loading this section…</p>}>
    <AccountSection />
  </Suspense>
</ErrorBoundary>
~~~

The outer boundary handles a failure. The inner Suspense boundary handles pending content.

## Server rendering and boundaries

Suspense also participates in React’s streaming server rendering and hydration. A server-rendered boundary can let React send ready parts of a page while another region is still pending, then stream that region when it is ready.

This is framework- and server-rendering behavior. A client-only app using **createRoot** still benefits from Suspense for supported client resources, but it is not automatically streaming HTML.

## Common mistakes

- Assuming any fetch or Promise activates Suspense.
- Fetching data in an Effect and expecting a nearby Suspense boundary to show its fallback.
- Creating a new Promise during every render instead of reusing a cached resource.
- Putting one boundary around an entire page when only one small region can be pending.
- Nesting boundaries without deciding whether content should appear together or progressively.
- Making a fallback that is as visually disruptive as the loading state it replaces.
- Expecting state from a tree that suspends before its first mount to be preserved.
- Expecting a fallback to handle a rejected Promise or rendering error.
- Expecting already revealed content to remain visible for every urgent update that suspends.
- Forgetting that nested boundaries can show their own fallbacks even when an outer boundary has already revealed.

## Summary

- A Suspense boundary shows its fallback when a supported child suspends while rendering.
- The closest boundary handles the suspended region; boundary placement controls what gets replaced.
- One boundary groups content for a coordinated reveal; nested boundaries allow progressive reveal.
- Effects and event-handler requests do not activate Suspense by themselves.
- Cached Promise reads through **use**, lazy-loaded components, and Suspense-enabled data sources can activate it.
- A Transition or deferred update can keep already revealed content visible while React prepares a new suspended result.
- Use an Error Boundary for failures; use Suspense fallbacks for pending content.
