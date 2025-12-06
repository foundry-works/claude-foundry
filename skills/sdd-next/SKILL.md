---
name: sdd-next
description: Task preparation skill for spec-driven workflows. Reads specifications, identifies next actionable tasks, and creates detailed execution plans. Use when ready to implement a task from an existing spec - bridges the gap between planning and coding.
---

# SDD-Next: Concise Playbook

## Overview

- **Purpose:** Repeatable workflow for discovering specs, selecting actionable tasks, and keeping the user in the loop from planning through wrap-up.
- **Scope:** Single-task pulls and phase-focused loops; work happens inside project root.
- **Audience:** AI agents orchestrating SDD workflows who must respect human checkpoints and CLI guardrails.

### High-Level Flow

```
Start -> Read Work Mode (Step 0)
           |
           +-- single mode --> Select Task -> Plan -> Approve -> Implement -> Complete
           |
           +-- autonomous mode --> Phase Loop (auto-complete all tasks) -> Summary
```

---

## MCP Tooling

- This skill interacts solely with the Foundry MCP server (`foundry-mcp`). Tool names use the `mcp__foundry-mcp__<tool-name>` syntax.
- The agent never invokes the CLI directly. When the narrative says "call" a tool, it refers to issuing an MCP invocation.
- Stay inside the repo root, avoid chained shell commands, and never read raw spec JSON outside of MCP helpers.

---

## Step 0: Read Work Mode Configuration

**CRITICAL: This must be the FIRST step when sdd-next is invoked.**

Call `mcp__foundry-mcp__session-work-mode` to read the user's configuration:
```json
{"work_mode": "single", "agent_type": "claude-code", "modes_available": ["single", "autonomous"]}
```

**Routing:**
- `"single"`: Follow **Single Task Workflow** below
- `"autonomous"`: Follow **Autonomous Mode** (see `reference.md#autonomous-mode-phase-completion`)

---

## CRITICAL: Global Requirements

### Spec Reading Rules (NEVER VIOLATE)

The skill **only** interacts with specs via MCP tools:
- `mcp__foundry-mcp__task-prepare`
- `mcp__foundry-mcp__task-info`
- `mcp__foundry-mcp__specs-find`
- `mcp__foundry-mcp__task-query`

Direct JSON access (`Read()`, `cat`, `jq`, `grep`, etc.) is prohibited.

### User Interaction Requirements

**Gate key decisions with `AskUserQuestion` (MANDATORY):**
- Spec selection (when multiple available)
- Task selection (recommended vs alternatives)
- Plan approval (before implementation)
- Blocker handling (alternative tasks or resolve)
- Completion verification (for verify tasks)

**Anti-Pattern:** Never use text-based numbered lists. Always use `AskUserQuestion` for structured choices.

### Anti-Recursion Rule (NEVER VIOLATE)

This skill must NEVER invoke itself or `Skill(sdd-next)`. The only valid callers are:
- The `/sdd-next` command (entry point)
- Direct user invocation

If you find yourself about to call `Skill(sdd-next)` from within this skill, **STOP** and proceed with the workflow instead. The skill handles the complete task lifecycle - there is no need to re-invoke it.

> For detailed context gathering patterns, see `reference.md#context-gathering-best-practices`
> For agent delegation patterns (when to use sdd-planner), see `reference.md#agent-delegation`

---

## CRITICAL: Context Checking Pattern

**Before checking context, you MUST generate a session marker first.**

This is a **two-step process** that must run **sequentially**:

1. Call `mcp__foundry-mcp__session-generate-marker` to mint a new marker.
2. Immediately call `mcp__foundry-mcp__session-context`, passing the marker string.

### Context Thresholds

- **< 85%**: Safe to continue
- **>= 85%**: Stop and recommend `/clear` before the next task

### CRITICAL: Never Anticipate Context Usage

**ONLY check actual context percentage - NEVER speculate about future consumption:**

- DO NOT stop early because "upcoming work will consume context"
- DO ONLY stop when context is CURRENTLY at or above 85%

### When to Check Context

- **Autonomous mode**: After EVERY task completion (REQUIRED)
- **Single-task mode**: After task completion (recommended)
- **Session start**: Before intensive work (optional)

---

## Work Mode Behavior

**Single Task Mode** (`"work_mode": "single"`) - Default
- Plan and execute one task at a time with explicit user approval
- After task completion, surface next recommendation and wait for user decision

**Autonomous Mode** (`"work_mode": "autonomous"`)
- Complete all tasks in current phase automatically within context limits
- Check context after EVERY task completion (required)
- Stop only for blockers, plan deviations, or when context >=85%

> For autonomous mode workflow details, see `reference.md#autonomous-mode-phase-completion`

---

## Single Task Workflow

Use this workflow when work mode is **single**. Execute one task at a time with explicit user approval.

### 3.1 Choose the Spec (Skip if called from /sdd-next command)

**IMPORTANT:** If this skill was invoked from the `/sdd-next` command with context indicating "Active spec detected", the spec has already been identified. **Skip this step** and proceed directly to Step 3.2.

Only execute this step if entering the skill directly (not via command):
- If the user supplies a spec identifier, confirm via `mcp__foundry-mcp__specs-find`
- Otherwise list candidates with `mcp__foundry-mcp__specs-find` (filter `active`)

### 3.2 Select Task

- **Recommendation path**: `mcp__foundry-mcp__prepare-task` -> surface task id, file, estimates, blockers
- **Browsing path**: Use `mcp__foundry-mcp__task-query` -> present shortlist via `AskUserQuestion`

### 3.3 Deep Dive & Plan Approval

Invoke `mcp__foundry-mcp__prepare-task` with the target `spec_id`. The response contains:
- `task_data`: title, metadata, instructions
- `dependencies`: blocking status (can_start, blocked_by list)
- `context`: previous sibling, parent task, phase, sibling files, journal, dependencies

Treat `context` as the authoritative source. Only fall back to `mcp__foundry-mcp__task-info` when the spec explicitly requires absent data.

> For context field details and JSON structure, see `reference.md#deep-dive-context-structure`

**Draft execution plan:**
1. Confirm previous edits in `context.sibling_files`
2. Align deliverables with `context.parent_task`
3. Call out open risks via `context.phase`
4. Reference `context.previous_sibling.summary` for continuity

**Present plan via `AskUserQuestion`:**
- Options: "Approve & Start", "Request Changes", "More Details", "Defer"

### 3.4 Implementation Handoff

**Before coding:**
```bash
mcp__foundry-mcp__task-update-status {spec-id} {task-id} in_progress --note "context"
```

**During implementation:**
- Follow execution plan
- Document any deviations immediately

> For using `doc-scope --view implement` during implementation, see `reference.md#using-doc-scope-during-implementation`

**After implementation:**

Mark task complete using the sdd-update skill:
```bash
Skill(foundry:sdd-update) "Complete task {task-id} in spec {spec-id}. Completion note: [Summary of what was accomplished, tests run, verification performed]."
```

### 3.5 Surface Next Recommendation

**Immediately after completion:**
```bash
mcp__foundry-mcp__prepare-task {spec-id}
```

- Summarize next task's scope and blockers
- Check with user before proceeding
- If no pending work or spec complete, surface that clearly

> For post-implementation checklist, see `reference.md#post-implementation-checklist`

---

## CRITICAL: Completion Requirements

### Never Mark Complete If:

- Tests are failing
- Implementation is partial
- You encountered unresolved errors
- You couldn't find necessary files or dependencies
- Blockers exist that prevent verification

### If Blocked or Incomplete:

- Keep task as `in_progress`
- Create new task describing what needs resolution
- Document blocker using `mcp__foundry-mcp__task-block`
- Present alternatives to user via `AskUserQuestion`

### Resolving Blocked Tasks

```bash
mcp__foundry-mcp__task-unblock {spec-id} {task-id} --resolution "Brief description"
```

### Completion Journal Requirements

**MUST provide journal content** describing:
- What was accomplished
- Tests run and results
- Verification performed
- Any deviations from plan
- Files created/modified

**Example:**
```bash
Skill(foundry:sdd-update) "Complete task task-2-3 in spec my-spec-001. Completion note: Implemented JWT auth middleware with PKCE flow. All 12 unit tests passing. Manual verification: login flow works in dev environment. Created src/middleware/auth.ts (180 lines) and tests/middleware/auth.spec.ts (45 tests)."
```

---

## CRITICAL: Verification Tasks

### Detecting Verification Tasks

Check task metadata for `type: verify` or `verification_type` field via `mcp__foundry-mcp__task-info`.

### Dispatch by Verification Type

| verification_type | Action |
|-------------------|--------|
| `"run-tests"` | Invoke `Skill(foundry:run-tests)` or use `mcp__foundry-mcp__test-run` |
| `"fidelity"` | Invoke `Skill(foundry:sdd-fidelity-review)` or use `mcp__foundry-mcp__spec-review-fidelity` |

**After verification completes:**
1. Present findings to user
2. Use `AskUserQuestion` to get approval before marking complete
3. Options: "Accept & Complete", "Fix Failures", "Review Details"

---

## Quick Reference

### Core Commands

```bash
# Discovery
mcp__foundry-mcp__specs-find --status active
mcp__foundry-mcp__list-phases {spec-id}

# Task Selection
mcp__foundry-mcp__prepare-task {spec-id}              # Primary command - includes all context
mcp__foundry-mcp__task-next {spec-id}                 # Simpler alternative - just task ID

# Context Checking (TWO STEPS - SEQUENTIAL)
mcp__foundry-mcp__session-generate-marker
mcp__foundry-mcp__session-context --session-marker "SESSION_MARKER_<hash>"

# Advanced
mcp__foundry-mcp__task-query {spec-id} --status pending --parent {phase-id}
mcp__foundry-mcp__list-blockers {spec-id}
mcp__foundry-mcp__task-unblock {spec-id} {task-id} [--resolution "reason"]
```

### Critical Patterns Summary

| Pattern | Requirement |
|---------|-------------|
| **Spec reading** | Always use MCP commands, NEVER `Read()` or `cat` on JSON |
| **Context checking** | Two-step sequential (marker -> context), never combined |
| **Task completion** | Never mark complete if tests failing/partial/errors |
| **Autonomous mode** | Check context after EVERY task, stop at >=85% |
| **Verification** | Dispatch to appropriate skill by `verification_type` |
| **User decisions** | Always use `AskUserQuestion`, never text lists |

---

## Detailed Reference

For comprehensive documentation including:
- Context gathering best practices and anti-patterns
- Tool value matrix
- Deep dive context JSON structure
- Using doc-scope during implementation
- Post-implementation checklist
- Autonomous mode workflow (full)
- Phase loop with human checkpoints
- Troubleshooting

See **[reference.md](./reference.md)**
