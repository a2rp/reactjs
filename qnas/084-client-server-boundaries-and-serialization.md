# Client and server boundaries, serialization, and security — Q&A

## Boundaries and module graphs

### 1. What is a server/client boundary?

It is the point where code is assigned to a server or client module graph. In frameworks that use `"use client"`, that directive marks a client entry point and its transitive imports become part of the client graph.

### 2. Does "use client" apply only to the component written in that file?

No. It applies to the module and its transitive dependencies. A component imported by that module becomes part of the client graph even if its own file does not have the directive.

### 3. Does every component under a Client Component become a Client Component?

The module graph determines which implementation code is included in the client bundle. A Server Component can still provide already-rendered JSX as a supported prop such as `children`; the client component should not import the Server Component's module directly.

### 4. Does a JSX parent-child relationship tell me where both components run?

No. The rendered component tree and module dependency graph are different. A Server Component can create a child and pass the resulting element through a prop to a Client Component, so their source modules can belong to different environments.

### 5. Can one module be evaluated on both server and client?

Depending on how it is imported, a shared module can be evaluated in both environments. Keep shared modules free of privileged APIs and secrets, and use explicit framework boundary markers for code that must run in only one environment.

### 6. Why keep Client Component boundaries small?

The client entry point and its dependencies need to be available to the browser. A focused boundary limits client JavaScript and makes it easier to review which code has browser capabilities.

## Serialization

### 7. What does serialization mean at this boundary?

The framework transports values between different rendering environments in a supported format. React's Server Component protocol supports more than JSON, but not every JavaScript value can be sent.

### 8. Which values can commonly be passed from a Server Component to a Client Component?

React supports primitives such as strings, numbers, booleans, null, undefined, and bigint; globally registered symbols; plain objects and supported iterables; Date; ArrayBuffer and typed arrays; supported React elements; promises; and Server Function references. Follow the supported types for the React and framework versions in use.

### 9. Can I pass an array or plain object?

Yes, when its contents are serializable. Prefer small plain objects with explicit fields instead of sending an entire database record.

### 10. Can I pass a Date, Map, or Set?

React's Server Component protocol supports Date and supported iterables such as Map and Set, subject to the React version and framework implementation. This differs from JSON serialization, which does not preserve all these values in the same way.

### 11. Can I pass a class instance?

Generally no. Custom class instances and objects with unsupported prototypes are not ordinary serializable props. Convert them to a supported plain data shape before crossing the boundary.

### 12. Can I pass an event handler from a Server Component to a Client Component?

Not as an ordinary function prop. A locally created callback cannot be serialized into browser code. Put browser event handling in a Client Component, or use a framework-supported Server Function when an interaction must request server work.

### 13. Are Server Functions an exception to the callback rule?

A Server Function reference is a special framework-supported reference, not an ordinary JavaScript callback. Its arguments still cross a request boundary and must be treated as untrusted input. Authenticate, authorize, and validate inside the function.

### 14. Can I pass a React element as a prop?

React's Server Component protocol supports React elements, including rendered UI passed through a slot such as `children`. This is useful for composing server-rendered content inside an interactive client wrapper.

### 15. Can I pass a Promise?

Promises are among the supported Server Component transport values in React. How and when a framework consumes or streams a Promise is framework-specific. Do not assume all ordinary HTTP JSON or component APIs handle it the same way.

### 16. Is `JSON.stringify` the definition of serializable here?

No. The React Server Component transport supports values that JSON does not, and may reject values JSON accepts in lossy or misleading ways. Use React's supported type set rather than inventing your own JSON-only rule.

## Safe data transfer

### 17. What is a DTO?

A Data Transfer Object is a deliberately shaped object containing only the values needed by the receiving component or layer. For a Client Component, it makes the server-to-client data boundary explicit.

### 18. Does a DTO automatically make data safe?

No. The server must still check that the current user is allowed to see each field. A DTO helps minimize what crosses the boundary after authorization has been applied.

### 19. Why should I avoid passing a full database row?

A full row may contain fields the UI does not need, such as internal flags, private contact information, tokens, or audit data. Passing it can expose those values through the rendered payload and make future changes risky.

### 20. Is data private because the component that fetched it is server-only?

No. The component implementation may stay on the server, but rendered HTML, the RSC payload, Client Component props, and returned action values can reach the user. Do not render or pass secrets.

### 21. What should a profile DTO look like?

It should contain only fields that the current viewer is allowed to see and that the UI requires, for example `{ displayName, avatarUrl }`. If a phone number is visible only to some viewers, enforce that rule before building the DTO.

### 22. Can I safely pass an environment variable to a Client Component?

Do not pass private secret values. Frameworks may expose variables with a public prefix to the browser bundle. Keep access to private environment variables in server-only code and pass only safe derived values.

## Server-only modules and authorization

### 23. What does a server-only marker do?

A framework-supported server-only marker helps catch an accidental import of a server module into client code, often by reporting a build error. It protects the module boundary; it does not check user permissions or hide data already returned to the client.

### 24. Does the `server-only` package work in every React build system?

No. Support and exact conventions are framework-specific. Use this marker only in a framework that supports it, or use the equivalent boundary protection provided by the chosen build system.

### 25. When should authorization happen?

Before returning protected data or performing a protected mutation, at the server-side data access or operation boundary. Do not rely only on which buttons the page chooses to render.

### 26. Is a page-level authentication check enough to protect a Server Function?

No. A Server Function is a separate callable entry point. Check the caller's identity and permissions inside the function or the protected data layer it invokes.

### 27. Should I trust form fields, route parameters, or Client Component props?

No. They can be changed or fabricated by the caller. Validate types, formats, ranges, and allowed values before using them in queries or operations.

### 28. How do I prevent query injection?

Use parameterized queries or the database library's safe query API. Do not concatenate user-controlled strings into a query.

### 29. Should a Server Function return the full updated database record?

Usually not. Return the minimum result the client needs, such as a success flag or a filtered DTO. The return value is serialized back to the caller.

### 30. Can experimental taint APIs replace DTOs and authorization?

No. If supported, tainting can be an additional safeguard against accidentally passing marked values to the client. It does not replace permission checks or selecting only safe fields.

## Caching, errors, and review

### 31. How can caching leak data across users?

A shared cache can return a result created for one user to a different user if the cache scope or key does not account for identity, tenant, or permission differences. Follow the framework's caching model and avoid caching personalized data without a safe scope.

### 32. Is a module-level variable a safe place to store the current user?

No. A server process may serve multiple requests. A mutable process-wide variable can accidentally share request-specific information. Keep user data request-scoped or in an explicitly identity-aware cache.

### 33. What should a visitor see when a server lookup fails?

Show a useful, non-sensitive error state. Keep stack traces, database messages, file paths, tokens, and internal service details out of the rendered response.

### 34. What security questions should I ask before passing a value to a Client Component?

Ask whether the browser needs the value, whether this user is allowed to see it, whether the value uses a supported serialization type, whether it is a minimal DTO, and whether its source module is safe to include in the client graph.

### 35. What is the key distinction between code secrecy and data secrecy?

Server-only code can remain out of the browser bundle, but its rendered result or returned data may still be sent to the browser. Keeping code on the server does not make every value it reads secret.

### 36. What is a practical review checklist for a new boundary?

Trace imports from every client entry point, inspect all props and rendered output, reduce data to explicit DTOs, check authorization at each read and mutation, validate caller-controlled values, review error output, and confirm cache scope cannot mix users.
