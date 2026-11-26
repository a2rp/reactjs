# Security Basics: Escaping, URLs, and Untrusted Data - Q&A

## Untrusted data and React rendering

### 1. What counts as untrusted data?

Any value a user, URL, browser, external service, or user-editable record can influence. Examples include form values, route and query parameters, comments, API responses, imported records, and browser storage.

### 2. Does data become trusted when it comes from our database?

No. A database can contain values users entered earlier or data imported from another system. Evaluate trust based on who could control the value and how the application uses it.

### 3. How does React render a string child?

A string child such as a name inside a paragraph is rendered as text. React does not interpret that ordinary string as JSX or parse it as an HTML element.

### 4. Should I escape text manually before rendering it in JSX?

Usually no. Render the original string as a child and let React handle normal text rendering. Manual HTML encoding can cause visible entities such as ampersands and angle brackets to appear as text, and it does not make a value safe for every other context.

### 5. Does React make all values safe in every context?

No. Normal JSX rendering provides useful protections for ordinary text and attributes, but it does not validate URL destinations, authorize operations, sanitize arbitrary HTML, validate business rules, or make code-execution APIs safe.

### 6. What is cross-site scripting (XSS)?

XSS is a class of vulnerabilities where untrusted data reaches a browser context that treats it as executable markup or code. It can let an attacker run actions in the victim's page context.

### 7. What should I use for a plain-text comment?

Render it as a normal React child, such as a paragraph's contents. Do not insert it as raw HTML.

### 8. What does dangerouslySetInnerHTML do?

It sends an HTML string to the browser for parsing and overrides the element's normal children. It bypasses React's ordinary text rendering and can introduce XSS if the string contains untrusted markup.

### 9. Is dangerouslySetInnerHTML safe if the value came from an API?

Not automatically. The API may store user-authored content or return data from another system. Only use raw HTML after an appropriate, maintained sanitizer has processed the HTML for the intended feature.

### 10. Can a regular expression safely sanitize arbitrary HTML?

No. HTML parsing and browser behavior are too complex for ad hoc regular-expression filtering. Use a maintained HTML sanitizer with an explicit policy if rich HTML is required.

### 11. Is a sanitizer required for every user comment?

No. For plain text, render a string child. Sanitization is for a feature that intentionally accepts and renders HTML structure or formatting.

### 12. Does sanitizing once mean the value can be modified freely afterward?

No. Later transformations can reintroduce unsafe markup or attributes. Keep sanitization close to HTML generation and avoid unsafe post-processing.

## URLs and navigation

### 13. Why does an href need validation if React escapes attributes?

Attribute escaping protects the markup syntax, but it does not decide whether the destination is allowed. A URL can be correctly quoted and still use a scheme, origin, or host the feature should reject.

### 14. What URL schemes should a normal web link usually allow?

For a feature that accepts web links, an application commonly allows only HTTP and HTTPS. Use a parsed URL and an explicit policy. A different feature may need a narrower rule, such as same-origin paths or a partner-host allowlist.

### 15. Is checking for http or https enough to prove a destination is trustworthy?

No. It only restricts the scheme. An arbitrary external HTTP or HTTPS site can still be deceptive or unwanted. Apply a host or origin allowlist when the product has a known destination set, and make external destinations clear to the user.

### 16. How should a URL value be parsed?

Use the URL constructor inside error handling. Provide a deliberate base URL if relative paths are accepted, then inspect properties such as protocol, origin, hostname, username, and password according to the feature's policy.

### 17. What does resolving a relative URL against window.location.origin do?

It turns a relative path into an absolute URL using the current application's origin. This is useful when the feature accepts internal paths, but a separate same-origin check is still needed if external destinations must be prohibited.

### 18. Is a protocol-relative URL such as //other.example internal?

No. The browser resolves it using the current page's scheme but it can still point to a different host. Compare the parsed origin or hostname when enforcing an internal-only policy.

### 19. How do I restrict a link to the current site?

Parse the URL and require its origin to equal the expected application origin. Also decide which paths are valid if only certain internal routes should be reachable.

### 20. How do I restrict links to a small set of external sites?

Parse the URL, check its protocol, and compare the normalized hostname against an explicit allowlist. Avoid loose suffix checks that accidentally accept a look-alike host.

### 21. What is an open redirect?

An open redirect occurs when an application redirects users to a destination controlled by a request parameter or other untrusted value. Attackers can use a trusted site's redirect link to make a malicious destination appear more credible.

### 22. How should an application handle a returnTo or next parameter?

Prefer mapping a known identifier to a route the application controls. If arbitrary internal paths are necessary, enforce a same-origin policy and restrict the accepted path shape. Do not redirect to any URL merely because it parses successfully.

### 23. Does encodeURIComponent validate a complete URL?

No. It encodes a URL component such as one query parameter value. It does not determine whether an entire URL is acceptable.

### 24. How should I construct a query string?

Use URL and URLSearchParams. Set each parameter value explicitly rather than concatenating user-controlled text into the URL string. Still validate the values against the feature's rules.

### 25. Is a link that opens a new tab different from a same-tab link?

It changes navigation behavior but does not remove the need to validate the destination. For a new tab, explicitly use rel="noopener noreferrer" with target="_blank" as a defensive convention.

### 26. Can a URL field's input type replace validation?

No. An input type such as url can improve browser feedback, but client-side controls can be bypassed. Parse and enforce the required policy in application code, and validate again on the server before accepting the data.

## Forms, APIs, and authorization

### 27. Is client-side validation a security boundary?

No. It is useful for fast feedback and reducing accidental errors. A caller can bypass the interface and send a request directly, so the server must repeat validation.

### 28. What is the difference between validation and authorization?

Validation checks whether a value follows an accepted format or business rule. Authorization checks whether the current user may perform an action on a particular resource. Protected operations require server-side authorization even when their input is valid.

### 29. Is hiding a button enough to protect an operation?

No. A hidden or disabled control only changes the interface. The server endpoint must verify the user's permissions for every protected action.

### 30. Should data in localStorage or sessionStorage be trusted?

No. Users and other scripts in the page can change browser storage. Validate its shape when reading it, and do not use it as a secure store for secrets or as evidence that a user is authorized.

### 31. Are TypeScript types runtime validation?

No. Types help catch mistakes during development, but they do not verify arbitrary JSON received from a server at runtime. Validate data at boundaries when its shape matters.

### 32. Is a database value safe because the backend returned it?

Not automatically. It may have originated from user input. Render it according to its destination context and apply the same URL or HTML policies that the feature requires.

### 33. Should user input ever be passed to eval?

No. Do not turn input into source code. Use trusted application functions and pass the value as ordinary data.

### 34. How should button behavior be attached?

Use a function defined by the application with a React event prop such as onClick. Avoid constructing HTML event-handler strings from user data.

## Defense in depth and review

### 35. Does Content Security Policy fix unsafe rendering?

No. A Content Security Policy can limit some execution and resource-loading paths as an additional defense. It does not replace safe rendering, URL validation, input validation, or authorization.

### 36. What should I review when tracing data through a component?

Identify the source, determine whether the value is user-controlled, identify its final context, and check the right control for that context. Review whether a server repeats validation and authorization when an action or protected resource is involved.

### 37. What is the first choice for displaying user text?

Render it as a normal React child. Use a raw HTML path only when the product intentionally supports rich HTML and a maintained sanitizer applies an explicit policy.

### 38. What is the key difference between text escaping and URL validation?

Text escaping prevents ordinary text from being parsed as markup in a text context. URL validation decides whether a parsed destination, scheme, origin, host, or route is allowed. One does not replace the other.

### 39. What should I test for a URL input?

Test an allowed relative path, allowed HTTP and HTTPS examples, malformed input, empty input, disallowed schemes, credentials in the URL if the policy rejects them, and a different origin when the feature is internal-only. Also test what the interface shows when the value is rejected.

### 40. What is the central rule for handling untrusted data?

Keep values as data. Render text with normal React children, validate URLs against an explicit policy, avoid raw HTML and code execution, and enforce validation and permissions on the server.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="111-code-splitting-and-asset-loading.md">111 - Code Splitting, Asset Loading, and Bundle Analysis - Q&amp;A</a></td><td><a href="../lessons/112-security-and-untrusted-data.md">Lesson</a></td><td><a href="../examples/112-security-and-untrusted-data.md">Example</a></td><td><a href="113-html-injection-and-trusted-types.md">113 - HTML Injection, Sanitization, and Trusted Types - Q&amp;A</a></td></tr>
  </tbody>
</table>
