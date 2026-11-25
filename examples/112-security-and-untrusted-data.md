# Safe Text and URL Handling — Complete Example

This example is a small Vite React feature for previewing a user-written message and checking a destination URL before rendering a link. It demonstrates three separate rules:

- Display user text as a normal React child.
- Parse a URL and allow only HTTP or HTTPS.
- Build query strings with URLSearchParams.

The URL policy here allows any HTTP or HTTPS destination. That restricts schemes, but it does not certify that an external site is trustworthy. For production features with known destinations, use a host allowlist or require same-origin navigation.

## 1. Files

In an existing Vite React project, add:

    src/App.jsx
    src/App.css
    src/url-policy.js

Keep the existing Vite entry point and package setup.

## 2. Create src/url-policy.js

    const ALLOWED_PROTOCOLS = new Set(['http:', 'https:']);

    export function getSafeWebUrl(value, baseUrl) {
      if (typeof value !== 'string') {
        return null;
      }

      const trimmed = value.trim();

      if (!trimmed) {
        return null;
      }

      try {
        const base = new URL(baseUrl);
        const url = new URL(trimmed, base);

        if (!ALLOWED_PROTOCOLS.has(url.protocol)) {
          return null;
        }

        if (url.username || url.password) {
          return null;
        }

        return {
          href: url.href,
          hostname: url.hostname,
          isExternal: url.origin !== base.origin,
        };
      } catch {
        return null;
      }
    }

    export function buildSearchUrl(query, baseUrl) {
      const url = new URL('/search', baseUrl);
      url.searchParams.set('q', query);
      return url.href;
    }

The baseUrl is supplied by the application, not by the user. URL resolves relative paths against that trusted base. The parsed protocol is checked explicitly. This helper rejects malformed input and URLs with embedded username or password values.

If a feature should permit only internal destinations, add a policy check that url.origin equals the expected application origin. If it should permit only partner sites, compare url.hostname against an explicit allowlist.

## 3. Create src/App.jsx

    import { useState } from 'react';
    import './App.css';
    import { buildSearchUrl, getSafeWebUrl } from './url-policy.js';

    export default function App() {
      const [message, setMessage] = useState(
        'Try typing text that looks like <strong>HTML</strong>.'
      );
      const [rawUrl, setRawUrl] = useState('');
      const [checkedUrl, setCheckedUrl] = useState(null);
      const [hasCheckedUrl, setHasCheckedUrl] = useState(false);
      const [query, setQuery] = useState('');

      function handleUrlSubmit(event) {
        event.preventDefault();
        setCheckedUrl(rawUrl);
        setHasCheckedUrl(true);
      }

      const destination = hasCheckedUrl
        ? getSafeWebUrl(checkedUrl, window.location.origin)
        : null;

      const searchPreview = buildSearchUrl(query, window.location.origin);

      return (
        <main className="page-shell">
          <header className="page-header">
            <p className="eyebrow">React security example</p>
            <h1>Text and URL handling</h1>
            <p>
              User text stays text. A destination is parsed and checked
              before it becomes a link.
            </p>
          </header>

          <section className="panel" aria-labelledby="message-title">
            <h2 id="message-title">Plain-text message</h2>

            <label className="field-label" htmlFor="message">
              Write a message
            </label>
            <textarea
              id="message"
              value={message}
              maxLength={400}
              rows={4}
              onChange={(event) => setMessage(event.target.value)}
            />

            <div className="preview">
              <h3>Preview</h3>
              <p>{message || 'Your message preview will appear here.'}</p>
            </div>

            <p className="help-text">
              Text that resembles HTML is rendered as text because it is
              passed as a normal React child.
            </p>
          </section>

          <section className="panel" aria-labelledby="url-title">
            <h2 id="url-title">Check a web destination</h2>

            <form onSubmit={handleUrlSubmit}>
              <label className="field-label" htmlFor="destination">
                URL or relative path
              </label>
              <input
                id="destination"
                name="destination"
                type="text"
                inputMode="url"
                autoComplete="url"
                value={rawUrl}
                onChange={(event) => setRawUrl(event.target.value)}
                placeholder="https://example.com or /help"
                aria-describedby="url-help"
              />

              <p id="url-help" className="help-text">
                This demo permits HTTP and HTTPS URLs. Other schemes are
                rejected.
              </p>

              <button type="submit">Check destination</button>
            </form>

            {hasCheckedUrl && (
              <div className="result" aria-live="polite">
                {destination ? (
                  <>
                    <p>
                      Allowed web URL for this demo:
                      {' '}
                      <strong>{destination.hostname}</strong>
                    </p>
                    <a
                      href={destination.href}
                      target={destination.isExternal ? '_blank' : undefined}
                      rel={
                        destination.isExternal
                          ? 'noopener noreferrer'
                          : undefined
                      }
                    >
                      Open checked destination
                    </a>
                    <p className="help-text">
                      An allowed scheme does not prove that an external site
                      is trustworthy. Use an origin or host allowlist when
                      the feature needs a limited set of destinations.
                    </p>
                  </>
                ) : (
                  <p className="error" role="alert">
                    No link was created. Enter a valid HTTP or HTTPS URL, or
                    a relative path.
                  </p>
                )}
              </div>
            )}
          </section>

          <section className="panel" aria-labelledby="search-title">
            <h2 id="search-title">Build a search URL</h2>

            <label className="field-label" htmlFor="query">
              Search query
            </label>
            <input
              id="query"
              type="search"
              value={query}
              onChange={(event) => setQuery(event.target.value)}
            />

            <p className="help-text">
              The query is added as a URL parameter, not concatenated into
              markup.
            </p>
            <output className="url-output">{searchPreview}</output>
          </section>
        </main>
      );
    }

The message preview never parses the message as HTML. The URL field is checked only when the form is submitted. If parsing or policy checking fails, the application shows an error and renders no anchor.

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
    input,
    textarea {
      font: inherit;
    }

    button {
      min-height: 2.75rem;
      padding: 0.65rem 1rem;
      border: 0;
      border-radius: 0.5rem;
      color: white;
      background: #174ea6;
      cursor: pointer;
    }

    button:hover {
      background: #123e85;
    }

    button:focus-visible,
    a:focus-visible,
    input:focus-visible,
    textarea:focus-visible {
      outline: 3px solid #f1a900;
      outline-offset: 3px;
    }

    a {
      color: #174ea6;
      overflow-wrap: anywhere;
    }

    .page-shell {
      width: min(100% - 2rem, 48rem);
      margin-inline: auto;
      padding-block: 3rem;
    }

    .page-header {
      margin-bottom: 1.5rem;
    }

    .page-header h1,
    .panel h2,
    .preview h3 {
      margin-top: 0;
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
      margin-bottom: 0;
      font-size: 1.2rem;
    }

    .field-label {
      margin-top: 0.3rem;
      font-weight: 650;
    }

    input,
    textarea {
      width: 100%;
      padding: 0.7rem 0.8rem;
      border: 1px solid #aeb9ca;
      border-radius: 0.45rem;
      color: inherit;
      background: white;
    }

    textarea {
      resize: vertical;
    }

    .preview,
    .result {
      padding: 1rem;
      border-radius: 0.55rem;
      background: #f2f5fa;
    }

    .preview h3 {
      margin-bottom: 0.4rem;
      font-size: 0.95rem;
    }

    .preview p,
    .result p {
      margin-block: 0.35rem;
      overflow-wrap: anywhere;
    }

    .help-text {
      margin: 0;
      color: #4b5c78;
      font-size: 0.9rem;
      line-height: 1.5;
    }

    .error {
      color: #a12222;
      font-weight: 650;
    }

    .url-output {
      display: block;
      padding: 0.8rem;
      border: 1px solid #d5dce8;
      border-radius: 0.45rem;
      background: #f8faff;
      font-family: ui-monospace, monospace;
      overflow-wrap: anywhere;
    }

## 5. Try these inputs

For the URL field, submit each value and observe whether a link is created:

| Input | Expected result |
| --- | --- |
| /help | Accepted as a relative path on the current origin |
| https://example.com/docs | Accepted as an HTTPS URL |
| http://example.com | Accepted by this demo's HTTP/HTTPS policy |
| javascript:alert(1) | Rejected because its protocol is not HTTP or HTTPS |
| data:text/html,hello | Rejected because its protocol is not HTTP or HTTPS |
| not a valid URL [ | Rejected if the browser URL parser cannot parse it |
| An empty value | Rejected |

The text preview accepts angle brackets and other ordinary characters. They remain visible text; they are not converted into HTML elements.

For the search query, try spaces, ampersands, and question marks. The output should keep the input within the q parameter because URLSearchParams encodes it as a parameter value.

## 6. Make the policy match the product

This example permits arbitrary HTTP and HTTPS hosts for demonstration. Change the policy before using this pattern in an application with narrower requirements:

- For internal navigation only, reject a parsed URL whose origin differs from the app's expected origin.
- For a partner directory, allow only explicit, normalized hostnames.
- For a redirect, prefer a mapping from known identifiers to application-owned routes.
- For URLs in resource attributes such as images or frames, define a separate policy for allowed sources.
- For rich text, use a maintained HTML sanitizer instead of this URL helper.

Do not use a URL that passed this demo's protocol check as proof that a user is authorized to access a resource. Authorization belongs on the server.

## 7. Security review

Before shipping a similar feature:

1. Confirm every user-controlled string is rendered as text unless rich HTML is explicitly required.
2. Confirm rejected URLs produce no active link or redirect.
3. Decide whether the feature allows any HTTP/HTTPS host, only HTTPS, the same origin, or a fixed allowlist.
4. Enforce the same data validation and permissions on the server.
5. Keep user input out of HTML strings, event-handler strings, eval, and other code execution paths.
6. Review any use of dangerouslySetInnerHTML separately; this example does not need it.

## Key points

- React children are suitable for ordinary text.
- Escaped attributes do not establish a URL destination policy.
- URL parsing, scheme checks, and origin or host rules are separate decisions.
- URLSearchParams is for structured query construction, not whole-URL trust.
- External HTTP or HTTPS destinations can still be deceptive.
- Client checks improve the interface; the server must enforce validation and authorization.
