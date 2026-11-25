# React DOM Resource Hints and Browser APIs

A browser can begin preparing a resource before the element or module that uses it is rendered. React DOM exposes resource-hint APIs for DNS lookup, connection setup, downloads, and eager script or stylesheet initialization.

These calls are hints to the browser, not promises that a request will finish or a resource will be used. Use them when you know a resource is likely to be needed soon. React-based frameworks may already manage these hints, so first understand what the framework emits.

## Learning goals

After this lesson, you should be able to:

- distinguish DNS prefetching, preconnecting, preloading, and preinitializing;
- import resource APIs from `react-dom`;
- choose the correct API for a classic script, stylesheet, image, or ESM module;
- call hints while rendering or before a likely navigation or state transition;
- avoid wasteful hints, duplicate work, and unsafe eager script execution;
- understand when framework or server-rendering behavior affects a hint.

## What is a resource hint?

A resource hint tells the browser to start work before normal discovery or use would start that work. For example, the browser can resolve a host name, open a connection, download a file, or evaluate a script early.

React provides functions for these hints. They do not return a promise or a handle you can await. The browser decides how to carry out a hint, and it may ignore or delay it based on support, resource priority, connection state, and available bandwidth.

Import the APIs from `react-dom`:

~~~jsx
import {
  prefetchDNS,
  preconnect,
  preload,
  preloadModule,
  preinit,
  preinitModule,
} from "react-dom";
~~~

These are browser DOM APIs. They are not imported from `react` or `react-dom/client`.

## Choose the amount of work

| API | Starts | Use it when |
| --- | --- | --- |
| `prefetchDNS(origin)` | A DNS lookup for a host | You expect a future resource request from that host, but do not yet know which resource |
| `preconnect(origin)` | DNS lookup and connection setup | A request to a particular origin is likely soon |
| `preload(href, { as })` | A resource download | You know which image, font, stylesheet, script, or other resource will be needed soon |
| `preloadModule(href, { as: "script" })` | An ESM module download | You expect to import a particular module soon but do not need to evaluate it yet |
| `preinit(href, { as: "script" })` | Downloads and evaluates a classic script | A trusted non-module script should be loaded and run as soon as possible |
| `preinit(href, { as: "style", precedence })` | Downloads and inserts a stylesheet | A stylesheet should be fetched and take effect immediately |
| `preinitModule(href, { as: "script" })` | Downloads and evaluates an ESM module | A specific module should be fetched and evaluated as soon as possible |

The more work a hint starts, the more certain you should be that the resource will be needed. Do not use `preconnect` when DNS lookup alone is enough, and do not preinit executable code just because it might be used someday.

## DNS and connection setup

### `prefetchDNS`

`prefetchDNS(origin)` asks the browser to resolve a host name before the application requests a resource from it:

~~~jsx
import { prefetchDNS } from "react-dom";

function App() {
  prefetchDNS(reportingOrigin);
  return <main>Dashboard</main>;
}
~~~

DNS prefetching does not download a file and does not open the full connection. It can help when an external host is likely to be used, but you do not yet know which resource will be requested. There is no benefit to prefetching the host that already served the page; its address has already been resolved.

### `preconnect`

`preconnect(origin)` asks the browser to resolve the host and prepare a connection, including the connection setup the browser needs for requests to that origin:

~~~jsx
import { preconnect } from "react-dom";

function App() {
  preconnect(mediaOrigin);
  return <main>Media gallery</main>;
}
~~~

A preconnection can cost network resources even if no later request uses it. Reserve it for a small number of important origins. If you expect a particular file, use a resource-specific hint such as `preload` instead.

For credentialed cross-origin requests, `preconnect` can receive an options object such as `{ crossOrigin: "use-credentials" }`. The request's CORS and credential behavior must agree with the hint.

## Download a known resource

### `preload`

`preload(href, options)` asks the browser to fetch a resource early. The `as` option is required because the browser uses the resource type to apply the right fetching and priority rules:

~~~jsx
import { preload } from "react-dom";

function ProductPage() {
  preload("/assets/product-hero.webp", {
    as: "image",
    fetchPriority: "high",
  });

  return <main>Product details</main>;
}
~~~

The `as` value describes what is being fetched, such as `image`, `font`, `style`, `script`, or `fetch`. Use the value that matches how the resource will be consumed. For a `fetch` preload, set the appropriate `crossOrigin` behavior.

Preloading a stylesheet downloads it; it does not insert the stylesheet into the document or apply its rules. Use `preinit` when the stylesheet should be inserted and take effect immediately.

For responsive images, `preload` can receive `imageSrcSet` and `imageSizes` so the browser can choose the same candidate that the eventual image element will use. Keep those values consistent with the image's `srcSet` and `sizes`.

### `preloadModule`

Use `preloadModule` for an ES module that is expected to be imported:

~~~jsx
import { preloadModule } from "react-dom";

function prepareReports() {
  preloadModule("/assets/reports-panel.js", { as: "script" });
}
~~~

This starts fetching the module but does not evaluate it. `preloadModule` uses `as: "script"`. Do not confuse it with `preload(..., { as: "script" })`, which is for a script resource and has different module semantics.

## Download and activate a resource

### `preinit`

`preinit` is more eager than `preload`: it starts fetching and makes the resource take effect when it is ready.

~~~jsx
import { preinit } from "react-dom";

function enablePrintStyles() {
  preinit("/assets/print-theme.css", {
    as: "style",
    precedence: "low",
  });
}
~~~

For a stylesheet, `as: "style"` and `precedence` are required. Precedence controls where React places the stylesheet relative to other React-managed stylesheets and how those styles interact. Use the framework's styling conventions when it manages stylesheet ordering.

A script can be preinitialized with `as: "script"`. The browser evaluates it when it finishes downloading. Only do this for code you trust and intend to run; a side-effectful script may execute earlier than expected.

### `preinitModule`

`preinitModule` eagerly fetches and evaluates an ES module:

~~~jsx
import { preinitModule } from "react-dom";

function prepareFeature() {
  preinitModule("/assets/feature-startup.js", { as: "script" });
}
~~~

The module's code can run as soon as it downloads. Use this only when early evaluation is intentional. If you want to start the download without evaluating the module yet, use `preloadModule` instead.

## When to call a hint

Call a resource hint when the application has a useful reason to predict a resource will be needed:

1. **While rendering:** if a rendered component knows that it or its children will soon need an image, font, stylesheet, or module, it can declare the hint while rendering.
2. **Before a transition:** an event handler can prepare a known resource before changing to a state or route that will use it.

~~~jsx
import { preload } from "react-dom";

function OpenEditorButton({ openEditor }) {
  function handleClick() {
    preload("/assets/editor-theme.css", { as: "style" });
    openEditor();
  }

  return <button onClick={handleClick}>Open editor</button>;
}
~~~

In the browser, the resource functions can be called in different situations. During server rendering and Server Component rendering, React can only associate a hint with the response when the call occurs while rendering a component or in an async context that originated from rendering a component. Calls in unrelated server code are ignored for that render.

Frameworks often know route boundaries, assets, and server responses better than an individual component. If a framework already emits resource hints, avoid adding duplicate manual logic without measuring a real gap.

## Hints are not guarantees

Resource hints are requests to the browser, not commands that guarantee a specific start time, completion time, or performance improvement.

- A browser may ignore a hint based on support, network conditions, or its own scheduling.
- A preload does not guarantee that application code will use the resource.
- A hint does not return a promise; it cannot be awaited to know when the resource is ready.
- Repeated equivalent calls are generally deduplicated by React or the browser.
- `prefetchDNS` and `preconnect` are most useful for origins that are not already connected.
- `preinit` can execute scripts or apply CSS before the user reaches the feature.

Treat a hint as a performance tradeoff. A wrong or unused preload can compete with the page's genuinely important resources. Preloading too many files can consume bandwidth and delay critical content.

## Security and request consistency

A hint must agree with the resource's eventual use:

- match the resource type in `as`;
- use compatible `crossOrigin` settings for cross-origin resources;
- keep `integrity` and `referrerPolicy` consistent when they apply;
- use the required Content Security Policy nonce when the page's policy requires one;
- choose the correct `fetchPriority` instead of marking every resource high priority;
- never build executable script URLs from untrusted user input.

For external resources, use origins and assets that your application is authorized to request. A resource hint is not a substitute for CORS configuration, Content Security Policy, integrity checking, or safe script loading.

## Automatic hints and framework ownership

React can generate resource hints for resources it discovers during server rendering. For example, server rendering may emit image preload hints based on rendered image elements. Frameworks can also manage route-based code and asset loading.

This means manual calls are not always necessary. Before adding them:

1. inspect what the server or framework already emits;
2. identify a resource that is arriving too late;
3. add the smallest targeted hint that addresses that delay;
4. measure whether it helps without harming other critical requests.

Avoid managing the same resource through conflicting hints and framework directives.

## Common mistakes

- **Importing from the wrong entry point:** resource hint functions come from `react-dom`.
- **Mixing up the levels:** DNS lookup, connection setup, download, and eager evaluation are different amounts of work.
- **Using `preinit` when only a download is needed:** this can execute scripts or apply styles earlier than intended.
- **Using `preload` for an ESM module:** use `preloadModule` or `preinitModule` depending on whether it should be evaluated.
- **Using the wrong `as` value:** browsers use it to determine how to fetch and prioritize the resource.
- **Preloading every asset:** speculative work competes with content the user needs now.
- **Preconnecting to the current page's own origin:** the connection is already in use.
- **Assuming the hint completed:** these APIs provide no promise or completion result.
- **Ignoring framework behavior:** frameworks may already generate the correct hints and deduplicate assets.
- **Treating hints as a security boundary:** validate origins and keep script execution under a deliberate trust policy.

## Practical checklist

1. Which exact resource or origin is likely to be needed, and how soon?
2. Is DNS lookup enough, or should the browser connect or download?
3. Should the browser only fetch the resource, or should it also evaluate or apply it?
4. Does the hint match the resource's actual type, credentials, and security policy?
5. Is the framework already producing a hint for this resource?
6. Could the resource be unused or compete with more important requests?
7. Did measurement show the hint improves the real user path?

## Summary

React DOM exposes `prefetchDNS` for DNS lookup, `preconnect` for connection setup, `preload` and `preloadModule` for downloads, and `preinit` and `preinitModule` for resources that should become active or evaluated immediately. These are browser hints, not guarantees. Use the least eager hint that helps a likely user path, and let a framework manage assets when it already does that well.
