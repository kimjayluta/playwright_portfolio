# Testing Standards

This file defines the naming conventions, test structure, assertion guidelines, and locator rules for all tests in this project. Follow these rules consistently so every generated test file looks the same.

---

## File Naming

| What | Convention | Example |
|---|---|---|
| Spec files | kebab-case, feature name | `login.spec.js`, `cart.spec.js` |
| Page Objects | PascalCase, page name | `LoginPage.js`, `InventoryPage.js` |
| Components | PascalCase + `Component` suffix | `HeaderComponent.js` |
| Fixtures | single entry point | `fixtures/index.js` |

---

## Test Structure

### describe / test Naming

```js
const { test, expect } = require('../fixtures');

test.describe('Login', () => {          // feature or page name
  test('displays error for locked out user', async ({ page }) => { ... });
  test('redirects to inventory on successful login', async ({ page }) => { ... });
});
```

**describe block:**
- Name = feature or page: `'Login'`, `'Inventory'`, `'Cart'`, `'Checkout'`
- One `describe` block per spec file
- No nested `describe` blocks unless separating positive vs. negative cases

**test name:**
- Starts with a verb, describes the expected outcome
- GOOD: `'displays error for locked out user'`
- GOOD: `'adds item to cart and updates badge count'`
- GOOD: `'redirects to inventory on successful login'`
- BAD: `'test login'` (too vague)
- BAD: `'locked_out_user test'` (uses internal credential name, not behavior)

---

## Arrange-Act-Assert Pattern

Every test MUST follow this structure:

```js
test('adds item to cart and updates badge count', async ({ page, inventoryPage }) => {
  // Arrange — navigate to starting state
  await page.goto('https://www.saucedemo.com/');
  await page.getByTestId('username').fill('standard_user');
  await page.getByTestId('password').fill('secret_sauce');
  await page.getByTestId('login-button').click();

  // Act — perform the action under test
  await page.getByText('Add to cart').first().click();

  // Assert — verify the expected outcome
  await expect(inventoryPage.header.cartBadge).toHaveText('1');
});
```

---

## Assertion Guidelines

### Preferred Matchers

| Matcher | When to use |
|---|---|
| `toBeVisible()` | Element is rendered and visible to the user |
| `toHaveText(string \| regex)` | Exact or partial text content |
| `toHaveURL(string \| regex)` | Current page URL after navigation |
| `toHaveValue(string)` | Input field value |
| `toHaveCount(number)` | Number of matching elements in the DOM |
| `not.toBeVisible()` | Element is hidden or absent |
| `toContainText(string)` | Substring match when full text varies |

### Forbidden Patterns

- MUST NOT use `page.waitForTimeout()` — Playwright's auto-waiting handles timing
- MUST NOT use `page.waitForSelector()` before an assertion — `expect()` already auto-waits
- MUST NOT assert on CSS class names or computed styles (fragile, implementation detail)
- MUST NOT use `toHaveText('0')` for cart badge — badge element is absent when cart is empty, use `not.toBeVisible()`

---

## Locator Priority

Use locators in this order of preference:

1. `page.getByTestId('...')` — `data-test` attribute (PREFER when available — see business-rules.md for full list)
2. `page.getByRole('button', { name: '...' })` — semantic role + accessible name
3. `page.getByLabel('...')` — form elements with a visible label
4. `page.getByText('...')` — last resort for non-interactive visible text

MUST NOT use:
- `page.locator('.css-class')` — CSS class selectors (fragile)
- XPath selectors
- `page.locator('#id')` — ID selectors unless no `data-test` attribute exists

---

## Test Independence

- Each test MUST work when run in isolation (`npx playwright test --grep "test name"`)
- MUST NOT rely on state left by a previous test (cart contents, login session, page URL)
- Use `test.beforeEach` to navigate to the starting URL for each test
- MUST NOT use `test.beforeAll` for login — not parallel-safe

```js
test.beforeEach(async ({ page }) => {
  await page.goto('https://www.saucedemo.com/');
});
```

---

## Import Pattern

```js
// Always import test and expect from fixtures when a fixture covers the page object
const { test, expect } = require('../fixtures');

// Import Page Objects manually only for pages not yet in fixtures
const { LoginPage } = require('../pages/LoginPage');
```

---

## One Spec File Per Feature

| Feature | File |
|---|---|
| Login / Auth | `tests/specs/login.spec.js` |
| Inventory / Products | `tests/specs/inventory.spec.js` |
| Cart | `tests/specs/cart.spec.js` |
| Checkout (all steps) | `tests/specs/checkout.spec.js` |

Do NOT create separate spec files per individual test case.
