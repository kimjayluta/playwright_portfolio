# Project Context

This file is the single source of truth for the project setup. Read this first before generating any test code so you know the tech stack, URLs, and directory layout.

---

## Application Under Test

- **Name:** SauceDemo
- **URL:** https://www.saucedemo.com/
- **Type:** Demo e-commerce site for testing practice — no real backend, no real transactions
- **Environment:** Single environment only (no staging/prod split)

---

## Tech Stack

- **Runtime:** Node.js
- **Module system:** CommonJS (`"type": "commonjs"` in package.json)
- **Test framework:** `@playwright/test` v1.60.0
- **Language:** JavaScript (`.js`) — NEVER generate TypeScript
- **Config file:** `playwright.config.js`
  - `testDir: './tests'`
  - `fullyParallel: true`
  - Browsers: Chromium, Firefox, WebKit (Desktop)
  - Reporter: HTML
  - Trace: on-first-retry
  - **baseURL is NOT configured** — always use full absolute URLs

---

## Application Page Map

| Page | URL |
|---|---|
| Login | https://www.saucedemo.com/ |
| Inventory (product list) | https://www.saucedemo.com/inventory.html |
| Product Detail | https://www.saucedemo.com/inventory-item.html |
| Cart | https://www.saucedemo.com/cart.html |
| Checkout — Info | https://www.saucedemo.com/checkout-step-one.html |
| Checkout — Overview | https://www.saucedemo.com/checkout-step-two.html |
| Checkout — Complete | https://www.saucedemo.com/checkout-complete.html |

---

## Project Directory Structure

```
playwright_portfolio/
├── playwright.config.js
├── package.json
├── tests/
│   ├── example.spec.js          # existing baseline — do not delete
│   ├── pages/                   # Page Object classes (to be created)
│   │   ├── LoginPage.js
│   │   ├── InventoryPage.js
│   │   ├── ProductDetailPage.js
│   │   ├── CartPage.js
│   │   ├── CheckoutInfoPage.js
│   │   ├── CheckoutOverviewPage.js
│   │   └── CheckoutCompletePage.js
│   ├── components/              # Reusable components (to be created)
│   │   └── HeaderComponent.js
│   ├── fixtures/                # Custom test fixtures (to be created)
│   │   └── index.js
│   └── specs/                   # Feature test specs (to be created)
│       ├── login.spec.js
│       ├── inventory.spec.js
│       ├── cart.spec.js
│       └── checkout.spec.js
└── .claude/
    └── skills/
        └── playwright-cli/
            ├── SKILL.md
            ├── references/      # General Playwright tooling references
            └── context/         # THIS folder — project-specific knowledge
```

---

## Key Technical Observations

- App uses `data-test` attributes extensively on interactive elements — always prefer `getByTestId()`
- No dynamic routes — all URLs are static HTML pages
- No persistent auth tokens — session state is managed by browser cookies only
- Playwright creates a **new browser context per test** — cart and session state reset automatically between tests
- Do not use `storageState` for this app
