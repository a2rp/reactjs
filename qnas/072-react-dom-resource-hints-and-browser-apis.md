# React DOM Resource Hints and Browser APIs - Q&A

## API basics

### 1. What are React DOM resource hints?

They are APIs that ask the browser to start resource-related work early, such as resolving a host, preparing a connection, downloading a file, or evaluating a script.

### 2. Which package exports the resource hint APIs?

Import them from `react-dom`, not from `react` or `react-dom/client`.

~~~jsx
import { preload, preconnect } from "react-dom";
~~~

### 3. Which APIs are the main React DOM resource hint APIs?

`prefetchDNS`, `preconnect`, `preload`, `preloadModule`, `preinit`, and `preinitModule`.

### 4. Do these functions return a promise?

No. They return no completion promise or resource handle. The browser decides when and whether it can carry out the hint.

### 5. Can I await a preload to know when the file is ready?

No. A resource hint is not an async loading API. If application logic needs to know whether a module or data request has completed, use the corresponding import or fetch workflow.

## DNS and connections

### 6. What does `prefetchDNS` do?

It asks the browser to resolve the IP address for an origin before a later request needs it. It does not open the full connection or download a resource.

### 7. What does `preconnect` do?

It asks the browser to resolve the host and prepare a connection to the origin. The connection may then be reused for later resource requests.

### 8. How do `prefetchDNS` and `preconnect` differ?

`prefetchDNS` performs DNS lookup. `preconnect` goes further by setting up the network connection. Use the less costly hint that suits the likely request.

### 9. Should I preconnect to every origin my app might use?

No. A preconnection uses network resources even if it is never used. Reserve it for a small number of likely, important origins.

### 10. Is there a benefit to preconnecting to the origin that served the current page?

Usually not. The browser is already connected to the page's origin by the time the hint could be issued.

## Downloads and activation

### 11. What does `preload` do?

It asks the browser to download a known resource early, such as an image, font, stylesheet, or script.

### 12. Why is the `as` option required for `preload`?

The browser needs to know the resource type to select the right fetching and priority behavior. Choose the value that matches how the resource will be used.

### 13. Does preloading a stylesheet apply its rules?

No. `preload` fetches a stylesheet but does not insert it as an active stylesheet. Use `preinit` when the stylesheet should be inserted and applied.

### 14. What does `preloadModule` do?

It starts fetching an ES module that the application expects to import. It does not evaluate the module when it is merely preloaded.

### 15. What is the required `as` value for `preloadModule`?

Use `{ as: "script" }`. This API is specifically for an ES module.

### 16. What does `preinit` do for a classic script?

It starts fetching and evaluates the script when the download completes. The code can run before the user reaches the feature.

### 17. What does `preinit` do for a stylesheet?

It fetches and inserts the stylesheet so its rules take effect. The stylesheet call requires `as: "style"` and a `precedence` value.

### 18. How is `preinitModule` different from `preloadModule`?

`preloadModule` fetches a module without evaluating it. `preinitModule` fetches and evaluates the module as soon as possible.

### 19. Which APIs download without activating code?

`preload` and `preloadModule` start downloads. Use `preinit` or `preinitModule` only when early activation or evaluation is intentional.

### 20. Why is `preinit` more sensitive than `preload`?

A preinitialized script executes and a preinitialized stylesheet takes effect. This can change the page before the user opens the feature, so only preinitialize trusted resources that should become active early.

## Call timing and framework behavior

### 21. Can a component call a resource hint while rendering?

Yes. If the component knows that it or its children will need a resource, it can declare the hint while rendering.

### 22. Can an event handler call a resource hint?

Yes. A handler can start preparing resources before a navigation or state transition that will use them.

### 23. How can a button prepare the next screen's stylesheet?

Call `preload` in the click handler before switching screens, using `{ as: "style" }`. The browser can begin downloading it while the transition proceeds.

### 24. Do server-rendering calls work from anywhere in server code?

No. During server rendering or Server Component rendering, React only associates a hint with the response when the call occurs during component rendering or in an async context originating from that rendering. Unrelated server calls can be ignored.

### 25. Do React frameworks already manage these hints?

Often they do. A framework may know which route assets, images, or modules are needed and emit hints automatically. Check what it does before adding manual hints.

### 26. Can React automatically preload an image during server rendering?

React can generate a preload hint for some rendered images during server rendering. Frameworks may further manage images, so check the behavior of the actual rendering stack.

### 27. Are repeated resource hints duplicated?

Equivalent repeated hints are generally deduplicated by React or the browser. Do not use repeated calls as a mechanism for retrying or tracking completion.

## Options and security

### 28. When should I set `crossOrigin`?

Set it when the resource's cross-origin request needs a particular CORS or credential mode. It is required for some `preload` fetch resources, and it should agree with the eventual request.

### 29. What is the purpose of `integrity`?

It lets the browser verify that a fetched resource matches an expected cryptographic hash. Keep the integrity value consistent with the resource that will actually be used.

### 30. What does `fetchPriority` do?

It suggests a relative fetch priority such as `high`, `low`, or `auto`. It is a hint, not a guarantee. Do not mark every resource as high priority.

### 31. What are `imageSrcSet` and `imageSizes` for?

They let an image preload describe the same responsive image candidates as the eventual image. Keep them aligned with the image element's `srcSet` and `sizes` so the browser does not fetch a different candidate unnecessarily.

### 32. Can hints replace Content Security Policy or CORS configuration?

No. Hints do not grant access or make a resource trusted. Keep CORS, Content Security Policy, integrity, and script-loading rules configured correctly.

### 33. Can a resource hint URL come from user input?

Do not construct script or stylesheet URLs from untrusted user input. Validate the allowed origins and resource paths, especially for APIs that execute scripts or apply styles.

## Performance and debugging

### 34. Why can preloading too many files slow a page down?

Speculative requests compete for bandwidth and browser priority with resources the user needs immediately. Unused downloads waste network and device resources.

### 35. How do I decide whether to use `preload` or `preinit`?

Ask whether the resource should merely be downloaded or should also become active immediately. Choose `preload` for fetch-only behavior; use `preinit` when activation is deliberately desired.

### 36. Why is my resource still loading late after I add a hint?

A hint is not a guarantee. The browser may ignore or deprioritize it, the URL may not match the resource actually used, or the hint may be issued too late. Inspect requests and priorities in browser developer tools.

### 37. Why did my preloaded file get downloaded twice?

The hint and actual resource use may not match in URL, type, credentials, responsive image candidate, or other fetch settings. Make the hint describe the same resource request the browser will later make.

### 38. Can a preload be awaited before navigating?

No. These APIs return no promise. Start the hint before the transition and let the browser manage the request; use an actual module import or data request if the application must wait for a result.

### 39. Should I use `prefetchDNS` if I know the exact file I need?

If the resource URL is already known and should start downloading, use a specific preload API. DNS lookup alone does not fetch the file.

### 40. What is the safest resource-hint strategy?

Let the framework handle routine assets, identify a real delay, choose the least eager matching hint, keep its request options consistent, and verify the change with network measurements.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="071-flushsync.md">071 - Synchronous DOM Updates with flushSync - Q&amp;A</a></td><td><a href="../lessons/072-react-dom-resource-hints-and-browser-apis.md">Lesson</a></td><td><a href="../examples/072-react-dom-resource-hints-and-browser-apis.md">Example</a></td><td><a href="073-client-side-data-fetching-and-use-api.md">073 - Client-Side Data Fetching, the `use` API, and Async Values - Q&amp;A</a></td></tr>
  </tbody>
</table>
