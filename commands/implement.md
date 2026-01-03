---
name: implement
description: Resume or start spec-driven development work by detecting active tasks and providing interactive options
argument-hint: [--auto] [--delegate] [--parallel]
---

# SDD Implement Command

When invoked, follow these steps:

## Step 0: Flag Parsing and Session Check

### Load TOML Defaults

Call the environment tool to read configuration (returns both `implement` and `git` sections):

```bash
mcp__plugin_foundry_foundry-mcp__environment action="get-config"
```

This returns:
```json
{
  "success": true,
  "data": {
    "sections": {
      "implement": {"auto": false, "delegate": false, "parallel": false},
      "git": {"enabled": true, "commit_cadence": "task", ...}
    }
  }
}
```

Use `implement` values as the baseline. CLI flags override TOML values.
The `git` section is available for commit cadence decisions during implementation.

**If the config file is missing or section not found:** Use defaults (all false for implement).

### Parse Command Flags

Three orthogonal flags that can be combined:

| Flag | Effect |
|------|--------|
| `--auto` | Skip prompts between tasks (autonomous execution) |
| `--delegate` | Use subagent(s) for implementation |
| `--parallel` | Run subagents concurrently (implies `--delegate`) |

**Resolution order:** TOML defaults → CLI flags (CLI wins)

**Resulting modes from combinations:**

| Flags | Subagent | Concurrent | Prompts | Description |
|-------|----------|------------|---------|-------------|
| (none) | ❌ | ❌ | ✅ | Interactive, inline |
| `--auto` | ❌ | ❌ | ❌ | Autonomous, inline |
| `--delegate` | ✅ | ❌ | ✅ | Interactive, sequential subagent |
| `--auto --delegate` | ✅ | ❌ | ❌ | Autonomous, sequential subagent |
| `--delegate --parallel` | ✅ | ✅ | ✅ | Interactive, concurrent subagents |
| `--auto --delegate --parallel` | ✅ | ✅ | ❌ | Autonomous, concurrent subagents |

Note: `--parallel` without `--delegate` implies `--delegate`.

### Check for Existing Session

Before proceeding, check if a paused session exists:

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-config" spec_id={spec-id} command="status"
```

**If session is `paused`:**
```
AskUserQuestion:
"Found paused session (reason: {pause_reason}). Last task: {current_task}"
Options:
- "Resume session" → session-config command="resume", then continue
- "Start fresh" → session-config command="end", then proceed normally
- "Exit"
```

**If session is `active`:**
Warn user that a session is already running. Offer to view status or exit.

**If session is `idle` or no session:**
Proceed to Step 1.

### Interactive Fallback (No Flags and No TOML Defaults)

When no CLI flags provided AND TOML defaults are all false, offer mode selection:

```
AskUserQuestion:
"Select execution mode:"
Options:
- "Interactive, inline (default)" → Proceed to Step 1
- "Autonomous, inline (--auto)" → Proceed with --auto
- "Interactive, delegated (--delegate)" → Proceed with --delegate
- "Autonomous, delegated (--auto --delegate)" → Proceed with --auto --delegate
- "Autonomous, parallel (--auto --delegate --parallel)" → Proceed with all flags
```

**If TOML has defaults set:** Skip this prompt and use TOML values directly.

## Step 1: Check for Active Specifications

Use `mcp__plugin_foundry_foundry-mcp__spec action="list"` with status "active" to find specifications in progress.

## Step 2: Route Based on Results

### If active specs exist:
Pass the active spec context to the `sdd-implement` skill. The skill will proceed directly to task selection - it does NOT need to re-check for active specs.

**IMPORTANT:** Do NOT invoke this command or skill recursively. The skill handles the full workflow.

```
Skill(sdd-implement) "Active spec detected via /implement command. Skip Step 3.1 (spec detection) and proceed directly to Step 3.2 (task selection)."
```

### If no active specs exist:
Check for pending specs using `mcp__plugin_foundry_foundry-mcp__spec action="list"` with status "pending".

**If pending specs found:**
Ask user if they want to activate one using `AskUserQuestion`.

**If no specs at all:**
Inform user no specifications exist and offer options:
1. Create a new spec using the sdd-plan skill
2. View completed/archived specs
3. Exit

## Step 3: Hand Off to Skill

Once routing is determined, the sdd-implement skill handles the full workflow:
- Task preparation and context gathering
- Implementation guidance
- Progress tracking and verification

This command is the user entry point; the skill contains the detailed workflow logic.
