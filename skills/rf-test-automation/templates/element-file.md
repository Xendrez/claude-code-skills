# Element File Template

Use this template when creating a new element locator file for a page.

**Location**: `resources/elements/<page>_elements.robot`

```robot
*** Settings ***
Documentation    Element locators for <Page Name>

*** Variables ***
# <Page Name> Locators
${LOC_ELEMENT_NAME}             data-test=element-identifier
${LOC_ANOTHER_ELEMENT}          id=element-id
${LOC_CSS_ELEMENT}              .css-class-name
```

## Rules

- **Variables section ONLY** -- no `*** Keywords ***` section
- **No Library imports** -- only `Documentation` in Settings
- **All locators use `${LOC_*}` prefix** in UPPER_SNAKE_CASE
- **One file per page** or major component
- **Group locators logically** with comments
- **Align values** for readability using consistent spacing

## Locator Strategy Priority

Choose locators in this order:

1. `data-test=value` or `data-testid=value` (most stable)
2. `id=value` (stable if unique)
3. `text=visible text` (for buttons/links)
4. `.css-class` or `tag.class` (structural)
5. `xpath=//expression` (last resort)

## Example: Login Page

```robot
*** Settings ***
Documentation    Element locators for Login Page

*** Variables ***
# Login Form
${LOC_USERNAME_FIELD}           id=user-name
${LOC_PASSWORD_FIELD}           id=password
${LOC_LOGIN_BUTTON}             id=login-button

# Error Display
${LOC_ERROR_MESSAGE}            data-test=error
${LOC_ERROR_CLOSE_BUTTON}      .error-button
```

## Example: Common Elements

The `common_elements.robot` file holds shared configuration, not locators:

```robot
*** Settings ***
Documentation    Common variables shared across all pages

*** Variables ***
# Application URLs
${APP_URL}                      https://example.com

# Credentials
${VALID_USERNAME}               standard_user
${VALID_PASSWORD}               secret_sauce

# Test Data Defaults
${DEFAULT_FIRST_NAME}           John
${DEFAULT_LAST_NAME}            Doe

# Configuration
${TAX_RATE}                     0.08
${HEADLESS}                     False
```
