# Products Page — Test Cases

## Page Under Test

**URL:** `https://codemify-demo-app.vercel.app/demo-app` (state-based, same URL for all views)  
**View:** Products (reached after successful login, "Continue Shopping", or logo click)  
**Prerequisite:** User must be logged in as `standard_user`

---

## Page Structure (verified live)

| Element                     | Locator / Selector                              | Notes                                     |
|-----------------------------|--------------------------------------------------|-------------------------------------------|
| Nav: Store logo heading     | `h1` — "🛒 Codemify Store"                       | Clickable, returns to Products            |
| Nav: Products button        | `button` — "Products"                            | CSS class `.nav-link`, no data-test       |
| Nav: Cart button            | `button` — "Cart" / "Cart N"                     | Badge `N` appears when items in cart      |
| Nav: Logout button          | `button` — "Logout"                              | CSS class `.nav-link logout-button`       |
| Page heading                | `h2` — "Products"                                | Confirms Products view is active          |
| Item count                  | `div` — "6 items"                                | Static, always 6                          |
| Product list                | `ul` containing 6 `li` items                     | —                                         |
| Product image               | `img` with alt = product name                    | No navigation on click                    |
| Product name                | `h3` — product name                              | No navigation on click                    |
| Product price               | `div` — "$X.XX"                                  | —                                         |
| Add to Cart button          | `button` — "Add to Cart", `data-test="add-to-cart-{id}"` | `[active]` when item is in cart |
| Footer                      | `p` — "© 2025 Codemify Store - Demo Application for Testing" | — |

### Product Inventory (all 6, static)

| # | Name                                  | Price   | data-test ID          |
|---|---------------------------------------|---------|-----------------------|
| 1 | Codemify Backpack                     | $29.99  | `add-to-cart-1`       |
| 2 | Codemify Bike Light                   | $9.99   | `add-to-cart-2`       |
| 3 | Codemify Bolt T-Shirt                 | $15.99  | `add-to-cart-3`       |
| 4 | Codemify Fleece Jacket                | $49.99  | `add-to-cart-4`       |
| 5 | Codemify Onesie                       | $7.99   | `add-to-cart-5`       |
| 6 | Test.allTheThings() T-Shirt (Red)     | $15.99  | `add-to-cart-6`       |

---

## Test Cases

### TC-PR-01 — Products page renders correctly after login

**Priority:** High  
**Type:** Smoke

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Log in as `standard_user` / `my_secret_code`

**Expected:**
- Heading "Products" (h2) is visible
- "6 items" count is visible
- All 6 product cards are displayed, each showing: image, name, price, and "Add to Cart" button
- Nav bar shows "Products", "Cart", and "Logout" buttons
- Cart button shows "Cart" with no count badge (cart is empty)
- URL remains `https://codemify-demo-app.vercel.app/demo-app`

---

### TC-PR-02 — All 6 products are listed with correct names and prices

**Priority:** High  
**Type:** Content / Regression

**Steps:**
1. Log in and reach the Products view

**Expected:**

| Product Name                          | Price   |
|---------------------------------------|---------|
| Codemify Backpack                     | $29.99  |
| Codemify Bike Light                   | $9.99   |
| Codemify Bolt T-Shirt                 | $15.99  |
| Codemify Fleece Jacket                | $49.99  |
| Codemify Onesie                       | $7.99   |
| Test.allTheThings() T-Shirt (Red)     | $15.99  |

---

### TC-PR-03 — Add single item to cart updates Cart badge

**Priority:** High  
**Type:** Positive / Core flow

**Steps:**
1. Log in and reach the Products view
2. Click "Add to Cart" for Codemify Backpack (`data-test="add-to-cart-1"`)

**Expected:**
- Cart nav button updates from "Cart" to "Cart 1"
- The Backpack's "Add to Cart" button enters `[active]` (visually pressed) state
- User stays on the Products page
- URL does not change

---

### TC-PR-04 — Add to Cart button becomes active for in-cart item

**Priority:** Medium  
**Type:** Positive / Visual state

**Steps:**
1. Log in and reach the Products view
2. Click "Add to Cart" for Codemify Bike Light (`data-test="add-to-cart-2"`)

**Expected:**
- The Bike Light "Add to Cart" button shows `[active]` visual state
- Other products' "Add to Cart" buttons remain in the default (inactive) state

---

### TC-PR-05 — Clicking active Add to Cart button adds another unit (not a toggle)

**Priority:** High  
**Type:** Behavior / Edge case

**Steps:**
1. Log in and reach the Products view
2. Click "Add to Cart" for Codemify Backpack once (cart = 1)
3. Click the same button again

**Expected:**
- Cart badge updates to "Cart 2"
- The button remains in `[active]` state
- The item is NOT removed — a second unit is added instead
- No toggle-off behavior on the product card

---

### TC-PR-06 — Add multiple different products to cart

**Priority:** High  
**Type:** Positive / Multi-item

**Steps:**
1. Log in and reach the Products view
2. Add Codemify Backpack (`data-test="add-to-cart-1"`)
3. Add Codemify Bike Light (`data-test="add-to-cart-2"`)

**Expected:**
- Cart badge shows "Cart 2"
- Both products' "Add to Cart" buttons show `[active]` state

---

### TC-PR-07 — Clicking product name does NOT navigate

**Priority:** Medium  
**Type:** Negative / Non-navigable element

**Steps:**
1. Log in and reach the Products view
2. Click on the product name heading (e.g., "Codemify Backpack")

**Expected:**
- User stays on the Products page
- No navigation, no modal, no detail view opens
- Page content is unchanged

---

### TC-PR-08 — Clicking product image does NOT navigate

**Priority:** Medium  
**Type:** Negative / Non-navigable element

**Steps:**
1. Log in and reach the Products view
2. Click on a product image (e.g., the Backpack image)

**Expected:**
- User stays on the Products page
- No navigation or detail view opens

---

### TC-PR-09 — Clicking logo navigates to Products view

**Priority:** Medium  
**Type:** Navigation

**Steps:**
1. Log in and navigate to the Cart view
2. Click the "🛒 Codemify Store" heading logo in the nav bar

**Expected:**
- Products page heading "Products" is visible
- All 6 products are listed
- Cart count badge is preserved (cart is not cleared)

---

### TC-PR-10 — Clicking Products nav button returns to Products view

**Priority:** Medium  
**Type:** Navigation

**Steps:**
1. Log in and navigate to Cart or Checkout
2. Click the "Products" button in the nav bar

**Expected:**
- Products page heading "Products" is visible
- All 6 products are listed

---

### TC-PR-11 — Cart badge count reflects total units across all products

**Priority:** Medium  
**Type:** Positive / Count accuracy

**Steps:**
1. Log in and reach the Products view
2. Click "Add to Cart" on Codemify Backpack (count = 1)
3. Click "Add to Cart" on Codemify Backpack again (count = 2)
4. Click "Add to Cart" on Codemify Bike Light (count = 3)

**Expected:**
- Cart badge shows "Cart 3" (total units, not total unique products)

---

### TC-PR-12 — Products page is not accessible without login (redirect to Login)

**Priority:** High  
**Type:** Security / Auth guard

**Steps:**
1. Navigate directly to `https://codemify-demo-app.vercel.app/demo-app` without logging in

**Expected:**
- Login form is shown (heading "🛒 Codemify Store" + "Please login to continue")
- Products view is NOT shown
- Nav bar is NOT visible

---

## Locator Reference (Products Page)

| Element                      | Recommended Locator                                |
|------------------------------|----------------------------------------------------|
| Products heading             | `page.getByRole('heading', { name: 'Products' })`  |
| Item count                   | `page.getByText('6 items')`                        |
| Add to Cart (by product id)  | `page.getByTestId('add-to-cart-1')` … `add-to-cart-6` |
| Cart nav button              | `page.getByRole('button', { name: /Cart/ })`       |
| Cart badge text              | `page.getByRole('button', { name: 'Cart 1' })`     |
| Logo / store heading         | `page.getByRole('heading', { name: '🛒 Codemify Store' })` |

---

## Notes for Test Implementation

- The URL never changes between views — do not assert `page.url()` to confirm navigation; assert headings instead.
- Cart state resets on full page reload — add items fresh in each test that needs cart state.
- `baseURL` is not configured in `playwright.config.js`; use the full URL in `page.goto()`.
- One console error fires on every load (image load failure) — safe to ignore.
- Add to Cart buttons use `data-test="add-to-cart-{id}"` where `{id}` corresponds to product position (1–6).
