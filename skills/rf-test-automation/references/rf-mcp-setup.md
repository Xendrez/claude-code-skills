# rf-mcp Setup

[rf-mcp](https://github.com/manykarim/rf-mcp) is an MCP server that enables interactive Robot Framework test development. It lets Claude execute test steps, validate results, and generate test suites from successful executions.

## Installation

Install rf-mcp with the extras matching your test target:

```bash
# Web testing (Browser Library + SeleniumLibrary)
pip install rf-mcp[web]

# API testing (RequestsLibrary)
pip install rf-mcp[api]

# Mobile testing (AppiumLibrary)
pip install rf-mcp[mobile]

# Everything
pip install rf-mcp[all]
```

If using Browser Library, initialize Playwright browsers after install:

```bash
rfbrowser init
```

## Claude Code Integration

Add rf-mcp as an MCP server in Claude Code:

```bash
claude mcp add rf-mcp -- uvx rf-mcp
```

This registers the MCP server so Claude can use rf-mcp tools like `analyze_scenario`, `execute_step`, and `build_test_suite`.

## Environment Variables

| Variable | Values | Description |
|----------|--------|-------------|
| `ROBOTMCP_TOOL_PROFILE` | `browser_exec`, `api_exec`, `discovery`, `minimal_exec`, `full` | Controls which tools are exposed. `browser_exec` is recommended for web testing. |
| `ROBOTMCP_OUTPUT_VERBOSITY` | `compact`, `standard`, `verbose` | Controls response detail level. |
| `ROBOTMCP_OUTPUT_MODE` | `auto`, `full`, `delta` | `delta` only returns changed sections (saves tokens). |
| `ROBOTMCP_PRE_VALIDATION` | `0`, `1` | Enable element pre-validation before actions. |
| `ROBOTMCP_INSTRUCTIONS` | `off`, `default`, `custom` | Controls built-in instruction injection. |
| `ROBOTMCP_INSTRUCTIONS_TEMPLATE` | `minimal`, `standard`, `detailed`, `browser-focused`, `api-focused` | Instruction template style. |

## Key Tools

| Tool | Purpose |
|------|---------|
| `analyze_scenario` | Convert natural language to structured test intent |
| `recommend_libraries` | Suggest the best test library for the task |
| `manage_session` | Initialize sessions, import libraries, set variables |
| `execute_step` | Run a single keyword and inspect the result |
| `execute_batch` | Run multiple keywords in one call |
| `intent_action` | Library-agnostic actions (click, fill, navigate, etc.) |
| `find_keywords` | Discover keywords by name or semantic search |
| `get_keyword_info` | Get keyword documentation and argument signatures |
| `get_locator_guidance` | Get locator strategy recommendations |
| `get_session_state` | Inspect variables, page source, application state |
| `build_test_suite` | Generate .robot files from validated steps |
| `run_test_suite` | Validate (dry) or execute test suites |
