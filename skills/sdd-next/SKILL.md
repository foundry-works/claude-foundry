---
name: sdd-next
description: Task preparation skill for spec-driven workflows. Reads specifications, identifies next actionable tasks, and creates detailed execution plans. Use when ready to implement a task from an existing spec - bridges the gap between planning and coding.
---

# SDD-Next: Concise Playbook

## Overview

- **Purpose:** Task execution workflow - select tasks, plan implementation, and track progress within an active spec.
- **Scope:** Single-task execution with user approval at key checkpoints.
- **Entry:** Invoked by `/next-cmd` command after spec has been identified.

### Flow

> `[x?]`=decision · `(GATE)`=user approval · `→`=sequence · `↻`=loop · `§`=section ref

```
- **Entry** → SelectTask
  - [recommend] → `task action="prepare"` → ShowRecommendation
  - [browse] → `task action="query"` → (GATE: task selection)
- → TypeDispatch
    - [type=verify?] → §VerifyWorkflow
    - [type=task?] → DeepDive
      - `task action="prepare"` → ExtractContext
      - DraftPlan → (GATE: approve plan)
        - [approved] → PreImpl
        - [changes] → ↻ back to DraftPlan
        - [defer] → **Exit**
      - PreImpl: LSP analysis → Explore subagent
      - `task action="update-status" status="in_progress"`
      - **Implement**
      - PostImpl: `Skill(sdd-update)` to complete
      - [success?] → SurfaceNext
        - `task action="prepare"` → ShowNextTask
        - (GATE: continue?) → ↻ back to SelectTask | **Exit**
      - [blocked?] → HandleBlocker
        - `task action="block"` → (GATE: alternatives)
        - [resolve] → ↻ back to Implement
        - [skip] → SurfaceNext

§VerifyWorkflow → see references/verification.md
```

> Flow notation: see [docs/flow-notation.md](../../docs/flow-notation.md)

---

## MCP Tooling

- This skill interacts solely with the Foundry MCP server (`foundry-mcp`). Tools use the router+action pattern: `mcp__plugin_foundry_foundry-mcp__<router>` with `action="<action>"`.
- The agent never invokes the CLI directly. When the narrative says "call" a tool, it refers to issuing an MCP invocation.
- Stay inside the repo root, avoid chained shell commands, and never read raw spec JSON outside of MCP helpers.

---

## CRITICAL: Global Requirements

### Spec Reading Rules (NEVER VIOLATE)

The skill **only** interacts with specs via MCP tools:
- `mcp__plugin_foundry_foundry-mcp__task action="prepare"`
- `mcp__plugin_foundry_foundry-mcp__task action="info"`
- `mcp__plugin_foundry_foundry-mcp__spec action="find"`
- `mcp__plugin_foundry_foundry-mcp__task action="query"`

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
- The `/next-cmd` command (entry point)
- Direct user invocation

If you find yourself about to call `Skill(sdd-next)` from within this skill, **STOP** and proceed with the workflow instead. The skill handles the complete task lifecycle - there is no need to re-invoke it.

> For detailed context gathering patterns, see `references/context-gathering.md`
> For agent delegation patterns (when to use sdd-planner), see `references/agent-delegation.md`

---

## Task Workflow

Execute one task at a time with explicit user approval.

**Assumption:** The `/next-cmd` command has already identified the active spec and passed it to this skill.

### Select Task

- **Recommendation path**: `mcp__plugin_foundry_foundry-mcp__task action="prepare"` -> surface task id, file, estimates, blockers
- **Browsing path**: Use `mcp__plugin_foundry_foundry-mcp__task action="query"` -> present shortlist via `AskUserQuestion`

### Task Type Dispatch

After selecting a task, check its type to determine the workflow path.

**Check task type** from the `task action="prepare"` response or via `mcp__plugin_foundry_foundry-mcp__task action="info"`:

| Task Type | Workflow Path |
|-----------|---------------|
| `type: "verify"` | Go to **Verification Task Workflow** |
| `type: "task"` (default) | Continue to **Deep Dive & Plan Approval** |

**CRITICAL:** Verification tasks must NOT go through the implementation workflow. They require MCP tool invocation, not code changes.

### Deep Dive & Plan Approval (Implementation Tasks Only)

**Note:** This section is for `type: "task"` only. Verification tasks (`type: "verify"`) use the Verification Task Workflow.

Invoke `mcp__plugin_foundry_foundry-mcp__task action="prepare"` with the target `spec_id`. The response contains:
- `task_data`: title, metadata, instructions
- `dependencies`: blocking status (can_start, blocked_by list)
- `context`: previous sibling, parent task, phase, sibling files, journal, dependencies

Treat `context` as the authoritative source. Only fall back to `mcp__plugin_foundry_foundry-mcp__task action="info"` when the spec explicitly requires absent data.

> For context field details and JSON structure, see `references/context-structure.md`

**Draft execution plan:**
1. Confirm previous edits in `context.sibling_files`
2. Align deliverables with `context.parent_task`
3. Call out open risks via `context.phase`
4. Reference `context.previous_sibling.summary` for continuity

**Present plan via `AskUserQuestion`:**
- Options: "Approve & Start", "Request Changes", "More Details", "Defer"

### Subagent Guidance (Pre-Implementation Exploration)

Before implementing, use Claude Code's built-in subagents for efficient codebase exploration:

| Scenario | Subagent | Thoroughness |
|----------|----------|--------------|
| Find related files/patterns | Explore | medium |
| Understand unfamiliar code areas | Explore | very thorough |
| Complex multi-file investigation | general-purpose | N/A |

**Example invocation:**
```
Use the Explore agent (medium thoroughness) to find:
- Existing implementations of similar patterns
- Test files for the target module
- Related documentation that may need updates
```

**Benefits of subagent exploration:**
- Prevents context bloat in main conversation
- Haiku model is faster for search operations
- Returns focused results for detailed analysis
- Keeps main context available for implementation

> For more subagent patterns including autonomous mode usage, see `references/subagent-patterns.md`

### LSP Dependency Analysis

Before implementing, use LSP tools to verify dependencies and preview impact:

1. **Verify dependencies exist**: Use `goToDefinition` on symbols the task modifies
2. **Preview impact**: Use `findReferences` to identify all affected files and call sites
3. **Include in plan**: Surface LSP findings (usage count, affected files, test coverage) in plan approval
4. **Fallback**: If LSP unavailable for file type, use Explore agent to find imports and usages

### Implementation Handoff

**Before coding:**
```bash
mcp__plugin_foundry_foundry-mcp__task action="update-status" spec_id={spec-id} task_id={task-id} status="in_progress" note="context"
```

**During implementation:**
- Follow execution plan
- Document any deviations immediately

**After implementation:**

Mark task complete using the sdd-update skill:
```bash
Skill(foundry:sdd-update) "Complete task {task-id} in spec {spec-id}. Completion note: [Summary of what was accomplished, tests run, verification performed]."
```

### Surface Next Recommendation

**Context Awareness:** The `context-monitor` hook automatically warns when context exceeds 85%. If you see `[CONTEXT X%]` warnings, follow the recommendation to `/clear` then `/next-cmd` after completing the current task.

**Surface next recommendation:**
```bash
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id}
```

- Summarize next task's scope and blockers
- Check with user before proceeding
- If no pending work or spec complete, surface that clearly

> For post-implementation checklist, see `references/checklist.md`

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
- Document blocker using `mcp__plugin_foundry_foundry-mcp__task action="block"`
- Present alternatives to user via `AskUserQuestion`

### Resolving Blocked Tasks

```bash
mcp__plugin_foundry_foundry-mcp__task action="unblock" spec_id={spec-id} task_id={task-id} resolution="Brief description"
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

## Verification Task Workflow

**Entry:** Routed here from Task Type Dispatch when task has `type: "verify"`

> For the complete verification workflow (mark in progress, detect type, dispatch, execute, complete/remediate), see [references/verification.md](./references/verification.md)

---

## Detailed Reference

For comprehensive documentation including:
- Context gathering best practices → `references/context-gathering.md`
- Agent delegation patterns → `references/agent-delegation.md`
- Deep dive context JSON structure → `references/context-structure.md`
- Built-in subagent patterns → `references/subagent-patterns.md`
- Post-implementation checklist → `references/checklist.md`
- Verification task workflow → `references/verification.md`
- Troubleshooting → `references/troubleshooting.md`
