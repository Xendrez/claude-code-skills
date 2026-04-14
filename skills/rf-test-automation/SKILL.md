---
name: rf-test-automation
description: Create and maintain Robot Framework test automation with Browser Library (Playwright). Use when writing robot tests, creating page objects, adding BDD keywords, setting up RF project structure, or when the user mentions Robot Framework, .robot files, or Browser Library.
argument-hint: [description of test or feature to automate]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Robot Framework Test Automation with Browser Library

Create well-structured Robot Framework test suites using Browser Library (Playwright) following BDD style and the three-tier Page Object architecture.

## When to Use

- Creating new Robot Framework test projects
- Writing new test cases in BDD format
- Adding page objects (elements + page keywords)
- Creating or extending BDD keyword layers
- Reviewing or refactoring existing .robot files
- Setting up project structure for RF + Browser Library

## Workflow

### 1. Understand the Project

Before writing any code, check the project structure:

1. Look for existing `resources/` and `tests/` directories
2. Read existing element files, page files, and BDD keyword files
3. Identify which pages and keywords already exist
4. Check `requirements.txt` for dependencies

### 2. Apply the Three-Tier Architecture

Every project follows this strict layered structure. See [references/project-structure.md](references/project-structure.md) for full details.

```
project/
    tests/                          # Test cases only (BDD format)
    resources/
        elements/                   # Layer 1: Locator variables only
            <page>_elements.robot
        pages/                      # Layer 2: Page action keywords
            <page>.robot
        <app>_bdd_keywords.robot    # Layer 3: BDD Given/When/Then keywords
    results/                        # Output directory (gitignored)
    requirements.txt
```

**Import direction is strictly downward:**
```
Tests → BDD Keywords → Page Keywords → Element Locators
```
Never import upward. BDD keywords should NOT import element files directly.

### 3. Write Files in Order

When adding automation for a new page:

1. **Element file** (`resources/elements/<page>_elements.robot`)
   - Variables section ONLY, no keywords
   - All locators use `${LOC_*}` prefix
   - See [templates/element-file.md](templates/element-file.md)

2. **Page file** (`resources/pages/<page>.robot`)
   - Import Browser Library and corresponding element file
   - Imperative verb phrase keywords (Title Case)
   - See [templates/page-file.md](templates/page-file.md)

3. **BDD keywords** (`resources/<app>_bdd_keywords.robot`)
   - Add Given/When/And/Then keywords that call page keywords
   - See [templates/bdd-keywords-file.md](templates/bdd-keywords-file.md)

4. **Test file** (`tests/<feature>_test.robot`)
   - Pure BDD scenarios, import only the BDD keywords resource
   - See [templates/test-file.md](templates/test-file.md)

### 4. Follow BDD Format

**All test cases MUST use Given/When/Then structure:**

```robot
*** Test Cases ***
User Can Complete a Purchase
    [Documentation]    Verifies the complete purchase flow
    [Tags]    e2e    purchase
    Given the user is logged in
    When the user adds an item to the cart
    And the user completes checkout
    Then the order confirmation is displayed
```

**BDD keyword definitions** - Robot Framework 5.0+ automatically strips Given/When/Then/And/But prefixes. Define keywords WITHOUT prefixes:

```robot
*** Keywords ***
the user is logged in
    [Documentation]    Precondition: user is authenticated
    Login With Credentials    ${VALID_USERNAME}    ${VALID_PASSWORD}
```

This single definition matches `Given the user is logged in`, `And the user is logged in`, etc.

**If the project already defines keywords WITH prefixes** (e.g., `Given the user is logged in`), follow the existing convention for consistency.

### 5. Apply Naming Conventions

See [references/naming-conventions.md](references/naming-conventions.md) for complete rules.

| Category | Convention | Example |
|----------|-----------|---------|
| Locator variables | `${LOC_DESCRIPTION}` | `${LOC_LOGIN_BUTTON}` |
| URL variables | `${APP_URL}` | `${SAUCEDEMO_URL}` |
| Test data constants | `${DEFAULT_*}` or descriptive | `${DEFAULT_FIRST_NAME}` |
| Local variables | `${lower_snake_case}` | `${item_price}` |
| Helper keywords | Title Case imperative | `Add Item To Cart` |
| BDD keywords | lowercase natural language | `the user is logged in` |
| Element files | `<page>_elements.robot` | `login_page_elements.robot` |
| Page files | `<page>.robot` | `login_page.robot` |
| Test files | `<feature>_test.robot` | `checkout_flow_test.robot` |
| Tags | lowercase, hyphenated | `e2e`, `product-detail` |

### 6. Use Browser Library Correctly

See [references/browser-library.md](references/browser-library.md) for complete patterns.

**Locator priority (most to least stable):**
1. `data-testid=value` or `data-test=value`
2. `id=value`
3. `text=visible text`
4. CSS selectors (`.class`, `tag.class`)
5. `xpath=//expression` (last resort)

**Browser lifecycle:**
```robot
New Browser    chromium    headless=${HEADLESS}
New Context
New Page       ${APP_URL}
# ... test actions ...
Close Browser    # In teardown
```

**Key rules:**
- NEVER use `Sleep` -- Browser Library has auto-waiting built in
- Use `Fill Text` (not `Input Text`)
- Use `Click` (not `Click Element`)
- Use `Get Text` (not `Get Element Text`)
- Use `Get Elements` for dynamic element lists, not hardcoded nth-child

### 7. Design Generic Tests

Tests should calculate expected values dynamically, never hardcode assertions:

```robot
# GOOD: Dynamic calculation
${price}    Get Item Price    ${item_locator}
${expected_total}    Evaluate    ${price} * ${quantity}
Should Be Equal As Numbers    ${actual_total}    ${expected_total}

# BAD: Hardcoded
Should Be Equal    ${actual_total}    $49.66
```

### 8. Handle Variable Scoping

When a BDD keyword captures data needed by a later BDD keyword in the same test:

```robot
the user captures the item price
    ${price}    Get Item Price    ${LOC_FIRST_ITEM}
    Set Test Variable    \${captured_price}    ${price}
```

The `\$` (escaped dollar) creates a test-level variable accessible to subsequent keywords.

**Python expressions in Evaluate:**
```robot
# WRONG: f-strings break because RF resolves variables first
${result}    Evaluate    f"${value:.2f}"

# CORRECT: Use .format()
${result}    Evaluate    "{:.2f}".format(${value})
```

## Guidelines

- Always read existing project files before making changes
- Follow the existing project's conventions when extending (consistency over personal preference)
- Every keyword MUST have `[Documentation]`
- Every test MUST have `[Documentation]` and `[Tags]`
- Use `Test Teardown    Close Browser` at suite level rather than per-test `[Teardown]`
- Use `Test Tags` for common tags, `[Tags]` only for test-specific additions
- Do not duplicate tags between `Test Tags` and `[Tags]`
- Separate browser lifecycle management from page-specific keywords
- Keep BDD keywords at business language level -- no technical details
- One keyword file per page, one element file per page
- Add `results/` to `.gitignore` and use `--outputdir results/`
