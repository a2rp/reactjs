# HTML Injection, Sanitization, and Trusted Types - Q&A

## HTML injection and React

### 1. What is HTML injection?

HTML injection occurs when data that should remain data reaches a browser API that parses it as markup. The inserted markup may change the page structure or add dangerous behavior, including cross-site scripting.

### 2. What is a browser injection sink?

An injection sink is an API that interprets a string as markup, script, or another active format. Examples include innerHTML, insertAdjacentHTML, document.write, and script execution APIs.

### 3. Does React sanitize dangerouslySetInnerHTML?

No. dangerouslySetInnerHTML passes markup to the browser for parsing. React warns that this is dangerous and expects the application to supply trusted and sanitized content.

### 4. When should I use dangerouslySetInnerHTML?

Only when the feature intentionally needs to render HTML, such as a reviewed rich-text or Markdown preview. If the value is plain text, render it as a normal React child.

### 5. Can a database value be passed directly to dangerouslySetInnerHTML?

Not safely by default. A database may contain user-authored or imported markup. Sanitize the final HTML with a maintained sanitizer and a policy appropriate for the feature before inserting it.

### 6. What happens when a string is rendered as a normal React child?

React treats it as text rather than parsing it as HTML. This is the preferred way to show a plain-text comment or label.

### 7. What is the difference between output escaping and sanitization?

Escaping makes characters display literally as text in a particular context. Sanitization parses markup and removes content that is not allowed while retaining permitted formatting. Use escaping for plain text and sanitization only when a feature needs HTML.

### 8. Can I use regular expressions to sanitize HTML?

No. HTML parsing rules and browser behavior are too complex for a reliable regular-expression filter. Use a maintained HTML sanitizer with an explicit policy.

## Sanitization policy

### 9. What is an allowlist sanitizer?

It defines which markup elements and attributes are permitted, then removes content outside that policy. A narrow allowlist is easier to review than trying to enumerate every dangerous string.

### 10. Which sanitizer does the lesson use?

DOMPurify is the example. It is a maintained sanitizer designed for HTML. Keep it updated and use its documented configuration for the installed version.

### 11. Should every feature use the same allowlist?

Not necessarily. A comment field might need paragraphs and emphasis, while a documentation preview might need code blocks and lists. Define the smallest useful set for each feature.

### 12. Is DOMPurify's default output always the right product policy?

Not automatically. Its default sanitizer policy is designed to remove unsafe markup, but a product may need a narrower subset. Review allowed elements, attributes, URL behavior, and feature requirements.

### 13. Should a user-authored rich-text feature allow script tags or event-handler attributes?

No. Those allow active code execution in the page and should not be allowed in user-authored rich text.

### 14. Does allowing an anchor tag mean every href destination is appropriate?

No. The sanitizer may remove unsafe URL schemes, but the product may require a stricter rule for destinations. Apply an explicit link policy such as permitted schemes, same-origin rules, or an allowed host set.

### 15. Does HTML sanitization make CSS safe?

No. CSS has its own parsing and resource-loading behavior. Avoid user-authored styles unless the feature has a separate, carefully constrained CSS policy.

### 16. Does HTML sanitization make scripts safe?

No. HTML sanitization is not a way to make arbitrary JavaScript executable. Do not pass user input to eval, Function, event-handler strings, or script URL sinks.

### 17. When should sanitation happen?

Sanitize after the transformations that generate or modify the HTML and before it reaches the intended HTML sink. Keep the operation close to the code that produces the final markup.

### 18. Can I alter sanitized HTML after sanitation?

Avoid doing so. String concatenation, templating, or later mutations can add unsafe content and undo the sanitizer's work. If content changes, reassess and sanitize the final result for its context.

### 19. Should I store sanitized HTML or the original source?

That depends on whether the product needs to let the author edit the original source. If storing rendered output, keep track of the policy that produced it and re-sanitize when the library or allowlist changes.

### 20. Does sanitization replace server-side validation or authorization?

No. Sanitization constrains HTML. Server validation checks business rules and input shape. Authorization decides whether a user may perform an action or access a resource. These are separate controls.

## React and Trusted Types

### 21. What does Trusted Types add?

Trusted Types provides browser types and policies that identify data transformed for a class of injection sink. With CSP enforcement in supported browsers, covered sinks can reject ordinary strings.

### 22. Does Trusted Types sanitize input by itself?

No. The application defines a policy and its transformation. For HTML, that transformation commonly calls a sanitizer. Trusted Types is a control around data flow, not a built-in sanitizer.

### 23. What is TrustedHTML?

TrustedHTML is a browser type for a value intended for an HTML injection sink after it has passed through a Trusted Types policy. Whether the value is actually safe depends on the policy that created it.

### 24. Can a policy return its input unchanged?

It can, but that defeats the protection for untrusted HTML. A policy should apply the reviewed transformation, such as calling DOMPurify with the correct options.

### 25. What should createHTML return when it calls DOMPurify?

The callback should return a string for the policy to wrap as TrustedHTML. With DOMPurify, set RETURN_TRUSTED_TYPE to false inside the createHTML callback; the outer policy creates the TrustedHTML value.

### 26. Can React receive a TrustedHTML value in dangerouslySetInnerHTML?

Current React documentation supports a raw HTML string or TrustedHTML value for the __html property. Still sanitize the value before passing it to React, and test the exact React version and CSP setup used by the application.

### 27. If a policy produces TrustedHTML, is that enough to trust it?

No. Review the policy implementation. An unsafe policy can create a TrustedHTML value from dangerous input. The type records that a policy ran; it does not certify the policy's logic.

## CSP and enforcement

### 28. Does creating a Trusted Types policy enforce its use everywhere?

No. The policy makes Trusted Types values, but CSP enforcement is what requires trusted values at covered injection sinks in supporting browsers.

### 29. What does require-trusted-types-for script do?

When supported and enforced by the site's CSP, it requires Trusted Types for covered injection sinks. Passing an ordinary string to a covered sink can cause a TypeError unless a permitted default policy handles it.

### 30. What does the trusted-types CSP directive do?

It restricts which named Trusted Types policies code may create. Keep the allowlist limited to policies the application and required dependencies actually use.

### 31. Why might a sanitizer need its own Trusted Types policy name?

A sanitizer may use HTML parsing internally and may create a policy to support that process. The CSP allowlist must match the policies required by the installed sanitizer version and configuration.

### 32. Should I copy a CSP header without testing it?

No. Inventory the policies used by the application and dependencies, test a report-only rollout where available, and verify the production build before enforcing. A missing policy name can break content rendering; an overly broad policy can weaken the boundary.

### 33. Does Trusted Types work in every browser?

Browser support varies. Check the browsers your application supports. Keep sanitization in the code path even where Trusted Types enforcement is unavailable.

### 34. Is a fallback string safe when Trusted Types is unsupported?

It can still be sanitized, but it does not get browser sink enforcement in a browser without Trusted Types. The sanitizer remains the protection for the HTML content.

### 35. Does Content Security Policy replace a sanitizer?

No. CSP is defense in depth. A raw HTML feature still needs safe content handling, and Trusted Types enforcement still depends on correctly implemented policies.

### 36. What is a default Trusted Types policy for?

A default policy can help during migration by observing or transforming legacy string assignments to covered sinks. It should not become a permissive production shortcut that silently accepts arbitrary untrusted content.

## Testing and maintenance

### 37. What should I test for a rich HTML feature?

Test intended formatting, disallowed tags and attributes, unsafe URL schemes, malformed markup, and content restored from storage or an API. Confirm that disallowed behavior is removed and intended text remains readable.

### 38. Should sanitizer versions be kept current?

Yes. Sanitizers and browsers evolve, and security bypasses may be discovered. Update the dependency and review release notes and policy changes.

### 39. How should the application handle content that fails sanitation?

Use the sanitized output produced by the library and show a clear result. Do not fall back to displaying the original raw HTML when sanitation throws or a policy is unavailable.

### 40. What is the safest rule for plain text?

Do not enter the HTML pipeline. Render it as normal React children.

### 41. What is the safest rule for rich text?

Define the smallest useful markup policy, sanitize the final HTML with a maintained library, and pass only that result to the HTML sink.

### 42. What is the central rule for Trusted Types?

Create narrowly named policies whose transformations are reviewed, then use CSP enforcement to require those values at covered sinks. Trusted Types makes the path auditable; the policy must still make the content safe.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="112-security-and-untrusted-data.md">112 - Security Basics: Escaping, URLs, and Untrusted Data - Q&amp;A</a></td><td><a href="../lessons/113-html-injection-and-trusted-types.md">Lesson</a></td><td><a href="../examples/113-html-injection-and-trusted-types.md">Example</a></td><td><a href="114-production-builds-and-deployment.md">114 - Production Builds, Configuration, and Deployment - Q&amp;A</a></td></tr>
  </tbody>
</table>
