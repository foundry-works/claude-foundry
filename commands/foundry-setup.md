---
name: foundry-setup
description: First-time setup for the claude-foundry plugin
argument-hint: [--check]
---

# Foundry Setup Command

When invoked, execute these phases in order. This command is idempotent and safe to run multiple times.

## Argument Handling

Check if `$ARGUMENTS` contains `--check` or `--preflight`:
- **If flag present:** Run only Phase 1 (Pre-flight Checks), display results, then stop
- **If no flag:** Run all phases (full setup)

## Phase 1: Pre-flight Checks

Run these checks in parallel to verify the environment:

1. Call `mcp__plugin_foundry_foundry-mcp__health action="liveness"` to verify MCP server connectivity
2. Call `mcp__plugin_foundry_foundry-mcp__environment action="verify-toolchain"` to check required tools (Python, git)
3. Call `mcp__plugin_foundry_foundry-mcp__environment action="verify"` to validate runtime versions

Present results in a status table:

```
## Pre-flight Checks

| Check | Status | Details |
|-------|--------|---------|
| MCP Server | [✓/✗] | foundry-mcp responding / connection failed |
| Python | [✓/✗] | version number |
| Git | [✓/✗] | version number |
```

**If MCP server check fails:** Stop and explain that foundry-mcp must be configured. Point to installation instructions:
- PyPI: `pip install foundry-mcp`
- Or: `uvx foundry-mcp`
- Configure in Claude Code MCP settings

**If other checks fail:** Warn but continue - some features may not work.

**If `--check` or `--preflight` flag was provided:** Stop here. Display the pre-flight results summary and exit. Do not continue to Phase 2 or beyond.

## Phase 1.5: Permission Management

Claude Code requires explicit permissions for plugin tools. This phase automatically detects, compares, and configures the required permissions.

### Step 1: Run Permission Check

Execute the permission check script:

```bash
WORKSPACE_DIR="$PWD" ${CLAUDE_PLUGIN_ROOT}/scripts/setup-permissions --diff
```

Parse the JSON output to determine the permission status.

### Step 2: Handle Based on Status

**If status is `complete`:**

Display:
> "Permissions are configured correctly. All required plugin permissions are present."

Continue to Phase 2.

---

**If status is `no_file`:**

Explain:
> "The `.claude/settings.local.json` file doesn't exist in this project. This file controls which tools Claude Code can use without prompting.
>
> The plugin requires permissions for:
> - All foundry-mcp MCP tools
> - Git commands (status, diff, log, etc.)
> - Test execution (pytest)
> - AI provider CLIs (codex, claude, gemini, cursor-agent, opencode)
> - Read/write access to specs/ directories"

Use `AskUserQuestion`:
- Question: "Create `.claude/settings.local.json` with plugin permissions?"
- Options:
  - "Yes, create the file (recommended)"
  - "No, I'll configure this manually"

**If "Yes":** Run the script with `--create`:
```bash
WORKSPACE_DIR="$PWD" ${CLAUDE_PLUGIN_ROOT}/scripts/setup-permissions --create
```

Display: "Created `.claude/settings.local.json` with plugin permissions."

**If "No":** Ask if they want to continue without permissions, then proceed or exit.

---

**If status is `invalid`:**

Explain:
> "Found `.claude/settings.local.json` but it contains invalid JSON. The file will be backed up before replacement."

Use `AskUserQuestion`:
- Question: "Replace invalid file with valid permissions? (Original will be backed up)"
- Options:
  - "Yes, backup and replace"
  - "No, I'll fix it manually"

**If "Yes":** Run the script with `--apply` (which backs up invalid files automatically).

---

**If status is `missing`:**

Display the categorized diff from `missing_by_category` as a table:

```
## Permission Comparison

| Category | Status |
|----------|--------|
| MCP Tools | {count or checkmark} |
| Git Commands | {count or checkmark} |
| Testing | {count or checkmark} |
| Foundry CLI | {count or checkmark} |
| AI Providers | {count or checkmark} |
| Specs Write | {count or checkmark} |
| Specs Edit | {count or checkmark} |
| Deny Rules | {count or checkmark} |
```

For categories with missing permissions, show the specific permissions:

```
**Missing Git Commands:**
+ Bash(git status:*)
+ Bash(git diff:*)
...
```

If `user_custom` is not empty, note:
> "Your custom permissions will be preserved: {list}"

Use `AskUserQuestion`:
- Question: "How would you like to update permissions?"
- Options:
  - "Add all missing permissions (recommended)"
  - "Skip - I'll update manually"

**If "Add all missing":** Run the script with `--apply`:
```bash
WORKSPACE_DIR="$PWD" ${CLAUDE_PLUGIN_ROOT}/scripts/setup-permissions --apply
```

Display: "Permissions updated! Added {X} allow permissions and {Y} deny rules."

**If "Skip":** Warn that some features may require manual approval, then continue.

## Phase 2: Workspace Setup

1. Call `mcp__plugin_foundry_foundry-mcp__environment action="detect-topology"` to analyze the project structure
2. Call `mcp__plugin_foundry_foundry-mcp__spec action="list"` to check for existing specifications

### If specs/ directory does not exist:

Use `AskUserQuestion` to ask:
- Question: "Would you like to initialize the specs directory structure?"
- Options: "Yes, initialize" / "No, skip"

If yes: Call `mcp__plugin_foundry_foundry-mcp__environment action="init-workspace"` to create:
- specs/active/
- specs/pending/
- specs/completed/
- specs/archived/

### If foundry-mcp.toml does not exist:

Use `AskUserQuestion` to ask:
- Question: "Would you like to create a foundry-mcp.toml configuration file?"
- Options: "Yes, create with defaults" / "No, skip"

If yes: Call `mcp__plugin_foundry_foundry-mcp__environment action="setup"` with `dry_run=false`

Report what was created or skipped.

## Setup Complete

Summarize what was configured:
- Pre-flight check results (what passed/failed)
- Permissions status (created/updated/skipped)
- Workspace setup (specs directory, foundry-mcp.toml)

**Important:** If permissions were added or modified, display:
> "**Restart Claude Code** for the permission changes to take effect. After restarting, run `/foundry-tutorial` if this is your first time using the plugin."

If no permissions were changed, end with:
> "Setup complete! If this is your first time using the plugin, run `/foundry-tutorial` to learn about Spec-Driven Development and see the workflow in action."
