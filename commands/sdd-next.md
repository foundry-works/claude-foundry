---
name: sdd-next
description: Resume or start spec-driven development work by detecting active tasks and providing interactive options
---

# SDD Next Command

When invoked, follow these steps:

## Step 1: Check for Active Specifications

Use `mcp__foundry-mcp__spec-list` with status "active" to find specifications in progress.

## Step 2: Route Based on Results

### If active specs exist:
Pass the active spec context to the `sdd-next` skill. The skill will proceed directly to task selection - it does NOT need to re-check for active specs.

**IMPORTANT:** Do NOT invoke this command or skill recursively. The skill handles the full workflow.

```
Skill(sdd-next) "Active spec detected via /sdd-next command. Skip Step 3.1 (spec detection) and proceed directly to Step 3.2 (task selection)."
```

### If no active specs exist:
Check for pending specs using `mcp__foundry-mcp__spec-list` with status "pending".

**If pending specs found:**
Ask user if they want to activate one using `AskUserQuestion`.

**If no specs at all:**
Inform user no specifications exist and offer options:
1. Create a new spec using the sdd-plan skill
2. View completed/archived specs
3. Exit

## Step 3: Hand Off to Skill

Once routing is determined, the sdd-next skill handles the full workflow:
- Work mode detection (single vs autonomous)
- Task preparation and context gathering
- Implementation guidance
- Progress tracking and verification

This command is the user entry point; the skill contains the detailed workflow logic.
