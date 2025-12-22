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
Start -> Check Mode (Step 0)
           |
           +-- minimal? --> EXIT: "Run /sdd-on, restart Claude, try again"
           |
           +-- full? --> Select Task (3.2) -> Check Task Type (3.2.1)
                                                    |
                                                    +-- type: "verify" --> Verification Workflow (Section 6) --+
                                                    |                                                          |
                                                    +-- type: "task" --> Plan -> Implement -> Complete ---------+
                                                                                                                |
                                                                                                                v
                                                                                                      Surface Next (3.5)
```

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
- The `/sdd-next` command (entry point)
- Direct user invocation

If you find yourself about to call `Skill(sdd-next)` from within this skill, **STOP** and proceed with the workflow instead. The skill handles the complete task lifecycle - there is no need to re-invoke it.

> For detailed context gathering patterns, see `reference.md#context-gathering-best-practices`
> For agent delegation patterns (when to use sdd-planner), see `reference.md#agent-delegation`

---

## Step 0: Verify SDD Tools Mode

Before proceeding, verify that the full SDD toolkit is available.

1. Call `mcp__plugin_foundry_foundry-ctl__get_sdd_mode`

2. If mode is `"full"`: Proceed to Task Workflow (Step 3.1)

3. If mode is `"minimal"`:
   - Display message:
     ```
     SDD tools are in minimal mode. To use /sdd-next:
     1. Run /sdd-on
     2. Restart Claude
     3. Run /sdd-next again
     ```
   - **Exit skill immediately** — do not proceed

---

## Task Workflow

Execute one task at a time with explicit user approval.

### 3.1 Choose the Spec (Skip if called from /sdd-next command)

**IMPORTANT:** If this skill was invoked from the `/sdd-next` command with context indicating "Active spec detected", the spec has already been identified. **Skip this step** and proceed directly to Step 3.2.

Only execute this step if entering the skill directly (not via command):
- If the user supplies a spec identifier, confirm via `mcp__plugin_foundry_foundry-mcp__spec action="find"`
- Otherwise list candidates with `mcp__plugin_foundry_foundry-mcp__spec action="find"` (filter `active`)

### 3.2 Select Task

- **Recommendation path**: `mcp__plugin_foundry_foundry-mcp__task action="prepare"` -> surface task id, file, estimates, blockers
- **Browsing path**: Use `mcp__plugin_foundry_foundry-mcp__task action="query"` -> present shortlist via `AskUserQuestion`

### 3.2.1 Task Type Dispatch

After selecting a task, check its type to determine the workflow path.

**Check task type** from the `task action="prepare"` response or via `mcp__plugin_foundry_foundry-mcp__task action="info"`:

| Task Type | Workflow Path |
|-----------|---------------|
| `type: "verify"` | Go to **Section 6: Verification Task Workflow** |
| `type: "task"` (default) | Continue to **3.3 Deep Dive & Plan Approval** |

**CRITICAL:** Verification tasks must NOT go through the implementation workflow (3.3-3.4). They require MCP tool invocation, not code changes.

### 3.3 Deep Dive & Plan Approval (Implementation Tasks Only)

**Note:** This section is for `type: "task"` only. Verification tasks (`type: "verify"`) are handled in Section 6.

Invoke `mcp__plugin_foundry_foundry-mcp__task action="prepare"` with the target `spec_id`. The response contains:
- `task_data`: title, metadata, instructions
- `dependencies`: blocking status (can_start, blocked_by list)
- `context`: previous sibling, parent task, phase, sibling files, journal, dependencies

Treat `context` as the authoritative source. Only fall back to `mcp__plugin_foundry_foundry-mcp__task action="info"` when the spec explicitly requires absent data.

> For context field details and JSON structure, see `reference.md#deep-dive-context-structure`

**Draft execution plan:**
1. Confirm previous edits in `context.sibling_files`
2. Align deliverables with `context.parent_task`
3. Call out open risks via `context.phase`
4. Reference `context.previous_sibling.summary` for continuity

**Present plan via `AskUserQuestion`:**
- Options: "Approve & Start", "Request Changes", "More Details", "Defer"

### 3.3.1 Subagent Guidance (Pre-Implementation Exploration)

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

> For more subagent patterns including autonomous mode usage, see `reference.md#built-in-subagent-patterns`

### 3.3.2 LSP-Enhanced Dependency Preview

For tasks modifying existing code, Claude Code's LSP tools can provide precise dependency information:

**Pre-Flight Dependency Check:**

Before starting implementation, verify dependencies exist:

```
# Task modifies AuthService.login() - verify it exists
definition = goToDefinition(file="src/auth/service.py", symbol="login", line=25)

if definition found:
    Proceed with implementation
else:
    Surface missing dependency to user
    Consider blocking task or checking spec accuracy
```

**Cross-File Impact Preview:**

Show which files will be affected by this task:

```
# Task modifies a shared function
references = findReferences(file="src/utils/helpers.py", symbol="format_date", line=10)

# Present to user before implementation:
"This task modifies format_date() which is used in 7 files:
- src/reports/generator.py (3 calls)
- src/api/handlers.py (2 calls)
- tests/test_reports.py (5 calls)
Consider impact on these files during implementation."
```

**Enhanced Plan Presentation:**

Include LSP-derived context in plan approval:

```markdown
## Task: Modify AuthService.login() for 2FA

**Impact Analysis (via LSP):**
- Direct usages: 12 call sites across 5 files
- Test coverage: 8 tests reference this method
- Dependent methods: validate_credentials(), create_session()

**Recommended approach:** [details]

Proceed? [Approve & Start] [Request Changes] [More Details]
```

**Fallback:**

If LSP unavailable for the file type:
```
Use the Explore agent (medium thoroughness) to find:
- All files importing the target module
- Usages of the function/class being modified
- Related test files
```

### 3.4 Implementation Handoff

**Before coding:**
```bash
mcp__plugin_foundry_foundry-mcp__task action="update-status" spec_id={spec-id} task_id={task-id} status="in_progress" note="context"
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

**Context Awareness:** The `context-monitor` hook automatically warns when context exceeds 85%. If you see `[CONTEXT X%]` warnings, follow the recommendation to `/clear` then `/sdd-next` after completing the current task.

**Surface next recommendation:**
```bash
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id}
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

## 6. Verification Task Workflow

**Entry:** Routed here from 3.2.1 when task has `type: "verify"`

### 6.1 Mark Task In Progress

```bash
mcp__plugin_foundry_foundry-mcp__task action="start" spec_id={spec-id} task_id={task-id}
```

### 6.2 Detect Verification Type

Read `verification_type` from task metadata (already available from `task action="prepare"` or `task action="info"`):

```bash
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}
```

### 6.3 Dispatch by Verification Type

| verification_type | Skill Invocation |
|-------------------|------------------|
| `"run-tests"` | `Skill(foundry:run-tests)` |
| `"fidelity"` | `Skill(foundry:sdd-fidelity-review)` |

**For fidelity reviews**, extract `scope` and `target` from task metadata to pass to the skill.

### 6.4 Execute Verification Skill

**MANDATORY:** You MUST invoke the Skill. Do NOT perform manual verification by reading files.

**For fidelity review:**
```bash
Skill(foundry:sdd-fidelity-review) "Review {scope} {target} in spec {spec-id}"
```

Example: `Skill(foundry:sdd-fidelity-review) "Review phase phase-1 in spec my-spec-001"`

**For run-tests:**
```bash
Skill(foundry:run-tests) "Run tests for spec {spec-id}"
```

### 6.5 Present Results

After the skill returns:
1. Display the verdict (pass/fail/partial)
2. List any deviations or failures found
3. Show recommendations from the review

### 6.6 Complete or Remediate

**If verdict = pass:**
```bash
Skill(foundry:sdd-update) "Complete verify task {task-id}. Fidelity review passed with verdict: {verdict}."
```

**If verdict = fail or partial:**
- Do NOT mark the verify task complete
- Present failures to user via `AskUserQuestion`
- Options: "Fix Issues & Re-run", "Create Remediation Tasks", "Override & Complete"
- If user chooses to fix, address deviations then re-run verification skill (6.4)

### 6.7 Return to Main Workflow

After verification task is complete, go to **3.5 Surface Next Recommendation** to continue with the next task.

---

## Quick Reference

### Core Commands

```bash
# Discovery
mcp__plugin_foundry_foundry-mcp__spec action="find" status="active"
mcp__plugin_foundry_foundry-mcp__spec action="list" status="active"

# Task Selection
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id}    # Primary - includes all context
mcp__plugin_foundry_foundry-mcp__task action="next" spec_id={spec-id}       # Simpler alternative - just task ID

# Task Operations
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="pending"
mcp__plugin_foundry_foundry-mcp__task action="list-blocked" spec_id={spec-id}
mcp__plugin_foundry_foundry-mcp__task action="unblock" spec_id={spec-id} task_id={task-id} resolution="reason"
```

### Critical Patterns Summary

| Pattern | Requirement |
|---------|-------------|
| **Spec reading** | Always use MCP tools, NEVER `Read()` or `cat` on JSON |
| **Task completion** | Never mark complete if tests failing/partial/errors |
| **Verification tasks** | Route via 3.2.1 to Section 6; MUST invoke MCP tools, NOT manual file reads |
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
