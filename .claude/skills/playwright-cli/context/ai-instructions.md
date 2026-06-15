# AI Instructions — Playwright Test Generation

These are the rules Claude MUST follow when generating any Playwright test code for this project. Read this file first, then read the referenced context files before writing a single line of code.

---

## Step 1 — Read These Files Before Generating Any Test

| File | What it tells you |
|---|---|
| `context/project-context.md` | Tech stack, URLs, directory layout |
| `context/business-rules.md` | App behavior, credentials, data-test attrs, edge cases |
| `context/framework-architecture.md` | POM class pattern, fixture pattern, component pattern |
| `context/testing-standards.md` | Naming rules, assertion rules, locator priority |
| `context/playwright-skills.md` | Common helpers and reusable patterns |

Do NOT generate code based on general Playwright knowledge alone. The project-specific rules here override generic best practices.

---

## Code Generation Rules

### Language

- JavaScript ONLY — NEVER generate TypeScript
- CommonJS ONLY — use `require()` and `module.exports`
- NEVER use `import` / `export` (ESM)
- NEVER add type annotations, `interface`, generics, or `.ts` extensions

### File Placement

| What | Where |
|---|---|
| New Page Object | `tests/pages/[PageName].js` |
| New Component | `tests/components/[Name]Component.js` |
| Test spec | `tests/specs/[feature].spec.js` |
| Fixtures | `tests/fixtures/index.js` |

When creating a new spec, generate the Page Object first, then the spec.

### Locator Rules

- MUST use `page.getByTestId('...')` when the element has a `data-test` attribute (see `business-rules.md`)
- MUST NOT use `page.locator('.css-class')` CSS selectors
- MUST NOT use XPath
- PREFER `page.getByRole()` over `page.getByText()` for interactive elements without `data-test`

### URL Rules

- MUST use full absolute URLs: `'https://www.saucedemo.com/'`
- MUST NOT use relative paths like `'/inventory.html'` — `baseURL` is not configured in `playwright.config.js`

### Assertion Rules

- MUST include at least one `expect()` assertion per test
- MUST use auto-waiting matchers (`toBeVisible()`, `toHaveURL()`, etc.)
- MUST NOT use `page.waitForTimeout()` — it is a banned anti-pattern
- MUST NOT use `page.waitForSelector()` before assertions
- MUST NOT place `expect()` calls inside Page Object methods — they belong in spec files only

### Test Independence

- MUST write each test to run successfully in isolation
- MUST NOT create tests that depend on the outcome of a previous test
- MUST use `test.beforeEach` for repeated setup, NEVER `test.beforeAll` for login

---

## What NOT to Generate Without Being Asked

- `test.only()` — breaks parallel execution across all spec files
- `page.waitForTimeout()` — explicit waits are banned
- CSS selector locators when a `data-test` attribute exists
- TypeScript syntax in any form
- `import` / `export` ESM statements
- Relative page URLs

---

## When Business Rules Are Unclear

If the behavior of a page, element, or flow is not documented in `business-rules.md`:

1. **Say so explicitly** before generating the test — do not guess
2. Add a comment in the generated code:

```js
// TODO: Verify expected behavior for [scenario] — not documented in business-rules.md
```

3. Do NOT invent expected values for assertions (e.g., do not guess error message text)
4. Ask the user to clarify or update `business-rules.md` first

---

## When a Generated Test Fails

If a test you generated fails after running it:

1. **First diagnose**: is this a test bug or an app bug?
2. If the **test is wrong** (selector off, wrong flow): fix the test
3. If the **app is wrong** (behavior contradicts `business-rules.md`):
   - Reference `context/bug-report-template.md`
   - Mark the test with `test.fixme()` and a `// BUG-XXX:` comment
   - Do NOT change the assertion to match broken app behavior

---

## Preferred Test Generation Workflow

When asked to write tests for a feature:

```
1. Read project-context.md       → confirm URLs and language
2. Read business-rules.md        → understand the scenario and expected outcomes
3. Check if a Page Object exists → if not, generate it first (framework-architecture.md pattern)
4. Generate the spec file        → follow testing-standards.md
5. Apply helpers from            → playwright-skills.md where applicable
6. Output in order               → Page Object first, then spec file
```

---

## Output Format

When generating multiple files, clearly label each one:

```
// FILE: tests/pages/LoginPage.js
[code here]

// FILE: tests/specs/login.spec.js
[code here]
```

Keep code clean — no explanatory comments inside the code unless the logic is genuinely non-obvious. The MD files provide the "why"; the code should speak for itself.
