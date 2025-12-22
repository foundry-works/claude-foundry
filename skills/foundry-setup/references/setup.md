# Foundry Setup - Detailed Phase Instructions

## Phase 0: Ensure Full Mode

Before running pre-flight checks, ensure the MCP server is in full mode so all tools are available.

1. Call `mcp__plugin_foundry_foundry-ctl__get_sdd_mode` to check current mode
2. **If mode is "minimal":** Call `mcp__plugin_foundry_foundry-ctl__set_sdd_mode` with `mode="full"` to enable all tools. Wait for the server restart (~1-2 seconds).
3. **If mode is already "full":** Continue to Phase 1

**Why this matters:** In minimal mode, only the `wake` tool is available. Without full mode, the setup skill cannot use the MCP tools for pre-flight checks and will fall back to slow Bash commands.

---

## Phase 1: Pre-flight Checks

Run these checks in parallel to verify the environment:

1. Call `mcp__plugin_foundry_foundry-mcp__health action="liveness"` to verify MCP server connectivity
2. Call `mcp__plugin_foundry_foundry-mcp__environment action="verify-toolchain"` to check required tools (Python, git)
3. Call `mcp__plugin_foundry_foundry-mcp__environment action="verify-env"` to validate runtime versions

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

**Recommended MCP configuration** (with mode toggling support):

```json
{
  "mcpServers": {
    "foundry-mcp": {
      "command": "python",
      "args": ["-m", "foundry_mcp_ctl", "wrap", "--name", "foundry-mcp", "--", "python", "-m", "foundry_mcp.server"]
    },
    "foundry-ctl": {
      "command": "python",
      "args": ["-m", "foundry_mcp_ctl", "helper"]
    }
  }
}
```

This configuration enables `/on-cmd` and `/off-cmd` commands to toggle between full mode (17 tools) and minimal mode (1 tool) to save context tokens.

**If other checks fail:** Warn but continue - some features may not work.

**If `--check` or `--preflight` flag was provided:** Stop here. Display the pre-flight results summary and exit. Do not continue to Phase 2 or beyond.

---

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

---

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

---

## Phase 2.5: AI Provider Configuration

**Only run this phase if foundry-mcp.toml was created in Phase 2.**

This phase detects available AI providers and configures the `[consultation]` section for plan reviews and other multi-model features.

### Step 1: Discover Available Providers

Call `mcp__plugin_foundry_foundry-mcp__provider action="list"` to get all registered providers.

Parse the response and filter to providers where `available: true`.

### Step 2: Get Provider Metadata

For each available provider, call in parallel:
```
mcp__plugin_foundry_foundry-mcp__provider action="status" provider_id="<provider_id>"
```

Extract `data.metadata.default_model` from each response.

### Step 3: Build Consultation Priority List

Construct priority entries in format: `[cli]{provider_id}:{default_model}`

Example entries:
- `[cli]gemini:gemini-2.5-flash`
- `[cli]claude:claude-sonnet-4-20250514`
- `[cli]opencode:gpt-4`

Sort entries by provider priority from Step 1 (lower number = higher priority in list).

### Step 4: Append to TOML

Read the existing `foundry-mcp.toml` file.

**If the file already contains `[consultation]`:** Skip this step and inform the user that existing configuration is preserved.

**Otherwise:** Append the consultation section:

```toml

[consultation]
# Auto-generated based on detected providers
# Edit priority order or models as needed
priority = [
    "<entry1>",
    "<entry2>",
]
default_timeout = 300
```

Use the Write tool to save the updated file.

### Step 5: Display Results

Present configured providers in a table:

```
## AI Provider Configuration

Detected {N} available provider(s):

| Provider | Model | Priority |
|----------|-------|----------|
| gemini | gemini-2.5-flash | 1 |
| claude | claude-sonnet-4 | 2 |

Configuration written to foundry-mcp.toml.
```

**If no providers are available:**

```
## AI Provider Configuration

No AI providers detected. The [consultation] section was added with an empty priority list.

To enable multi-model features like plan reviews, install one of:
- gemini CLI: https://github.com/google-gemini/gemini-cli
- opencode CLI: https://github.com/opencode-ai/opencode
- codex CLI: https://github.com/openai/codex

Then re-run `/setup-cmd` or manually edit foundry-mcp.toml.
```

---

## Phase 2.6: Test Configuration

**Only run this phase if foundry-mcp.toml exists (either created in Phase 2 or already present).**

This phase detects the appropriate test runner for the project and optionally configures the `[test]` section.

### Step 1: Check Existing Configuration

Read the `foundry-mcp.toml` file and check if it already contains a `[test]` section.

**If `[test]` section exists:** Skip this phase and display:
> "Test configuration already present in foundry-mcp.toml. Your existing settings are preserved."

Continue to Setup Complete.

### Step 2: Detect Test Runner

Call `mcp__plugin_foundry_foundry-mcp__environment action="detect-test-runner"` to analyze the project.

Parse the response to get:
- `detected_runners`: Array of detected runners with confidence levels
- `recommended_default`: The recommended runner based on project type

### Step 3: Present Results and Get User Choice

**If runners were detected:**

Display the detection results:

```
## Test Runner Detection

Detected test runner(s) for this project:

| Runner | Project Type | Confidence | Reason |
|--------|--------------|------------|--------|
| pytest | python | high | pyproject.toml found |
```

Use `AskUserQuestion`:
- Question: "Configure test runner in foundry-mcp.toml?"
- Options:
  - "Yes, use {recommended_default} (recommended)"
  - "Skip test configuration"

**If "Yes":** Proceed to Step 4.
**If "Skip":** Continue to Setup Complete.

---

**If no runners were detected:**

Display:
> "No test runners detected for this project. You can manually configure the `[test]` section in foundry-mcp.toml if needed."

Continue to Setup Complete.

### Step 4: Append Test Section to TOML

Read the existing `foundry-mcp.toml` file content.

Append the test configuration section using textual append (preserves comments and formatting):

```toml

[test]
# Auto-configured based on project detection
default_runner = "{recommended_default}"
```

Use the Write tool to save the updated file.

Display:
> "Added [test] section to foundry-mcp.toml with default_runner = \"{recommended_default}\""

### Step 5: Verify TOML is Valid

After appending, read the file back and verify it's valid TOML by checking that `mcp__plugin_foundry_foundry-mcp__health action="readiness"` doesn't report configuration errors.

**If validation fails:** Warn the user that the file may need manual review.

---

## Setup Complete

Summarize what was configured:
- Pre-flight check results (what passed/failed)
- Permissions status (created/updated/skipped)
- Workspace setup (specs directory, foundry-mcp.toml)
- AI providers configured (list providers added to consultation priority, or note if skipped)
- Test runner configured (runner name, or note if skipped/already configured)

**Important:** If permissions were added or modified, display:
> "**Restart Claude Code** for the permission changes to take effect. After restarting, run `/tutorial-cmd` if this is your first time using the plugin."

If no permissions were changed, end with:
> "Setup complete! If this is your first time using the plugin, run `/tutorial-cmd` to learn about Spec-Driven Development and see the workflow in action."
