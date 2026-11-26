# HTML Injection, Sanitization, and Trusted Types

## Learning objectives

After this lesson, you should be able to:

- Explain why raw HTML insertion creates an injection risk.
- Choose between rendering plain text and supporting user-authored rich HTML.
- Describe how an allowlist HTML sanitizer reduces risk.
- Use React's dangerouslySetInnerHTML only with intentionally sanitized output.
- Explain what Trusted Types policies and Content Security Policy enforcement do.
- Identify why a Trusted Types value is not automatically safe.
- Review the path from untrusted input to an HTML injection sink.

## 1. HTML injection is a context problem

A browser interprets HTML markup. If an application sends untrusted text to an API that parses HTML, an attacker may be able to add elements, attributes, links, or other active behavior to the page. This class of issue includes cross-site scripting (XSS).

The source might be a comment form, a Markdown field, imported content, a database record, or an API response. Storing the value in a database does not remove the risk. The important question is whether untrusted data can reach a browser parsing or execution context.

A value that is safe to show as ordinary text is not automatically safe to interpret as markup. Use the simplest rendering mode that provides the feature users need.

## 2. Prefer ordinary React children for plain text

For names, comments, labels, and text that should not contain rendered markup, use a normal React child:

    function Comment({ text }) {
      return <p>{text}</p>;
    }

React renders this string as text. A value containing angle brackets remains visible text rather than becoming an HTML element.

Do not convert a plain-text string into HTML just to display it:

    // Avoid this for plain text.
    function Comment({ text }) {
      return <div dangerouslySetInnerHTML={{ __html: text }} />;
    }

The raw HTML path is unnecessary here and bypasses React's normal text rendering.

## 3. What dangerouslySetInnerHTML changes

React's dangerouslySetInnerHTML prop sets the element's HTML content using the browser's HTML parser. It accepts an object whose __html value is a raw string or, where supported, a TrustedHTML value.

    const markup = { __html: '<p>Formatted content</p>' };

    return <article dangerouslySetInnerHTML={markup} />;

This prop is an escape hatch. If its value is based on untrusted input and has not been sanitized for HTML, it can introduce XSS. React does not sanitize the markup for you.

Create the markup object close to the code that generates the final sanitized HTML. This makes the trust boundary visible during review. Do not pass both children and dangerouslySetInnerHTML to the same element.

## 4. Decide whether the feature needs HTML

If a field only needs readable text, render a React child. If it needs formatting, consider whether a structured format such as Markdown is enough. A Markdown renderer still needs a safe policy for raw HTML, generated links, and other output features.

If users truly need to author HTML, define the allowed subset. For example, a comment feature might allow paragraphs, emphasis, lists, code, and links while excluding scripts, forms, embedded frames, event-handler attributes, and styles.

A smaller feature surface is easier to explain and maintain. Avoid enabling tags, attributes, SVG, CSS, or URL schemes just because a sanitizer makes them available.

## 5. Sanitize HTML with a maintained library

HTML sanitization parses the markup and removes elements or attributes that are not allowed by the sanitizer's policy. Use a maintained library with an explicit allowlist appropriate for the product, such as DOMPurify:

    import DOMPurify from 'dompurify';

    const allowedMarkup = DOMPurify.sanitize(untrustedHtml, {
      ALLOWED_TAGS: [
        'p',
        'br',
        'strong',
        'em',
        'ul',
        'ol',
        'li',
        'blockquote',
        'a',
        'code',
        'pre',
      ],
      ALLOWED_ATTR: ['href', 'title'],
    });

    const markup = { __html: allowedMarkup };

    return <article dangerouslySetInnerHTML={markup} />;

The sanitizer should be configured for the feature and kept up to date. Its default policy may be a good starting point, but product requirements can justify a narrower allowlist. Do not allow script execution or event-handler attributes in user-authored content.

Sanitization is different from HTML escaping. Escaping makes markup characters display as text. Sanitization retains intended, permitted formatting while removing content outside the policy.

## 6. Keep the sanitized value intact

Sanitize after transformations that generate or change HTML, and pass that result to the intended HTML sink. If code modifies sanitized markup afterward, it can undo the protection.

Avoid sending sanitized markup through a later template engine, string concatenation, parser, or library that can add unsafe markup. If content is transformed, re-evaluate the final output for the actual context and sanitize again where needed.

Do not write a sanitizer with regular expressions. HTML has parsing rules that cannot be safely covered by a few text replacements.

## 7. Sanitization does not make all contexts safe

An HTML sanitizer is designed to constrain HTML. It does not authorize a user, validate business rules, make arbitrary JavaScript safe, or automatically make every URL, CSS, script, or template context safe.

For links and resource attributes, ensure the sanitizer's URL handling and the application's product policy agree. A rich text link may need to allow only certain schemes. A link that points to any external HTTPS site may still be deceptive. A user-authored style block or embedded frame can greatly expand the policy's complexity; omit such features unless they are necessary and separately reviewed.

Do not interpret sanitized HTML as JavaScript. Do not pass raw input to eval, Function, script URLs, or event-handler strings.

## 8. What Trusted Types provides

Trusted Types is a browser API that helps applications control data passed to DOM injection sinks. The browser distinguishes values such as TrustedHTML from ordinary strings. A Trusted Types policy contains the transformation that creates an accepted value.

For HTML, the transformation should sanitize input or otherwise apply the application's reviewed HTML policy. A TrustedHTML value represents that the policy was used; the type itself does not prove that the policy is safe.

A policy that simply returns the input unchanged defeats the purpose:

    // Unsafe policy: it labels input without sanitizing it.
    const unsafePolicy = trustedTypes.createPolicy('app-html', {
      createHTML: (input) => input,
    });

Only create policies that apply the required transformation. Keep policy creation centralized and allow only the intended policy names.

## 9. Trusted Types with a sanitizer

A sanitizer can be used inside a Trusted Types policy. When a policy's createHTML callback calls DOMPurify, configure that call to return a normal string because createHTML itself is responsible for producing the TrustedHTML value:

    const appHtmlPolicy = trustedTypes.createPolicy('app-html', {
      createHTML: (dirtyHtml) =>
        DOMPurify.sanitize(dirtyHtml, {
          ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a'],
          ALLOWED_ATTR: ['href', 'title'],
          RETURN_TRUSTED_TYPE: false,
        }),
    });

    const trustedMarkup = appHtmlPolicy.createHTML(untrustedHtml);

    return (
      <article
        dangerouslySetInnerHTML={{ __html: trustedMarkup }}
      />
    );

When the Trusted Types API is unavailable, the application still needs the sanitizer. A compatibility path can return sanitized strings, but it does not provide browser sink enforcement in that browser.

Do not create multiple competing policies casually. Centralize policy creation, define clear ownership, and ensure the CSP permits only the policies the app and its dependencies require.

## 10. Enforce Trusted Types with CSP

Creating a policy alone does not force every part of the application to use it. A Content Security Policy can require Trusted Types for supported injection sinks and allowlist policy names.

A policy may include directives in this form:

    require-trusted-types-for 'script';
    trusted-types app-html dompurify;

These directives are delivered as part of the site's Content-Security-Policy response header. The allowed names must match the policies the application and its libraries actually create. Test the production build and all relevant dependencies before enforcing the policy.

Roll out enforcement carefully. A report-only policy can help discover existing sink usage before enforcement, where the hosting setup supports it. Check browser support and application compatibility because not every browser enforces Trusted Types.

## 11. Trusted Types is not a sanitizer

Trusted Types does not provide an HTML sanitizer or decide what your application considers acceptable. It verifies that values passed to covered sinks have gone through an allowed policy when enforcement is active.

A permissive or identity policy can still pass dangerous input. A sanitizer bug or a policy mistake can still create unsafe markup. Trusted Types is a way to centralize and enforce a transformation boundary, not a substitute for a correct transformation.

## 12. Server rendering and content storage

Sanitize at a clearly defined trust boundary and store the original content only when the product needs to preserve it for later editing. If sanitized HTML is stored, record enough context to know which sanitizer policy produced it and re-sanitize when policies or libraries change.

For server-side rendering, use a sanitizer environment supported by the chosen library and keep its DOM implementation current. Do not assume a browser-only sanitizer call can be copied unchanged into a server runtime. Regardless of where sanitation occurs, render only the intended sanitized result and keep authorization on the server.

## 13. A practical review workflow

For every rich content feature:

1. Confirm whether plain text or structured content can meet the need.
2. List permitted HTML elements and attributes before enabling the feature.
3. Identify every input source, including saved records and imported content.
4. Sanitize with a maintained library configured for the output context.
5. Keep the sanitizer call next to generation of the final HTML.
6. Pass only that output to dangerouslySetInnerHTML.
7. Check links and embedded resources against a separate destination policy.
8. If using Trusted Types, centralize policies and test CSP enforcement.
9. Test allowed formatting, rejected markup, unsafe attributes, and malformed input.
10. Revisit stored output when the sanitizer or allowlist changes.

## Common mistakes

- Using dangerouslySetInnerHTML for plain text.
- Assuming React sanitizes HTML supplied to dangerouslySetInnerHTML.
- Treating a TrustedHTML value as automatically safe without reviewing its policy.
- Writing an identity Trusted Types policy that returns input unchanged.
- Using a regular expression as an HTML sanitizer.
- Sanitizing content and then concatenating untrusted strings into it.
- Allowing a broad set of tags, attributes, or URL schemes without a product need.
- Assuming HTML sanitation also validates CSS, script, authorization, or business rules.
- Enforcing Trusted Types without inventorying policy names created by dependencies.
- Assuming Content Security Policy replaces a maintained sanitizer.

## Key takeaways

- Render plain text as normal React children.
- Use dangerouslySetInnerHTML only when the feature intentionally renders HTML.
- Sanitize rich HTML with a maintained library and a narrow, reviewed policy.
- Keep sanitized output unchanged until it reaches its intended HTML sink.
- Trusted Types centralizes transformations and can block strings at covered sinks when CSP enforces it.
- Trusted Types does not sanitize data on its own; the policy must be safe.
- Keep server validation, authorization, URL policy, and CSP as distinct controls.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="112-security-and-untrusted-data.md">112 - Security Basics: Escaping, URLs, and Untrusted Data</a></td><td><a href="../qnas/113-html-injection-and-trusted-types.md">Q&amp;A</a></td><td><a href="../examples/113-html-injection-and-trusted-types.md">Example</a></td><td><a href="114-production-builds-and-deployment.md">114 - Production Builds, Configuration, and Deployment</a></td></tr>
  </tbody>
</table>
