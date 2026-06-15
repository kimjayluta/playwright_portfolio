# Playwright Skills — Project Helpers

This file documents approved patterns, helpers, and common setups for this project. Use these instead of re-inventing them. Do not deviate from these patterns unless the test explicitly requires something different.

---

## Login Setup Pattern

When login is a **precondition** (not what's being tested), perform it in `beforeEach`:

```js
const { test, expect } = require('../fixtures');
const { LoginPage } = require('../pages/LoginPage');

test.describe('Inventory', () => {
  test.beforeEach(async ({ page }) => {
    const loginPage = new LoginPage(page);
    await page.goto('https://www.saucedemo.com/');
    await loginPage.login('standard_user', 'secret_sauce');
    // After login, Playwright lands on /inventory.html automatically
  });

  test('shows products after login', async ({ page }) => {
    await expect(page).toHaveURL('https://www.saucedemo.com/inventory.html');
  });
});
```

**Do NOT use `storageState`** for this app. SauceDemo doesn't use persistent auth tokens — login is session-only (browser cookie). `storageState` won't help here.

---

## Cart State

Playwright creates a **new browser context per test** by default, which means:
- Cart is automatically empty at the start of each test
- You do NOT need to manually clear the cart in `afterEach`
- Tests are cart-isolated without any extra setup

---

## Navigation Pattern

Always use the full absolute URL. The `playwright.config.js` does NOT currently have a `baseURL` set:

```js
// CORRECT
await page.goto('https://www.saucedemo.com/');
await page.goto('https://www.saucedemo.com/inventory.html');
await page.goto('https://www.saucedemo.com/cart.html');

// WRONG — do not use relative paths yet
await page.goto('/inventory.html');
```

---

## Common Assertions

### Login succeeded

```js
await expect(page).toHaveURL('https://www.saucedemo.com/inventory.html');
```

### Login failed (error visible)

```js
await expect(page.getByTestId('error')).toBeVisible();
await expect(page.getByTestId('error')).toContainText('Epic sadface');
```

### Cart badge shows a count

```js
await expect(page.getByTestId('shopping-cart-badge')).toHaveText('1');
await expect(page.getByTestId('shopping-cart-badge')).toHaveText('3');
```

### Cart badge is gone (cart is empty)

```js
await expect(page.getByTestId('shopping-cart-badge')).not.toBeVisible();
```

### Checkout complete

```js
await expect(page.getByTestId('complete-header')).toHaveText('Thank you for your order!');
await expect(page).toHaveURL('https://www.saucedemo.com/checkout-complete.html');
```

### Checkout field error

```js
await expect(page.getByTestId('error')).toContainText('First Name is required');
```

---

## Test Data Constants

Until `tests/fixtures/test-data.js` is created, inline credentials are acceptable. When creating it, use this structure:

```js
// tests/fixtures/test-data.js
const USERS = {
  standard:          { username: 'standard_user',          password: 'secret_sauce' },
  locked:            { username: 'locked_out_user',         password: 'secret_sauce' },
  problem:           { username: 'problem_user',            password: 'secret_sauce' },
  performanceGlitch: { username: 'performance_glitch_user', password: 'secret_sauce' },
  error:             { username: 'error_user',              password: 'secret_sauce' },
  visual:            { username: 'visual_user',             password: 'secret_sauce' },
};

const CHECKOUT_DATA = {
  valid: { firstName: 'John', lastName: 'Doe', postalCode: '12345' },
};

module.exports = { USERS, CHECKOUT_DATA };
```

Usage:
```js
const { USERS } = require('../fixtures/test-data');
await loginPage.login(USERS.standard.username, USERS.standard.password);
```

---

## Parallel Safety Rules

`playwright.config.js` has `fullyParallel: true`. Tests run concurrently across workers.

- MUST NOT use `test.beforeAll` for login — `beforeAll` runs once per `describe` but shares state across workers, causing flakiness
- MUST NOT use module-level variables to share data between tests
- MUST use `test.beforeEach` for per-test setup

---

## Slow Test Handling (performance_glitch_user)

When testing `performance_glitch_user`, login has a ~5 second delay. Playwright's default timeout is 30 seconds, so no timeout change is needed. Just be aware that assertions after login may need to wait slightly longer for navigation to settle.

```js
await loginPage.login('performance_glitch_user', 'secret_sauce');
// Navigation is slow — expect() auto-waits up to 5s by default
await expect(page).toHaveURL('https://www.saucedemo.com/inventory.html');
```
