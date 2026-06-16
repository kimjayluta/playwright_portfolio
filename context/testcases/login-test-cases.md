# Login Page — Test Cases

## Page Under Test

**URL:** `https://codemify-demo-app.vercel.app/demo-app`  
**View:** Login (initial load, or after Logout)

---

## Page Structure (verified live)

| Element               | Locator                          | Input type |
|-----------------------|----------------------------------|------------|
| Page heading          | `h1` — "🛒 Codemify Store"       | —          |
| Subheading            | `p` — "Please login to continue" | —          |
| Username field        | `[data-test="username"]`         | `text`     |
| Password field        | `[data-test="password"]`         | `password` |
| Login button          | `[data-test="login-button"]`     | button     |
| Error message area    | appears between password + button after submit | —  |
| Credentials hint box  | static text listing accepted usernames + shared password | — |

---

## Test Cases

### TC-LG-01 — Login page renders on initial load

**Priority:** High  
**Type:** Smoke

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`

**Expected:**
- Page title is "E-Commerce Demo"
- Heading "🛒 Codemify Store" is visible
- Subheading "Please login to continue" is visible
- Username input (`[data-test="username"]`) is visible and empty
- Password input (`[data-test="password"]`) is visible and empty
- Login button (`[data-test="login-button"]`) with text "Login" is visible
- Credentials hint box is visible listing `standard_user`, `locked_out_user`, and `my_secret_code`
- Nav bar (Products / Cart / Logout) is **not** visible

---

### TC-LG-02 — Successful login with valid credentials

**Priority:** High  
**Type:** Positive / Happy path

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Fill `[data-test="username"]` with `standard_user`
3. Fill `[data-test="password"]` with `my_secret_code`
4. Click `[data-test="login-button"]`

**Expected:**
- Login form disappears
- "Products" heading (h2) is visible
- "6 items" count is visible
- Nav bar shows "Products", "Cart", and "Logout" buttons
- URL remains `https://codemify-demo-app.vercel.app/demo-app` (no URL change)
- No error message is shown

---

### TC-LG-03 — Submit with both fields empty

**Priority:** High  
**Type:** Negative / Validation

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Leave both fields empty
3. Click `[data-test="login-button"]`

**Expected:**
- User stays on the Login page
- Error message "Username is required" is visible
- No navigation occurs

---

### TC-LG-04 — Submit with username filled, password empty

**Priority:** High  
**Type:** Negative / Validation

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Fill `[data-test="username"]` with `standard_user`
3. Leave `[data-test="password"]` empty
4. Click `[data-test="login-button"]`

**Expected:**
- User stays on the Login page
- Error message "Password is required" is visible
- No navigation occurs

---

### TC-LG-05 — Submit with password filled, username empty

**Priority:** Medium  
**Type:** Negative / Validation

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Leave `[data-test="username"]` empty
3. Fill `[data-test="password"]` with `my_secret_code`
4. Click `[data-test="login-button"]`

**Expected:**
- User stays on the Login page
- Error message "Username is required" is visible
- No navigation occurs

---

### TC-LG-06 — Login with invalid username and invalid password

**Priority:** High  
**Type:** Negative / Security

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Fill `[data-test="username"]` with `wrong_user`
3. Fill `[data-test="password"]` with `wrong_password`
4. Click `[data-test="login-button"]`

**Expected:**
- User stays on the Login page
- Error message "Username and password do not match any user in this service" is visible
- No navigation occurs

---

### TC-LG-07 — Login with valid username but wrong password

**Priority:** High  
**Type:** Negative / Security

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Fill `[data-test="username"]` with `standard_user`
3. Fill `[data-test="password"]` with `wrong_pass`
4. Click `[data-test="login-button"]`

**Expected:**
- User stays on the Login page
- Error message "Username and password do not match any user in this service" is visible
- No navigation occurs

---

### TC-LG-08 — Login with locked out user

**Priority:** High  
**Type:** Negative / Access Control

**Steps:**
1. Navigate to `https://codemify-demo-app.vercel.app/demo-app`
2. Fill `[data-test="username"]` with `locked_out_user`
3. Fill `[data-test="password"]` with `my_secret_code`
4. Click `[data-test="login-button"]`

**Expected:**
- User stays on the Login page
- Error message "Sorry, this user has been locked out." is visible
- No navigation occurs

---

### TC-LG-09 — Logout returns user to Login page

**Priority:** High  
**Type:** Positive / Session

**Steps:**
1. Log in as `standard_user` with `my_secret_code` (see TC-LG-02)
2. Click the "Logout" button in the nav bar

**Expected:**
- Products view disappears
- Login form is visible again with empty fields
- Heading "🛒 Codemify Store" and subheading "Please login to continue" are visible
- Nav bar (Products / Cart / Logout) is no longer visible
- URL remains `https://codemify-demo-app.vercel.app/demo-app`

---

### TC-LG-10 — Login page is not accessible after login (session guard)

**Priority:** Medium  
**Type:** Positive / Session

**Steps:**
1. Log in as `standard_user` with `my_secret_code`
2. Reload the page (`page.reload()`)

**Expected:**
- Products view is still shown (session persists across reload)
- Login form is NOT shown
- Nav bar remains visible

---

## Error Message Reference (confirmed live)

| Scenario                                      | Exact error text                                             |
|-----------------------------------------------|--------------------------------------------------------------|
| Both fields empty                             | `Username is required`                                       |
| Username filled, password empty               | `Password is required`                                       |
| Password filled, username empty               | `Username is required`                                       |
| Invalid username + invalid password           | `Username and password do not match any user in this service`|
| Valid username + wrong password               | `Username and password do not match any user in this service`|
| Locked out user + correct password            | `Sorry, this user has been locked out.`                      |

---

## Notes for Test Implementation

- Use `[data-test="username"]`, `[data-test="password"]`, `[data-test="login-button"]` as primary locators — they are stable.
- The URL never changes between views; do not assert `page.url()` to confirm navigation — assert visible headings instead.
- `baseURL` is not configured in `playwright.config.js`; use the full URL in `page.goto()` calls.
- One console error fires on every load (image load failure) — safe to ignore in tests.
- Cart state resets on full page reload; session state does not.
