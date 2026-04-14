# Robot Framework Project Structure

## Recommended Directory Layout

```
project-name/
├── tests/                              # Test suites only
│   ├── login_test.robot
│   ├── checkout_flow_test.robot
│   └── product_search_test.robot
├── resources/
│   ├── elements/                       # Layer 1: Locator variables
│   │   ├── common_elements.robot       # URLs, credentials, shared constants
│   │   ├── login_page_elements.robot
│   │   ├── home_page_elements.robot
│   │   └── checkout_page_elements.robot
│   ├── pages/                          # Layer 2: Page action keywords
│   │   ├── login_page.robot
│   │   ├── home_page.robot
│   │   └── checkout_page.robot
│   └── <app>_bdd_keywords.robot        # Layer 3: BDD keywords
├── libraries/                          # Optional: Custom Python libraries
├── data/                               # Optional: Test data (CSV, JSON)
├── results/                            # Output directory (gitignored)
│   ├── log.html
│   ├── report.html
│   └── output.xml
├── requirements.txt
├── .gitignore
└── README.md
```

## The Three-Tier Architecture

### Layer 1: Element Locators (`resources/elements/`)

Pure data files. Contain ONLY `*** Variables ***` sections.

**Purpose**: Centralize all element locators so changes to the UI only require updating one file.

**Rules:**
- No `*** Keywords ***` section
- No `Library` imports
- Only `Documentation` in `*** Settings ***`
- All locator variables use `${LOC_*}` prefix
- One file per page or component
- `common_elements.robot` holds shared data: URLs, credentials, configuration constants

**Import pattern**: Element files import nothing (or only `common_elements.robot` if they need shared constants).

### Layer 2: Page Keywords (`resources/pages/`)

Action-oriented keywords for each page of the application.

**Purpose**: Encapsulate all interactions with a specific page into reusable keywords.

**Rules:**
- Import `Library Browser` (always)
- Import corresponding element file(s)
- Import additional libraries only when needed (`String`, `Collections`)
- Keywords use imperative verb phrases in Title Case
- Each keyword represents an atomic page interaction
- Never reference locators from other pages (import those page files instead)

**Import pattern**: `Library Browser` + `Resource ../elements/<page>_elements.robot` + optionally `Resource ../elements/common_elements.robot`

### Layer 3: BDD Keywords (`resources/<app>_bdd_keywords.robot`)

High-level business-readable keywords with Given/When/Then semantics.

**Purpose**: Bridge between human-readable test scenarios and technical page interactions.

**Rules:**
- Import all page keyword files needed
- Do NOT import element files directly (page keywords handle locators)
- Keywords represent business actions, not technical steps
- Define keywords without BDD prefixes (RF 5.0+ auto-strips them)
- Each keyword calls one or more page-level keywords

**Import pattern**: `Resource pages/login_page.robot` + `Resource pages/checkout_page.robot` + etc.

**Exception**: If a BDD keyword needs a specific locator variable (e.g., to pass as an argument to a page keyword), it may import that element file. Prefer creating a page keyword that accepts a meaningful parameter instead.

### Test Files (`tests/`)

Pure BDD test scenarios that read like specifications.

**Rules:**
- Import ONLY the BDD keywords resource file
- No `Library` imports (libraries belong in resource files)
- All tests use Given/When/Then format
- Every test has `[Documentation]` and `[Tags]`
- Use `Test Teardown    Close Browser` at suite level
- Use `Test Tags` for common tags, `[Tags]` only for unique additions

## Import Chain

```
tests/<feature>_test.robot
    └── Resource ../resources/<app>_bdd_keywords.robot
            ├── Resource pages/login_page.robot
            │     ├── Library Browser
            │     └── Resource ../elements/login_page_elements.robot
            ├── Resource pages/home_page.robot
            │     ├── Library Browser
            │     └── Resource ../elements/home_page_elements.robot
            └── Resource pages/checkout_page.robot
                  ├── Library Browser
                  ├── Library String
                  └── Resource ../elements/checkout_page_elements.robot
```

## Scaling for Larger Projects

When the BDD keywords file grows beyond ~50 keywords, split by feature area:

```
resources/
    bdd/
        authentication_keywords.robot
        cart_keywords.robot
        checkout_keywords.robot
        product_keywords.robot
    <app>_bdd_keywords.robot    # Aggregator that imports all bdd/*.robot files
```

When pages have many shared components, extract to component files:

```
resources/
    elements/
        components/
            navigation_elements.robot
            footer_elements.robot
    pages/
        components/
            navigation.robot
            footer.robot
```

## Configuration Files

### requirements.txt
```
robotframework>=7.0
robotframework-browser>=19.0
```

### .gitignore additions
```
# Robot Framework output
results/
log.html
report.html
output.xml
screenshots/
*.png
```

### Running tests
```bash
# Run all tests with output directory
robot --outputdir results tests/

# Run specific test suite
robot --outputdir results tests/login_test.robot

# Run by tag
robot --outputdir results --include e2e tests/
robot --outputdir results --exclude slow tests/

# Run headless (for CI)
robot --outputdir results --variable HEADLESS:True tests/
```
