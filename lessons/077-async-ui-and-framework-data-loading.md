# Async UI, Suspense, and Framework Data Loading

Asynchronous work takes longer than a render. A component may need data or code that is not ready yet, but the page still needs to explain what is happening. React's Suspense lets a subtree declare a fallback while a supported dependency is pending.

This lesson covers what can suspend, how Suspense boundaries shape the loading experience, how Promise reads relate to Error Boundaries, and what frameworks add to data loading.

## Learning goals

- Explain what it means for a component to suspend.
- Use Suspense fallbacks for supported asynchronous resources.
- Place boundaries around meaningful UI regions.
- Distinguish Suspense-enabled data from requests made in Effects and event handlers.
- Keep a Promise stable across render retries.
- Handle rejected resources and implement a real retry.
- Explain framework loaders, waterfalls, server rendering, and streaming.
- Choose explicit loading state when the resource does not support Suspense.

## 1. The asynchronous UI problem

A request can be idle, pending, successful, failed, or refreshing old data. You can model these states with component state or a data library. Suspense provides a rendering boundary for a different model: a component tries to read a supported resource, and React temporarily shows a fallback for the nearest boundary when that resource is pending.

Suspense does not start every asynchronous operation and it is not itself a query cache.

## 2. How a boundary works

A descendant suspends when it cannot finish rendering because a supported resource is pending. React searches upward for the nearest Suspense boundary and renders its fallback instead of that boundary's children.

    import { Suspense, use } from 'react';

    function Profile({ profilePromise }) {
      const profile = use(profilePromise);
      return <h2>{profile.name}</h2>;
    }

    function ProfileArea({ profilePromise }) {
      return (
        <Suspense fallback={<p role="status">Loading profile…</p>}>
          <Profile profilePromise={profilePromise} />
        </Suspense>
      );
    }

The Promise must come from a source that owns and reuses it, such as a framework loader, a Suspense-compatible library, or a deliberate resource cache. Creating it during every render is not safe.

## 3. What can suspend

React's documented cases include code loaded with lazy, a Promise read with the use API, and data integrations provided by a Suspense-enabled framework or library. React also coordinates some resource loading of its own.

These do not automatically activate Suspense:

- Calling fetch in useEffect.
- Waiting for a request inside a click handler.
- Creating an arbitrary Promise without reading it through a supported integration.
- Setting an isLoading boolean.

An Effect can still be a valid client-fetching strategy. The component or custom Hook then owns loading, success, and error state. It also needs cleanup or cancellation so an old response cannot overwrite newer data. Suspense will not observe that Effect's pending request.

## 4. Reading Promises with use

React's use API can read a Promise during render. While it is pending, the component suspends. After it resolves, React retries rendering and use returns the value. If it rejects, the error reaches the nearest Error Boundary.

Do not create the Promise inline:

    function Albums() {
      const albums = use(fetch('/api/albums').then(response => response.json()));
      return <AlbumList albums={albums} />;
    }

A suspended render can be retried. That code creates a new request each time, so React cannot reuse the pending Promise. Instead, use a framework loader or library, or get a stable Promise from a cache keyed by the resource:

    const requests = new Map();

    function getAlbums(url) {
      if (!requests.has(url)) {
        requests.set(url, fetch(url).then(response => {
          if (!response.ok) throw new Error('Could not load albums');
          return response.json();
        }));
      }
      return requests.get(url);
    }

    function Albums() {
      const albums = use(getAlbums('/api/albums'));
      return <AlbumList albums={albums} />;
    }

This illustrates Promise identity, not a complete production cache. Production caching also needs freshness, invalidation, retries, cancellation, user-data isolation, and memory limits.

The use API can be called conditionally or in loops, unlike most Hooks. It must still be called from a component or custom Hook. Do not put it inside try/catch: Suspense uses internal thrown signals. Use an Error Boundary for rejected resources.

## 5. Pair pending and error handling

Suspense handles pending work. Error Boundaries handle errors thrown while rendering descendants, including rejected Promises read with use.

    <ErrorBoundary fallback={<ProfileError />}>
      <Suspense fallback={<ProfileSkeleton />}>
        <Profile />
      </Suspense>
    </ErrorBoundary>

A try/catch around JSX does not catch a child's rendering error. A retry also needs to replace or clear the failed Promise. Merely hiding the error while reusing the same rejected Promise will fail again.

The data owner should define recovery: retry the resource, return to a safe route, preserve user input, or offer a reload if local recovery is not possible.

## 6. Choose boundary placement by user experience

A boundary decides which region is replaced. Put related content together when it should appear as one unit. Use nested boundaries when independent regions can become useful separately:

    <PageLayout>
      <PageHeading />
      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />
      </Suspense>
      <Suspense fallback={<ActivitySkeleton />}>
        <RecentActivity />
      </Suspense>
    </PageLayout>

The page heading stays visible while the chart and activity panel load independently. Avoid both extremes: one global fallback can hide useful content, while too many unrelated boundaries can make the page flicker as a collection of placeholders.

A good fallback describes what is pending, keeps layout stable when practical, and uses accessible status semantics. A spinner by itself may not communicate progress to assistive technology.

## 7. Updates and already visible content

On an initial load there may be no old content to preserve. During an update, replacing a useful screen with a large fallback can be disruptive. React transitions can mark an update as non-urgent; when an already revealed region suspends because of that update, React can keep the old content visible while preparing the next version.

The UI still needs to communicate freshness when it matters. A transition changes presentation, not data correctness. A query library may have its own stale-data and fetching indicators; use the behavior it documents.

## 8. What frameworks add

React supplies rendering primitives. Frameworks can connect data loading to routes, server rendering, navigation, and caching. Depending on the framework, its data system may provide:

- route loaders or server-side data access;
- request deduplication, caching, and preloading;
- parallel data work instead of parent-child waterfalls;
- server-rendered initial content;
- streaming around Suspense boundaries;
- coordinated navigation and error handling.

The exact APIs and cache lifetime differ among frameworks and versions. Follow the chosen framework's data-loading conventions. A framework route loader can start independent requests together before rendering. A Server Component may await data on the server or pass a Promise deeper into the tree, depending on that framework's model. A Client Component can read an appropriate Promise with use, but use does not make an unsupported fetch source Suspense-compatible.

A waterfall occurs when request B waits for request A even though both could start at once. Deep component fetching can cause this accidentally. Route-level loading can start independent work in parallel, while Suspense boundaries determine when regions can appear.

With streaming server rendering, a framework can send a ready page shell and fallback for a slower region, then stream that region later. A client-only Vite app does not acquire server streaming just by rendering Suspense; the server and framework must support it.

## 9. When to use explicit state

Use Suspense when your framework or data library explicitly supports it and boundary-based loading fits the interface.

Use explicit status, data, and error state when the request is started by an interaction, the source is not Suspense-enabled, a form needs a submission state machine, or users should keep interacting with old content during refresh. Do not let two independent systems issue and cache the same request.

## 10. Common mistakes

- Expecting an Effect fetch or click-handler request to activate Suspense.
- Creating a new Promise on each render.
- Treating Suspense as the data cache.
- Replacing the entire application for one slow panel.
- Adding boundaries without considering how regions should appear together.
- Catching use's suspension with try/catch instead of using an Error Boundary.
- Retrying the same rejected Promise.
- Assuming client rendering and server streaming are the same thing.

## Practical checklist

1. Which system owns this request and cache?
2. Does that system support Suspense?
3. Is the Promise stable across render retries?
4. What is the smallest useful boundary?
5. What happens on rejection?
6. Does retry create a new attempt?
7. Can independent requests start in parallel?
8. Should old content stay visible during an update?
9. Does the fallback communicate progress accessibly?
10. Am I following the framework's own data-loading model?

## Summary

Suspense shows a fallback when a supported dependency is pending. Reading a stable Promise with use connects it to Suspense; rejection belongs to an Error Boundary. Requests started by Effects or event handlers do not suspend automatically. Frameworks and data libraries supply the integrations, caching, and server behavior needed for production data loading.

## Key terms

- **Suspension:** A render cannot finish because a supported dependency is pending.
- **Boundary:** The Suspense component that selects a fallback for a subtree.
- **Promise identity:** Reusing the same Promise across render retries for one resource.
- **Error Boundary:** A component that shows recovery UI for errors in descendant rendering.
- **Waterfall:** Serial requests that could have started in parallel.
- **Streaming:** Delivering server-rendered content in parts as regions become ready.
- **Transition:** An update React may prepare without immediately replacing already revealed content.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="076-caching-and-revalidation.md">076 - Caching, Deduplication, Invalidation, and Revalidation</a></td><td><a href="../qnas/077-async-ui-and-framework-data-loading.md">Q&amp;A</a></td><td><a href="../examples/077-async-ui-and-framework-data-loading.md">Example</a></td><td><a href="078-react-form-actions.md">078 - React Form Actions and Asynchronous Submissions</a></td></tr>
  </tbody>
</table>
