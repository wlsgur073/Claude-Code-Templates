# Claude Code Project Setup

You are a Claude Code configuration expert. Analyze the current project and set up optimal Claude Code configuration through interactive conversation.

Follow these phases in order.

---

## Phase 1: Analyze

Scan the project silently:

- Read dependency files: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `Gemfile`, etc.
- Identify primary language, framework, project type, and directory structure
- Detect test frameworks, linters, formatters, and build tools from dependencies
- Check for existing `CLAUDE.md`, `.claude/` directory, or `.claude/rules/`

Do NOT output your analysis yet. Use it to inform your questions.

## Phase 2: Ask Questions

Ask the user the following questions **one at a time**. Use detected defaults from Phase 1 when possible (e.g., "I found Jest in your devDependencies. Is `npm test` your test command?").

1. **Project overview** — Confirm detected language/framework. Ask for a 1-2 sentence description of what the project does.
2. **Build & run** — Confirm or ask for build command, dev server command, and how to run the project.
3. **Testing** — Confirm test runner and command. Ask about coverage targets and preferred test patterns (factories, mocks, integration, etc.).
4. **Code style** — Ask about naming conventions, formatting preferences (indentation size, line length), and import ordering.
5. **Workflow** — Ask about branch naming convention, commit message style, and any pre-development checklist items.
6. **Advanced features** — Ask: "Would you like to set up any of these optional features?"
   - (a) Auto-linting hooks — automatically runs linter after every file edit
   - (b) File protection hooks — blocks edits to `.env` and sensitive files
   - (c) Custom agent roles — specialized agents for different parts of the codebase
   - (d) Custom skill commands — reusable multi-step workflow automations
   - (e) None for now

## Phase 3: Generate Files

Create all files based on user answers. Follow these rules strictly:

- **Be specific and verifiable** — "Use 2-space indentation" not "Format code properly"
- **Omit obvious information** — don't state what Claude can infer from reading the code
- **All commands must be copy-pasteable** — use actual project commands, not placeholders
- **Use actual directory names** — reference the project's real structure, not generic examples
- **CLAUDE.md must stay under 200 lines** — be concise
- **Merge, don't overwrite** — if `CLAUDE.md` or `settings.json` already exists, merge new content into the existing file

### Always generate:

**`CLAUDE.md`** with these sections:

```
# Project Overview        ← user's description + language/framework
## Build & Run            ← exact commands from user answers
## Testing                ← test commands + verification commands Claude can run
## Code Style & Conventions ← only rules that differ from language defaults
## Workflow               ← branch/commit conventions, pre-dev checklist
## Project Structure      ← key directories and purposes (from your analysis)
## Important Context      ← non-obvious things discovered during analysis
## References             ← @import links to relevant docs if they exist
```

**`.claude/settings.json`**:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [],
    "deny": []
  }
}
```

- `allow`: add test, lint, and build commands (e.g., `"Bash(npm test)"`, `"Bash(npm run lint)"`)
- `deny`: add sensitive file patterns (e.g., `"Read(.env)"`, `"Read(.env.*)"`)

**`.claude/rules/code-style.md`**:

```
# Code Style
## Naming Conventions     ← from user answers
## Formatting             ← indentation, line length, trailing commas, etc.
## Imports                ← import grouping and ordering
```

Include `✗ bad / ✓ good` code examples for non-obvious rules.

**`.claude/rules/testing.md`**:

```
# Testing Conventions
## Test Framework         ← detected framework
## Test Structure         ← directory layout, file naming
## Coverage               ← targets from user answers
## Patterns               ← factories, mocks, integration patterns
```

**`.claude/rules/architecture.md`**:

```
# Architecture
## Layer Structure        ← detected layers from project analysis
## Dependency Direction   ← inferred dependency rules
## Module Boundaries      ← module organization patterns
```

**`.claude/rules/workflow.md`**:

```
# Workflow
## Pre-Development Checklist
- Read relevant CLAUDE.md sections
- Review existing tests for the area to modify
- Check git status for uncommitted changes
[+ user's specific checklist items]

## Review Gates
- All tests pass
- Lint has zero warnings
- Build succeeds
[+ user's specific gates]
```

**`.gitignore`** — append this line if not already present:

```
.claude/settings.local.json
```

### Conditionally generate (only if user opted in):

**Auto-linting hooks** — add `hooks` key to `.claude/settings.json`:

```json
"hooks": {
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "[detected-linter-command] --fix \"$CLAUDE_FILE_PATH\" 2>/dev/null || true",
          "statusMessage": "Auto-linting edited file"
        }
      ]
    }
  ]
}
```

Replace `[detected-linter-command]` with the actual linter (e.g., `npx eslint`, `ruff`, `gofmt`).

**File protection hooks** — add `PreToolUse` to hooks:

```json
"PreToolUse": [
  {
    "matcher": "Edit|Write",
    "hooks": [
      {
        "type": "command",
        "command": "echo \"$CLAUDE_FILE_PATH\" | grep -qE '\\.(env|pem|key)' && echo 'BLOCK: Protected file' && exit 1 || exit 0",
        "statusMessage": "Checking for protected files"
      }
    ]
  }
]
```

**Agent roles** — ask user for role name and scope, then create `.claude/agents/<name>.md`:

```markdown
---
name: "[Role Name]"
description: "[Specialization]"
tools:
  - Read
  - Edit
  - Write
  - Bash
  - Grep
  - Glob
model: "sonnet"
---

# Scope
[Directories this agent modifies]

## Rules
[Domain-specific rules]
```

**Skill commands** — ask user for skill purpose, then create `.claude/skills/<name>/SKILL.md`:

```markdown
---
name: "[skill-name]"
description: "[What this skill automates]"
---

# Steps

## Step 1: Gather Information
[What to ask the user]

## Step 2: Validate
[Pre-checks before proceeding]

## Step 3: Execute
[Files to create or modify]

## Step 4: Verify
[Build/test commands to confirm]
```

## Phase 4: Wrap Up

After generating all files:

1. Print a summary table listing every created/modified file and what it contains
2. If you merged into existing files, explain what was added
3. Tell the user: "Run `/memory` to verify all configuration files are loaded"
4. Suggest trying a simple task to test the configuration works
