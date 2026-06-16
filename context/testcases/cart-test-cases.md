# Cart Page — Test Cases

## Page Under Test

**URL:** `https://codemify-demo-app.vercel.app/demo-app` (state-based, URL does not change)  
**View:** Cart (reached by clicking "Cart" or "Cart N" in the nav bar)  
**Prerequisite:** User must be logged in as `standard_user`

---

## Page Structure (verified live)

### Cart with items

| Element                    | Locator / Selector                                   | Notes                                        |
|----------------------------|------------------------------------------------------|----------------------------------------------|
| Page heading               | `h2` — "Your Shopping Cart"                          | Confirms Cart view is active                 |
| Cart item list             | `ul` — one `li` per unique product                   | —                                            |
| Item image                 | `img` with alt = product name                        | —                                            |
| Item name heading          | `h3` — product name                                  | —                                            |
| Item price + qty formula   | `div` — "$X.XX × N = $Y.YY"                          | Updates live as qty changes                  |
| Decrease qty button        | `button` — "−", `data-test="decrease-quantity-{id}"` | Becomes `[active]` at qty=1; auto-removes at 0 |
| Quantity display           | `div` — current quantity number                      | —                                            |
| Increase qty button        | `button` — "+", `data-test="increase-quantity-{id}"` | —                                            |
| Remove button              | `button` — "Remove", `data-test="remove-from-cart-{id}"` | Removes item entirely regardless of qty  |
| Total label                | `div` — "Total:"                                     | —                                            |
| Total amount               | `div` — "$X.XX"                                      | Sum of all items × quantities                |
| Continue Shopping button   | `button` — "Continue Shopping"                       | Returns to Products view                     |
| Proceed to Checkout button | `button` — "Proceed to Checkout", `data-test="checkout-button"` | Goes to Checkout view            |
| Cart nav button (active)   | `button` — "Cart N" `[active]`                       | Active state while on Cart view              |

### Empty cart

| Element                    | Locator / Notes                                      |
|----------------------------|------------------------------------------------------|
| Page heading               | `h2` — "Your Shopping Cart"                          |
| Empty message              | `p` — "Your cart is empty"                           |
| Continue Shopping button   | `button` — "Continue Shopping"                       |
| No item list or total shown | — |

---

## Test Cases

### TC-CT-01 — Cart view renders correctly with one item

**Priority:** High  
**Type:** Smoke

**Steps:**
1. Log in as `standard_user`
2. Add Codemify Backpack (`data-test="add-to-cart-1"`) to cart
3. Click the "Cart 1" nav button

**Expected:**
- Heading "Your Shopping Cart" is visible
- Codemify Backpack is listed with: image, name, description, price `$29.99 × 1 = $29.99`
- Quantity controls show: "−" button, "1", "+" button
- "Remove" button is visible
- Total shows `$29.99`
- "Continue Shopping" button is visible
- "Proceed to Checkout" button is visible
- Cart nav button shows `[active]` state

---

### TC-CT-02 — Cart view shows "Your cart is empty" when no items added

**Priority:** High  
**Type:** Edge case / Empty state

**Steps:**
1. Log in as `standard_user`
2. Click the "Cart" nav button without adding any items

**Expected:**
- Heading "Your Shopping Cart" is visible
- Text "Your cart is empty" is displayed
- "Continue Shopping" button is visible
- No item list, quantity controls, total, or "Proceed to Checkout" button are shown

---

### TC-CT-03 — Increase quantity with "+" button updates price and total

**Priority:** High  
**Type:** Positive / Quantity control

**Steps:**
1. Log in, add Codemify Bike Light (`data-test="add-to-cart-2"`), go to Cart
2. Click the "+" button (`data-test="increase-quantity-2"`)

**Expected:**
- Quantity display updates to "2"
- Item line updates to `$9.99 × 2 = $19.98`
- Total updates to `$19.98`
- Cart nav badge updates to "Cart 2"

---

### TC-CT-04 — Decrease quantity with "−" button updates price and total

**Priority:** High  
**Type:** Positive / Quantity control

**Steps:**
1. Log in, add Codemify Bike Light twice (qty = 2), go to Cart
2. Click the "−" button (`data-test="decrease-quantity-2"`)

**Expected:**
- Quantity display updates to "1"
- Item line updates to `$9.99 × 1 = $9.99`
- Total updates to `$9.99`
- Cart nav badge updates to "Cart 1"

---

### TC-CT-05 — "−" button at quantity 1 becomes active (minimum reached)

**Priority:** Medium  
**Type:** Visual state / Edge case

**Steps:**
1. Log in, add Codemify Backpack once (qty = 1), go to Cart
2. Observe the "−" button state

**Expected:**
- The "−" button shows `[active]` visual state when quantity is exactly 1
- This indicates 1 is the minimum quantity before auto-removal

---

### TC-CT-06 — Clicking "−" at quantity 1 auto-removes item and shows empty cart

**Priority:** High  
**Type:** Edge case / Auto-remove

**Steps:**
1. Log in, add Codemify Backpack once (qty = 1), go to Cart
2. Click the "−" button (`data-test="decrease-quantity-1"`)

**Expected:**
- Codemify Backpack is removed from the cart
- "Your cart is empty" message appears
- Cart nav badge disappears (button shows "Cart" with no number)
- Total and item list are no longer visible

---

### TC-CT-07 — "Remove" button removes item entirely regardless of quantity

**Priority:** High  
**Type:** Positive / Remove

**Steps:**
1. Log in, add Codemify Backpack twice (qty = 2), go to Cart
2. Click the "Remove" button (`data-test="remove-from-cart-1"`)

**Expected:**
- Codemify Backpack is completely removed from the cart (not just reduced to 1)
- Cart count drops by the item's full quantity (2 → 0 in this case)
- If cart becomes empty: shows "Your cart is empty"
- Total updates to reflect removal

---

### TC-CT-08 — Cart displays multiple different products with correct totals

**Priority:** High  
**Type:** Positive / Multi-item

**Steps:**
1. Log in, add Codemify Backpack ($29.99) and Codemify Bike Light ($9.99)
2. Go to Cart

**Expected:**
- Both products are listed separately
- Each shows qty 1: `$29.99 × 1 = $29.99` and `$9.99 × 1 = $9.99`
- Total shows `$39.98`
- Cart nav badge shows "Cart 2"

---

### TC-CT-09 — Total updates correctly when mixing quantities of multiple items

**Priority:** Medium  
**Type:** Positive / Calculation accuracy

**Steps:**
1. Log in, add Codemify Backpack ($29.99) and Codemify Bike Light ($9.99)
2. Go to Cart
3. Increase Backpack qty to 2

**Expected:**
- Backpack line: `$29.99 × 2 = $59.98`
- Bike Light line: `$9.99 × 1 = $9.99`
- Total: `$69.97`
- Cart badge: "Cart 3"

---

### TC-CT-10 — "Continue Shopping" returns to Products view with cart preserved

**Priority:** High  
**Type:** Navigation / Session

**Steps:**
1. Log in, add Codemify Backpack to cart, go to Cart
2. Click "Continue Shopping"

**Expected:**
- Products page heading "Products" is visible
- Cart nav badge still shows "Cart 1" (cart is NOT cleared)
- User is back on the Products view

---

### TC-CT-11 — "Continue Shopping" from empty cart also navigates to Products

**Priority:** Medium  
**Type:** Navigation / Empty state

**Steps:**
1. Log in, navigate to Cart without any items
2. Click "Continue Shopping"

**Expected:**
- Products page heading "Products" is visible
- Cart nav button shows no badge (remains empty)

---

### TC-CT-12 — "Proceed to Checkout" navigates to Checkout view

**Priority:** High  
**Type:** Navigation / Core flow

**Steps:**
1. Log in, add at least one item to cart, go to Cart
2. Click "Proceed to Checkout" (`data-test="checkout-button"`)

**Expected:**
- Heading "Checkout" (h2) is visible
- Order Summary section shows the items from the cart
- Shipping Information form is displayed
- Payment Information form is displayed

---

### TC-CT-13 — Cart state resets on full page reload

**Priority:** Medium  
**Type:** Behavior / Session

**Steps:**
1. Log in, add Codemify Backpack to cart
2. Perform a full page reload (`page.reload()`)

**Expected:**
- Cart nav badge disappears (shows "Cart" with no count)
- Navigating to Cart shows "Your cart is empty"

---

## Locator Reference (Cart Page)

| Element                       | Recommended Locator                                          |
|-------------------------------|--------------------------------------------------------------|
| Cart heading                  | `page.getByRole('heading', { name: 'Your Shopping Cart' })` |
| Empty cart message            | `page.getByText('Your cart is empty')`                       |
| Decrease qty (by product id)  | `page.getByTestId('decrease-quantity-1')`                    |
| Increase qty (by product id)  | `page.getByTestId('increase-quantity-1')`                    |
| Remove item (by product id)   | `page.getByTestId('remove-from-cart-1')`                     |
| Proceed to Checkout button    | `page.getByTestId('checkout-button')`                        |
| Continue Shopping button      | `page.getByRole('button', { name: 'Continue Shopping' })`   |
| Total amount                  | — (assert via text, e.g., `page.getByText('$39.98')`)        |
| Cart nav button               | `page.getByRole('button', { name: /Cart/ })`                 |

---

## Notes for Test Implementation

- Cart state lives in React component state and resets on full page reload — always add items fresh within the same test session.
- The `−` button `data-test` follows the pattern `decrease-quantity-{product-id}` (same IDs as `add-to-cart-{id}`).
- Clicking `−` at quantity 1 auto-removes the item; the button does NOT prevent the action.
- The URL never changes between views — assert headings to confirm navigation.
- `baseURL` is not configured; use the full URL in `page.goto()`.
