# Suspense Boundaries and Fallbacks — Q&A

## Boundaries and fallbacks

### 1. What is a Suspense boundary?

It is a region of the React tree that can show a fallback while supported child content suspends during rendering.

### 2. What does the Suspense fallback prop represent?

It is the placeholder React displays when a child in that boundary is not ready to render, such as a loading message or skeleton.

### 3. What does Suspense receive besides fallback?

It wraps child content. The boundary can display that content when it is ready or its fallback while the content suspends.

### 4. Which boundary handles a suspension?

The closest Suspense boundary above the component that suspends handles it.

### 5. What happens if no Suspense boundary contains the suspended component?

There is no local fallback to show. Rendering may fail or be handled by an outer rendering mechanism; place a boundary around regions that are allowed to suspend.

### 6. Can the fallback itself suspend?

Yes. If the fallback suspends, the closest parent Suspense boundary may handle that suspension.

### 7. Can fallback be any React node?

Yes. In practice, keep it lightweight and ready to render, such as a skeleton or local loading message.

### 8. Does a boundary catch errors?

No. Suspense represents pending content. Use an Error Boundary for errors thrown during rendering or by a rejected resource.

### 9. Can a Suspense boundary wrap the whole app?

It can, but that is often too broad. One slow region could replace the entire app. Place boundaries around meaningful loading regions.

### 10. Does every pending component need its own boundary?

No. Group content that should appear together. Use nested boundaries when parts should reveal at different times.

## What activates Suspense

### 11. Does any pending network request activate Suspense?

No. The data source must integrate with Suspense. A normal Promise or request does not suspend React just because it is pending.

### 12. Does fetching in useEffect activate a Suspense boundary?

No. Suspense does not detect data fetched inside an Effect. Use the usual loading state pattern or a Suspense-enabled data source.

### 13. Does fetching in an event handler activate Suspense?

No. A request started in a click or submit handler does not activate a boundary by itself.

### 14. Can lazy-loaded component code activate Suspense?

Yes. A component declared with **lazy** suspends while its code is being loaded. The next lesson covers lazy loading in detail.

### 15. Can reading a Promise with React use activate Suspense?

Yes. A component that reads a pending Promise with React’s **use** API suspends. The Promise must be cached so the same instance is reused across renders.

### 16. What is a Suspense-enabled framework?

It is a framework or data source that integrates its data loading with React Suspense and maintains the resources needed for that behavior.

### 17. Does wrapping a regular fetch call in Suspense make it Suspense-enabled?

No. The boundary does not convert ordinary request code into a supported data source.

### 18. Do images automatically activate Suspense?

Not in the ordinary case. React documents selected resource-loading integrations, but ordinary image loading is not automatically a Suspense data source.

### 19. Do fonts automatically activate Suspense?

No, not by default for normal client rendering. React has selected integrations for coordinated resource and View Transition behavior.

### 20. Does Suspense support stylesheet loading?

React DOM supports selected stylesheet-loading behavior, including stylesheet links with the required React DOM properties. This is separate from ordinary data fetching.

## Grouping and nested boundaries

### 21. What does one boundary do when several children suspend?

It shows the same fallback for the boundary’s content until the suspended children needed for that reveal are ready.

### 22. When should content share one boundary?

When the pieces form one meaningful unit and showing only part of them would be confusing.

### 23. When should I use nested boundaries?

Use them when an outer section can be useful before a slower inner section is ready.

### 24. Can the outer content appear while inner content is pending?

Yes. If the outer content is ready and the pending inner content has its own boundary, React can reveal the outer section and show the inner fallback.

### 25. Does nesting boundaries guarantee every child appears at an exact time?

No. Boundaries define reveal groups and fallbacks, but network, scheduling, and rendering determine when each group becomes ready.

### 26. Can an outer fallback replace an inner fallback?

If the outer boundary’s own content suspends before it reveals, its fallback is shown. Once revealed, a nested boundary may handle later inner suspension separately.

### 27. Should every paragraph or card have a boundary?

Usually not. A boundary should correspond to a meaningful loading unit, not every small element.

### 28. How can I keep a page heading visible while a panel loads?

Place the heading outside the Suspense boundary that wraps the panel.

## Suspensions during updates

### 29. What happens if a tree suspends before its first mount?

React does not preserve state from that uncommitted initial render. When the content becomes ready, React retries rendering the tree from scratch.

### 30. What happens if already revealed content suspends again?

For an ordinary update, the boundary may show its fallback again and hide the revealed content inside it.

### 31. How can I avoid hiding already revealed content during a navigation?

Mark the navigation as a Transition using an appropriate router API or **useTransition**, so React can keep the current revealed content while preparing the new view.

### 32. Can useDeferredValue keep old content visible?

Yes. A slow consumer can keep using its deferred value while React prepares the new value. If the background render suspends, the previous revealed content can remain visible.

### 33. Does Suspense always keep previously revealed content visible?

No. This behavior depends on the update and boundary. An urgent update that suspends may show the fallback again.

### 34. Does a Transition remove nested Suspense fallbacks?

No. A nested boundary may still show its own fallback for its content. The Transition can avoid hiding already revealed content at the outer level.

### 35. What happens to layout Effects when visible content is hidden?

React cleans up layout Effects in the hidden content. When that content is revealed again, the layout Effects run again.

### 36. Do ordinary Effects run for a suspended render attempt?

Effects run only for a committed render. An attempt that is abandoned before commit does not run its Effects.

## Errors and accessibility

### 37. What should the fallback say?

Use concise text that describes which region is loading. Keep the fallback understandable even if users do not see the surrounding page.

### 38. Should a fallback use role=status?

It can, especially for a concise local loading message. Avoid unnecessary live announcements for decorative skeleton elements.

### 39. Does a Suspense fallback handle a rejected Promise?

No. A rejected Promise read during rendering is an error. Place an Error Boundary where the app can show an appropriate failure or recovery view.

### 40. Where should I place Error Boundary and Suspense?

A common arrangement places an Error Boundary outside a Suspense boundary: the inner boundary handles pending content, and the outer one handles failures.

### 41. Can an Error Boundary show a retry button?

Yes, if its implementation supports resetting its error state and the data source can retry. Suspense itself does not provide retry behavior for arbitrary data sources.

## Server rendering and other details

### 42. Does Suspense participate in server rendering?

Yes. React’s streaming server rendering and hydration integrate with Suspense boundaries.

### 43. Does a client-only app automatically stream HTML because it uses Suspense?

No. Streaming requires a server-rendering setup that supports React’s streaming APIs or a framework built on them.

### 44. Can Suspense reveal server-rendered regions as they arrive?

With streaming server rendering, React can send fallback HTML first and stream a boundary’s content later when it is ready.

### 45. Does a fallback have to be a spinner?

No. A skeleton or layout-shaped placeholder is often less disruptive. Match it to the region being replaced.

### 46. Should the fallback contain the same amount of content as the real region?

It does not have to, but a similarly shaped placeholder can reduce layout shifts and make the pending region clear.

### 47. Does lazy loading need a Suspense boundary?

Yes, if the UI should show a loading fallback while the lazy component’s code is loading.

### 48. What if I declare a lazy component inside another component?

It may be recreated during parent renders, which can reset child state. Declare lazy components at module scope.

### 49. What is the difference between pending and failed content?

Pending content is not ready and may use a Suspense fallback. Failed content has produced an error and should be handled by an Error Boundary or explicit error state.

### 50. What is the core mental model?

Suspense defines a reveal boundary for supported resources. Its position determines which content can be replaced while waiting; Error Boundaries handle failures instead.
