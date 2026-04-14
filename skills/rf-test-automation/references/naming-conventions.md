# Robot Framework Naming Conventions

## Variable Names

### Constants and Configuration (UPPER_SNAKE_CASE)

```robot
*** Variables ***
# URLs
${APP_URL}                  https://example.com
${API_BASE_URL}             https://api.example.com/v1

# Credentials
${VALID_USERNAME}           standard_user
${VALID_PASSWORD}           secret_sauce

# Test data defaults
${DEFAULT_FIRST_NAME}       John
${DEFAULT_LAST_NAME}        Doe
${DEFAULT_EMAIL}            john@example.com

# Configuration constants
${TAX_RATE}                 0.08
${TIMEOUT}                  10s
${BROWSER}                  chromium
${HEADLESS}                 False
```

### Locator Variables (`${LOC_*}` prefix, UPPER_SNAKE_CASE)

```robot
*** Variables ***
# Login Page
${LOC_USERNAME_FIELD}       id=user-name
${LOC_PASSWORD_FIELD}       id=password
${LOC_LOGIN_BUTTON}         id=login-button
${LOC_ERROR_MESSAGE}        data-test=error

# Inventory Page
${LOC_PRODUCT_LIST}         .inventory_list
${LOC_SORT_DROPDOWN}        data-test=product-sort-container
${LOC_CART_BADGE}           .shopping_cart_badge
```

### Local/Temporary Variables (lower_snake_case)

Used within keywords for intermediate values:

```robot
*** Keywords ***
Calculate Total
    ${subtotal}    Evaluate    ${price1} + ${price2}
    ${tax_amount}    Evaluate    round(${subtotal} * ${TAX_RATE}, 2)
    ${total}    Evaluate    round(${subtotal} + ${tax_amount}, 2)
    RETURN    ${total}
```

### Lists and Dictionaries

```robot
*** Variables ***
# Lists use @
@{VALID_BROWSERS}           chromium    firefox    webkit
@{REQUIRED_FIELDS}          username    password    email

# Dictionaries use &
&{DEFAULT_USER}             name=John    email=john@test.com    role=admin
```

## Keyword Names

### Helper/Page Keywords (Title Case, Imperative Verb)

Keywords should start with a verb and describe the action:

```robot
*** Keywords ***
# Actions
Login With Credentials
Add Item To Cart
Fill Checkout Information
Navigate To Product Page
Submit Order

# Getters
Get Item Price
Get Cart Item Count
Get Error Message Text

# Verifiers
Verify Order Success
Verify Cart Is Empty
Verify Page Title
```

### BDD Keywords (Lowercase Natural Language)

Define without Given/When/Then prefix (RF 5.0+ auto-strips):

```robot
*** Keywords ***
the user is logged in
the shopping cart is empty
the user adds an item to the cart
the user completes checkout with default information
the order confirmation is displayed
the error message says "${expected_message}"
```

If the project convention uses explicit prefixes, follow that:

```robot
*** Keywords ***
Given the user is logged in
And the shopping cart is empty
When the user adds an item to the cart
Then the order confirmation is displayed
```

### Documentation Patterns

```robot
# Helper keyword documentation: imperative description
Login With Credentials
    [Documentation]    Logs into the application with the given credentials

# BDD keyword documentation: prefix the BDD type
the user is logged in
    [Documentation]    Precondition: User is authenticated and on the main page

the order confirmation is displayed
    [Documentation]    Assertion: Verifies the order success message is shown
```

## File Names

All `.robot` files use `snake_case`:

| File Type | Pattern | Example |
|-----------|---------|---------|
| Element locators | `<page>_elements.robot` | `login_page_elements.robot` |
| Page keywords | `<page>.robot` | `login_page.robot` |
| BDD keywords | `<app>_bdd_keywords.robot` | `myapp_bdd_keywords.robot` |
| Test suites | `<feature>_test.robot` | `checkout_flow_test.robot` |
| Common elements | `common_elements.robot` | `common_elements.robot` |

## Test Case Names

Use descriptive Title Case names that state the scenario:

```robot
*** Test Cases ***
# Good: Descriptive and specific
User Can Complete Purchase With Two Items
Verify Cart Total Calculates Tax Correctly
Product Search Returns Relevant Results
Login Fails With Invalid Password

# Bad: Vague or generic
Test 1
Purchase Test
Check Cart
```

## Tags

- **Lowercase** with hyphens for multi-word: `e2e`, `smoke`, `product-detail`
- **Suite-level** (`Test Tags`): tags common to all tests in the file
- **Test-level** (`[Tags]`): tags unique to a specific test case
- Never duplicate tags between `Test Tags` and `[Tags]`

Common tag categories:

| Category | Examples |
|----------|---------|
| Test type | `smoke`, `regression`, `e2e`, `integration` |
| Feature area | `login`, `cart`, `checkout`, `search` |
| Priority | `critical`, `high`, `medium`, `low` |
| Style | `bdd` |
| Environment | `ci-safe`, `requires-network` |

## Indentation and Spacing

- Use **4 spaces** for indentation (not tabs)
- Use **4 spaces** minimum between columns (keyword name and arguments)
- Align variable values for readability in element files
- Use blank lines to separate logical groups of variables or keywords
