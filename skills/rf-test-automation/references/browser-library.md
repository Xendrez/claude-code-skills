# Browser Library (Playwright) Patterns

> **Note:** When rf-mcp is available, prefer using `intent_action` and `execute_step` to interact with Browser Library keywords interactively. The patterns below remain the reference for understanding and reviewing generated .robot files.

## Browser Lifecycle

### The Three-Tier Hierarchy: Browser > Context > Page

```robot
# Full lifecycle
New Browser    chromium    headless=${HEADLESS}    # Opens browser process
New Context                                        # Isolated session (cookies, storage)
New Page       ${APP_URL}                          # Navigates to URL
# ... test actions ...
Close Browser                                      # Closes everything
```

**Context isolation**: Each `New Context` gets its own cookies, local storage, and cache. Use this for test isolation without the overhead of launching a new browser.

### Recommended Browser Setup Keyword

```robot
*** Keywords ***
Open Application
    [Documentation]    Opens browser and navigates to the application
    [Arguments]    ${url}=${APP_URL}
    New Browser    chromium    headless=${HEADLESS}
    New Context
    New Page    ${url}

Close Application
    [Documentation]    Closes the browser and all contexts
    Close Browser
```

### Headless Configuration

Make headless mode configurable for CI/CD:

```robot
*** Variables ***
${HEADLESS}    False    # Override with: robot --variable HEADLESS:True
```

## Locator Strategies

### Priority Order (Most to Least Stable)

1. **`data-testid=` or `data-test=`** -- Designed for testing, immune to CSS/layout changes
   ```robot
   ${LOC_SUBMIT_BUTTON}    data-test=submit-button
   ${LOC_USERNAME}          data-testid=username-input
   ```

2. **`id=`** -- Stable when IDs are consistent and unique
   ```robot
   ${LOC_LOGIN_BUTTON}    id=login-button
   ${LOC_EMAIL_FIELD}     id=email
   ```

3. **`text=`** -- Good for buttons and links with visible text
   ```robot
   ${LOC_CHECKOUT}    text=Checkout
   ${LOC_SUBMIT}      text=Submit Order
   ```

4. **CSS selectors** -- Flexible, use for structure-based selection
   ```robot
   ${LOC_PRICE_LIST}      .product-price
   ${LOC_FIRST_ITEM}      .inventory_item:first-child
   ${LOC_NAV_MENU}        nav.main-menu
   ```

5. **`xpath=`** -- Last resort, use only when CSS cannot express the query
   ```robot
   ${LOC_PARENT_BY_CHILD}    xpath=//div[contains(@class,'item') and .//span[text()='Sale']]
   ```

### Locator Tips

- **Prefer `data-test` attributes** -- Ask developers to add them; they survive refactoring
- **Avoid fragile selectors** -- Don't rely on CSS classes used for styling (they change)
- **Avoid deep nesting** -- `.container > .row > .col > .card > .card-body > button` is fragile
- **Use `:nth-child()` sparingly** -- It breaks when list order changes
- **For dynamic lists, use `Get Elements`** instead of hardcoded indices

## Common Keywords

### Text Input

```robot
# Browser Library uses Fill Text (not Input Text)
Fill Text    ${LOC_USERNAME_FIELD}    ${username}
Fill Text    ${LOC_PASSWORD_FIELD}    ${password}

# Clear and type
Clear Text    ${LOC_SEARCH_FIELD}
Fill Text     ${LOC_SEARCH_FIELD}    ${search_term}
```

### Clicking

```robot
# Simple click
Click    ${LOC_LOGIN_BUTTON}

# Click with options
Click    ${LOC_MENU_ITEM}    button=right    # Right-click
Click    ${LOC_LINK}         force=True       # Force click (skip actionability checks)
```

### Reading Text

```robot
# Get element text
${text}    Get Text    ${LOC_PRODUCT_NAME}

# Get attribute value
${value}    Get Attribute    ${LOC_INPUT}    value
${href}     Get Attribute    ${LOC_LINK}     href
```

### Dropdowns

```robot
# Select by value
Select Options By    ${LOC_SORT_DROPDOWN}    value    price-low-to-high

# Select by label (visible text)
Select Options By    ${LOC_COUNTRY_DROPDOWN}    label    United States

# Select by index
Select Options By    ${LOC_MONTH_DROPDOWN}    index    3
```

### Working with Element Lists

```robot
# GOOD: Dynamic element collection
@{elements}    Get Elements    .inventory_item_price
${count}       Get Length    ${elements}
FOR    ${element}    IN    @{elements}
    ${text}    Get Text    ${element}
    Append To List    ${prices}    ${text}
END

# BAD: Hardcoded nth-child (fragile)
${price1}    Get Text    .item:nth-child(1) .price
${price2}    Get Text    .item:nth-child(2) .price
```

## Waiting and Synchronization

### Auto-Waiting (Default Behavior)

Browser Library automatically waits for elements to be actionable before interacting. **Do NOT use `Sleep`.**

```robot
# WRONG: Using Sleep
Click    ${LOC_SUBMIT}
Sleep    2s
Get Text    ${LOC_RESULT}

# CORRECT: Auto-waiting handles it
Click    ${LOC_SUBMIT}
Get Text    ${LOC_RESULT}    # Playwright waits automatically
```

### Explicit Waits (When Needed)

```robot
# Wait for element to be visible
Wait For Elements State    ${LOC_LOADING_SPINNER}    hidden    timeout=10s

# Wait for element to appear
Wait For Elements State    ${LOC_SUCCESS_MESSAGE}    visible    timeout=5s

# Wait for condition
Wait For Condition    Text    ${LOC_STATUS}    ==    Complete    timeout=10s
```

### Timeout Configuration

```robot
# Set global timeout (in Suite Setup or keyword)
Set Browser Timeout    10s

# Per-action timeout
Click    ${LOC_SLOW_BUTTON}    timeout=15s
```

## Assertions

```robot
# Text assertions
Get Text    ${LOC_HEADER}    ==    Welcome
Get Text    ${LOC_MESSAGE}    contains    successfully
Get Text    ${LOC_ERROR}    should not be    ${EMPTY}

# Visibility assertions
Get Element States    ${LOC_MODAL}    contains    visible
Get Element States    ${LOC_SPINNER}    not contains    visible

# Count assertions
${count}    Get Element Count    .list-item
Should Be Equal As Integers    ${count}    5

# URL assertions
Get Url    ==    ${EXPECTED_URL}
Get Url    contains    /dashboard
```

## Screenshots

```robot
# Take screenshot (saved to output directory)
Take Screenshot

# Take screenshot with specific filename
Take Screenshot    filename=login_error

# Screenshot on failure (automatic with Test Teardown)
# Configure in suite settings or use Run Keyword If Test Failed
```

## Keyboard and Special Input

```robot
# Press key
Keyboard Key    press    Enter
Keyboard Key    press    Tab
Keyboard Key    press    Escape

# Key combinations
Keyboard Key    press    Control+a
Keyboard Key    press    Meta+c    # Cmd+C on macOS
```
