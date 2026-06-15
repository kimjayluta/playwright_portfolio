# Bug Report Template

When a test reveals that the **app** is wrong (not the test), document it here using this template. Do NOT silently adjust test assertions to match broken app behavior.

---

## When to File a Bug Report vs When to Fix the Test

| Situation | Action |
|---|---|
| Assertion fails because **expected behavior changed** (intentional update) | Update the test |
| Assertion fails because **the app has a real defect** | File a bug report, mark test with `test.fixme()` |
| New behavior contradicts what's in `business-rules.md` | File a bug report |
| Element is missing, broken image, wrong error message | File a bug report |

---

## How to Mark a Test as Blocked by a Bug

When you find a bug, mark the failing test with `test.fixme()` and reference the Bug ID in a comment. MUST NOT delete the test or change the assertion to pass against broken behavior.

```js
test.fixme('displays correct product images on inventory page', async ({ page }) => {
  // BUG-001: problem_user sees broken images — filed 2026-06-15
  // Expected: all product images load correctly
  // Actual: images show broken icon (404 src)
});
```

MUST NOT use `test.skip()` without a bug reference comment — skipped tests with no context become invisible debt.

---

## Bug Report Template

Copy the block below for each bug found. File it in the appropriate spec file as a comment block above the `test.fixme()`, or in a separate `bugs/` folder if the project grows.

```
---
Bug ID:        BUG-[sequential number, e.g. BUG-001]
Date Found:    [YYYY-MM-DD]
Found By:      [test name or "manual exploration"]
Status:        Open

Summary:
  [One sentence: what is broken and where]

Severity:
  [ ] Critical — app is unusable or data is corrupted
  [ ] High — major feature broken, no workaround
  [x] Medium — feature broken, workaround exists
  [ ] Low — cosmetic or minor inconvenience

Steps to Reproduce:
  1. Go to [URL]
  2. [Action]
  3. [Action]
  4. Observe result

Expected Result:
  [What should happen, per business-rules.md or reasonable expectation]

Actual Result:
  [What actually happened — include exact error messages verbatim]

Affected User Types:
  [ ] standard_user
  [ ] locked_out_user
  [ ] problem_user
  [ ] performance_glitch_user
  [ ] error_user
  [ ] visual_user

Test File:
  [tests/specs/feature.spec.js — line number if known]

Notes:
  [Console errors, Playwright trace URL, screenshot path, or any extra context]
---
```

---

## Example — Completed Bug Report

```
---
Bug ID:        BUG-001
Date Found:    2026-06-15
Found By:      tests/specs/inventory.spec.js — "displays product images"
Status:        Open

Summary:
  problem_user sees broken/missing product images on the inventory page.

Severity:
  [x] Medium — feature broken, workaround exists (use standard_user)

Steps to Reproduce:
  1. Go to https://www.saucedemo.com/
  2. Login with username: problem_user, password: secret_sauce
  3. Observe the product list on /inventory.html

Expected Result:
  All product images load correctly (per business-rules.md: problem_user has "broken images").

Actual Result:
  All product images show a broken image icon — the src attribute points to an invalid path.

Affected User Types:
  [ ] standard_user
  [ ] locked_out_user
  [x] problem_user
  [ ] performance_glitch_user
  [ ] error_user
  [ ] visual_user

Test File:
  tests/specs/inventory.spec.js

Notes:
  This is a known intentional defect in SauceDemo (used to test visual regression tools).
  The test remains as test.fixme() to document the expected state.
---
```
