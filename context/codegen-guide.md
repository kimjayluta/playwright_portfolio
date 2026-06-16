# Code Generation Guide: Codemify Store

This is the single source of truth for generating Playwright tests in this project.  
Read this file in full before writing any code. App facts (URLs, credentials, page map) live in `app-context.md`.

---

## 1. Language & Module System

| Rule             | Value                                                                 |
|------------------|-----------------------------------------------------------------------|
| Language         | JavaScript (`.js`) — no TypeScript                                    |
| Module system    | ESM — always `import`, never `require()`                              |
| First line       | `// @ts-check` on every spec and page object file                     |
| `package.json`   | No `"type": "module"` declared — Playwright's transform handles ESM   |

```js
// @ts-check
import { test, expect } from '@playwright/test';
```

---

## 2. File Placement

| File type    | Directory            | Naming convention         |
|--------------|----------------------|---------------------------|
| Specs        | `tests/`             | `[feature].spec.js`       |
| Page Objects | `tests/pages/`       | `[ViewName]Page.js`       |
| Components   | `tests/components/`  | `[Name]Component.js`      |
| Fixtures     | `tests/fixtures/`    | `[name].fixture.js`       |
| Test data    | `tests/data/`        | `[name].data.js`          |

---

## 3. Locator Priority Rules

Use the first locator type that works stably. Do not fall back further than necessary.

| Priority | Locator                                         | When to use                                              |
|----------|-------------------------------------------------|----------------------------------------------------------|
| 1        | `page.locator('[data-test="..."]')`             | Login form only — username, password, login-button       |
| 2        | `page.getByRole('...', { name: '...' })`        | Buttons, headings, links identified by visible text      |
| 3        | `page.getByLabel('...')`                        | Checkout form fields — linked via `<label>` text         |
| 4        | `page.locator('#id')`                           | Checkout inputs — stable camelCase `id` attributes       |
| 5        | `page.getByText('...')`                         | Read-only static text content                            |
| 6        | `.locator('.class-name')`                       | Last resort — prefer semantic locators above             |

**Never use:** XPath, `:nth-child()`, positional `.nth(0)`, or attribute selectors for CSS class values.

Scope product-level locators with `.filter({ hasText: productName })` to avoid ambiguity across the 6-item grid:

```js
const card = page.locator('.product-card').filter({ hasText: 'Codemify Backpack' });
await card.getByRole('button', { name: 'Add to Cart' }).click();
```

---

## 4. Assertion Rules

| Rule                                              | Pattern                                                    |
|---------------------------------------------------|------------------------------------------------------------|
| Always use Playwright auto-retry matchers         | `await expect(locator).toBeVisible()`                      |
| Assert user-visible outcomes, not internal state  | heading text, button text, cart count — not CSS classes     |
| Exact text match                                  | `await expect(el).toHaveText('Products')`                  |
| Partial text / dynamic values                     | `await expect(el).toContainText('Cart')`                   |
| Dynamic cart badge (any count)                    | `await expect(cartBtn).toContainText(/Cart \d+/)`          |
| Element not present                               | `await expect(el).not.toBeVisible()`                       |

**Never:**
- `await page.waitForTimeout(n)` — use event-driven assertions instead
- `expect(await locator.isVisible()).toBe(true)` — use `await expect(locator).toBeVisible()`
- Assert on CSS class names or aria attributes as a proxy for business outcome

---

## 5. Page Object Pattern

One Page Object per view. Never merge multiple views into one class.

```js
// @ts-check
import { expect } from '@playwright/test';

export class ProductsPage {
  /** @param {import('@playwright/test').Page} page */
  constructor(page) {
    this.page = page;
    this.heading   = page.getByRole('heading', { name: 'Products' });
    this.cartButton = page.getByRole('button', { name: /Cart/ });
    this.logoutButton = page.getByRole('button', { name: 'Logout' });
  }

  async addToCart(productName) {
    const card = this.page.locator('.product-card').filter({ hasText: productName });
    await card.getByRole('button', { name: 'Add to Cart' }).click();
  }

  async goToCart() {
    await this.cartButton.click();
  }
}
```

**Rules:**
- Constructor assigns all locators to `this.*` — never build locators inside action methods
- Action methods are `async`, perform one user action each, and return `void`
- No assertions inside action methods — assertions belong in the spec
- Export as **named export** (not `export default`)

---

## 6. Component Pattern

Use a Component (not a Page Object) for UI blocks that appear on multiple views.

```js
// @ts-check
export class NavBar {
  /** @param {import('@playwright/test').Page} page */
  constructor(page) {
    this.productsButton = page.getByRole('button', { name: 'Products' });
    this.cartButton     = page.getByRole('button', { name: /Cart/ });
    this.logoutButton   = page.getByRole('button', { name: 'Logout' });
  }
}
```

Use **Component** for: `NavBar`, `ProductCard`.  
Use **Page Object** for: `LoginPage`, `ProductsPage`, `CartPage`, `CheckoutPage`.

---

## 7. Fixture Pattern

Fixtures provide pre-authenticated state and pre-built page objects so specs avoid repeating login setup.

```js
// @ts-check
// tests/fixtures/auth.fixture.js
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage.js';
import { ProductsPage } from '../pages/ProductsPage.js';

export const test = base.extend({
  productsPage: async ({ page }, use) => {
    const login = new LoginPage(page);
    await login.goto();
    await login.loginAs('standard_user', 'my_secret_code');
    await use(new ProductsPage(page));
  },
});

export { expect } from '@playwright/test';
```

Specs that need auth import `test` and `expect` from the fixture file:

```js
import { test, expect } from '../fixtures/auth.fixture.js';
```

---

## 8. Test Structure Rules

### Naming

| Block      | Convention                                          | Example                                         |
|------------|-----------------------------------------------------|-------------------------------------------------|
| `describe` | Feature or view name                                | `'Login'`, `'Cart'`, `'Checkout'`               |
| `test`     | User action + expected outcome (no "should")        | `'shows error for locked out user'`             |

### AAA Pattern

```js
test('adds item to cart and updates badge count', async ({ productsPage }) => {
  // Arrange — handled by fixture (already on Products view)

  // Act
  await productsPage.addToCart('Codemify Backpack');

  // Assert
  await expect(productsPage.cartButton).toContainText('1');
});
```

### beforeEach vs beforeAll

- `beforeEach` — navigation or mutable state that must be clean for every test
- `beforeAll` — one-time read-only setup only (rare in this app)
- Never share mutable state between tests via `beforeAll`

---

## 9. Test Independence Rules

- Every test must pass when run in isolation (`--grep` on a single test name)
- Never rely on cart items added by a previous test — each test adds its own items
- Never rely on execution order within a `describe` block
- Never use `test.only` in committed code (`forbidOnly: true` in CI config)

---

## 10. What NOT to Generate Without Being Asked

- `try/catch` in test code
- Retry logic — `playwright.config.js` handles retries
- Helper utilities beyond Page Objects and fixtures
- Network mocks unless the test is specifically about error/offline states
- `page.screenshot()` calls
- Comments explaining what the code does — method names must be self-describing

---

## 11. Preferred Generation Workflow

1. Identify which view(s) are involved and check if a Page Object already exists
2. If missing, generate the Page Object first
3. Generate or extend the fixture if auth is needed
4. Generate the spec file, importing from page objects and the fixture
5. Output files in this order: **Page Objects → Fixtures → Specs**

---

## 12. Output Format

Label each file with a comment header when outputting multiple files:

```
// FILE: tests/pages/CartPage.js
[content]

// FILE: tests/cart.spec.js
[content]
```

---

## 13. When Business Rules Are Unclear

If a behavior is not documented in `app-context.md` and cannot be confirmed from the live app:

```js
// UNCLEAR: Does removing the last cart item redirect to Products automatically?
// Assumption: stays on Cart view and shows an empty state. Verify before shipping.
```

- Use the prefix `// UNCLEAR:` followed by the assumption made
- Call out the assumption in the PR description so it can be verified manually

---

## 14. When a Test Fails

Diagnose in this order:

1. **Test bug** — locator stale, assertion too strict, or race condition
   - Re-run the flow manually in the browser with the same user actions
   - Compare the current snapshot to the locator being used
2. **App bug** — the feature itself is broken
   - Confirm the failure is reproducible manually
   - Mark with `test.fail()` and add a comment referencing the bug
   - Never delete a failing test to make the suite green
