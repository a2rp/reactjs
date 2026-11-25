# Large Lists, Virtualization, and Responsive Rendering

## Learning objectives

After this lesson, you should be able to:

- Recognize whether a slow list is limited by data work, React rendering, or browser layout.
- Distinguish pagination, infinite loading, and virtualization.
- Explain how a virtualized list keeps the scroll experience while mounting only a small visible range.
- Choose fixed or measured row sizes and understand the trade-offs.
- Preserve stable item identity, accessible semantics, and keyboard behavior.
- Adapt rows to container width without making rendering depend on a guessed viewport.
- Choose an appropriate list strategy based on profiling and product requirements.

## 1. Why large lists can become slow

Rendering a collection with map is a clear and useful pattern:

    <ul>
      {products.map((product) => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>

For a short list, this is usually all that is needed. As the list grows, the cost can come from several places:

- Creating a large number of React elements.
- Mounting and maintaining many DOM nodes.
- Calculating, styling, and laying out many rows.
- Filtering or sorting a large dataset on the main thread.
- Loading images, fonts, or other row content.
- Re-rendering rows during unrelated updates.

Do not assume that React rendering is the bottleneck. Profile the interaction and identify what work is expensive.

## 2. Three different strategies

These approaches solve related but different problems:

| Strategy | Data loaded | DOM rows mounted | Useful when |
| --- | --- | --- | --- |
| Pagination | A page or slice at a time | Usually only the current page | Users need clear navigation, page links, or a manageable result set |
| Infinite loading | More data as the user approaches the end | Usually all loaded rows remain mounted | Users browse a feed and more data arrives incrementally |
| Virtualization | It may have all or paged data | Only the visible range plus a small overscan range | The DOM cost of a very long list is the bottleneck |

Pagination limits the amount of data shown at a time. Infinite loading controls when more data is fetched. Virtualization controls how many rows are mounted. An application can combine them, such as virtualizing the currently loaded segment of a server-backed feed.

## 3. What virtualization does

A virtualizer measures or estimates the scroll viewport and calculates which rows are visible. It renders that range, often with a few extra rows before and after it, while reserving the total scrollable space for all items.

When the user scrolls, the rendered range moves. Rows outside that range are unmounted or recycled, and new visible rows are rendered in their place.

This reduces mounted DOM work. It does not automatically reduce:

- The size of an already-loaded dataset in memory.
- The cost of filtering or sorting every item.
- Network transfer or server processing.
- The work performed by each visible row.

For very large remote datasets, combine virtualization with server-side pagination, filtering, or incremental loading as appropriate.

## 4. Choose virtualization only when it helps

Consider virtualization when profiling shows that mounting or laying out many rows makes a real interaction slow. It can be unnecessary for a small list, and it adds complexity around focus, measurement, scrolling, and content that is not currently mounted.

Before adopting it, consider:

1. Can the result set be reduced with search, filters, pagination, or a server query?
2. Are rows individually expensive or is the number of DOM nodes the problem?
3. Must users navigate the complete result with a screen reader, browser find, or print?
4. Does the list contain variable-height content?
5. Is maintaining scroll position and focus manageable for this interface?

Use the least complex approach that meets the user need and performance target.

## 5. Stable identity and keys

Every item should have a stable identity from its data, such as a database ID. Use it as the React key. Do not create keys during render and do not use an array index for data that can be inserted, deleted, filtered, or reordered.

Virtualizers may use their own row key option or use the index by default. Set the item key to the stable data identity when list contents can reorder or when rows hold local state. A row's current index is a position, not a permanent identity.

Stable keys help React associate state and DOM with the correct item. They do not make filtering or sorting free; those operations still need to be measured separately.

## 6. Fixed and variable row sizes

### Fixed size

When every row has a known height, the virtualizer can calculate offsets efficiently. Fixed size is often the best starting point for compact tables, menus, and uniform result rows.

Make the CSS agree with the configured row size. Borders and padding count toward the rendered box; using box-sizing: border-box helps keep dimensions predictable.

### Variable size

When rows have different heights, the virtualizer needs an estimate or a measurement mechanism. It may use a ResizeObserver or a size cache and adjust offsets after measuring rendered rows.

Variable-size lists are more complex and can be less efficient:

- An inaccurate estimate can cause scroll position corrections.
- Images, fonts, or expanding content can change row height after initial rendering.
- Measurement results may need to be invalidated after content or width changes.
- Server rendering cannot know client-measured dimensions in advance.

Use known or predictable row sizes when possible. If variable heights are essential, choose a virtualizer that supports measurement and test resizing, image loading, and scroll restoration.

## 7. Overscan

Overscan renders a few rows outside the visible viewport. It can prevent blank gaps when scrolling quickly and can keep the next row ready for keyboard movement.

More overscan means more mounted rows and more work. Too little overscan can reveal flicker near the viewport edge; too much reduces the performance benefit. Tune it using realistic data and devices rather than maximizing it.

## 8. Filtering, sorting, and loading

Virtualization decides which rows to render; it does not choose the correct data model.

- Filter and sort the intended data before passing it to the virtualizer.
- For huge datasets, consider performing filtering and sorting on the server.
- Fetch more data before the user reaches the end if using an infinite feed.
- Keep loaded pages and result counts clear to the user.
- Avoid rebuilding expensive derived data on every render without measuring it.
- If filtering itself blocks typing, consider reducing the data work or using an appropriate deferred or asynchronous strategy.

Keep item IDs stable across pages and updates. If a query replaces the dataset, decide deliberately whether to reset scroll position and focused item.

## 9. Responsive rows and container sizing

Use CSS for visual changes such as column visibility, spacing, and typography. CSS media queries can change the row layout without reading window width during render.

A virtualizer still needs a usable viewport size. Use a list API that measures its container or observes resizing. When a container's width changes, text wrapping may change row heights, so measured-size caches may need to update.

Avoid setting a row's measured height from a one-time window.innerWidth read. It can become stale after rotation, split-screen resizing, a sidebar opening, or a parent container changing width.

For predictable fixed-height rows, design the compact layout to fit the same configured row height. If content wraps to arbitrary heights, use a dynamic-size strategy or redesign the row so its height remains predictable.

## 10. Accessibility and interaction

Virtualization changes which content exists in the DOM at a given moment. Treat that as part of the interaction design.

- Use the correct list, grid, or table semantics for the content.
- Preserve each item's accessible name and native interactive controls.
- Keep keyboard focus on an item that is still mounted; if focused content is about to be removed, move or preserve focus intentionally.
- Provide keyboard navigation when the interface pattern requires it.
- Expose the item's position and total count when the virtualizer supports suitable ARIA attributes.
- Do not add ARIA roles to replace native elements without a reason.
- Ensure overscan and scroll-to-item behavior support focus movement.
- Test with a keyboard and a screen reader, including at the first and last item.

The browser's find-in-page and screen-reader virtual navigation generally see only mounted content. If access to every row at once is important, pagination or a non-virtualized alternative may be better.

## 11. Server rendering and initial layout

A server cannot measure the user's client viewport. Many virtualized list components therefore accept an initial or default viewport size for server rendering, then measure the real container after hydration.

Choose an initial size that matches the expected layout when the library requires one. Keep server and initial client markup compatible, and test the loading and hydration behavior. A virtualized list may initially render only its first rows; that may not suit pages where every item must be present in server-rendered HTML.

## 12. Common mistakes

- Virtualizing a small list before profiling it.
- Assuming virtualization makes filtering, sorting, or data transfer cheaper.
- Using unstable keys or indexes for reorderable data.
- Choosing variable heights when a simple fixed row design would work.
- Forgetting to spread the virtualizer's provided row styles or accessibility attributes.
- Hiding important controls when the responsive layout changes.
- Letting a focused row unmount without a focus plan.
- Using one-time viewport measurements that become stale on resize.
- Assuming browser find or printing can access unmounted rows.
- Combining infinite loading with virtualization without handling the loaded range and end-of-list state.

## 13. A practical decision workflow

1. Measure a representative list interaction with realistic data.
2. Identify whether the cost is rendering, DOM/layout, data transformation, network, or row content.
3. Reduce unnecessary data or work first when that addresses the problem.
4. Compare pagination, incremental loading, and virtualization against the product requirements.
5. Choose a stable key and decide whether row size is fixed or measured.
6. Implement responsive behavior and accessibility with virtualization in mind.
7. Test scrolling, focus, keyboard navigation, resizing, empty results, and loading boundaries.
8. Profile again on representative devices and keep the extra complexity only if it helps.

## Key takeaways

- Virtualization mounts only a window of rows while representing the full scrollable range.
- It primarily reduces DOM and rendering work; it does not make all data operations cheap.
- Pagination, infinite loading, and virtualization solve different problems and can be combined.
- Stable IDs, deliberate row sizing, overscan, focus handling, and accessible semantics are essential.
- Use CSS for responsive presentation and a measuring virtualizer for changing container dimensions.
- Profile first, choose a strategy that fits the experience, then measure again.
