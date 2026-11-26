# React Example: Testing an Async Username Form and API Boundary

This example builds a username availability form. It submits through the real form controls, passes the normalized value to an injected API function, shows pending feedback, and handles available, already-taken, and service-failure outcomes.

The tests never call a live server. A deferred promise lets the test keep the request pending long enough to inspect the loading state, then resolve or reject it. The React component and form submission path remain real.

## What this example demonstrates

- Labeled fields and named form data.
- Submission through a user interaction.
- A controlled API boundary instead of a live network call.
- A pending-state assertion before resolving the request.
- Separate tests for success, business rejection, and service failure.
- Awaiting the result instead of adding a fixed delay.

## Prerequisites

The application already has React and React DOM installed. Install these development dependencies from the project root if needed:

    npm install --save-dev vitest jsdom @testing-library/react @testing-library/dom @testing-library/user-event

Use a Vitest config that selects jsdom and registers the cleanup setup. If the project already has the configuration from the component-test lesson, keep that setup and do not duplicate it.

Create or update vitest.config.ts:

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

The package.json scripts should include a test command. Merge this into the existing scripts instead of replacing the project's other commands:

    "scripts": {
      "test": "vitest run"
    }

## File structure

    src/
    ├── UsernameForm.tsx
    └── test/
        ├── setup.ts
        └── UsernameForm.test.tsx
    vitest.config.ts

## 1. Create the form component

Create src/UsernameForm.tsx:

    import { useState, type FormEvent } from 'react';

    export type CheckUsername = (username: string) => Promise<boolean>;

    type UsernameFormProps = {
      checkUsername: CheckUsername;
    };

    export function UsernameForm({ checkUsername }: UsernameFormProps) {
      const [pending, setPending] = useState(false);
      const [message, setMessage] = useState('');

      async function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        if (pending) {
          return;
        }

        const formData = new FormData(event.currentTarget);
        const submittedValue = formData.get('username');
        const username =
          typeof submittedValue === 'string' ? submittedValue.trim() : '';

        if (username.length === 0) {
          setMessage('Enter a username.');
          return;
        }

        setPending(true);
        setMessage('Checking username availability.');

        try {
          const available = await checkUsername(username);
          setMessage(
            available
              ? 'Username is available.'
              : 'That username is already taken.',
          );
        } catch {
          setMessage('Could not check username. Try again.');
        } finally {
          setPending(false);
        }
      }

      return (
        <form onSubmit={handleSubmit}>
          <label htmlFor="username">Username</label>
          <input
            id="username"
            name="username"
            type="text"
            autoComplete="username"
            required
            disabled={pending}
          />

          <button type="submit" disabled={pending}>
            {pending ? 'Checking username...' : 'Check username'}
          </button>

          <p role="status">{message}</p>
        </form>
      );
    }

The API is a required prop so this component does not decide how a request reaches the server. The input has both a label and a name: the label supports accessible interaction, and the name lets FormData include the entered value. The form keeps the pending and result messages visible through one status region.

## 2. Add a deferred promise helper to the test

Create src/test/UsernameForm.test.tsx:

    import { render, screen } from '@testing-library/react';
    import userEvent from '@testing-library/user-event';
    import { describe, expect, it, vi } from 'vitest';
    import { UsernameForm } from '../UsernameForm';

    function createDeferred<T>() {
      let resolve!: (value: T) => void;
      let reject!: (reason?: unknown) => void;

      const promise = new Promise<T>((resolvePromise, rejectPromise) => {
        resolve = value => resolvePromise(value);
        reject = reason => rejectPromise(reason);
      });

      return { promise, resolve, reject };
    }

The helper exposes controls the test can use to finish a request. It does not change the component's state or bypass the form submission.

## 3. Test pending and success states

Add this test inside the file:

    describe('UsernameForm', () => {
      it('shows pending feedback and then reports an available username', async () => {
        const user = userEvent.setup();
        const request = createDeferred<boolean>();
        const checkUsername = vi.fn(() => request.promise);

        render(<UsernameForm checkUsername={checkUsername} />);

        await user.type(
          screen.getByRole('textbox', { name: 'Username' }),
          '  ada  ',
        );
        await user.click(
          screen.getByRole('button', { name: 'Check username' }),
        );

        expect(checkUsername).toHaveBeenCalledTimes(1);
        expect(checkUsername).toHaveBeenCalledWith('ada');
        expect(screen.getByRole('status').textContent).toBe(
          'Checking username availability.',
        );
        expect(
          screen.getByRole('button', { name: 'Checking username...' })
            .disabled,
        ).toBe(true);

        request.resolve(true);

        expect(
          await screen.findByText('Username is available.'),
        ).toBeTruthy();
        expect(
          screen.getByRole('button', { name: 'Check username' }).disabled,
        ).toBe(false);
      });

The promise remains unresolved while the test checks that the normalized username reached the service and the pending state is visible. Only then does the test resolve the request and wait for the success message.

## 4. Test a business rejection

Add this case to the same describe block:

    it('explains when the requested username is taken', async () => {
      const user = userEvent.setup();
      const checkUsername = vi.fn(async (_username: string) => false);

      render(<UsernameForm checkUsername={checkUsername} />);

      await user.type(
        screen.getByRole('textbox', { name: 'Username' }),
        'ada',
      );
      await user.click(
        screen.getByRole('button', { name: 'Check username' }),
      );

      expect(
        await screen.findByText('That username is already taken.'),
      ).toBeTruthy();
      expect(checkUsername).toHaveBeenCalledWith('ada');
    });

A false response is a completed business result, not a thrown exception. The test verifies the correction message shown to the user.

## 5. Test an unexpected service failure

Add a separate failure case:

    it('shows a recovery message when the service request fails', async () => {
      const user = userEvent.setup();
      const request = createDeferred<boolean>();
      const checkUsername = vi.fn(() => request.promise);

      render(<UsernameForm checkUsername={checkUsername} />);

      await user.type(
        screen.getByRole('textbox', { name: 'Username' }),
        'ada',
      );
      await user.click(
        screen.getByRole('button', { name: 'Check username' }),
      );

      expect(screen.getByRole('status').textContent).toBe(
        'Checking username availability.',
      );

      request.reject(new Error('Connection refused'));

      expect(
        await screen.findByText('Could not check username. Try again.'),
      ).toBeTruthy();
      expect(
        screen.getByRole('button', { name: 'Check username' }).disabled,
      ).toBe(false);
    });

The component catches the transport failure and presents a stable recovery message. The test intentionally does not expose the raw exception to the user or assert its internal text.

## 6. Run the tests

Run the test script from the project root:

    npm test

The test submits through the form button, so it checks the same React path used by a person. If a later version uses a React 19 function action instead of onSubmit, keep the real submit interaction and update the assertions to match that action's pending and field-reset behavior.

## 7. Extend the example thoughtfully

Possible additions include:

- Verify that a blank username does not call the service and shows the intended validation feedback.
- Preserve the submitted value after a failure and verify it remains in the field.
- Disable repeated submissions while pending, then verify a second activation does not create another request.
- Add an error boundary test if the application intentionally lets unexpected Action errors propagate.
- Add a browser check for native required-field validation.

Keep the test at the service boundary. Do not mock useState, call handleSubmit directly, or wait a fixed number of milliseconds for the request.

## Review checklist

- The input is located by its associated label.
- The actual form submit control triggers the request.
- FormData contains the named input value.
- The API receives the normalized value.
- Pending feedback is checked before a deferred request is completed.
- Success, business rejection, and transport failure are separate outcomes.
- The asynchronous result is awaited through visible UI.
- No test uses a live network or arbitrary sleep.
- React form Action reset behavior is considered if the app uses that pattern.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="102-testing-user-interactions.md">102 - React Example: Testing a Complete User Interaction Flow</a></td><td><a href="../lessons/103-testing-forms-and-async-ui.md">Lesson</a></td><td><a href="../qnas/103-testing-forms-and-async-ui.md">Q&amp;A</a></td><td><a href="104-browser-level-and-end-to-end-tests.md">104 - React Example: Browser-Test a Complete Checkout Journey</a></td></tr>
  </tbody>
</table>
