# Large Lists and Virtualization - Q&A

## Concepts and choosing a strategy

### 1. What is list virtualization?

Virtualization renders only the rows near the scroll viewport and positions them within a scrollable space that represents the full list. As the user scrolls, the rendered range changes.

### 2. Is virtualization built into React?

React provides list rendering and reconciliation, but virtualization is usually provided by a library or a custom implementation. Choose a library that supports the row sizes, scrolling, accessibility, and framework features your app needs.

### 3. Does virtualization make a list load faster?

It can reduce the amount of DOM created and updated, which may improve rendering and layout performance. It does not automatically make data fetching, sorting, filtering, or calculations faster.

### 4. How is virtualization different from pagination?

Pagination presents a limited page of results and navigation between pages. Virtualization can present one continuous scroll surface while only mounting a visible subset. They can be combined.

### 5. How is virtualization different from infinite loading?

Infinite loading fetches additional data as the user approaches the end. Virtualization decides which already-loaded rows remain mounted. A feed can use both.

### 6. When should I use virtualization?

Use it when profiling shows that a large number of mounted rows causes a user-visible rendering or layout problem, and the interface can support the constraints virtualization introduces.

### 7. When should I use pagination instead?

Pagination may be better when users need page numbers, direct page navigation, a compact result set, full-page semantics, or reliable print and browser find behavior.

### 8. Should every list with hundreds of items be virtualized?

No. Item count alone does not determine whether virtualization helps. Row complexity, device performance, interactions, data shape, and accessibility needs all matter. Measure first.

### 9. Can virtualization and pagination be combined?

Yes. A page or server-loaded segment can itself be virtualized if its mounted DOM is still too large. Keep page boundaries, result counts, and scroll behavior understandable.

## Rendering and keys

### 10. Why do virtualized rows need stable keys?

A key helps React associate a rendered row with the same data item when items are filtered, inserted, deleted, or reordered. Use a stable ID from the data, not an ID generated during render.

### 11. Is the row index a safe key?

Only when the item order never changes and index identity truly matches the item's lifetime. For filterable or reorderable data, indexes can associate a row's component state or DOM with the wrong item.

### 12. Does a virtualizer use keys the same way as React map rendering?

It must identify rendered items consistently. Libraries may accept a custom row-key function or use an index by default. Follow the library's current API and pass stable data identity when rows can change order.

### 13. Does virtualization reduce JavaScript data memory?

Usually not. If the whole collection is already loaded, it remains in memory. Use server queries, paging, or incremental loading when data size or transfer is also the problem.

### 14. Does virtualization make filtering and sorting cheap?

No. Filtering and sorting can still inspect or transform the full dataset. Profile those operations separately and consider server-side work, indexing, or reducing the dataset.

### 15. Does a virtualized list render all items for server rendering?

Typically it renders only an initial range because the server does not know the client's scroll position or measured viewport. Check the library's server-rendering behavior and choose an initial viewport size when supported.

### 16. Why can a virtualized list jump after resizing?

The viewport size or row measurements changed. A fixed-height design can avoid many recalculations; a variable-height list must update its measurement data and offsets when content or container dimensions change.

## Row sizing and overscan

### 17. What is a fixed-size row?

Every item uses the same known height. A virtualizer can calculate row offsets directly, which is usually efficient and predictable.

### 18. What is a variable-size row?

Rows can have different heights, so a virtualizer estimates or measures their size and updates its position model as needed. This is useful for content that wraps or expands, but it can be more complex and costly.

### 19. Should I use dynamic row heights by default?

No. Prefer a fixed or predictable size when it meets the design. Use dynamic measurement only when variable heights are necessary and test how resizing, fonts, images, and expanded content affect measurements.

### 20. What does overscan mean?

Overscan is the number of extra rows rendered just outside the visible range. It can make fast scrolling and focus movement feel smoother, but increases the amount of mounted work.

### 21. How much overscan should I use?

Start with a modest value and test realistic scrolling. Increase it if users see blank gaps or focus navigation feels abrupt; decrease it if the extra rows erode performance.

### 22. What if row content changes height after it loads?

The virtualizer needs to measure the updated row and adjust its size model. This can happen when images load, fonts swap, or expandable content opens. Reserve image dimensions when possible and use a library that supports measurement if rows truly vary.

### 23. Why is my row clipped or overlapping?

Check that the virtualizer's configured row size matches the rendered CSS box, including padding and borders. Ensure that the row receives the style and position values supplied by the virtualizer.

## Accessibility and interaction

### 24. Is a virtualized list accessible automatically?

No. A library can provide useful row attributes, but the application still needs correct semantics, labels, keyboard behavior, focus management, and screen-reader testing.

### 25. Why can browser find-in-page miss list items?

Items outside the virtualized range are not mounted in the DOM, so browser find and screen-reader virtual navigation generally cannot inspect them. Pagination or another non-virtualized view can be more suitable when users must browse all content at once.

### 26. What happens to keyboard focus when a row unmounts?

Focus can be lost if the focused item is removed from the DOM while scrolling or changing the dataset. Preserve focus deliberately, keep the focused item rendered when appropriate, or move focus to a meaningful mounted control.

### 27. Should a virtualized list use list, grid, or table semantics?

Use semantics that match the content and interaction. A sequence of links or options may be a list; tabular data should retain meaningful table structure where possible. Do not choose ARIA roles only to match a library example.

### 28. What do aria-posinset and aria-setsize communicate?

They can tell assistive technology an item's position and the total number of items when only part of the list is mounted. Use the attributes provided by the virtualizer where appropriate, and confirm the resulting semantics with assistive technology.

### 29. Is overscan enough for keyboard navigation?

Not by itself. Ensure the next focus target is mounted or can be scrolled into view before focus moves to it. Test moving through the first, middle, and final items with the keyboard.

### 30. How should filtering affect focus and scroll position?

Decide explicitly. A new query may need to reset to the first result, keep focus in the search field, and announce the updated result count. Avoid leaving focus tied to a row that no longer exists in the filtered data.

## Responsive layouts and loading

### 31. Should I read window.innerWidth during render to choose row layout?

Usually not. Use CSS media queries for visual layout. A one-time JavaScript width read can become stale when the window, split pane, sidebar, or parent container resizes, and can complicate server rendering.

### 32. When does JavaScript need to know the container size?

The virtualizer needs the viewport dimensions and may need to remeasure rows when width affects their height. Prefer a library that observes the scrolling element or uses ResizeObserver rather than a one-time viewport guess.

### 33. How can I make rows responsive without breaking fixed-height virtualization?

Use a compact layout that still fits within the configured row height, such as hiding a secondary column at narrow widths. If text must wrap to arbitrary heights, use a variable-size strategy or redesign the row.

### 34. Does loading more data require virtualization?

No. Infinite loading and virtualization solve different problems. If an infinite feed retains every loaded row in the DOM, virtualization may help; if only a small page is mounted, it may not be needed.

### 35. Can loading=lazy on images replace virtualization?

No. Lazy image loading can defer image downloads, but the rows and their non-image DOM still exist. Use it for image resources; use virtualization when the mounted row count is the bottleneck.

### 36. What about empty, loading, and error states?

Render these states outside the virtualizer when there are no rows, and keep loading or retry controls reachable. For incremental loading, make the end-of-list trigger and loading indicator compatible with the virtualized range and keyboard navigation.

## Diagnostics

### 37. How can I tell whether virtualization helped?

Profile the same scroll and update interactions before and after with realistic data. Compare mounted row counts, scripting time, layout work, and visible smoothness on representative devices.

### 38. What should I check if scrolling looks blank or flickers?

Check the viewport measurement, row-size estimates, style application, overscan, and whether data is available for the requested range. Variable-height rows may need accurate measurement or a better initial estimate.

### 39. Should I virtualize a plain table?

Only if the number of rows causes a measured problem and the table's semantics and interaction can be preserved. A custom layout can accidentally lose column headers, row relationships, keyboard behavior, and screen-reader context.

### 40. What is the safest rollout checklist?

1. Profile and identify the actual bottleneck.
2. Choose pagination, infinite loading, virtualization, or a combination based on product needs.
3. Use stable item IDs and suitable semantics.
4. Decide fixed versus measured row sizes.
5. Tune overscan with real scrolling.
6. Check focus, keyboard navigation, and screen-reader output.
7. Resize the container and test loading, filtering, and empty states.
8. Profile again and keep the added complexity only if the experience improves.

## Key takeaway

Virtualization limits mounted rows; it does not solve every large-data problem. Choose it from measured evidence, and design row identity, sizing, accessibility, focus, responsive behavior, and loading together.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="109-memo-and-optimization-decisions.md">109 - Memoization and Optimization Decisions - Q&amp;A</a></td><td><a href="../lessons/110-large-lists-and-virtualization.md">Lesson</a></td><td><a href="../examples/110-large-lists-and-virtualization.md">Example</a></td><td><a href="111-code-splitting-and-asset-loading.md">111 - Code Splitting, Asset Loading, and Bundle Analysis - Q&amp;A</a></td></tr>
  </tbody>
</table>
