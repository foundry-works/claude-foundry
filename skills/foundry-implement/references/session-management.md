# Session Management

Autonomous execution session tracking for foundry-implement. Manages state persistence across `/clear` boundaries and pause/resume workflows.

## Contents

- [Session State Model](#session-state-model)
- [Session MCP Actions](#session-mcp-actions)
- [Session-Step MCP Actions](#session-step-mcp-actions)
- [Session Events](#session-events)
- [Pause Reasons](#pause-reasons)
- [Recovery Patterns](#recovery-patterns)
- [State Persistence](#state-persistence)
- [Anti-patterns](#anti-patterns)

## Session State Model

Sessions track autonomous execution progress when using `--auto` mode.

### States

| State | Description | Transitions To |
|-------|-------------|----------------|
| `idle` | No active autonomous session | `active` |
| `active` | Executing tasks autonomously | `paused`, `idle` |
| `paused` | Execution halted, awaiting recovery | `active`, `idle` |

### State Diagram

```
idle ──[start]──> active ──[complete]──> idle
                    │
                    ├──[pause trigger]──> paused ──[resume]──> active
                    │                        │
                    │                        └──[abandon]──> idle
                    │
                    └──[error/abort]──> idle
```

### Session Data

| Field | Type | Description |
|-------|------|-------------|
| `session_id` | string | Unique identifier (uuid) |
| `spec_id` | string | Associated specification |
| `state` | enum | `idle`, `active`, `paused` |
| `started_at` | datetime | Session start timestamp |
| `paused_at` | datetime | When pause triggered (if paused) |
| `pause_reason` | enum | Reason for pause (if paused) |
| `completed_tasks` | list | Task IDs completed this session |
| `current_task` | string | Task ID in progress (if any) |
| `error_count` | int | Consecutive errors encountered |
| `context_percentage` | int | Last known context usage |

## Session MCP Actions

Manage autonomous execution sessions via the task router using the canonical `session` action.

> **Legacy aliases:** The old `session-config` action name still works but responses include deprecation metadata. Use canonical forms for new code.

### Start Session

```bash
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="start"
```

**Returns:** `{ session_id, state: "active", started_at }`

### Check Status

```bash
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="status"
```

**Returns:** Full session data including state, progress, pause reason if applicable.

### Pause Session

```bash
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="pause" \
  reason={pause-reason}
```

### Resume Session

```bash
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="resume"
```

**Precondition:** Session must be in `paused` state.

### End Session

```bash
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="end"
```

Clears session state, returns summary of completed tasks.

## Session-Step MCP Actions

Orchestrate individual step execution within an active session using the canonical `session-step` action.

> **Legacy aliases:** The old hyphenated forms (`session-step-next`, etc.) still work but responses include deprecation metadata. Use canonical forms for new code.

### Get Next Step

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-step" \
  spec_id={spec-id} \
  command="next"
```

**Returns:** Next step to execute with task context. Response includes `loop_signal` and `recommended_actions` (see below).

### Report Step Result

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-step" \
  spec_id={spec-id} \
  command="report" \
  step_id={step-id} \
  outcome={outcome} \
  completion_note="Summary of what was done"
```

**Returns:** Updated session state with `loop_signal` and `recommended_actions`.

### Replay Step

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-step" \
  spec_id={spec-id} \
  command="replay" \
  step_id={step-id}
```

Re-execute a previously completed or failed step.

### Heartbeat

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-step" \
  spec_id={spec-id} \
  command="heartbeat"
```

Signal that the agent is still active. Prevents stale session detection.

### Response Fields: `loop_signal` and `recommended_actions`

Session-step responses include two fields that guide autonomous continuation:

**`data.loop_signal`** — Determines whether to auto-continue:

| Signal | Meaning | Action |
|--------|---------|--------|
| `phase_complete` | Current phase finished, more phases remain | Auto-continue |
| `spec_complete` | All phases/tasks done | End session |
| `paused_needs_attention` | Requires user intervention | Pause session |
| `failed` | Step failed | Increment errors, check threshold |
| `blocked_runtime` | Runtime blocker encountered | Pause session |

**`data.recommended_actions`** — Array of suggested next steps:

```json
{
  "data": {
    "loop_signal": "phase_complete",
    "recommended_actions": ["start_next_phase", "run_verification"]
  }
}
```

Use `recommended_actions` as guidance, but always respect `loop_signal` for continuation decisions.

## Session Events

Inspect the journal-backed event feed for debugging, audit, or progress review.

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-events" \
  spec_id={spec-id} \
  session_id={session-id}
```

**Pagination:** Use `cursor` and `limit` parameters for large event histories:

```bash
mcp__plugin_foundry_foundry-mcp__task action="session-events" \
  spec_id={spec-id} \
  session_id={session-id} \
  limit=20 \
  cursor={last-cursor}
```

**Returns:** Chronological events including task starts, completions, pauses, errors, and phase transitions.

## Pause Reasons

Autonomous execution pauses when specific thresholds are reached.

### Pause Triggers

| Reason | Trigger | Threshold | Auto-Resume |
|--------|---------|-----------|-------------|
| `context_limit` | Context exceeds threshold | >= 85% | No |
| `error_threshold` | Consecutive task failures | >= 3 | No |
| `blocked_task` | Current task blocked | N/A | No |
| `task_limit` | Session task count reached | Configurable | No |
| `user_requested` | User manually paused | N/A | No |

### context_limit

**Trigger:** Context monitor reports >= 85% usage.

**Resolution:**
1. Finish current task at full quality -- the remaining headroom is sufficient, do not rush
2. Pause: `task action="session" command="pause" reason="context_limit"`
3. Instruct user to run `/clear` then `foundry-implement --auto` to resume

**Example recovery message:**
```
Context at 87%. Session paused.
Run `/clear` then `foundry-implement --auto` to resume from task-3-2.
```

### error_threshold

**Trigger:** 3+ consecutive tasks fail (blocked, errors, cannot complete).

**Resolution:**
1. Pause: `task action="session" command="pause" reason="error_threshold"`
2. Surface failed tasks and error summaries
3. User investigates and fixes issues manually
4. Resume with `foundry-implement --auto`

**Example recovery message:**
```
3 consecutive tasks failed. Session paused.
Review: task-2-1 (import error), task-2-2 (test failure), task-2-3 (blocked)
Fix issues, then run `foundry-implement --auto` to resume.
```

### blocked_task

**Trigger:** Next recommended task has unresolved blockers.

**Resolution:**
1. Pause: `task action="session" command="pause" reason="blocked_task"`
2. Surface blocker details
3. User resolves blocker or skips task
4. Resume with `foundry-implement --auto`

### task_limit

**Trigger:** Completed N tasks in current session (configurable, default: 10).

**Purpose:** Checkpoint for user review of progress.

**Resolution:**
1. Pause: `task action="session" command="pause" reason="task_limit"`
2. Summarize completed tasks
3. User reviews and continues with `foundry-implement --auto`

### user_requested

**Trigger:** User explicitly requests pause (via Ctrl+C or interrupt).

**Resolution:**
1. Complete current atomic operation if safe
2. Record session state
3. User resumes when ready with `foundry-implement --auto`

## Recovery Patterns

### After /clear

Session state persists across `/clear` boundaries via MCP storage.

**Recovery flow:**
1. User runs `foundry-implement --auto`
2. Check status: `task action="session" command="status"`
3. If session exists and is `paused`:
   - Surface pause reason and last task
   - Offer: "Resume session?" / "Start fresh?"
4. If resumed, continue from `current_task`

### After Errors

```bash
# Check session status
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="status"

# If paused due to errors, user fixes issues, then:
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="resume"
```

### Abandoning a Session

If user wants to start fresh:

```bash
mcp__plugin_foundry_foundry-mcp__task action="session" \
  spec_id={spec-id} \
  command="end"
```

This clears all session state and returns to `idle`.

## State Persistence

### Storage Model

Session state stored in MCP server's workspace data:
- Path: `~/.foundry-mcp/sessions/{spec_id}.json`
- Survives `/clear` and conversation restarts
- Cleaned up when session ends or spec completes

### What Persists

| Data | Persists Across /clear | Notes |
|------|------------------------|-------|
| Session state | Yes | Core state machine |
| Completed tasks | Yes | For progress tracking |
| Current task | Yes | For resume point |
| Error count | Yes | Resets on successful task |
| Context % | No | Re-evaluated on resume |

### What Doesn't Persist

- In-progress file edits (complete task first)
- Conversation context (use `/clear` guidance)
- Subagent state (re-launch on resume)

## Anti-patterns

### Starting Session Without Checking Status

**Wrong:**
```bash
# Starting fresh without checking for existing session
mcp__plugin_foundry_foundry-mcp__task action="session" command="start" ...
```

**Right:**
```bash
# Always check status first
mcp__plugin_foundry_foundry-mcp__task action="session" command="status" ...
# Then decide: resume existing or start new
```

### Ignoring Pause Reasons

**Wrong:**
```
Session paused (context_limit). Continuing anyway...
[Proceeds to execute more tasks]
```

**Right:**
```
Session paused (context_limit).
Run `/clear` then `foundry-implement --auto` to resume safely.
```

### Manual State Manipulation

**Wrong:**
- Directly editing session JSON files
- Using `task action="update-status"` to bypass session tracking

**Right:**
- Always use canonical `session` and `session-step` actions for session operations
- Let the system manage state transitions

### Resuming Without Addressing Blockers

**Wrong:**
```bash
# Session paused due to blocked_task
# User immediately resumes without fixing blocker
mcp__plugin_foundry_foundry-mcp__task action="session" command="resume" ...
# Will immediately pause again
```

**Right:**
```bash
# First resolve the blocker
mcp__plugin_foundry_foundry-mcp__task action="unblock" ...
# Or skip the blocked task
mcp__plugin_foundry_foundry-mcp__task action="update-status" status="blocked" ...
# Then resume
mcp__plugin_foundry_foundry-mcp__task action="session" command="resume" ...
```

### Not Completing Tasks Atomically

**Wrong:**
```
# Context at 84%, near limit
# Starting a large multi-file task
```

**Right:**
```
# Context at 84%, near limit
# Check if next task is completable within remaining context
# If not, pause proactively before starting
```
