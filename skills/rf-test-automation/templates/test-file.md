# Test File Template

Use this template when creating a new test suite.

**Location**: `tests/<feature>_test.robot`

```robot
*** Settings ***
Documentation    <Brief description of what this test suite validates>
...              <Additional context if needed>
Resource         ../resources/<app>_bdd_keywords.robot
Test Teardown    Close Browser

*** Test Cases ***
<Descriptive Test Name>
    [Documentation]    <Detailed description of the test scenario>
    [Tags]    <test-specific-tags>
    Given <precondition>
    When <action>
    And <additional action>
    Then <expected outcome>
```

## Rules

- **Import ONLY the BDD keywords resource** -- no `Library` imports
- **Use `Test Teardown    Close Browser`** at suite level (not per-test)
- **Use `Test Tags`** for tags common to all tests in the file
- **Use `[Tags]`** only for tags unique to a specific test
- **Never duplicate tags** between `Test Tags` and `[Tags]`
- **Every test has `[Documentation]` and `[Tags]`**
- **All tests use Given/When/Then BDD format**
- **Test names are descriptive Title Case**

## Example: Simple E2E Test

```robot
*** Settings ***
Documentation    Test suite for the complete purchase flow
Resource         ../resources/myapp_bdd_keywords.robot
Test Tags        e2e    bdd
Test Teardown    Close Browser

*** Test Cases ***
User Can Complete a Purchase With Two Items
    [Documentation]    Verifies that a user can log in, add items to cart,
    ...                complete checkout, and receive order confirmation.
    [Tags]    purchase
    Given the user is logged in
    When the user adds two items to the cart
    And the user completes checkout
    Then the order confirmation is displayed
```

## Example: Verification Test with Dynamic Calculations

```robot
*** Settings ***
Documentation    Test suite for cart total and tax calculation verification
Resource         ../resources/myapp_bdd_keywords.robot
Test Tags        e2e    bdd    calculation
Test Teardown    Close Browser

*** Test Cases ***
Cart Total Calculates Tax Correctly
    [Documentation]    Validates cart totals by dynamically capturing prices,
    ...                computing expected values, and verifying displayed totals match.
    [Tags]    cart
    Given the user is logged in
    And the shopping cart is empty
    When the user captures prices for two items
    And the user proceeds to checkout with default information
    Then the cart totals should be calculated correctly
```

## Example: Multiple Tests in One Suite

```robot
*** Settings ***
Documentation    Test suite for product search functionality
Resource         ../resources/myapp_bdd_keywords.robot
Test Tags        bdd    search
Test Teardown    Close Browser

*** Test Cases ***
Search Returns Relevant Results
    [Documentation]    Verifies search returns products matching the query
    [Tags]    smoke
    Given the user is logged in
    When the user searches for "laptop"
    Then the search results contain relevant products

Empty Search Shows All Products
    [Documentation]    Verifies an empty search displays all available products
    Given the user is logged in
    When the user clears the search field
    Then all products are displayed

Search With No Results Shows Message
    [Documentation]    Verifies appropriate message when no products match
    [Tags]    negative
    Given the user is logged in
    When the user searches for "xyznonexistent"
    Then the no results message is displayed
```

## Tag Strategy

```robot
*** Settings ***
# Suite-level: applies to ALL tests in this file
Test Tags    e2e    bdd    saucedemo

*** Test Cases ***
Test One
    # Gets tags: e2e, bdd, saucedemo, smoke (from below)
    [Tags]    smoke
    ...

Test Two
    # Gets tags: e2e, bdd, saucedemo, cart (from below)
    [Tags]    cart
    ...
```

Common tags to consider:

| Tag | When to Use |
|-----|-------------|
| `e2e` | Full end-to-end workflow tests |
| `smoke` | Quick sanity checks for CI |
| `bdd` | Tests using Given/When/Then format |
| `<feature>` | Feature area: `cart`, `login`, `checkout` |
| `negative` | Tests for error/failure scenarios |
| `critical` | Must-pass tests for releases |
