# Unattended Posture Behavior

Continuous task execution without user prompts, driven by the `unattended` autonomy posture.

## Contents

- [Overview](#overview)
- [Enabling Unattended Posture](#enabling-unattended-posture)
- [Pause Triggers](#pause-triggers)
- [Recovery Patterns](#recovery-patterns)
- [Gate Comparison](#gate-comparison)
- [Best Practices](#best-practices)

## Overview

When `autonomy.posture.profile` is `"unattended"`, the skill executes tasks continuously without requiring user approval between each task. The system automatically:

1. Completes the current task
2. Selects the next recommended task
3. Executes it immediately
4. Repeats until a pause trigger fires

> **Note:** Unattended posture skips *user prompts* between tasks, but still respects all safety limits (context >= 85%, error thresholds, blockers). These limits are mandatory, not optional.

### When to Use

| Scenario | Recommended Posture |
|----------|---------------------|
| Implementing a well-defined spec | Unattended |
| Exploratory work or unclear requirements | Supervised |
| Large batch of similar tasks | Unattended |
| Tasks requiring frequent decisions | Supervised |
| Overnight/unattended execution | Unattended |

### Session Tracking

Unattended posture uses canonical session actions to track state:
- `task action="session" command="start"` — begin session
- `task action="session" command="status"` — check state
- `task action="session" command="pause"` — pause execution
- `task action="session" command="resume"` — resume from pause
- `task action="session" command="end"` — end session

Progress persists across `/clear` boundaries. Session ends when spec completes or user abandons.

> See [session-management.md](./session-management.md) for full session and session-step MCP action details.

## Enabling Unattended Posture

### Via TOML Config

Set in `foundry-mcp.toml`:
```toml
[autonomy_posture]
profile = "unattended"

# Optional session defaults (shown with defaults)
# [autonomy_session_defaults]
# max_tasks_per_session = 100
# max_consecutive_errors = 3
# stop_on_phase_completion = false
# auto_retry_fidelity_gate = false
# max_fidelity_review_cycles_per_phase = 3
```

### How the Skill Reads Posture

At entry, the skill calls:
```bash
mcp__plugin_foundry_foundry-mcp__environment action="get-config" sections='["autonomy", "git"]'
```

The response includes `autonomy.posture.profile`. When set to `"unattended"`, all user gates are skipped.

### Resuming a Paused Session

```bash
foundry-implement
# If session exists and is paused, you'll be prompted:
# "Resume session?" / "Start fresh?"
```

## Pause Triggers

Unattended execution pauses when specific thresholds are reached. Thresholds are sourced from `autonomy.session_defaults`.

### Trigger Table

| Trigger | Threshold | Source | Auto-Resume |
|---------|-----------|--------|-------------|
| Context limit | >= 85% | Context monitor hook | No |
| Error threshold | >= `max_consecutive_errors` | Task completion failures | No |
| Blocked task | Any unresolved blocker | `can_start: false` | No |
| Task limit | `max_tasks_per_session` | Session counter | No |
| Phase boundary | Phase completes | `stop_on_phase_completion` | No |
| User interrupt | Ctrl+C or interrupt | Signal handler | No |
| Spec complete | All tasks done | Progress check | N/A |

### Context Limit (>= 85%)

**Detection:** The `context-monitor` hook reports `[CONTEXT X%]` warnings.

**Behavior:**
1. Continue working on the current task at full quality -- do not rush or cut corners (the remaining ~15% headroom is sufficient)
2. Pause session: `task action="session" command="pause" reason="context_limit"`
3. Output recovery guidance

**Output:**
```
Current task completed normally. Context at 87%. Session paused at task-3-2.
Completed 5 tasks this session.
Run `/clear` then `foundry-implement` to resume.
```

### Error Threshold

**Detection:** Task completion returns error or task marked blocked N+ times in a row (threshold from `max_consecutive_errors`, default 3).

**Behavior:**
1. Pause session: `task action="session" command="pause" reason="error_threshold"`
2. List failed tasks with error summaries
3. Wait for user investigation

**Output:**
```
3 consecutive task failures. Session paused.
Failed tasks:
- task-2-1: ImportError in module X
- task-2-2: Test assertion failed
- task-2-3: Blocked by missing dependency
Fix issues manually, then `foundry-implement` to resume.
```

### Blocked Task

**Detection:** `task action="prepare"` returns `can_start: false`.

**Behavior:**
1. Check if alternative tasks exist
2. If yes, skip to next available task (no pause)
3. If no, pause session: `task action="session" command="pause" reason="blocked_task"`

**Output:**
```
Next task (task-4-1) is blocked by: task-3-2 (incomplete)
No alternative tasks available. Session paused.
Resolve blocker or mark task as skipped, then resume.
```

### Task Limit

**Detection:** Session `completed_tasks` count reaches `max_tasks_per_session` (default 100).

**Purpose:** Periodic checkpoint for user review.

**Output:**
```
Completed 100 tasks this session. Checkpoint pause.
Progress: 100/250 tasks (40%)
Review changes, then `foundry-implement` to continue.
```

### Phase Boundary

**Detection:** A phase completes and `stop_on_phase_completion` is `true`.

**Behavior:**
1. Pause session: `task action="session" command="pause" reason="phase_complete"`
2. Summarize completed phase
3. User reviews before starting next phase

### User Interrupt

**Detection:** Ctrl+C or SIGINT signal.

**Behavior:**
1. Complete current atomic operation if safe
2. Pause session: `task action="session" command="pause" reason="user_requested"`
3. Save state for resume

## Loop Signals

Session-step responses include a `data.loop_signal` field that guides autonomous continuation decisions.

### Signal Values

| Signal | Meaning | Auto-Continue? |
|--------|---------|----------------|
| `phase_complete` | Current phase finished, more phases remain | Yes |
| `spec_complete` | All phases/tasks done | No — report completion |
| `paused_needs_attention` | Requires user intervention | No — pause session |
| `failed` | Step failed | No — increment error count |
| `blocked_runtime` | Runtime blocker encountered | No — pause session |

### Auto-Continue Rule

**CRITICAL:** Auto-continue should ONLY happen when `loop_signal` is `phase_complete`. All other signals require either pausing or ending the session.

```
session-step response received:
    │
    ├─ loop_signal == "phase_complete" → Auto-continue to next phase
    ├─ loop_signal == "spec_complete" → End session, report completion
    ├─ loop_signal == "paused_needs_attention" → Pause session
    ├─ loop_signal == "failed" → Increment errors, check threshold
    └─ loop_signal == "blocked_runtime" → Pause session
```

### Recommended Actions

Session-step responses also include a `data.recommended_actions` array suggesting next steps:

```json
{
  "data": {
    "loop_signal": "phase_complete",
    "recommended_actions": [
      "start_next_phase",
      "run_verification"
    ]
  }
}
```

Use `recommended_actions` as guidance for what to do next, but always respect the `loop_signal` for continuation decisions.

## Session Events

Inspect the journal-backed event feed for debugging or audit:

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-events" \
  spec_id={spec-id} \
  session_id={session-id}
```

Supports pagination via `cursor` and `limit` parameters. Returns chronological events including task starts, completions, pauses, and errors.

## Recovery Patterns

### After /clear

Session state persists in MCP storage. Recovery flow:

```
User runs: foundry-implement
    │
    ├─ Check session status
    │
    ├─ Session exists and paused?
    │       │
    │       ├─ Yes → "Resume session? (paused at task-X)"
    │       │           ├─ Resume → Continue from current_task
    │       │           └─ Fresh → End session, start new
    │       │
    │       └─ No → Start new session
    │
    └─ Begin unattended execution
```

### After Context Limit

1. Run `/clear` to reset context
2. Run `foundry-implement`
3. Accept "Resume session" prompt
4. Execution continues from paused task

### After Error Threshold

1. Review error output from pause message
2. Fix underlying issues (imports, tests, dependencies)
3. Run `foundry-implement`
4. Accept "Resume session" prompt
5. Error counter resets on successful task completion

### After Blocked Task

**Option A: Resolve the blocker**
1. Complete the blocking task manually
2. Run `foundry-implement` to resume

**Option B: Skip the blocked task**
1. Mark task as blocked/skipped via MCP
2. Run `foundry-implement` to continue with next available

### Abandoning a Session

To start completely fresh:
```bash
foundry-implement
# When prompted "Resume session?", select "Start fresh"
```

Or explicitly end:
```bash
mcp__plugin_foundry_foundry-mcp__task action="session" spec_id={spec-id} command="end"
```

## Gate Comparison

How decision points differ between postures:

| Gate | Supervised Posture | Unattended Posture |
|------|-------------------|-------------------|
| **Task selection** | AskUserQuestion with options | Auto-select recommended task |
| **Plan approval** | AskUserQuestion required | Skip (use recommended plan) |
| **Implementation start** | Explicit "Approve & Start" | Auto-start after prepare |
| **Task completion** | Mark complete, surface next | Mark complete, auto-continue |
| **Continue prompt** | "Continue to next task?" | Skip (auto-continue) |
| **Blocked task** | Present alternatives | Auto-skip or pause |
| **Context warning** | Suggest /clear | Auto-pause session |
| **Spec complete** | Report completion | Report completion (same) |

### Gates That Always Apply

Even in unattended posture, these gates require interaction:

| Gate | Reason |
|------|--------|
| Pause recovery | User must confirm resume vs fresh start |
| Error investigation | User must fix issues before continuing |
| Spec activation | Initial spec selection is always interactive |

### Gates That Are Skipped (Unattended)

| Gate | Unattended Behavior |
|------|---------------------|
| Task selection | Use `task action="prepare"` recommendation |
| Plan details | Accept generated plan |
| Continue prompt | Proceed immediately |

## Auto-Commit Behavior

After marking a task complete, check the response for `suggest_commit`:

### When `suggest_commit: true` and `commit_scope: "task"`

The git cadence is set to "task" - commit after each task:

1. Stage all changes: `git add -A`
2. Commit with message from `commit_message_hint` (e.g., "task: Implement user validation")
3. Do NOT prompt - this is silent auto-commit per user config
4. Continue to next task

### When `suggest_commit: true` and `commit_scope: "phase"`

The git cadence is set to "phase" and a phase just completed:

1. Stage all changes: `git add -A`
2. Commit with message from `commit_message_hint` (e.g., "phase: Core API Implementation")
3. Do NOT prompt - this is silent auto-commit per user config
4. Continue to next task

### When `suggest_commit: false`

No commit action needed. The git cadence is either "manual" or no phase boundary was crossed. Continue to next task.

### Example Task Completion Response

```json
{
  "status": "success",
  "data": {
    "task_id": "task-1-2",
    "completed_at": "2025-01-03T10:00:00Z",
    "progress": {"completed_tasks": 5, "total_tasks": 12, "percentage": 42},
    "suggest_commit": true,
    "commit_scope": "task",
    "commit_message_hint": "task: Add input validation"
  }
}
```

## Auto-Push Behavior

After a successful commit, check the git config for `auto_push`:

### When `auto_push: true`

Automatically push to remote after commit:

1. **Detect upstream:** `git rev-parse --abbrev-ref --symbolic-full-name @{upstream}`
2. **If upstream exists:** `git push`
3. **If no upstream (first push):** `git push -u origin <branch>`
4. Continue to next task

### When `auto_push: false` (Default)

No push action. User handles pushing manually. Continue to next task.

### Push Failure Handling

Push failures are **non-blocking**:

| Failure Type | Behavior |
|--------------|----------|
| Network error | Log warning, continue workflow |
| Auth failure | Log warning, continue workflow |
| Remote rejected | Log warning, continue workflow |
| Detached HEAD | Skip push with warning, continue |
| No remote origin | Skip push with warning, continue |

**Rationale:** Push failures don't affect local work. The user can resolve push issues independently without blocking task execution.

### Example Push Flow

```
After commit succeeds:
    │
    ├─ Check git config: auto_push?
    │       │
    │       ├─ false → Skip push, continue to next task
    │       │
    │       └─ true → Attempt push
    │                   │
    │                   ├─ Check upstream: git rev-parse ... @{upstream}
    │                   │       │
    │                   │       ├─ exists → git push
    │                   │       │
    │                   │       └─ missing → git push -u origin <branch>
    │                   │
    │                   ├─ Success → Continue silently
    │                   │
    │                   └─ Failure → Log warning, continue
```

## Best Practices

### Before Starting Unattended Execution

1. **Review the spec** - Ensure tasks are well-defined
2. **Check dependencies** - Verify external deps are available
3. **Commit current work** - Clean git state recommended
4. **Review session defaults** - Ensure task/error limits are appropriate

### During Unattended Execution

1. **Monitor output** - Watch for warnings
2. **Don't interrupt mid-task** - Wait for task boundaries
3. **Check git status** - Periodically verify changes

### After Pause

1. **Review completed work** - Check git diff
2. **Address pause reason** - Don't just resume blindly
3. **Run tests** - Verify accumulated changes work together

### Anti-patterns

| Anti-pattern | Problem | Solution |
|--------------|---------|----------|
| Ignoring context notifications | OOM or truncation | After finishing current task, pause and /clear |
| Resuming without fixing errors | Immediate re-pause | Investigate root cause first |
| Running on unclear specs | Poor quality output | Use supervised posture for exploration |
| Never reviewing progress | Drift from intent | Use task limits for checkpoints |
