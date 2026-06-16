# App Context: Codemify Store

## App Overview

| Field       | Value                                            |
|-------------|--------------------------------------------------|
| App Name    | Codemify Store                                   |
| URL         | https://codemify-demo-app.vercel.app/demo-app    |
| App Type    | Demo e-commerce — no real payments, no real data |
| Auth Method | Username / Password form on the login view       |

---

## Page Map

All views render at the same URL (`https://codemify-demo-app.vercel.app/demo-app`).  
Navigation is **state-based** — the URL never changes between views.

| View       | How to reach                                           | Heading on page                                      |
|------------|--------------------------------------------------------|------------------------------------------------------|
| Login      | Initial load, or after Logout                          | "🛒 Codemify Store" + "Please login to continue"     |
| Products   | After successful login, "Continue Shopping", logo click| "Products"                                           |
| Cart       | Click "Cart" in the nav bar                            | "Your Shopping Cart"                                 |
| Checkout   | Click "Proceed to Checkout" from the Cart view         | "Checkout"                                           |

There is **no product detail view** and **no order confirmation view**.

---

## Auth Method and Test Credentials

Credentials are displayed on the login page itself under "Accepted usernames are:" and "Password for all users:".

| Username         | Password       | Behavior                            |
|------------------|----------------|-------------------------------------|
| `standard_user`  | `my_secret_code` | Full access, reaches Products view |
| `locked_out_user`| `my_secret_code` | Login fails — locked out           |

`baseURL` is **not yet configured** in `playwright.config.js`. Use the full URL in all `page.goto()` calls until it is set.

---

## Key Technical Observations

### Locator Attributes

| Element                      | Attribute available                        | Example value              |
|------------------------------|--------------------------------------------|----------------------------|
| Login username input         | `data-test="username"`                     | —                          |
| Login password input         | `data-test="password"`                     | —                          |
| Login submit button          | `data-test="login-button"`                 | —                          |
| Checkout form inputs         | `id` + `name` (camelCase, matching)        | `id="firstName"`, `id="cardNumber"` |
| Nav buttons, product cards   | CSS classes only — no stable test IDs      | `.nav-link`, `.product-card` |

### Product Inventory (static, 6 items)

| Name                                | Price   |
|-------------------------------------|---------|
| Codemify Backpack                   | $29.99  |
| Codemify Bike Light                 | $9.99   |
| Codemify Bolt T-Shirt               | $15.99  |
| Codemify Fleece Jacket              | $49.99  |
| Codemify Onesie                     | $7.99   |
| Test.allTheThings() T-Shirt (Red)   | $15.99  |

### Key Behaviors

- Cart count badge appears on the "Cart" nav button after adding an item (e.g., button text becomes "Cart 1")
- "Add to Cart" button becomes visually active (pressed state) when that product is in the cart
- Cart supports quantity adjustment (− / + buttons) and per-item Remove
- After "Complete Order" in Checkout, the app redirects to the Products view and the cart is cleared
- Cart state lives in React component state — it resets on full page reload

### Checkout Form Fields

Shipping: `firstName`, `lastName`, `email`, `address`, `city`, `state`, `zipCode`  
Payment: `cardNumber`, `expiryDate` (format `MM/YY`), `cvv`

All fields are required (`*`). No real validation beyond required-field checks.

---

## Known Quirks

- One console error fires on every page load (image loading failure — not a test concern, safe to ignore)
- Clicking a product name or image does **not** navigate anywhere — there is no product detail view
- No order confirmation page — "Complete Order" lands back on Products
- `baseURL` is commented out in the config — tests must use the full URL for now
- The URL never changes between views — do not rely on `page.url()` to assert navigation

---

## What This App Does NOT Have

- Product detail / single-product page
- Order confirmation / thank you page
- Product search, sort, or filter controls
- Account management or profile page
- Real payment processing
- URL-based navigation
- Data persistence across browser sessions
