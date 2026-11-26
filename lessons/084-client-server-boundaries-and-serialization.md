# Client and server boundaries, serialization, and security

## Learning goals

After this lesson, you should be able to:

- Explain how the client boundary changes a module dependency graph.
- Distinguish the component tree from the server/client module boundary.
- Identify values that React can serialize across a Server-to-Client Component boundary.
- Design a small, safe data-transfer object for a Client Component.
- Protect server-only modules and keep secrets out of rendered output.
- Validate and authorize data at the server entry point that uses it.
- Avoid request-specific data leaks caused by unsafe caching.

## Two environments, one React UI

An RSC-enabled application can render parts of the UI in a server environment and parts in a browser environment. These environments have different capabilities:

- **Server code** can use trusted server resources such as a database connection, private environment variables, or a filesystem API supported by the deployment.
- **Client code** can use state, effects, event handlers, and browser APIs. It must be treated as code that runs for the visitor.
- **Rendered output and values crossing the boundary** can be sent to the browser, even when the code that produced them stays on the server.

The server/client distinction is a security and bundling boundary. It does not remove the need for authentication, authorization, input validation, or careful data handling.

## The module graph boundary

In frameworks that use the `"use client"` directive, it marks a module as a client entry point. The module and its transitive imports are part of the client module graph.

```jsx
"use client";

import { useState } from "react";
import { formatPrice } from "./format-price";
import Button from "./button";

export default function CartControls({ initialQuantity }) {
  const [quantity, setQuantity] = useState(initialQuantity);

  return (
    <section>
      <p>{formatPrice(quantity)}</p>
      <Button onClick={() => setQuantity((value) => value + 1)}>
        Add one
      </Button>
    </section>
  );
}
```

Even though `format-price.js` and `button.jsx` do not contain `"use client"`, they are imported by a client entry point and become part of its client-side dependency graph. Do not import database drivers, secret-bearing configuration, or server-only data access modules from this graph.

A module may be evaluated in different environments when it is imported by both server and client code. Do not assume that a file is server-only simply because it has no directive. Make environment boundaries explicit and follow the framework's rules.

## Module graph is not the rendered component tree

The parent/child relationship seen in JSX does not always mean both components run in the same environment. A Server Component can create a server-rendered element and pass it into a Client Component through a supported prop such as `children`.

```jsx
// Server Component
import ExpandablePanel from "./expandable-panel";
import AccountDetails from "./account-details";

export default async function AccountPage() {
  const account = await getAccountForCurrentUser();

  return (
    <ExpandablePanel title="Account details">
      <AccountDetails account={account} />
    </ExpandablePanel>
  );
}
```

```jsx
// expandable-panel.jsx
"use client";

import { useState } from "react";

export default function ExpandablePanel({ title, children }) {
  const [open, setOpen] = useState(false);

  return (
    <section>
      <button type="button" onClick={() => setOpen((value) => !value)}>
        {open ? "Hide" : "Show"} {title}
      </button>
      {open ? <div>{children}</div> : null}
    </section>
  );
}
```

The Server Component imports and creates `AccountDetails`; the client module does not import that server module. It receives the rendered child through the framework's component transport. This lets the panel handle interaction while keeping server-owned data access outside the client dependency graph.

## Serialization across the boundary

Values passed from a Server Component to a Client Component cross a transport boundary. They must use types supported by React's Server Component serialization. This protocol supports more than plain JSON, but it is still a defined set of values; do not assume any JavaScript value can cross it.

Supported values include:

- Primitive values such as strings, numbers, booleans, `null`, `undefined`, and `bigint`.
- Symbols registered with `Symbol.for`.
- Plain objects and iterables containing supported values, including arrays, maps, and sets.
- `Date`, `ArrayBuffer`, and typed-array values.
- React elements (JSX) supported by the framework.
- Promises supported by the React Server Components transport.
- Server Function references that the framework supports.

Values that generally do not cross as ordinary props include:

- A locally defined event handler or callback function.
- A component function or arbitrary function.
- Class instances and custom class objects.
- Objects with unsupported prototypes.
- A symbol created with `Symbol()` instead of the global registry.

Frameworks and React versions can refine these details. When a value crosses the boundary, use the framework's supported types and check its diagnostics. A server function reference is a special supported reference; it does not make arbitrary functions serializable.

### A local callback is not a server callback

This does not work as a normal Server-to-Client prop:

```jsx
// Server Component: do not pass a local callback this way.
<ClientButton onSave={() => saveRecord(record.id)} />
```

The local function cannot be serialized as a browser prop. If the browser needs to request a server mutation, use the framework's supported Server Function or form Action mechanism. That function must validate its input and authorize the caller every time it runs.

### Prefer explicit data shapes

Use a plain object containing only the fields the Client Component needs:

```jsx
const accountSummary = {
  displayName: account.displayName,
  avatarUrl: account.avatarUrl,
  canEdit: permissions.canEdit,
};

return <AccountControls account={accountSummary} />;
```

This is often called a data-transfer object (DTO). A DTO makes the boundary visible in code review and reduces accidental exposure of private model fields. Its shape alone does not authorize access; the server must still verify that this viewer is allowed to receive those values.

## The rendered result can be public

Keeping a component implementation on the server does not keep all of its output secret. Values can become observable through:

- HTML sent for the initial page.
- The React Server Component payload used to render or update the UI.
- Props passed to Client Components.
- A Server Function's returned state or value.
- Error output that exposes internal details.

If the visitor can see or inspect a value in the rendered page, it is not a secret. Do not put credentials, private tokens, password hashes, internal notes, or hidden authorization data into rendered output or client props.

## Protect the data access layer

For an application with private data, put data access and authorization in a server-only layer. Keep route components and Client Components from reaching database internals directly.

A server-only data module can return a safe DTO:

```js
import "server-only";

export async function getPublicProfile(profileId, viewerId) {
  const profile = await database.profile.findById(profileId);

  if (!profile) {
    return null;
  }

  const viewerCanSeePhone =
    await permissions.canViewPhoneNumber(viewerId, profile.id);

  return {
    displayName: profile.displayName,
    phoneNumber: viewerCanSeePhone ? profile.phoneNumber : null,
  };
}
```

The example uses a framework-supported `server-only` marker. In frameworks that do not provide this exact package or convention, use their equivalent boundary protection. This marker helps catch an accidental client import; it does not replace authorization or data minimization.

Do not rely on environment-variable naming alone to protect a secret. Some frameworks deliberately expose variables with a public prefix to client bundles. Keep private environment access inside server-only code and never pass the secret's value into rendered output.

## Validate and authorize every server entry point

A URL parameter, form field, header, cookie, or Client Component prop can be missing, malformed, stale, or controlled by the caller. Validate values before using them in a query or mutation.

For mutations, verify both:

1. **Authentication:** who is making the request?
2. **Authorization:** may that user perform this operation on this specific resource?

A page-level check does not automatically protect a separate Server Function. Each callable entry point must enforce the permissions it needs because requests can be made without following the visible UI flow.

Use parameterized database operations or a safe query API. Never build a query by concatenating raw user input. Return only the result the UI needs, such as `{ success: true }`, rather than a full database record.

## Prevent environment poisoning

Environment poisoning happens when a module intended for one environment is accidentally included in the other environment. For example, a shared helper might import a server database client and later be imported from a Client Component.

Use the framework's server-only and client-only markers where available. Keep modules focused:

- Server data modules can import database clients, private configuration, and server APIs.
- Client UI modules can import browser-safe formatting and presentation helpers.
- Shared modules should avoid secrets and APIs that exist in only one environment.
- Review transitive imports when adding `"use client"`; the whole imported graph matters.

A build-time boundary error is useful feedback. Do not work around it by exposing a secret or moving a privileged operation into browser code.

## Caching and user-specific data

Server rendering can be cached or reused according to framework settings. A cache key that ignores the current user, tenant, or permission scope can expose one person's result to another.

Before caching sensitive data, establish:

- Which values vary by user, tenant, role, or request.
- Whether the framework's cache is per request or shared across requests.
- Whether the identity and permission scope are represented safely in the cache key.
- How data is invalidated after a mutation or permission change.

Do not place a request-specific user object in a process-wide mutable variable. Avoid shared caches for personalized output unless the framework and cache key explicitly prevent cross-user reuse.

## Errors and logging

Show visitors a useful, non-sensitive error state. Keep stack traces, SQL messages, file paths, tokens, and internal service responses in protected server logs. Avoid logging credentials or full request objects that may contain secrets.

When a lookup fails, distinguish between expected cases (such as a missing public profile) and unexpected server failures. Do not reveal whether a protected record exists if that would expose private information.

## Security review checklist

For each server-to-client boundary, ask:

- Is this value required in the browser?
- Is it safe for this specific user to see?
- Is the shape a minimal DTO rather than a raw database record?
- Are all values supported by the React/framework serialization protocol?
- Did a client entry point pull server-only imports into its dependency graph?
- Are user-controlled values validated before data access?
- Is authorization checked at the operation that reads or changes the resource?
- Can a cache reuse personalized data across users?
- Do errors or logs reveal secrets or internal details?

## Common mistakes

- **"It was fetched on the server, so it is secret."** Rendered and serialized values can still reach the browser.
- **Passing the full database row to a Client Component.** Build and return a minimal DTO.
- **Passing an ordinary callback as a prop across the server boundary.** Use a supported Server Function for server mutations, with validation and authorization.
- **Treating `"use client"` as a component-only annotation.** It marks the module and its transitive dependencies as client code.
- **Assuming a child component always shares its parent's environment.** Component composition and module dependency boundaries are different.
- **Relying on a hidden button or page check for access control.** Check authorization inside the protected read or mutation.
- **Relying on experimental tainting as the main defense.** Filter data and enforce permissions first; optional framework protections are additional safeguards.
- **Caching personalized data without identity-aware scope.** A shared cache can serve the wrong user's data.

## Summary

The `"use client"` boundary affects the module graph and its imports, while the rendered component tree can interleave server output with client interaction. Values sent across that boundary must use React's supported serialization types. Treat rendered output and props as visible to the visitor, return minimal DTOs, keep privileged code in a server-only layer, validate untrusted input, and authorize each protected operation at its server entry point.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="083-react-server-components.md">083 - React Server Components and server rendering</a></td><td><a href="../qnas/084-client-server-boundaries-and-serialization.md">Q&amp;A</a></td><td><a href="../examples/084-client-server-boundaries-and-serialization.md">Example</a></td><td><a href="085-routing-concepts-and-router-choice.md">085 - Routing concepts and choosing a router</a></td></tr>
  </tbody>
</table>
