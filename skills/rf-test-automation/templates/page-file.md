# Page File Template

Use this template when creating page keywords for a new page.

**Location**: `resources/pages/<page>.robot`

```robot
*** Settings ***
Documentation    Keywords for <Page Name> functionality
Library          Browser
Resource         ../elements/<page>_elements.robot

*** Keywords ***
<Action Keyword Name>
    [Documentation]    <Imperative description of what this keyword does>
    [Arguments]    ${arg1}    ${arg2}=${DEFAULT_VALUE}
    # Implementation using Browser Library keywords and locator variables
    RETURN    ${result}
```

## Rules

- **Always import `Library Browser`**
- **Import corresponding element file** via `Resource`
- **Import `common_elements.robot`** only when you reference its variables
- **Import additional libraries** (`String`, `Collections`) only when needed
- **Keywords use Title Case imperative verb phrases**
- **Every keyword has `[Documentation]`**
- **Use `RETURN`** (not `[Return]`) for return values (RF 5.0+ syntax)
- **Each keyword represents ONE atomic page interaction**

## Example: Login Page

```robot
*** Settings ***
Documentation    Keywords for Login Page functionality
Library          Browser
Resource         ../elements/login_page_elements.robot
Resource         ../elements/common_elements.robot

*** Keywords ***
Login With Credentials
    [Documentation]    Logs into the application with given credentials
    [Arguments]    ${username}=${VALID_USERNAME}    ${password}=${VALID_PASSWORD}
    Fill Text    ${LOC_USERNAME_FIELD}    ${username}
    Fill Text    ${LOC_PASSWORD_FIELD}    ${password}
    Click    ${LOC_LOGIN_BUTTON}

Get Login Error Message
    [Documentation]    Gets the error message displayed on the login page
    ${message}    Get Text    ${LOC_ERROR_MESSAGE}
    RETURN    ${message}

Verify Login Error
    [Documentation]    Verifies the login error message matches expected text
    [Arguments]    ${expected_message}
    ${actual}    Get Login Error Message
    Should Be Equal    ${actual}    ${expected_message}
```

## Example: Page with Calculations

When a page keyword needs to compute values, import additional libraries:

```robot
*** Settings ***
Documentation    Keywords for Checkout Overview Page functionality
Library          Browser
Library          String
Resource         ../elements/checkout_overview_page_elements.robot
Resource         ../elements/common_elements.robot

*** Keywords ***
Get Cart Totals From Page
    [Documentation]    Reads subtotal, tax, and total from the checkout overview
    ${subtotal_text}    Get Text    ${LOC_SUBTOTAL_LABEL}
    ${tax_text}    Get Text    ${LOC_TAX_LABEL}
    ${total_text}    Get Text    ${LOC_TOTAL_LABEL}
    RETURN    ${subtotal_text}    ${tax_text}    ${total_text}

Calculate Expected Cart Totals
    [Documentation]    Calculates expected totals from item prices
    [Arguments]    @{prices}
    ${subtotal}    Evaluate    sum(${prices})
    ${tax}    Evaluate    round(${subtotal} * ${TAX_RATE}, 2)
    ${total}    Evaluate    round(${subtotal} + ${tax}, 2)
    RETURN    ${subtotal}    ${tax}    ${total}
```

## Common Keyword Patterns

### Getter Keywords
```robot
Get Product Name
    [Documentation]    Gets the product name text
    ${name}    Get Text    ${LOC_PRODUCT_NAME}
    RETURN    ${name}
```

### Action Keywords
```robot
Add Item To Cart
    [Documentation]    Adds specified item to the shopping cart
    [Arguments]    ${item_locator}
    Click    ${item_locator}
```

### Navigation Keywords
```robot
Navigate To Cart
    [Documentation]    Navigates to the shopping cart page
    Click    ${LOC_CART_LINK}
```

### Verification Keywords
```robot
Verify Page Loaded
    [Documentation]    Verifies the page has loaded by checking a key element
    Wait For Elements State    ${LOC_PAGE_HEADER}    visible    timeout=5s
```
