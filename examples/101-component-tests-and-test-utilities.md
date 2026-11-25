# React Example: Component Tests and React Test Utilities

This example builds a small theme-aware greeting component and tests it with React Testing Library. It shows a provider wrapper, document-level queries, user-event, a prop rerender, unmount, an asynchronous findBy query, explicit cleanup setup, and a focused renderHook example.

The component and its React updates remain real. The tests use a DOM environment for component rendering; that environment does not reproduce every detail of a real browser or assistive technology.

## What this example demonstrates

- Rendering a component with a required context provider.
- Reusing provider setup with a small custom render helper.
- Finding content by role and accessible name through screen.
- Choosing getBy, queryBy, or findBy based on the expected timing.
- Using rerender and unmount only when those lifecycle behaviors are relevant.
- Registering cleanup explicitly when the test runner does not provide global hooks.
- Using renderHook and act only for a Hook that has its own public contract.

## Prerequisites

The application already has React and React DOM installed. Install these development dependencies from the project root:

    npm install --save-dev vitest jsdom @testing-library/react @testing-library/dom @testing-library/user-event

Add a test script to the existing package.json scripts object. Preserve the project's other scripts:

    "scripts": {
      "test": "vitest run"
    }

## File structure

    src/
    ├── ThemeGreeting.tsx
    ├── DelayedNotice.tsx
    ├── useCounter.ts
    └── test/
        ├── renderWithTheme.tsx
        ├── setup.ts
        ├── ThemeGreeting.test.tsx
        ├── DelayedNotice.test.tsx
        └── useCounter.test.ts
    vitest.config.ts

## 1. Create the theme provider and component

Create src/ThemeGreeting.tsx:

    import {
      createContext,
      useContext,
      useState,
      type PropsWithChildren,
    } from 'react';

    export type Theme = 'light' | 'dark';

    const ThemeContext = createContext<Theme | null>(null);

    export function ThemeProvider({
      theme,
      children,
    }: PropsWithChildren<{ theme: Theme }>) {
      return (
        <ThemeContext.Provider value={theme}>
          {children}
        </ThemeContext.Provider>
      );
    }

    type ThemeGreetingProps = {
      name: string;
    };

    export function ThemeGreeting({ name }: ThemeGreetingProps) {
      const theme = useContext(ThemeContext);
      const [viewCount, setViewCount] = useState(0);

      if (theme === null) {
        throw new Error('ThemeGreeting must be rendered inside ThemeProvider.');
      }

      return (
        <section>
          <h1>Hello, {name}</h1>
          <p>Theme: {theme}</p>
          <p>Viewed {viewCount} times</p>
          <button
            type="button"
            onClick={() => setViewCount(count => count + 1)}
          >
            Mark viewed
          </button>
        </section>
      );
    }

The component has a required provider, visible theme information, and a button that updates local state. Its test can observe all of these through the rendered DOM without accessing the context or state variable directly.

## 2. Add a reusable render helper

Create src/test/renderWithTheme.tsx:

    import type { PropsWithChildren, ReactElement } from 'react';
    import { render } from '@testing-library/react';
    import { ThemeProvider, type Theme } from '../ThemeGreeting';

    export function renderWithTheme(
      ui: ReactElement,
      theme: Theme = 'light',
    ) {
      function Wrapper({ children }: PropsWithChildren) {
        return <ThemeProvider theme={theme}>{children}</ThemeProvider>;
      }

      return render(ui, { wrapper: Wrapper });
    }

This helper keeps the required provider setup in one place. A test can override the theme when that value matters. Do not add unrelated global providers to this helper; keep each test's dependencies explicit.

## 3. Configure Vitest and cleanup

Create vitest.config.ts at the project root:

    import { defineConfig } from 'vitest/config';

    export default defineConfig({
      test: {
        environment: 'jsdom',
        setupFiles: ['./src/test/setup.ts'],
      },
    });

Create src/test/setup.ts:

    import { cleanup } from '@testing-library/react';
    import { afterEach } from 'vitest';

    afterEach(() => {
      cleanup();
    });

Vitest does not expose its test hooks as globals by default. Registering cleanup with its imported afterEach ensures each rendered React tree is unmounted and its DOM container is removed after the test. If a project's configuration enables globals and the library already performs cleanup automatically, do not register a duplicate cleanup hook.

## 4. Test the component with screen and user-event

Create src/test/ThemeGreeting.test.tsx:

    import { render, screen } from '@testing-library/react';
    import userEvent from '@testing-library/user-event';
    import { describe, expect, it } from 'vitest';
    import { ThemeGreeting } from '../ThemeGreeting';
    import { renderWithTheme } from './renderWithTheme';

    describe('ThemeGreeting', () => {
      it('renders the heading and the selected theme', () => {
        renderWithTheme(<ThemeGreeting name="Ada" />, 'dark');

        expect(
          screen.getByRole('heading', { name: 'Hello, Ada' }),
        ).toBeTruthy();
        expect(screen.getByText('Theme: dark')).toBeTruthy();
      });

      it('updates visible output after a user activates the button', async () => {
        const user = userEvent.setup();
        renderWithTheme(<ThemeGreeting name="Ada" />);

        expect(screen.getByText('Viewed 0 times')).toBeTruthy();

        await user.click(screen.getByRole('button', { name: 'Mark viewed' }));

        expect(screen.getByText('Viewed 1 times')).toBeTruthy();
      });

      it('responds to a new name when the rendered props change', () => {
        const { rerender } = renderWithTheme(
          <ThemeGreeting name="Ada" />,
          'dark',
        );

        rerender(<ThemeGreeting name="Grace" />);

        expect(
          screen.getByRole('heading', { name: 'Hello, Grace' }),
        ).toBeTruthy();
        expect(screen.getByText('Theme: dark')).toBeTruthy();
        expect(
          screen.queryByRole('heading', { name: 'Hello, Ada' }),
        ).toBeNull();
      });

      it('removes the rendered heading when explicitly unmounted', () => {
        const { unmount } = renderWithTheme(
          <ThemeGreeting name="Ada" />,
        );

        expect(
          screen.getByRole('heading', { name: 'Hello, Ada' }),
        ).toBeTruthy();

        unmount();

        expect(
          screen.queryByRole('heading', { name: 'Hello, Ada' }),
        ).toBeNull();
      });
    });

The first test uses role and accessible name for a heading. The interaction test creates a user session before rendering, activates the real button, and checks visible output. The prop test demonstrates rerender; in an application, test a real parent instead when its state change is the behavior that matters. The unmount test is useful when teardown is part of the contract; routine tests can rely on configured cleanup.

## 5. Use findBy when content appears later

Create src/DelayedNotice.tsx:

    import { useEffect, useState } from 'react';

    export function DelayedNotice() {
      const [ready, setReady] = useState(false);

      useEffect(() => {
        const timer = window.setTimeout(() => {
          setReady(true);
        }, 20);

        return () => {
          window.clearTimeout(timer);
        };
      }, []);

      if (!ready) {
        return null;
      }

      return <p role="status">Preferences are ready.</p>;
    }

Create src/test/DelayedNotice.test.tsx:

    import { render, screen } from '@testing-library/react';
    import { describe, expect, it } from 'vitest';
    import { DelayedNotice } from '../DelayedNotice';

    describe('DelayedNotice', () => {
      it('waits for the status to appear', async () => {
        render(<DelayedNotice />);

        expect(screen.queryByRole('status')).toBeNull();

        const status = await screen.findByRole('status');

        expect(status.textContent).toBe('Preferences are ready.');
      });
    });

The notice is initially absent, so queryByRole is appropriate for the immediate absence check. It appears after a scheduled update, so findByRole waits for the actual DOM condition. The test uses no arbitrary sleep.

## 6. Use renderHook only when the Hook is the API

For most application Hooks, test a component that consumes the Hook. A hook-library author may also want a focused test of the Hook's own public return contract.

Create src/useCounter.ts:

    import { useState } from 'react';

    export function useCounter(initialCount = 0) {
      const [count, setCount] = useState(initialCount);

      function increment() {
        setCount(current => current + 1);
      }

      return { count, increment };
    }

Create src/test/useCounter.test.ts:

    import { act, renderHook } from '@testing-library/react';
    import { describe, expect, it } from 'vitest';
    import { useCounter } from '../useCounter';

    describe('useCounter', () => {
      it('increments the value exposed by its public Hook API', async () => {
        const { result } = renderHook(() => useCounter(2));

        expect(result.current.count).toBe(2);

        await act(async () => {
          result.current.increment();
        });

        expect(result.current.count).toBe(3);
      });
    });

This test calls the Hook through renderHook rather than calling it as an ordinary function. It uses act because it invokes a state-changing function directly instead of using a rendered control. If the same Hook is used by an application component, also test the visible effect through that component.

## 7. Run the suite

Run the test script from the project root:

    npm test

The component tests use Testing Library's wrapped render and user-event helpers, so they do not manually wrap ordinary rendering or clicking in act. The Hook test uses act for its direct update. If your React and testing library versions differ from this setup, use APIs supported by the versions installed in the project.

## Review checklist

- Required providers are included through a small wrapper.
- Tests query user-facing content through screen and semantic queries.
- getBy, queryBy, and findBy match the expected timing.
- User actions use a per-test user-event session.
- Rerender and unmount appear only where those behaviors matter.
- The test runner has an explicit cleanup strategy.
- renderHook is used only for a Hook-level public contract.
- Direct Hook updates are wrapped in async act.
- The tests avoid private component state and raw CSS selectors.
