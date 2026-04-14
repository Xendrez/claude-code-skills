# Claude Code Skills

A collection of reusable [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) to share with others.

## Available Skills

| Skill | Description |
|-------|-------------|
| [rf-test-automation](skills/rf-test-automation/) | Robot Framework test automation with Browser Library (Playwright). Three-tier Page Object architecture, BDD-style tests. |
| [spec](skills/spec/) | Feature specification developer — creates phased specs with research-backed reference documents, user stories, and Gherkin acceptance tests. |

## Skill Prerequisites

### rf-test-automation

- **Python** 3.8+
- **Node.js** (required by Browser Library for Playwright)
- **Robot Framework** and **Browser Library**:
  ```bash
  pip install robotframework>=7.0 robotframework-browser>=19.0
  rfbrowser init
  ```
  The `rfbrowser init` step downloads the Playwright browser binaries.
- **rf-mcp** (MCP server for interactive test development):
  ```bash
  pip install rf-mcp[web]
  ```
  Then register it with Claude Code:
  ```bash
  claude mcp add rf-mcp -- uvx rf-mcp
  ```
  rf-mcp enables Claude to execute test steps interactively and validate them before generating the final test suite. See the [rf-mcp setup guide](skills/rf-test-automation/references/rf-mcp-setup.md) for configuration options.

### spec

No additional dependencies. Works out of the box with Claude Code's built-in tools (Read, Write, Edit, WebSearch, WebFetch, Agent).

## How to Use

### 1. Clone this repository

```bash
git clone https://github.com/Xendrez/claude-code-skills.git
```

### 2. Add a skill to your project

Copy the skill folder into your project's `.claude/skills/` directory:

```bash
# From your project root
mkdir -p .claude/skills
cp -r /path/to/claude-code-skills/skills/rf-test-automation .claude/skills/
```

Alternatively, to make a skill available across all your projects, copy it into your user-level skills directory:

```bash
mkdir -p ~/.claude/skills
cp -r /path/to/claude-code-skills/skills/rf-test-automation ~/.claude/skills/
```

### 3. Use the skill

Once installed, Claude Code automatically detects the skill and activates it when relevant. You can also invoke it directly:

```
/rf-test-automation create login page tests for the checkout flow
```

## Repository Structure

```
claude-code-skills/
    skills/
        <skill-name>/
            SKILL.md              # Skill definition (required)
            references/           # Reference docs the skill can read
            templates/            # Templates the skill uses
```

## Adding a New Skill

1. Create a new directory under `skills/` with your skill name
2. Add a `SKILL.md` file with frontmatter (`name`, `description`, `argument-hint`, `allowed-tools`) and instructions
3. Optionally add `references/` and `templates/` directories for supporting material
