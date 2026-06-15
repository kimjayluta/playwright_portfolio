# Business Rules — SauceDemo

This file documents how the SauceDemo application behaves: user credentials, page flows, UI behaviors, and edge cases. Read this before writing any test assertion — do not invent expected behavior that is not documented here.

---

## User Credentials

All users share the same password: `secret_sauce`

| Username | Password | Behavior |
|---|---|---|
| `standard_user` | `secret_sauce` | Fully working — use for happy path tests |
| `locked_out_user` | `secret_sauce` | Login blocked — error message displayed, never reaches inventory |
| `problem_user` | `secret_sauce` | Product images broken, some interactions glitchy |
| `performance_glitch_user` | `secret_sauce` | Login takes ~5 seconds (artificial delay) |
| `error_user` | `secret_sauce` | Some cart/form interactions throw errors |
| `visual_user` | `secret_sauce` | Visual layout differences vs standard user |

**Default test user:** `standard_user` — use this unless you are explicitly testing user-specific behavior.

---

## Page Flows

### Happy Path (standard_user)

```
Login → Inventory → (optional: Product Detail) → Add to Cart → Cart →
Checkout Info → Checkout Overview → Checkout Complete → Back to Inventory
```

### Flow Rules

- Accessing `/inventory.html` without login redirects to `/` (login page)
- Cart persists within the same browser context session
- Checkout requires three fields: **First Name**, **Last Name**, **Zip/Postal Code**
- After Checkout Complete, the "Back Home" button returns to `/inventory.html`
- You cannot reach Checkout Overview without completing Checkout Info

---

## Key UI Behaviors

### Cart Badge

- Displays the count of items currently in the cart
- `data-test="shopping-cart-badge"`
- Appears on all post-login pages (inside the header)
- **Disappears entirely** when the cart is empty (element not in DOM — do not assert `toHaveText('0')`)

### Inventory Sorting

- Dropdown on the Inventory page: `data-test="product-sort-container"`
- Options: **Name (A to Z)**, **Name (Z to A)**, **Price (low to high)**, **Price (high to low)**
- Sorting changes visible product order immediately, no page reload

### Add to Cart / Remove

- Each product on Inventory page has an **Add to cart** button
- After clicking, the button label changes to **Remove**
- Same toggle behavior on the Product Detail page
- Clicking **Remove** removes the item and decrements the cart badge

---

## data-test Attribute Reference

These are the verified `data-test` values for key elements. MUST use `getByTestId()` for all of these.

| Element | data-test value |
|---|---|
| Username input | `username` |
| Password input | `password` |
| Login button | `login-button` |
| Error message container | `error` |
| Inventory list container | `inventory-list` |
| Product sort dropdown | `product-sort-container` |
| Cart link (header icon) | `shopping-cart-link` |
| Cart badge (item count) | `shopping-cart-badge` |
| Checkout button (in cart) | `checkout` |
| First name field | `firstName` |
| Last name field | `lastName` |
| Postal code field | `postalCode` |
| Continue button (checkout info) | `continue` |
| Finish button (checkout overview) | `finish` |
| Complete header (thank you message) | `complete-header` |
| Back Home button (checkout complete) | `back-to-products` |

---

## Edge Cases Worth Testing

### locked_out_user

- Exact error message: `"Epic sadface: Sorry, this user has been locked out."`
- Error container: `data-test="error"`
- User stays on login page — URL remains `https://www.saucedemo.com/`

### Empty Checkout Fields

- Submitting without First Name → error: `"Error: First Name is required"`
- Submitting without Last Name → error: `"Error: Last Name is required"`
- Submitting without Postal Code → error: `"Error: Postal Code is required"`
- Each field triggers its own distinct error message

### Cart Persistence Within Session

- Cart count survives navigation between Inventory and Product Detail pages
- Cart is cleared on logout (session ends)
- Playwright resets cart automatically between tests (new browser context per test)

### Add/Remove Consistency

- Adding item from Inventory page and viewing in Product Detail — both show "Remove"
- Cart badge count stays accurate through multiple add/remove cycles

---

## What the App Does NOT Have

- No email verification
- No real payment processing
- No persistent user accounts (credentials are hardcoded)
- No admin panel
- No search functionality
- No product stock limits
