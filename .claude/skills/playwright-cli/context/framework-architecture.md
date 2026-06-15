# Framework Architecture — Page Object Model

This is the required code structure for all Playwright tests in this project. Every generated Page Object, Component, Fixture, and Spec file MUST follow the patterns defined here. This is the most important architecture reference.

---

## Directory Layout

```
tests/
├── pages/                    # One class per full page
│   ├── LoginPage.js
│   ├── InventoryPage.js
│   ├── ProductDetailPage.js
│   ├── CartPage.js
│   ├── CheckoutInfoPage.js
│   ├── CheckoutOverviewPage.js
│   └── CheckoutCompletePage.js
├── components/               # Shared UI elements that appear on multiple pages
│   └── HeaderComponent.js
├── fixtures/                 # Custom test fixtures (extends base test)
│   └── index.js
└── specs/                    # Test spec files, one per feature
    ├── login.spec.js
    ├── inventory.spec.js
    ├── cart.spec.js
    └── checkout.spec.js
```

---

## Page Object Rules

### Rule 1 — Constructor Pattern

- Accept `page` as the ONLY constructor argument
- Define ALL locators as `this.xxx` properties in the constructor
- NEVER create locators inside action methods
- NEVER use inheritance between Page Objects

```js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.getByTestId('username');
    this.passwordInput = page.getByTestId('password');
    this.loginButton  = page.getByTestId('login-button');
    this.errorMessage = page.getByTestId('error');
  }
}
```

### Rule 2 — Action Methods

- Each public method is `async` and represents a user action
- Method names use camelCase verbs: `login()`, `addToCart()`, `proceedToCheckout()`
- Methods return `void` (or a value only when the caller needs it for the next step)
- NEVER put `expect()` assertions inside Page Object methods — assertions belong in spec files

```js
async login(username, password) {
  await this.usernameInput.fill(username);
  await this.passwordInput.fill(password);
  await this.loginButton.click();
}
```

### Rule 3 — Exports

- Named exports only: `module.exports = { LoginPage };`
- CommonJS only — NEVER use `import` / `export` ESM syntax

---

## Full Example: LoginPage.js

```js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.getByTestId('username');
    this.passwordInput = page.getByTestId('password');
    this.loginButton  = page.getByTestId('login-button');
    this.errorMessage = page.getByTestId('error');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}

module.exports = { LoginPage };
```

---

## Full Example: InventoryPage.js

```js
class InventoryPage {
  constructor(page) {
    this.page = page;
    this.inventoryList  = page.getByTestId('inventory-list');
    this.sortDropdown   = page.getByTestId('product-sort-container');
    this.cartLink       = page.getByTestId('shopping-cart-link');
    this.cartBadge      = page.getByTestId('shopping-cart-badge');
  }

  async sortBy(optionLabel) {
    await this.sortDropdown.selectOption({ label: optionLabel });
  }

  async addToCartByName(productName) {
    const addButton = this.page.getByRole('button', { name: `Add to cart` }).filter({
      has: this.page.locator('..').filter({ hasText: productName })
    });
    await addButton.click();
  }

  async goToCart() {
    await this.cartLink.click();
  }

  async getProductNames() {
    return this.page.getByTestId('inventory-item-name').allTextContents();
  }
}

module.exports = { InventoryPage };
```

---

## Component Rules

### When to create a Component vs a Page Object

| Use a **Component** when... | Use a **Page Object** when... |
|---|---|
| The element appears on **multiple pages** | It represents an entire page |
| It has its own independent behavior | |

The Header (cart icon + badge) appears on all post-login pages → it is a Component.

### Full Example: HeaderComponent.js

```js
class HeaderComponent {
  constructor(page) {
    this.page      = page;
    this.cartLink  = page.getByTestId('shopping-cart-link');
    this.cartBadge = page.getByTestId('shopping-cart-badge');
  }

  async goToCart() {
    await this.cartLink.click();
  }

  async getCartCount() {
    return this.cartBadge.textContent();
  }

  async isCartBadgeVisible() {
    return this.cartBadge.isVisible();
  }
}

module.exports = { HeaderComponent };
```

### Composing Components into Page Objects

Page Objects that contain the header can hold a `HeaderComponent` instance:

```js
const { HeaderComponent } = require('../components/HeaderComponent');

class InventoryPage {
  constructor(page) {
    this.page   = page;
    this.header = new HeaderComponent(page);
    // ... other locators
  }
}
```

---

## Custom Fixtures

### Purpose

Fixtures provide pre-instantiated Page Objects to spec files, eliminating `new LoginPage(page)` boilerplate in every test.

### tests/fixtures/index.js

```js
const base = require('@playwright/test');
const { LoginPage }            = require('../pages/LoginPage');
const { InventoryPage }        = require('../pages/InventoryPage');
const { CartPage }             = require('../pages/CartPage');
const { CheckoutInfoPage }     = require('../pages/CheckoutInfoPage');
const { CheckoutOverviewPage } = require('../pages/CheckoutOverviewPage');
const { CheckoutCompletePage } = require('../pages/CheckoutCompletePage');

const test = base.test.extend({
  loginPage:            async ({ page }, use) => await use(new LoginPage(page)),
  inventoryPage:        async ({ page }, use) => await use(new InventoryPage(page)),
  cartPage:             async ({ page }, use) => await use(new CartPage(page)),
  checkoutInfoPage:     async ({ page }, use) => await use(new CheckoutInfoPage(page)),
  checkoutOverviewPage: async ({ page }, use) => await use(new CheckoutOverviewPage(page)),
  checkoutCompletePage: async ({ page }, use) => await use(new CheckoutCompletePage(page)),
});

module.exports = { test, expect: base.expect };
```

### Usage in spec files

```js
const { test, expect } = require('../fixtures');

test('adds item to cart', async ({ page, inventoryPage }) => {
  await page.goto('https://www.saucedemo.com/');
  // inventoryPage is already instantiated — no `new` needed
  await inventoryPage.goToCart();
});
```

---

## What NOT to Do

| Forbidden | Correct Alternative |
|---|---|
| `page.locator('.login-button')` when data-test exists | `page.getByTestId('login-button')` |
| `await expect(locator)` inside a Page Object method | Put assertions in the spec file |
| `import { LoginPage } from './LoginPage'` | `const { LoginPage } = require('./LoginPage')` |
| TypeScript syntax (`.ts`, `interface`, `: string`) | Plain JavaScript only |
| `class CartPage extends BasePage` | No inheritance — each Page Object is standalone |
| One spec file per individual test | Group tests by feature in one spec file |
