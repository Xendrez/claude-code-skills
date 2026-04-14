# BDD Keywords File Template

Use this template when creating or extending the BDD keywords layer.

**Location**: `resources/<app>_bdd_keywords.robot`

```robot
*** Settings ***
Documentation    BDD keywords for <Application Name> test automation

# Import page keyword files (Layer 2)
Resource         pages/login_page.robot
Resource         pages/home_page.robot
Resource         pages/checkout_page.robot

*** Keywords ***
# ============================================================================
# Authentication Keywords
# ============================================================================

the user is logged in
    [Documentation]    Precondition: User is authenticated and on the main page
    Open Application
    Login With Credentials    ${VALID_USERNAME}    ${VALID_PASSWORD}

# ============================================================================
# Cart Keywords
# ============================================================================

the shopping cart is empty
    [Documentation]    Precondition: Cart starts with zero items
    ${count}    Get Cart Item Count
    IF    ${count} > 0
        Clear Cart
    END

the user adds an item to the cart
    [Documentation]    Action: User adds the first available item
    Add First Available Item

# ============================================================================
# Checkout Keywords
# ============================================================================

the user completes checkout
    [Documentation]    Action: User goes through checkout with default info
    Navigate To Cart
    Start Checkout
    Fill Checkout Information
    Finish Order

# ============================================================================
# Verification Keywords
# ============================================================================

the order confirmation is displayed
    [Documentation]    Assertion: Order success message is shown
    Verify Order Success
```

## Rules

- **Import ONLY page keyword files** -- not element files
- **Do NOT import `Library Browser`** -- page files handle that
- **Define keywords WITHOUT BDD prefixes** (RF 5.0+ auto-strips them)
- **Every keyword has `[Documentation]`**
- **Group keywords by feature area** with section comments
- **Keywords call page-level keywords** -- no direct Browser Library calls
- **If the existing project uses explicit prefixes**, follow that convention

## Documentation Convention

Prefix documentation with the BDD step type:

```robot
the user is logged in
    [Documentation]    Precondition: User is authenticated

the user submits the form
    [Documentation]    Action: Submits the registration form

the success message is displayed
    [Documentation]    Assertion: Verifies the success notification appears
```

## Variable Sharing Pattern

When a keyword captures data needed by a later keyword:

```robot
the user captures the product price
    [Documentation]    Action: Reads and stores the product price
    ${price}    Get Item Price    ${LOC_FIRST_ITEM}
    Set Test Variable    \${captured_price}    ${price}

the receipt total matches the captured price
    [Documentation]    Assertion: Receipt total matches previously captured price
    ${receipt}    Get Receipt Total
    Should Be Equal    ${receipt}    ${captured_price}
```

## Scaling Pattern

When the file exceeds ~50 keywords, split by feature:

```
resources/
    bdd/
        authentication_keywords.robot
        cart_keywords.robot
        checkout_keywords.robot
    <app>_bdd_keywords.robot    # Aggregator
```

Aggregator file:
```robot
*** Settings ***
Documentation    BDD keywords aggregator for <Application Name>
Resource         bdd/authentication_keywords.robot
Resource         bdd/cart_keywords.robot
Resource         bdd/checkout_keywords.robot
```

## Handling Existing Projects with Explicit Prefixes

If the project already defines keywords with Given/When/And/Then prefixes:

```robot
# Existing convention -- follow it for consistency
Given the user is logged in
    [Documentation]    BDD Given: User is logged in
    Login With Credentials

And the shopping cart is empty
    [Documentation]    BDD And: Cart is empty
    Clear Cart
```

When the same step is used with different prefixes in different tests, you may need to define it multiple times or refactor to the prefix-free style.
