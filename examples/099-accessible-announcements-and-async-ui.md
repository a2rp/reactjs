# React Example: Accessible Async Search with Current-Request Protection

This example builds a small client-side search interface. It does not call a network service; a local function waits briefly and filters sample data so the UI can demonstrate asynchronous states without external dependencies.

The example includes:

- A visible label and form submit control.
- A persistent status region for loading, result counts, and empty results.
- A visible urgent error message when the simulated service fails.
- A results region marked busy while its contents are incomplete.
- A request identifier so a late response cannot replace newer results.
- Cleanup so a completed request does not update an unmounted screen.

## File structure

    src/
    ├── SearchPage.tsx
    └── SearchPage.css

Render SearchPage from the application's entry point. This example uses React with TypeScript.

## 1. Create the search screen

Create src/SearchPage.tsx:

    import { useEffect, useRef, useState, type FormEvent } from 'react';
    import './SearchPage.css';

    type SearchResult = {
      id: string;
      title: string;
      summary: string;
    };

    const catalog: SearchResult[] = [
      {
        id: 'components',
        title: 'React component fundamentals',
        summary: 'Build interfaces from reusable React components.',
      },
      {
        id: 'state',
        title: 'State and UI updates',
        summary: 'Understand how React state updates the rendered interface.',
      },
      {
        id: 'forms',
        title: 'Accessible forms',
        summary: 'Use labels, controls, and useful validation feedback.',
      },
      {
        id: 'async',
        title: 'Asynchronous user interfaces',
        summary: 'Show loading, success, empty, and error states clearly.',
      },
    ];

    async function searchCatalog(term: string): Promise<SearchResult[]> {
      // Even-length terms take longer so the example can demonstrate
      // two requests finishing out of order.
      const delay = term.length % 2 === 0 ? 900 : 350;

      await new Promise<void>(resolve => {
        window.setTimeout(resolve, delay);
      });

      if (term.toLowerCase() === 'error') {
        throw new Error('The local demonstration failed.');
      }

      const normalizedTerm = term.toLowerCase();

      return catalog.filter(result => {
        const searchableText = result.title + ' ' + result.summary;
        return searchableText.toLowerCase().includes(normalizedTerm);
      });
    }

    export default function SearchPage() {
      const [query, setQuery] = useState('');
      const [submittedQuery, setSubmittedQuery] = useState('');
      const [results, setResults] = useState<SearchResult[]>([]);
      const [isLoading, setIsLoading] = useState(false);
      const [statusMessage, setStatusMessage] = useState('');
      const [errorMessage, setErrorMessage] = useState('');
      const latestRequestId = useRef(0);

      useEffect(() => {
        return () => {
          // Ignore any request that finishes after this screen unmounts.
          latestRequestId.current += 1;
        };
      }, []);

      async function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        const term = query.trim();

        if (term.length === 0) {
          setStatusMessage('Enter a word to search.');
          return;
        }

        const requestId = ++latestRequestId.current;

        setSubmittedQuery(term);
        setResults([]);
        setErrorMessage('');
        setIsLoading(true);
        setStatusMessage('Searching for ' + term + '.');

        try {
          const nextResults = await searchCatalog(term);

          if (requestId !== latestRequestId.current) {
            return;
          }

          setResults(nextResults);

          if (nextResults.length === 0) {
            setStatusMessage('No results found for ' + term + '.');
          } else {
            setStatusMessage(
              nextResults.length + ' results found for ' + term + '.',
            );
          }
        } catch {
          if (requestId !== latestRequestId.current) {
            return;
          }

          setErrorMessage('Search could not be completed. Try again.');
          setStatusMessage('');
        } finally {
          if (requestId === latestRequestId.current) {
            setIsLoading(false);
          }
        }
      }

      return (
        <main className="search-page">
          <h1>Help article search</h1>
          <p className="intro">
            Search the local example catalog by entering a word or phrase.
          </p>

          <form className="search-form" onSubmit={handleSubmit}>
            <label htmlFor="search-query">Search articles</label>
            <p className="search-hint" id="search-hint">
              Try a word such as React, state, forms, or async.
              Enter error to see the simulated failure state.
            </p>

            <div className="search-controls">
              <input
                aria-describedby="search-hint"
                autoComplete="off"
                id="search-query"
                name="query"
                onChange={event => setQuery(event.target.value)}
                required
                type="search"
                value={query}
              />
              <button type="submit">Search</button>
            </div>
          </form>

          <p className="status-message" role="status">
            {statusMessage}
          </p>

          {errorMessage ? (
            <p className="error-message" role="alert">
              {errorMessage}
            </p>
          ) : null}

          <section
            aria-busy={isLoading}
            aria-labelledby="results-heading"
            className="results-panel"
            id="search-results"
          >
            <h2 id="results-heading">
              {submittedQuery
                ? 'Results for ' + submittedQuery
                : 'Results'}
            </h2>

            {isLoading ? (
              <p className="loading-copy">The result list is updating.</p>
            ) : null}

            {!isLoading && errorMessage ? (
              <p>The search did not finish. You can change the query and retry.</p>
            ) : null}

            {!isLoading &&
            !errorMessage &&
            submittedQuery &&
            results.length === 0 ? (
              <p>Try a different word or phrase.</p>
            ) : null}

            {!submittedQuery ? (
              <p>Submit a search to see matching articles.</p>
            ) : null}

            {results.length > 0 ? (
              <ul className="result-list">
                {results.map(result => (
                  <li className="result-card" key={result.id}>
                    <h3>{result.title}</h3>
                    <p>{result.summary}</p>
                  </li>
                ))}
              </ul>
            ) : null}
          </section>
        </main>
      );
    }

The status is a persistent element in the page. It announces a short message without taking focus. The results section is marked busy only while the list is being replaced. The error is visible and uses alert semantics because this demonstration reports the failure immediately.

## 2. Style the interface

Create src/SearchPage.css:

    :root {
      color: #172033;
      background: #f4f6fa;
      font-family: system-ui, sans-serif;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
    }

    button:focus-visible,
    input:focus-visible {
      outline: 3px solid #174dbb;
      outline-offset: 3px;
    }

    .search-page {
      width: min(100% - 2rem, 44rem);
      margin: 3rem auto;
    }

    h1 {
      margin-bottom: 0.5rem;
      font-size: clamp(1.8rem, 5vw, 2.4rem);
      line-height: 1.2;
    }

    h2 {
      margin-top: 0;
      font-size: 1.4rem;
    }

    .intro,
    .search-hint {
      color: #43516a;
      line-height: 1.6;
    }

    .search-form {
      display: grid;
      gap: 0.5rem;
      margin-top: 1.5rem;
    }

    .search-form label {
      font-weight: 700;
    }

    .search-hint {
      margin: 0;
    }

    .search-controls {
      display: flex;
      flex-wrap: wrap;
      gap: 0.75rem;
      margin-top: 0.25rem;
    }

    .search-controls input {
      min-height: 2.9rem;
      min-width: 12rem;
      flex: 1;
      padding: 0.65rem 0.75rem;
      border: 1px solid #718097;
      border-radius: 0.4rem;
      background: #ffffff;
      color: #172033;
      font: inherit;
    }

    .search-controls button {
      min-height: 2.9rem;
      padding: 0.65rem 1rem;
      border: 0;
      border-radius: 0.4rem;
      background: #174dbb;
      color: #ffffff;
      cursor: pointer;
      font: inherit;
      font-weight: 700;
    }

    .search-controls button:hover {
      background: #103b95;
    }

    .status-message {
      min-height: 1.5rem;
      margin: 1rem 0;
      color: #263852;
      font-weight: 600;
    }

    .error-message {
      padding: 0.8rem 1rem;
      border-left: 4px solid #a51f2b;
      background: #fff1f1;
      color: #7d1721;
      line-height: 1.5;
    }

    .results-panel {
      min-height: 10rem;
      margin-top: 1rem;
      padding: 1.25rem;
      border: 1px solid #cbd4e1;
      border-radius: 0.7rem;
      background: #ffffff;
    }

    .results-panel[aria-busy="true"] {
      border-color: #718097;
    }

    .loading-copy {
      color: #43516a;
    }

    .result-list {
      display: grid;
      gap: 0.8rem;
      margin: 0;
      padding: 0;
      list-style: none;
    }

    .result-card {
      padding: 1rem;
      border: 1px solid #d5dce6;
      border-radius: 0.5rem;
    }

    .result-card h3 {
      margin: 0 0 0.4rem;
      font-size: 1.05rem;
    }

    .result-card p {
      margin: 0;
      color: #43516a;
      line-height: 1.5;
    }

## 3. Understand the request handling

Each submission gets a new request id. A response updates the screen only if it still belongs to the latest request. The old local timer is not cancelled; its result is simply ignored. A real network client can use cancellation when supported, then still guard state updates against outdated results.

The Search button stays available while a request is pending. This allows the user to submit a newer query. The request id prevents a slow earlier search from replacing the newer answer.

The example marks only the results section busy. Its status message is outside that region, so “Searching…” can be announced while the list is being refreshed. The status region remains mounted while its text changes.

The query error is intentionally simulated by submitting the word error. It shows one visible alert message and a retry instruction. The error state is cleared when the user submits another search.

## 4. Try these behaviors

1. Submit React and observe the loading message followed by matching results.
2. Submit a word that is not in the catalog and inspect the empty state.
3. Submit error and inspect the failure message.
4. Submit dialog, then quickly change the query to React and submit again. The slower earlier request must not replace the latest result.
5. Use only the keyboard to reach the search field and button.
6. Use a screen reader to confirm that the status is announced, focus stays in the form, and results remain available through normal navigation.
7. Repeat with a slow network simulation if adapting the example to a real service.

## 5. Production considerations

- Replace the local catalog with a real data service and present a useful retry path.
- Prefer AbortController or the data client's cancellation API to stop obsolete network work.
- Keep request ordering protection even when cancellation is available; cancellation may race with completion.
- Keep messages concise and do not announce every keystroke.
- Clear aria-busy on every success, empty, and error path.
- Test announcements with the browsers and screen readers used by your audience.
- If the update contains multiple coordinated regions, set aria-busy for the duration of the partial update and expose the complete state when it is ready.

## Key lesson

A visible state change is not necessarily announced. Keep a status region ready, announce meaningful milestones at an appropriate priority, mark incomplete content busy, and ensure the latest user request owns the final result.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="098-keyboard-focus-and-dialogs.md">098 - React Example: A Keyboard-Friendly Native Confirmation Dialog</a></td><td><a href="../lessons/099-accessible-announcements-and-async-ui.md">Lesson</a></td><td><a href="../qnas/099-accessible-announcements-and-async-ui.md">Q&amp;A</a></td><td><a href="100-testing-strategy-and-component-design.md">100 - React Example: Test Strategy and Testable Component Design</a></td></tr>
  </tbody>
</table>
