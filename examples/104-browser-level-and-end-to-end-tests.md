# React Example: Browser-Test a Complete Checkout Journey

This example runs a small React checkout app in a real browser. The Playwright test adds a product, proceeds to checkout, submits an email address, and checks the confirmation. The order API is controlled by the test so the browser journey is deterministic and does not need a live backend.

## What this example demonstrates

- Starting a production-like Vite preview server for browser automation.
- Locating controls by role and accessible name, and fields by label.
- Testing a complete user journey through the rendered React app.
- Intercepting API requests and asserting the submitted data.
- Checking pending, success, and service-failure states.
- Testing native email validation in an actual browser.
- Waiting for visible outcomes instead of fixed delays.

## Prerequisites

Start from a React application that uses Vite and has `build` and `preview` scripts. Install Playwright Test and its Chromium browser:

```sh
npm install --save-dev @playwright/test
npx playwright install chromium
```

Add or merge these scripts into the existing `package.json`; keep its other scripts and dependencies:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test:e2e": "npm run build && playwright test"
  }
}
```

If the project already has these scripts, do not duplicate them. Use its own framework's build and local server commands if it is not a Vite app.

## File structure

```text
index.html
playwright.config.js
e2e/
  checkout.spec.js
src/
  App.jsx
  App.css
  main.jsx
```

## 1. Add the React application

Create `index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Trail Shop</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

Create `src/main.jsx`:

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './App.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

Create `src/App.jsx`:

```jsx
import { useState } from 'react';

const product = {
  id: 'trail-backpack',
  name: 'Trail Backpack',
  price: 48,
};

export default function App() {
  const [inCart, setInCart] = useState(false);
  const [showCheckout, setShowCheckout] = useState(false);
  const [email, setEmail] = useState('');
  const [pending, setPending] = useState(false);
  const [error, setError] = useState('');
  const [orderNumber, setOrderNumber] = useState('');

  async function handleSubmit(event) {
    event.preventDefault();
    setPending(true);
    setError('');

    try {
      const response = await fetch('/api/orders', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ productId: product.id, email }),
      });

      if (!response.ok) {
        throw new Error('The order service returned an error.');
      }

      const result = await response.json();
      setOrderNumber(result.orderNumber);
    } catch {
      setError('Order could not be placed. Try again.');
    } finally {
      setPending(false);
    }
  }

  if (orderNumber) {
    return (
      <main className="page">
        <h1>Trail Shop</h1>
        <section aria-labelledby="confirmation-title" className="card">
          <h2 id="confirmation-title">Order confirmed</h2>
          <p>Order number: {orderNumber}</p>
          <p>A confirmation will be sent to {email}.</p>
        </section>
      </main>
    );
  }

  return (
    <main className="page">
      <h1>Trail Shop</h1>

      <section aria-labelledby="product-title" className="card">
        <h2 id="product-title">{product.name}</h2>
        <p>Lightweight day pack</p>
        <p>${product.price.toFixed(2)}</p>
        <button type="button" onClick={() => setInCart(true)}>
          Add {product.name} to cart
        </button>
      </section>

      {inCart && (
        <section aria-labelledby="cart-title" className="card">
          <h2 id="cart-title">Cart</h2>
          <p>{product.name} - ${product.price.toFixed(2)}</p>
          <p role="status">{product.name} added to cart.</p>
          <button type="button" onClick={() => setShowCheckout(true)}>
            Continue to checkout
          </button>
        </section>
      )}

      {showCheckout && (
        <section aria-labelledby="checkout-title" className="card">
          <h2 id="checkout-title">Checkout</h2>
          <form onSubmit={handleSubmit}>
            <label htmlFor="email">Email</label>
            <input
              id="email"
              name="email"
              type="email"
              autoComplete="email"
              required
              value={email}
              onChange={(event) => setEmail(event.target.value)}
            />
            <button type="submit" disabled={pending}>
              {pending ? 'Placing order…' : 'Place order'}
            </button>
          </form>
          {pending && <p role="status">Placing your order.</p>}
          {error && <p role="alert">{error}</p>}
        </section>
      )}
    </main>
  );
}
```

Create `src/App.css`:

```css
:root {
  font-family: system-ui, sans-serif;
  color: #17202a;
  background: #f3f6f8;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  margin: 0;
}

button,
input {
  font: inherit;
}

button {
  min-height: 2.75rem;
  padding: 0.6rem 1rem;
  border: 0;
  border-radius: 0.4rem;
  color: white;
  background: #145c47;
  cursor: pointer;
}

button:disabled {
  cursor: wait;
  opacity: 0.7;
}

:focus-visible {
  outline: 3px solid #d47b00;
  outline-offset: 3px;
}

.page {
  width: min(100% - 2rem, 42rem);
  margin: 2rem auto;
}

.card {
  display: grid;
  gap: 0.75rem;
  margin-block: 1rem;
  padding: 1.25rem;
  border: 1px solid #d5dde2;
  border-radius: 0.75rem;
  background: white;
}

.card h2,
.card p {
  margin: 0;
}

form {
  display: grid;
  gap: 0.75rem;
}

input {
  min-height: 2.75rem;
  padding: 0.5rem 0.75rem;
  border: 1px solid #687780;
  border-radius: 0.35rem;
}

[role='alert'] {
  color: #9b1c1c;
}
```

## 2. Configure Playwright

Create `playwright.config.js`:

```js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: Boolean(process.env.CI),
  retries: process.env.CI ? 1 : 0,
  reporter: 'html',
  use: {
    baseURL: 'http://127.0.0.1:4173',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
  webServer: {
    command: 'npm run preview -- --host 127.0.0.1 --port 4173 --strictPort',
    url: 'http://127.0.0.1:4173',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
});
```

The `test:e2e` script builds the app before Playwright starts the preview server. The server setting waits for the configured local address to become ready. If your framework serves the production build differently, replace the preview command and readiness address together.

## 3. Write the browser tests

Create `e2e/checkout.spec.js`:

```js
import { expect, test } from '@playwright/test';

async function addProductAndOpenCheckout(page) {
  await page.goto('/');
  await page.getByRole('button', { name: 'Add Trail Backpack to cart' }).click();

  await expect(page.getByRole('status')).toHaveText(
    'Trail Backpack added to cart.',
  );

  await page.getByRole('button', { name: 'Continue to checkout' }).click();
  await expect(
    page.getByRole('heading', { name: 'Checkout' }),
  ).toBeVisible();
}

test('customer completes checkout and sees the order confirmation', async ({
  page,
}) => {
  await page.route('**/api/orders', async (route) => {
    const request = route.request();

    expect(request.method()).toBe('POST');
    expect(request.postDataJSON()).toEqual({
      productId: 'trail-backpack',
      email: 'ada@example.test',
    });

    await route.fulfill({
      status: 201,
      contentType: 'application/json',
      body: JSON.stringify({ orderNumber: 'ORDER-1042' }),
    });
  });

  await addProductAndOpenCheckout(page);
  await page.getByRole('textbox', { name: 'Email' }).fill('ada@example.test');
  await page.getByRole('button', { name: 'Place order' }).click();

  await expect(
    page.getByRole('heading', { name: 'Order confirmed' }),
  ).toBeVisible();
  await expect(page.getByText('Order number: ORDER-1042')).toBeVisible();
});

test('checkout shows pending feedback until the order request completes', async ({
  page,
}) => {
  let releaseResponse;
  const responseCanContinue = new Promise((resolve) => {
    releaseResponse = resolve;
  });

  await page.route('**/api/orders', async (route) => {
    await responseCanContinue;
    await route.fulfill({
      status: 201,
      contentType: 'application/json',
      body: JSON.stringify({ orderNumber: 'ORDER-PENDING' }),
    });
  });

  await addProductAndOpenCheckout(page);
  await page.getByRole('textbox', { name: 'Email' }).fill('ada@example.test');
  await page.getByRole('button', { name: 'Place order' }).click();

  await expect(page.getByRole('status')).toHaveText('Placing your order.');
  await expect(
    page.getByRole('button', { name: 'Placing order…' }),
  ).toBeDisabled();

  releaseResponse();
  await expect(
    page.getByRole('heading', { name: 'Order confirmed' }),
  ).toBeVisible();
});

test('checkout explains when the order service is unavailable', async ({
  page,
}) => {
  await page.route('**/api/orders', (route) =>
    route.fulfill({ status: 503, body: 'Service unavailable' }),
  );

  await addProductAndOpenCheckout(page);
  await page.getByRole('textbox', { name: 'Email' }).fill('ada@example.test');
  await page.getByRole('button', { name: 'Place order' }).click();

  await expect(page.getByRole('alert')).toHaveText(
    'Order could not be placed. Try again.',
  );
  await expect(
    page.getByRole('button', { name: 'Place order' }),
  ).toBeEnabled();
});

test('browser validation prevents checkout with an empty email', async ({
  page,
}) => {
  await addProductAndOpenCheckout(page);
  const email = page.getByRole('textbox', { name: 'Email' });

  await page.getByRole('button', { name: 'Place order' }).click();

  await expect(email).toBeFocused();
  await expect(
    page.getByRole('heading', { name: 'Checkout' }),
  ).toBeVisible();
});
```

The API route is intercepted only in the tests. The success case checks the actual request method and submitted JSON before returning a predictable order number. The pending case deliberately holds the response until the test has observed the pending interface. The failure case returns an unavailable-service response. No test waits for an arbitrary number of milliseconds.

## 4. Run and inspect the suite

Run the build and browser tests from the project root:

```sh
npm run test:e2e
```

Open the HTML report after a run:

```sh
npx playwright show-report
```

When a test fails, inspect its trace and screenshot, then check the app's console and request behavior. Fix the cause of a failure instead of adding a delay or weakening the expected result. These route-intercepted tests prove the browser UI's behavior against controlled API outcomes; add a separate test against a dedicated backend environment if the browser-to-service integration also needs coverage.

## Expected outcomes

- The customer can add the product and reach checkout.
- A valid submission sends the expected product and email to the order boundary.
- A successful response produces an order confirmation.
- While the response is pending, the page communicates progress and prevents a duplicate submission.
- A service error produces a recoverable message and re-enables the submit button.
- An empty required email is blocked by browser form validation.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="103-testing-forms-and-async-ui.md">103 - React Example: Testing an Async Username Form and API Boundary</a></td><td><a href="../lessons/104-browser-level-and-end-to-end-tests.md">Lesson</a></td><td><a href="../qnas/104-browser-level-and-end-to-end-tests.md">Q&amp;A</a></td><td><a href="105-react-devtools-and-debugging.md">105 - React Example: Debug and Profile a Task Board with React DevTools</a></td></tr>
  </tbody>
</table>
