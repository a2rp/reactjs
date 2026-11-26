# React Example: Test Strategy and Testable Component Design

This example builds a quantity stepper and tests it at two useful levels. A pure helper test checks the numeric boundary rule. A component test renders the real React component, activates its accessible buttons, and checks the text and disabled state a user can see.

The example uses TypeScript, Vitest, jsdom, React Testing Library, and user-event. It does not mock React state or the component's event handlers. The browser-like DOM gives the component test a repeatable environment; it is not a replacement for checking a critical workflow in a real browser.

## What this example demonstrates

- Isolating a meaningful domain rule as a pure helper.
- Keeping the component responsible for its own simple local quantity state.
- Finding controls by their accessible role and name.
- Simulating a real user interaction instead of calling an event handler directly.
- Checking both the displayed value and the maximum boundary.
- Keeping the test environment local and deterministic.

## Prerequisites

The application already has React and React DOM installed. Install the development dependencies from the project root:

    npm install --save-dev vitest jsdom @testing-library/react @testing-library/dom @testing-library/user-event

The package scripts should include a command for Vitest. Add or merge this entry into the existing package.json scripts object rather than replacing other scripts:

    "scripts": {
      "test": "vitest run"
    }

## File structure

    src/
    ├── quantity.ts
    ├── quantity.test.ts
    ├── QuantityStepper.tsx
    └── QuantityStepper.test.tsx
    vitest.config.ts

## 1. Write the pure quantity rule

Create src/quantity.ts:

    export function clampQuantity(
      value: number,
      min: number,
      max: number,
    ): number {
      if (min > max) {
        throw new RangeError('The minimum cannot be greater than the maximum.');
      }

      return Math.min(max, Math.max(min, value));
    }

This helper has no React or browser dependency. Its contract expects finite numeric inputs and a minimum that is no greater than the maximum. It returns the closest value inside the allowed range.

## 2. Test the pure rule directly

Create src/quantity.test.ts:

    import { describe, expect, it } from 'vitest';
    import { clampQuantity } from './quantity';

    describe('clampQuantity', () => {
      it('raises a value below the minimum to the minimum', () => {
        expect(clampQuantity(0, 1, 5)).toBe(1);
      });

      it('keeps a value already inside the range', () => {
        expect(clampQuantity(3, 1, 5)).toBe(3);
      });

      it('lowers a value above the maximum to the maximum', () => {
        expect(clampQuantity(8, 1, 5)).toBe(5);
      });

      it('rejects an invalid range', () => {
        expect(() => clampQuantity(3, 5, 1)).toThrow(RangeError);
      });
    });

These tests cover the rule's lower boundary, ordinary case, upper boundary, and invalid range. They do not need to render React because the behavior under test is a standalone calculation.

## 3. Build the React component

Create src/QuantityStepper.tsx:

    import { useState } from 'react';
    import { clampQuantity } from './quantity';

    type QuantityStepperProps = {
      initialValue?: number;
      min?: number;
      max?: number;
    };

    export function QuantityStepper({
      initialValue = 1,
      min = 1,
      max = 10,
    }: QuantityStepperProps) {
      const [quantity, setQuantity] = useState(() =>
        clampQuantity(initialValue, min, max),
      );

      return (
        <section aria-labelledby="quantity-heading">
          <h2 id="quantity-heading">Choose a quantity</h2>

          <p>Quantity: {quantity}</p>

          <button
            type="button"
            disabled={quantity <= min}
            onClick={() => {
              setQuantity(current => clampQuantity(current - 1, min, max));
            }}
          >
            Decrease quantity
          </button>

          <button
            type="button"
            disabled={quantity >= max}
            onClick={() => {
              setQuantity(current => clampQuantity(current + 1, min, max));
            }}
          >
            Increase quantity
          </button>
        </section>
      );
    }

The section has a heading, and each native button has a visible accessible name. The minimum and maximum states are represented with the button's native disabled property. Functional state updates use the latest queued value, which remains correct when React batches updates.

This component treats initialValue, min, and max as setup inputs. If a parent needs changing bounds after mount, define that behavior explicitly and add a test for it instead of assuming the initial state will be recalculated automatically.

## 4. Configure a browser-like test environment

Create vitest.config.ts at the project root:

    import { defineConfig } from 'vitest/config';

    export default defineConfig({
      test: {
        environment: 'jsdom',
      },
    });

Vitest's default environment is Node. jsdom supplies the DOM APIs that a React component test needs. Use the configuration file already present in the project if the application has one; merge this test setting into it.

## 5. Test the rendered component through its controls

Create src/QuantityStepper.test.tsx:

    import { render, screen } from '@testing-library/react';
    import userEvent from '@testing-library/user-event';
    import { describe, expect, it } from 'vitest';
    import { QuantityStepper } from './QuantityStepper';

    describe('QuantityStepper', () => {
      it('shows the initial quantity and normalizes it to the range', () => {
        render(<QuantityStepper initialValue={20} min={1} max={3} />);

        expect(screen.getByText('Quantity: 3')).toBeTruthy();
      });

      it('updates the quantity and stops at the maximum', async () => {
        const user = userEvent.setup();
        render(<QuantityStepper initialValue={2} min={1} max={3} />);

        const increaseButton = screen.getByRole('button', {
          name: 'Increase quantity',
        });

        await user.click(increaseButton);

        expect(screen.getByText('Quantity: 3')).toBeTruthy();
        expect(increaseButton.hasAttribute('disabled')).toBe(true);

        await user.click(
          screen.getByRole('button', { name: 'Decrease quantity' }),
        );

        expect(screen.getByText('Quantity: 2')).toBeTruthy();
        expect(increaseButton.hasAttribute('disabled')).toBe(false);
      });

      it('disables decrease at the minimum', () => {
        render(<QuantityStepper initialValue={1} min={1} max={3} />);

        const decreaseButton = screen.getByRole('button', {
          name: 'Decrease quantity',
        });

        expect(decreaseButton.hasAttribute('disabled')).toBe(true);
      });
    });

The test locates controls by role and accessible name, then uses an interaction helper to click them. Assertions cover the visible quantity and the native disabled state. The test does not inspect the state hook, call a handler directly, or depend on CSS class names.

## 6. Run and interpret the tests

Run the suite from the project root:

    npm test

A failure in a pure helper test points to the range rule. A failure in a component test may point to rendering, accessible naming, event wiring, or the visible boundary state. This distinction helps identify which contract changed.

## 7. Extend the example thoughtfully

Possible useful additions include:

- A test that pressing Enter or Space activates a focused button through normal keyboard behavior.
- A test for the minimum and maximum values the real product allows.
- A test for a parent callback if the component later accepts an onQuantityChange prop.
- A browser-level test if this quantity control becomes part of a high-risk checkout flow.

Do not add a test for every implementation detail. If a refactor preserves the behavior and the test breaks only because an internal variable or wrapper changed, revise the test to assert the user-facing contract instead.

## Review checklist

- The pure range rule is tested independently.
- The React component is rendered in a DOM-based test.
- Controls are found by accessible role and name.
- Interactions go through the rendered controls.
- Assertions check visible output and a meaningful boundary state.
- The tested UI and state updates remain real.
- Test inputs are local and do not need a live network.
- Any added browser test covers a complete, high-value workflow.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="099-accessible-announcements-and-async-ui.md">099 - React Example: Accessible Async Search with Current-Request Protection</a></td><td><a href="../lessons/100-testing-strategy-and-component-design.md">Lesson</a></td><td><a href="../qnas/100-testing-strategy-and-component-design.md">Q&amp;A</a></td><td><a href="101-component-tests-and-test-utilities.md">101 - React Example: Component Tests and React Test Utilities</a></td></tr>
  </tbody>
</table>
