# Checkout Page — Test Cases

## Page Under Test

**URL:** `https://codemify-demo-app.vercel.app/demo-app` (state-based, URL does not change)  
**View:** Checkout (reached from Cart by clicking "Proceed to Checkout")  
**Prerequisite:** User must be logged in as `standard_user` with at least one item in cart

---

## Page Structure (verified live)

### Checkout form

| Element                      | Locator                                      | Type / Notes                                     |
|------------------------------|----------------------------------------------|--------------------------------------------------|
| Page heading                 | `h2` — "Checkout"                            | Confirms Checkout view                           |
| Order Summary heading        | `h3` — "Order Summary"                       | —                                                |
| Order item line              | `div` — "Product Name × Qty"                 | One line per unique product                      |
| Order item price             | `div` — "$X.XX"                              | Per-line price                                   |
| Order total                  | `div` — "Total: $X.XX"                       | Sum of all items                                 |
| Shipping heading             | `h3` — "Shipping Information"                | —                                                |
| First Name input             | `data-test="first-name-input"`, `id="firstName"` | `type="text"`, `required`                   |
| Last Name input              | `data-test="last-name-input"`, `id="lastName"`   | `type="text"`, `required`                   |
| Email input                  | `data-test="email-input"`, `id="email"`          | `type="email"`, `required`                  |
| Address input                | `data-test="address-input"`, `id="address"`      | `type="text"`, `required`                   |
| City input                   | `data-test="city-input"`, `id="city"`            | `type="text"`, `required`                   |
| State input                  | `data-test="state-input"`, `id="state"`          | `type="text"`, `required`                   |
| Zip Code input               | `data-test="zip-code-input"`, `id="zipCode"`     | `type="text"`, placeholder `12345 or 12345-6789`, `required` |
| Payment heading              | `h3` — "Payment Information"                 | —                                                |
| Card Number input            | `data-test="card-number-input"`, `id="cardNumber"` | `type="text"`, placeholder `1234567890123456`, `required` |
| Expiry Date input            | `data-test="expiry-date-input"`, `id="expiryDate"` | `type="text"`, placeholder `MM/YY`, `required` |
| CVV input                    | `data-test="cvv-input"`, `id="cvv"`              | `type="text"`, placeholder `123`, `required` |
| Cancel button                | `button` — "Cancel"                          | Returns to Cart view with items preserved        |
| Complete Order button        | `data-test="complete-order-button"`          | Submits the form                                 |

### Order Confirmation state (after successful submission)

| Element                      | Locator / Notes                                             |
|------------------------------|-------------------------------------------------------------|
| Success heading              | `h3` — "✅ Order Placed Successfully!"                      |
| Confirmation text            | `p` — "Thank you for your purchase. Your order has been confirmed." |
| Order total confirmation     | `p` — "Order Total: $X.XX"                                  |
| Continue Shopping button     | `button` — "Continue Shopping"                              |

---

## Test Cases

### TC-CH-01 — Checkout page renders with correct Order Summary

**Priority:** High  
**Type:** Smoke

**Steps:**
1. Log in, add Codemify Bike Light ($9.99) to cart
2. Go to Cart and click "Proceed to Checkout" (`data-test="checkout-button"`)

**Expected:**
- Heading "Checkout" (h2) is visible
- "Order Summary" section shows: "Codemify Bike Light × 1" and "$9.99"
- Total shows "$9.99"
- All 10 form fields are empty and visible
- "Cancel" and "Complete Order" buttons are visible
- URL remains `https://codemify-demo-app.vercel.app/demo-app`

---

### TC-CH-02 — Order Summary reflects all cart items and quantities

**Priority:** High  
**Type:** Content / Regression

**Steps:**
1. Log in, add Codemify Backpack (qty 2) and Codemify Bike Light (qty 1) to cart
2. Navigate to Checkout

**Expected:**
- Order Summary shows:
  - "Codemify Backpack × 2" — "$59.98"
  - "Codemify Bike Light × 1" — "$9.99"
- Total shows "$69.97"

---

### TC-CH-03 — All form fields are present with correct labels and placeholders

**Priority:** Medium  
**Type:** Content / Regression

**Steps:**
1. Navigate to Checkout

**Expected:**

| Field       | Label               | Placeholder              |
|-------------|---------------------|--------------------------|
| firstName   | First Name *        | —                        |
| lastName    | Last Name *         | —                        |
| email       | Email *             | —                        |
| address     | Address *           | —                        |
| city        | City *              | —                        |
| state       | State *             | —                        |
| zipCode     | Zip Code *          | `12345 or 12345-6789`    |
| cardNumber  | Card Number *       | `1234567890123456`       |
| expiryDate  | Expiry Date *       | `MM/YY`                  |
| cvv         | CVV *               | `123`                    |

---

### TC-CH-04 — Successful order completion shows confirmation screen

**Priority:** High  
**Type:** Positive / Happy path — End-to-end

**Steps:**
1. Log in, add Codemify Bike Light to cart, go to Checkout
2. Fill all fields with valid data:
   - First Name: `Jane`, Last Name: `Doe`
   - Email: `jane.doe@example.com`
   - Address: `123 Main St`, City: `Springfield`, State: `IL`, Zip: `62701`
   - Card Number: `1234567890123456`, Expiry: `12/27`, CVV: `123`
3. Click "Complete Order" (`data-test="complete-order-button"`)

**Expected:**
- Heading "✅ Order Placed Successfully!" (h3) is visible
- Text "Thank you for your purchase. Your order has been confirmed." is visible
- "Order Total: $9.99" is shown
- "Continue Shopping" button is visible
- URL remains unchanged

---

### TC-CH-05 — "Continue Shopping" after order confirmation returns to Products with cleared cart

**Priority:** High  
**Type:** Positive / Post-order flow

**Steps:**
1. Complete a successful order (see TC-CH-04)
2. Click "Continue Shopping" from the confirmation screen

**Expected:**
- Products page heading "Products" is visible
- Cart nav badge is gone — button shows "Cart" with no count
- Cart is cleared

---

### TC-CH-06 — Empty form submission triggers HTML5 native validation

**Priority:** High  
**Type:** Negative / Validation

**Steps:**
1. Navigate to Checkout with an item in cart
2. Leave all fields empty
3. Click "Complete Order"

**Expected:**
- Form submission is blocked
- First Name field receives focus (browser highlights first invalid field)
- Browser-native validation tooltip is shown (e.g., "Please fill in this field")
- User remains on the Checkout page
- All 10 inputs are in the CSS `:invalid` state

---

### TC-CH-07 — Missing First Name blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout with an item in cart
2. Fill all fields except First Name
3. Click "Complete Order"

**Expected:**
- Form is not submitted
- First Name field is focused / highlighted by browser
- User stays on Checkout

---

### TC-CH-08 — Missing Last Name blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout with an item in cart
2. Fill all fields except Last Name
3. Click "Complete Order"

**Expected:**
- Form is not submitted
- Last Name field is focused / highlighted by browser
- User stays on Checkout

---

### TC-CH-09 — Missing Email blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout with an item in cart
2. Fill all fields except Email
3. Click "Complete Order"

**Expected:**
- Form is not submitted
- Email field is focused / highlighted
- User stays on Checkout

---

### TC-CH-10 — Invalid email format blocks form submission

**Priority:** Medium  
**Type:** Negative / Email validation

**Steps:**
1. Navigate to Checkout with an item in cart
2. Fill all fields, but set Email to `not-an-email`
3. Click "Complete Order"

**Expected:**
- Form is not submitted
- Browser-native email validation error is shown (e.g., "Please include an '@' in the email address")
- User stays on Checkout

---

### TC-CH-11 — Missing Address blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except Address, click "Complete Order"

**Expected:**
- Form not submitted; Address field is focused by browser

---

### TC-CH-12 — Missing City blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except City, click "Complete Order"

**Expected:**
- Form not submitted; City field is focused by browser

---

### TC-CH-13 — Missing State blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except State, click "Complete Order"

**Expected:**
- Form not submitted; State field is focused by browser

---

### TC-CH-14 — Missing Zip Code blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except Zip Code, click "Complete Order"

**Expected:**
- Form not submitted; Zip Code field is focused by browser

---

### TC-CH-15 — Missing Card Number blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except Card Number, click "Complete Order"

**Expected:**
- Form not submitted; Card Number field is focused by browser

---

### TC-CH-16 — Missing Expiry Date blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except Expiry Date, click "Complete Order"

**Expected:**
- Form not submitted; Expiry Date field is focused by browser

---

### TC-CH-17 — Missing CVV blocks form submission

**Priority:** High  
**Type:** Negative / Required field

**Steps:**
1. Navigate to Checkout, fill all fields except CVV, click "Complete Order"

**Expected:**
- Form not submitted; CVV field is focused by browser

---

### TC-CH-18 — "Cancel" returns to Cart view with items preserved

**Priority:** High  
**Type:** Navigation / Cancellation

**Steps:**
1. Log in, add Codemify Bike Light to cart, navigate to Checkout
2. Fill some form fields (partial data)
3. Click "Cancel"

**Expected:**
- "Your Shopping Cart" heading is visible
- Codemify Bike Light is still in the cart (cart is NOT cleared)
- Cart count badge is preserved
- Total still shows the correct amount

---

### TC-CH-19 — Checkout is not directly reachable without items in cart

**Priority:** Medium  
**Type:** Guard behavior

**Steps:**
1. Log in with no items in cart
2. Navigate to Cart (shows "Your cart is empty")
3. Observe whether "Proceed to Checkout" button is present

**Expected:**
- "Proceed to Checkout" button is NOT visible on the empty cart
- Only "Continue Shopping" is shown

---

## Locator Reference (Checkout Page)

| Element                     | Recommended Locator                                                 |
|-----------------------------|---------------------------------------------------------------------|
| Checkout heading            | `page.getByRole('heading', { name: 'Checkout' })`                  |
| First Name input            | `page.getByTestId('first-name-input')`                             |
| Last Name input             | `page.getByTestId('last-name-input')`                              |
| Email input                 | `page.getByTestId('email-input')`                                  |
| Address input               | `page.getByTestId('address-input')`                                |
| City input                  | `page.getByTestId('city-input')`                                   |
| State input                 | `page.getByTestId('state-input')`                                  |
| Zip Code input              | `page.getByTestId('zip-code-input')`                               |
| Card Number input           | `page.getByTestId('card-number-input')`                            |
| Expiry Date input           | `page.getByTestId('expiry-date-input')`                            |
| CVV input                   | `page.getByTestId('cvv-input')`                                    |
| Complete Order button       | `page.getByTestId('complete-order-button')`                        |
| Cancel button               | `page.getByRole('button', { name: 'Cancel' })`                     |
| Order success heading       | `page.getByRole('heading', { name: /Order Placed Successfully/ })` |
| Confirmation message        | `page.getByText('Thank you for your purchase')`                    |
| Post-order Continue Shopping| `page.getByRole('button', { name: 'Continue Shopping' })`          |

---

## Known Behaviors (confirmed live)

| Behavior                                               | Detail                                                                   |
|--------------------------------------------------------|--------------------------------------------------------------------------|
| Form validation type                                   | HTML5 native `required` attribute — browser-native tooltips, NOT custom messages |
| Email field validation                                 | `type="email"` — browser validates format automatically                  |
| Zip Code no format validation                          | No real validation beyond `required`; any non-empty string is accepted   |
| Card number no format validation                       | No real validation beyond `required`; any non-empty string is accepted   |
| After complete order                                   | Shows Order Confirmation screen (NOT a redirect directly to Products)    |
| Cart after order                                       | Cleared only after clicking "Continue Shopping" from the confirmation    |
| Cancel action                                          | Returns to Cart — cart items are preserved, NOT cleared                  |

---

## Notes for Test Implementation

- All 10 checkout fields have both `data-test` attributes and matching `id` attributes — prefer `data-test` for stability.
- The email field is `type="email"`, so the browser enforces format; no custom error message to assert.
- No real payment processing occurs — any value satisfies the required check.
- After a successful order, assert the confirmation heading (`✅ Order Placed Successfully!`) rather than a redirect.
- The URL never changes — do not rely on `page.url()` for navigation assertions.
- `baseURL` is not configured in `playwright.config.js`; use the full URL in `page.goto()`.
