---
name: rf-test-automation
description: Create and maintain Robot Framework test automation with Browser Library (Playwright). Use when writing robot tests, creating page objects, adding BDD keywords, setting up RF project structure, or when the user mentions Robot Framework, .robot files, or Browser Library.
argument-hint: [description of test or feature to automate]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, mcp__rf-mcp__analyze_scenario, mcp__rf-mcp__recommend_libraries, mcp__rf-mcp__manage_session, mcp__rf-mcp__execute_step, mcp__rf-mcp__execute_batch, mcp__rf-mcp__execute_flow, mcp__rf-mcp__find_keywords, mcp__rf-mcp__get_keyword_info, mcp__rf-mcp__get_session_state, mcp__rf-mcp__build_test_suite, mcp__rf-mcp__run_test_suite, mcp__rf-mcp__intent_action, mcp__rf-mcp__get_locator_guidance, mcp__rf-mcp__resume_batch, mcp__rf-mcp__check_library_availability, mcp__rf-mcp__set_library_search_order, mcp__rf-mcp__manage_library_plugins
---

# Robot Framework Test Automation with Browser Library

Create well-structured Robot Framework test suites using Browser Library (Playwright) following BDD style and the three-tier Page Object architecture. Uses [rf-mcp](https://github.com/manykarim/rf-mcp) to interactively execute and validate test steps before generating the final suite.

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

### 2. Analyze the Test Scenario (rf-mcp)

Use `analyze_scenario` to convert the natural language test description into a structured test intent:

```
analyze_scenario("User logs in, adds two items to cart, completes checkout, and verifies the order confirmation")
```

Then use `recommend_libraries` to confirm the right test library (typically Browser Library for web testing).

### 3. Initialize a Session (rf-mcp)

Use `manage_session` to start an interactive session and import the required libraries:

```
manage_session(action="init", library="Browser")
```

Use `check_library_availability` if unsure whether a library is installed.

### 4. Execute Steps Interactively (rf-mcp)

Run each test step one at a time, validating results as you go:

- **`execute_step`** — Run a single Robot Framework keyword and inspect the result
- **`execute_batch`** — Run multiple keywords in one call for efficiency
- **`intent_action`** — Use library-agnostic intents (`click`, `fill`, `navigate`, `assert_visible`, `extract_text`, `wait_for`, `hover`, `select`) when you don't need library-specific keywords
- **`get_locator_guidance`** — Get help choosing the right locator strategy
- **`find_keywords`** — Discover available keywords by name or semantic search
- **`get_keyword_info`** — Get documentation and argument signatures for a keyword

When a step fails, inspect the error, adjust the approach (e.g., fix a locator), and retry. Use `get_session_state` to inspect variables, page source, or application state for debugging.

Use `execute_flow` to build control structures (if/for_each/try) when needed.

### 5. Build the Test Suite (rf-mcp)

Once all steps execute successfully, use `build_test_suite` to generate the final .robot files. Request BDD format with Given/When/Then keywords:

```
build_test_suite(format="bdd")
```

The generated suite should follow the three-tier architecture described below. Review the output and adjust if needed.

### 6. Run the Test Suite (rf-mcp)

Use `run_test_suite` to validate or execute the generated suite:

- **Dry mode** — Validate syntax without executing
- **Execute mode** — Run the full suite and report results

### 7. Save to Project Structure

Write the generated .robot files into the correct project locations using the file tools (Write/Edit). Apply the three-tier architecture and naming conventions from the reference docs.

## Three-Tier Architecture

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

## File Creation Order

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

## BDD Format

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

See [references/bdd-patterns.md](references/bdd-patterns.md) for complete BDD patterns.

## Naming Conventions

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

## Browser Library Reference

See [references/browser-library.md](references/browser-library.md) for complete patterns.

**Locator priority (most to least stable):**
1. `data-testid=value` or `data-test=value`
2. `id=value`
3. `text=visible text`
4. CSS selectors (`.class`, `tag.class`)
5. `xpath=//expression` (last resort)

When using rf-mcp, use `get_locator_guidance` to get locator recommendations for specific elements.

**Key rules:**
- NEVER use `Sleep` -- Browser Library has auto-waiting built in
- Use `Fill Text` (not `Input Text`)
- Use `Click` (not `Click Element`)
- Use `Get Text` (not `Get Element Text`)
- Use `Get Elements` for dynamic element lists, not hardcoded nth-child

## rf-mcp Setup

See [references/rf-mcp-setup.md](references/rf-mcp-setup.md) for installation and configuration.

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
- Design generic tests that calculate expected values dynamically, never hardcode assertions
- When a BDD keyword captures data needed by a later keyword, use `Set Test Variable` with escaped `\$` syntax
