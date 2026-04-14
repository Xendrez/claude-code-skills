# BDD Patterns in Robot Framework

## Core Principles

### Given/When/Then Structure

Every test case follows this structure:

| Prefix | Purpose | Example |
|--------|---------|---------|
| **Given** | Preconditions -- set up the initial state | `Given the user is logged in` |
| **When** | Action -- the behavior being tested | `When the user submits the order` |
| **And** | Continuation of the previous step type | `And the shopping cart is empty` |
| **Then** | Assertion -- verify the expected outcome | `Then the confirmation page is displayed` |
| **But** | Negative continuation (rare) | `But the error count remains zero` |

### Prefix Auto-Stripping (RF 5.0+)

Robot Framework 5.0+ automatically strips Given/When/Then/And/But prefixes when matching keywords. This means:

```robot
# Define ONE keyword without prefix:
the user is logged in
    Login With Credentials    ${VALID_USERNAME}    ${VALID_PASSWORD}

# It matches ALL of these in tests:
Given the user is logged in
When the user is logged in
And the user is logged in
Then the user is logged in
```

**Benefit**: No need to define separate keywords for each prefix. One definition serves all uses.

**Legacy convention**: Some projects define keywords WITH explicit prefixes (`Given the user is logged in`). This still works but creates duplication when the same step needs `Given` in one test and `And` in another. When extending such projects, follow the existing convention.

## Writing Good BDD Keywords

### Business Language, Not Technical

```robot
# GOOD: Business-readable
the user adds the Sauce Labs Backpack to the cart
the order total includes 8% tax
the search results contain at least 5 products

# BAD: Technical implementation details
the user clicks id=add-to-cart-sauce-labs-backpack
the tax equals subtotal times 0.08 rounded to 2 decimals
the element count of css=.result-item is greater than 5
```

### Parameterized BDD Keywords

Use embedded arguments for flexible, reusable steps:

```robot
*** Keywords ***
the user adds "${product_name}" to the cart
    [Documentation]    Adds a specific product to the shopping cart
    Add Product By Name    ${product_name}

the cart should contain ${count} items
    [Documentation]    Verifies the number of items in the cart
    ${actual}    Get Cart Item Count
    Should Be Equal As Integers    ${actual}    ${count}

the error message says "${expected_message}"
    [Documentation]    Verifies the displayed error message
    ${actual}    Get Text    ${LOC_ERROR_MESSAGE}
    Should Be Equal    ${actual}    ${expected_message}
```

Usage in tests:
```robot
When the user adds "Sauce Labs Backpack" to the cart
And the user adds "Sauce Labs Bike Light" to the cart
Then the cart should contain 2 items
```

### BDD Keyword Implementation Patterns

**Precondition keywords (Given/And)** -- set up state:
```robot
the user is logged in
    [Documentation]    Precondition: User is authenticated
    Open Application
    Login With Credentials    ${VALID_USERNAME}    ${VALID_PASSWORD}

the shopping cart is empty
    [Documentation]    Precondition: Cart starts with zero items
    ${count}    Get Cart Item Count
    IF    ${count} > 0
        Clear Cart
    END
```

**Action keywords (When/And)** -- perform the behavior:
```robot
the user completes checkout with default information
    [Documentation]    Action: Fills checkout form and submits
    Navigate To Cart
    Start Checkout
    Fill Checkout Information
    Finish Order
```

**Assertion keywords (Then/And)** -- verify outcomes:
```robot
the order confirmation is displayed
    [Documentation]    Assertion: Order success message is shown
    Verify Order Success

the cart total should be calculated correctly
    [Documentation]    Assertion: Verify computed totals match displayed values
    ${actual_subtotal}    ${actual_tax}    ${actual_total}    Get Cart Totals From Page
    ${expected_subtotal}    ${expected_tax}    ${expected_total}    Calculate Expected Cart Totals
    Verify Cart Totals    ${actual_subtotal}    ${actual_tax}    ${actual_total}
    ...    ${expected_subtotal}    ${expected_tax}    ${expected_total}
```

## Variable Sharing Between BDD Steps

BDD keywords in the same test case often need to share data. Use `Set Test Variable`:

```robot
the user captures the item price
    [Documentation]    Action: Reads and stores the item price for later verification
    ${price}    Get Item Price    ${LOC_FIRST_ITEM}
    Set Test Variable    \${captured_price}    ${price}

the receipt shows the correct price
    [Documentation]    Assertion: Receipt matches the captured price
    ${receipt_price}    Get Text    ${LOC_RECEIPT_TOTAL}
    Should Be Equal    ${receipt_price}    ${captured_price}
```

**Important**: The `\$` (backslash-dollar) syntax is required. It tells Robot Framework to create a test-level variable rather than trying to resolve `${captured_price}` immediately.

For lists:
```robot
Set Test Variable    \@{captured_prices}    @{prices}
```

## Organizing BDD Keywords

### Single File (Small Projects)

When you have fewer than ~30 BDD keywords:

```
resources/
    myapp_bdd_keywords.robot    # All BDD keywords in one file
```

### Split by Feature (Larger Projects)

When the BDD keywords file grows large:

```
resources/
    bdd/
        authentication_keywords.robot
        cart_keywords.robot
        checkout_keywords.robot
        product_keywords.robot
    myapp_bdd_keywords.robot    # Aggregator
```

The aggregator imports all feature-specific files:

```robot
*** Settings ***
Resource    bdd/authentication_keywords.robot
Resource    bdd/cart_keywords.robot
Resource    bdd/checkout_keywords.robot
Resource    bdd/product_keywords.robot
```

## Test Case Design

### One Scenario per Test Case

```robot
# GOOD: Focused test
User Can Add Item To Cart
    Given the user is logged in
    When the user adds an item to the cart
    Then the cart badge shows 1 item

# BAD: Testing too many things
Full E2E Test
    Given the user is logged in
    When the user searches for a product
    Then search results are displayed
    When the user adds an item to the cart
    Then the cart badge shows 1 item
    When the user checks out
    Then the order is confirmed
    When the user views order history
    Then the order appears in history
```

### Generic Tests Over Specific Tests

Design tests that work with any valid data:

```robot
# GOOD: Dynamic, works with any items
Verify Cart Total Calculates Tax Correctly
    Given the user is logged in
    And the shopping cart is empty
    When the user captures prices for two items
    And the user proceeds to checkout with default information
    Then the cart totals should be calculated correctly

# BAD: Hardcoded to specific items and values
Verify Backpack And Bike Light Total Is 39.98
    Given the user is logged in
    When the user adds the backpack for 29.99
    And the user adds the bike light for 9.99
    Then the total should be 43.18
```

## Anti-Patterns

### Avoid These

1. **UI details in BDD keywords**: `the user clicks the green button in the top-right corner`
2. **Implementation leaking into tests**: `Given the database has 5 product records`
3. **Overly specific steps**: `the user types "john@test.com" into the email field`
4. **Missing Given**: Starting tests with `When` without establishing preconditions
5. **Multiple assertions in one Then**: Split into `Then X` / `And Y`
6. **BDD keywords that import element locators directly**: Use page keywords as intermediary
