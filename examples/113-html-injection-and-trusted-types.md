# Sanitized Rich Text with React and Trusted Types — Complete Example

This Vite React example lets a user enter a small amount of rich HTML and previews only the sanitized result. It demonstrates:

- A small DOMPurify allowlist.
- A Trusted Types policy that calls the sanitizer.
- A React dangerouslySetInnerHTML sink that receives only the policy result.
- A sanitized-string fallback in browsers without Trusted Types support.
- A Content Security Policy header pattern for deployment.

Use this pattern only when the feature genuinely needs HTML. For ordinary text, render a normal React child instead.

## 1. Install the sanitizer

In an existing Vite React project, install DOMPurify:

    npm install dompurify

Keep the dependency updated. This example is browser-side and expects the project's normal Vite React setup.

## 2. Create src/safe-html.js

    import DOMPurify from 'dompurify';

    const SANITIZE_OPTIONS = {
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
    };

    function sanitizeToString(dirtyHtml) {
      return DOMPurify.sanitize(dirtyHtml, {
        ...SANITIZE_OPTIONS,
        RETURN_TRUSTED_TYPE: false,
      });
    }

    const appHtmlPolicy =
      typeof globalThis.trustedTypes?.createPolicy === 'function'
        ? globalThis.trustedTypes.createPolicy('app-html', {
            createHTML(dirtyHtml) {
              return sanitizeToString(dirtyHtml);
            },
          })
        : null;

    export function createSafeMarkup(dirtyHtml) {
      const source =
        typeof dirtyHtml === 'string' ? dirtyHtml : String(dirtyHtml ?? '');

      const html = appHtmlPolicy
        ? appHtmlPolicy.createHTML(source)
        : sanitizeToString(source);

      return { __html: html };
    }

The Trusted Types policy returns the string produced by DOMPurify. The browser wraps that return value as TrustedHTML. RETURN_TRUSTED_TYPE is false inside the policy callback because createHTML is the function that creates the TrustedHTML value.

If Trusted Types is unavailable, the same sanitizer still runs and returns a sanitized string. That fallback preserves HTML sanitization but cannot provide browser sink enforcement in an unsupported browser.

Keep the policy centralized and create it once. If policy creation is blocked by CSP or the policy name already exists, this module fails instead of silently sending the original input to the sink.

## 3. Create src/App.jsx

    import { useState } from 'react';
    import { createSafeMarkup } from './safe-html.js';
    import './App.css';

    const initialHtml =
      '<p>Welcome to the <strong>rich text</strong> preview.</p>' +
      '<p>You can include a list:</p>' +
      '<ul><li>Readable content</li><li>Limited formatting</li></ul>';

    export default function App() {
      const [source, setSource] = useState(initialHtml);
      const safeMarkup = createSafeMarkup(source);

      return (
        <main className="page-shell">
          <header className="page-header">
            <p className="eyebrow">React security example</p>
            <h1>Sanitized rich text</h1>
            <p>
              The preview uses a limited HTML policy. The editor input is
              never inserted directly into the page as markup.
            </p>
          </header>

          <section className="panel" aria-labelledby="editor-title">
            <h2 id="editor-title">Rich text source</h2>

            <label className="field-label" htmlFor="html-source">
              Enter HTML for the preview
            </label>
            <textarea
              id="html-source"
              value={source}
              rows={10}
              maxLength={5000}
              spellCheck="false"
              onChange={(event) => setSource(event.target.value)}
              aria-describedby="editor-help"
            />

            <p id="editor-help" className="help-text">
              Allowed formatting includes paragraphs, emphasis, lists,
              quotations, links, and code. Other elements and attributes are
              removed by the sanitizer policy.
            </p>
          </section>

          <section className="panel" aria-labelledby="preview-title">
            <h2 id="preview-title">Sanitized preview</h2>

            <article
              className="rich-preview"
              dangerouslySetInnerHTML={safeMarkup}
            />
          </section>

          <aside className="notice">
            <strong>Security boundary:</strong>
            {' '}
            only createSafeMarkup supplies the value for the raw HTML sink.
            Do not pass source directly to dangerouslySetInnerHTML.
          </aside>
        </main>
      );
    }

The sanitized markup object is created close to its sink. The article intentionally has no React children because dangerouslySetInnerHTML supplies its content.

## 4. Create src/App.css

    * {
      box-sizing: border-box;
    }

    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172033;
      background: #f2f5fa;
      font-synthesis: none;
      text-rendering: optimizeLegibility;
    }

    body {
      min-width: 20rem;
      margin: 0;
    }

    button,
    textarea {
      font: inherit;
    }

    a {
      color: #174ea6;
      overflow-wrap: anywhere;
    }

    a:focus-visible,
    textarea:focus-visible {
      outline: 3px solid #f1a900;
      outline-offset: 3px;
    }

    .page-shell {
      width: min(100% - 2rem, 52rem);
      margin-inline: auto;
      padding-block: 3rem;
    }

    .page-header {
      margin-bottom: 1.5rem;
    }

    .page-header h1 {
      margin-block: 0.35rem;
    }

    .eyebrow {
      margin-bottom: 0.4rem;
      color: #4b5c78;
      font-size: 0.8rem;
      font-weight: 700;
      letter-spacing: 0.08em;
      text-transform: uppercase;
    }

    .panel {
      display: grid;
      gap: 0.9rem;
      margin-block: 1rem;
      padding: 1.25rem;
      border: 1px solid #d5dce8;
      border-radius: 0.8rem;
      background: white;
      box-shadow: 0 0.4rem 1.5rem rgb(20 40 80 / 6%);
    }

    .panel h2 {
      margin: 0;
      font-size: 1.2rem;
    }

    .field-label {
      font-weight: 650;
    }

    textarea {
      width: 100%;
      min-height: 12rem;
      padding: 0.8rem;
      border: 1px solid #aeb9ca;
      border-radius: 0.45rem;
      color: inherit;
      background: #fbfcff;
      font-family: ui-monospace, monospace;
      line-height: 1.5;
      resize: vertical;
    }

    .help-text {
      margin: 0;
      color: #4b5c78;
      font-size: 0.9rem;
      line-height: 1.5;
    }

    .rich-preview {
      min-height: 8rem;
      padding: 1rem;
      border: 1px solid #e0e5ed;
      border-radius: 0.55rem;
      overflow-wrap: anywhere;
    }

    .rich-preview > :first-child {
      margin-top: 0;
    }

    .rich-preview > :last-child {
      margin-bottom: 0;
    }

    .rich-preview pre {
      padding: 0.8rem;
      overflow-x: auto;
      border-radius: 0.4rem;
      background: #f2f5fa;
    }

    .notice {
      padding: 1rem;
      border-left: 0.3rem solid #174ea6;
      border-radius: 0.35rem;
      background: #eaf1ff;
      line-height: 1.5;
    }

## 5. Configure the Content Security Policy

Trusted Types enforcement is delivered through the site's Content-Security-Policy response header. The policy name in the application is app-html. DOMPurify may create its own policy for internal parsing; allow only the names required by the installed version and configuration.

A starting header pattern for this example is:

    Content-Security-Policy: require-trusted-types-for 'script'; trusted-types app-html dompurify;

Check the sanitizer's current policy naming and any suffix configured by your build. If a required policy name is absent, creating the policy or sanitizing markup may fail. Test with the actual production build and hosting configuration.

For an existing application, first inventory its injection sinks and dependencies. A report-only CSP rollout can help identify compatibility issues before enforcement where your host supports it. Do not weaken the policy by allowing arbitrary policy names just to silence an error.

## 6. Try the preview cases

Enter each sample in the editor and inspect the preview:

| Input | Expected behavior |
| --- | --- |
| A paragraph containing strong or emphasized text | Paragraph and permitted formatting remain |
| An unordered list with list items | The allowed list structure remains |
| A script element | The active script element is removed |
| A paragraph with an onclick attribute | The paragraph remains; the event attribute is removed |
| A link with a javascript URL | The unsafe href is removed by the sanitizer |
| An iframe or form | The element is removed because it is not on the allowed tag list |
| A paragraph with a title attribute | The title attribute is allowed by this example |

The preview should never use the raw textarea value directly. Change the allowlist only when the product needs the added feature and the security implications have been reviewed.

## 7. Understand the two protections

DOMPurify transforms the input according to its HTML policy. Trusted Types then marks the result as the output of the app-html policy, and CSP can require that trusted value at covered sinks in supported browsers.

These controls complement each other:

- If the sanitizer is missing, Trusted Types alone does not define safe HTML.
- If CSP enforcement is absent, a code path may still pass an ordinary string to a sink.
- If the policy is overly permissive, it can create a TrustedHTML value from unsafe input.
- If the sanitizer output is changed after sanitation, the resulting markup may no longer meet the policy.

## 8. Production checklist

Before using this pattern:

1. Confirm rich HTML is required; otherwise render a normal React child.
2. Keep the allowlist as small as possible.
3. Keep DOMPurify current and review its configuration when it is upgraded.
4. Confirm app-html and any sanitizer policy names match the enforced CSP.
5. Verify React and the browser support the TrustedHTML path used by the app.
6. Test allowed formatting, disallowed markup, unsafe attributes, and links.
7. Confirm no other component passes raw data to an HTML sink.
8. Sanitize again if content is transformed after the initial sanitation.
9. Validate and authorize data on the server separately.
10. Keep original editable source separate from derived sanitized output when the product needs to edit it later.

## Common mistakes

- Rendering plain text through dangerouslySetInnerHTML.
- Passing raw textarea, API, or database content directly to an HTML sink.
- Treating a TrustedHTML type as proof that its policy is safe.
- Creating an identity policy that returns untrusted markup unchanged.
- Returning TrustedHTML from inside the sanitizer callback instead of returning the string the outer policy expects.
- Mutating or concatenating untrusted data into sanitized output.
- Allowing a broad tag or attribute set without a feature requirement.
- Forgetting a sanitizer's own policy name when configuring CSP.
- Assuming the browser enforces Trusted Types when it does not support the API.
- Using Trusted Types or CSP as a substitute for the sanitizer.

## Key points

- Prefer normal React children for plain text.
- Sanitize intentionally supported rich HTML with a maintained library.
- Pass only the final sanitized or policy-created value to dangerouslySetInnerHTML.
- Trusted Types policy functions must apply a real transformation.
- CSP enforcement can make covered sinks reject ordinary strings in supported browsers.
- Keep sanitizer policy, Trusted Types policy, URL rules, server validation, and authorization explicit.
