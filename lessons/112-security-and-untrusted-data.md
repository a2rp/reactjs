# Security Basics: Escaping, URLs, and Untrusted Data

## Learning objectives

After this lesson, you should be able to:

- Identify untrusted data in a React application.
- Explain what React's normal JSX rendering protects and where those protections stop.
- Render user-provided text as text instead of interpreting it as markup or code.
- Validate URL schemes and construct query strings safely.
- Recognize dangerous escape hatches such as raw HTML insertion and code evaluation.
- Separate client-side validation from server-side validation and authorization.
- Review a feature for common cross-site scripting and navigation risks.

## 1. Security starts with tracking data

Untrusted data is any value that a user, URL, browser, external service, or persisted record can influence. It includes:

- Form fields and profile names.
- Route parameters, query parameters, and URL fragments.
- Comments, descriptions, filenames, and database records originally entered by users.
- Responses from APIs, webhooks, imports, and third-party services.
- Values restored from local storage or other browser storage.

A value does not become trustworthy just because it came from your own database. It may have been entered by a user earlier, imported from another system, or modified by an account with fewer privileges.

Security depends on how data is used. A string that is safe to display as a label might be unsafe as HTML, a URL, CSS, a redirect destination, or executable code. Treat data as data and choose protection for the destination context.

## 2. What normal JSX rendering does

React normally renders a string child as text:

    function ProfileName({ name }) {
      return <p>{name}</p>;
    }

If name contains characters that resemble markup, React displays those characters as text. It does not treat a normal string child as JSX or parse it as an HTML element. This is the right default for names, comments, search terms, and other plain text.

React also handles ordinary quoted text attributes through its renderer:

    function SearchInput({ query }) {
      return <input aria-label="Search" value={query} readOnly />;
    }

Keep tag names, prop names, and event handlers in your code. Pass dynamic values to ordinary data props rather than constructing markup strings.

This protection is useful but limited. It does not mean every value is safe in every place. In particular, normal text handling does not decide whether a URL is an acceptable destination, sanitize HTML for insertion, validate business rules, or authorize an operation.

## 3. Avoid interpreting text as HTML

Do not use dangerouslySetInnerHTML with user-controlled or otherwise untrusted strings:

    function UnsafeComment({ comment }) {
      return <div dangerouslySetInnerHTML={{ __html: comment }} />;
    }

This bypasses React's normal text rendering and asks the browser to parse the string as HTML. If the data can contain attacker-controlled markup or attributes, this can create a cross-site scripting (XSS) vulnerability.

For plain text, render it as a child:

    function Comment({ comment }) {
      return <p>{comment}</p>;
    }

If a product genuinely needs user-authored rich HTML, use a maintained HTML sanitizer configured with a narrow allowlist. Keep the sanitization step close to the point where the HTML is produced, and do not modify sanitized markup afterward in a way that reintroduces unsafe content. A sanitizer is a specialized control, not a reason to trust arbitrary HTML. The next lesson covers HTML insertion, sanitization, and Trusted Types in more depth.

Do not try to sanitize HTML with a regular expression. HTML parsing has rules and browser behaviors that make ad hoc filtering unreliable.

## 4. URL attributes need URL validation

React's escaping of attribute characters is not the same as validating a URL scheme or destination. A value can be syntactically quoted and still point somewhere your application should not send the visitor.

For a link that accepts a user-entered web address, parse it and explicitly allow the schemes your feature supports:

    function getSafeWebUrl(value) {
      const trimmed = value.trim();

      if (!trimmed) {
        return null;
      }

      try {
        const url = new URL(trimmed, window.location.origin);

        if (url.protocol !== 'https:' && url.protocol !== 'http:') {
          return null;
        }

        if (url.username || url.password) {
          return null;
        }

        return url.href;
      } catch {
        return null;
      }
    }

This example accepts relative paths by resolving them against the current origin, and accepts absolute HTTP or HTTPS URLs. If the feature should only navigate within your application, additionally require url.origin to equal window.location.origin. If it should link only to partner sites, compare the parsed hostname against an explicit allowlist.

Allowing HTTP and HTTPS only rejects other schemes, but it does not prove that an external site is trustworthy. Display the destination clearly and choose an allowlist when the feature has a known set of valid destinations.

Avoid building a complete URL by concatenating untrusted strings. Use URL and URLSearchParams for structured construction:

    const searchUrl = new URL('/search', window.location.origin);
    searchUrl.searchParams.set('q', userQuery);

    const href = searchUrl.href;

URLSearchParams encodes the parameter value for its URL component. It does not validate an entire URL supplied by a user.

## 5. Redirects require a destination policy

A redirect parameter such as next, returnTo, or redirect can be abused if the application accepts any destination. A user may be sent to a deceptive site after trusting your application's link.

For an internal redirect, accept a known route or a relative path that cannot change the origin. For a small set of destinations, map an identifier to a route you control:

    const destinations = {
      dashboard: '/dashboard',
      settings: '/settings',
    };

    const destination = destinations[requestedPage] ?? '/';

Do not treat a valid URL parse as sufficient redirect authorization. The allowed destination is a product rule, so encode that rule explicitly.

## 6. Keep event behavior in code

Do not turn strings into event-handler code or JavaScript:

    // Do not interpret data as source code.
    eval(userInput);

    // Do not place user data in an HTML event-handler string.
    const markup = '<button onclick="' + userInput + '">Open</button>';

Attach a function written by the application and pass user input as an ordinary value:

    function handleSearch() {
      runSearch(query);
    }

    return <button type="button" onClick={handleSearch}>Search</button>;

Avoid eval and similar code-execution APIs for application data. A string from a form or URL should never become a program.

## 7. Validate user input for the feature

Validate input according to the field's purpose. For example, a URL field can require a valid web URL, a quantity can require an integer in a product-defined range, and a username can follow the application's documented format.

Client-side validation improves feedback and prevents accidental invalid submissions. It is not a security boundary. A user can bypass the interface and send a request directly, so the server must validate the data again before storing it or acting on it.

Do not rely on a browser input type alone. An input with type="url" helps the user, but server-side validation and destination policy are still needed.

## 8. Do not confuse encoding with validation or authorization

These controls answer different questions:

- Output handling asks how to display or use a value safely in a particular context.
- Validation asks whether a value follows the feature's accepted shape and limits.
- Authorization asks whether this user may perform this action on this resource.

A string can be safely rendered but still violate business rules. A valid identifier can refer to a record the current user is not allowed to access. The server must enforce authorization for protected data and actions.

Never rely on a hidden button, disabled control, or client-side route guard as the only access control.

## 9. Treat browser storage and API data as untrusted

Local storage, session storage, URL state, and client caches can be changed by the user or by a prior script running in the page. Validate stored values when reading them. Do not store secrets there merely because the data is not visible in the interface.

API responses also need sensible shape checks before the interface uses them. A TypeScript type or PropTypes declaration helps development but does not validate data received at runtime. Consider schema validation at a trust boundary, and still render text and URLs safely afterward.

## 10. Safer link behavior

When opening an external destination in a new tab, use an ordinary anchor and make the relationship explicit:

    <a href={safeUrl} target="_blank" rel="noopener noreferrer">
      Open destination
    </a>

Use a normal navigation link when possible instead of assigning window.location from an arbitrary string. Do not use untrusted values as script URLs, iframe sources, CSS resource URLs, or other active resource destinations without a separate, narrow policy.

## 11. Defense in depth

Application code should use safe rendering, URL rules, server validation, and authorization. Additional controls can reduce impact:

- A Content Security Policy can restrict where scripts and other resources load from.
- Secure cookie settings can limit some ways browser scripts interact with session cookies.
- Dependency updates help address known defects in the framework and packages.
- Logging and monitoring can help identify suspicious behavior.

These controls support secure code; they do not make unsafe HTML insertion or missing authorization safe. A Content Security Policy is not a substitute for rendering text correctly or validating URLs.

## 12. A practical security review

For each value that enters a component, ask:

1. Where did it come from, and could a user or external system change it?
2. Is the UI treating it as plain text, a URL, HTML, a redirect, CSS, or code?
3. Does the framework's normal rendering handle this exact context?
4. Does this context need a scheme, origin, host, format, or length allowlist?
5. Does a server endpoint repeat validation and enforce authorization?
6. Is any escape hatch such as dangerouslySetInnerHTML, eval, or direct DOM insertion involved?
7. Can the intended behavior be tested with ordinary text, malformed input, and disallowed schemes?

Review data flow from its source to its final use. Filtering input once at the start is not enough when the value may later be used in a different context.

## Common mistakes

- Assuming React makes every value safe wherever it is used.
- Rendering plain text through dangerouslySetInnerHTML.
- Assuming attribute escaping makes an href destination safe.
- Allowing any redirect destination because it parses as a URL.
- Building query strings through manual concatenation.
- Treating client-side validation as server-side protection.
- Assuming data from an API, database, or local storage is automatically trusted.
- Trying to remove dangerous markup with a regular expression.
- Using eval or string-built event handlers for application data.
- Treating Content Security Policy as a replacement for safe rendering and authorization.

## Key takeaways

- Render ordinary untrusted strings as React children so they remain text.
- React's normal rendering does not validate URL schemes, redirect destinations, business rules, or permissions.
- Parse URLs and enforce an explicit protocol, origin, host, or route policy appropriate to the feature.
- Use URL and URLSearchParams rather than manual query-string concatenation.
- Avoid raw HTML insertion and never convert user data into executable code.
- Validate on the server and authorize every protected operation.
- Treat browser storage, API responses, and database values as potentially untrusted.
