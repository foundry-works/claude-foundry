# Parallel Behavior (`--delegate --parallel`)

Execute multiple independent tasks concurrently via subagents. Enabled with `--parallel` flag (implies `--delegate`).

## Contents

- [Overview](#overview)
- [Flag Combinations](#flag-combinations)
- [Enabling Parallel Mode](#enabling-parallel-mode)
- [Batch Operations API](#batch-operations-api)
- [Subagent Delegation](#subagent-delegation)
- [Conflict Detection](#conflict-detection)
- [Partial Failure Handling](#partial-failure-handling)
- [Best Practices](#best-practices)

## Overview

Parallel mode spawns multiple subagents to work on independent tasks simultaneously. Maximizes throughput for non-conflicting work.

### When to Use

| Scenario | Recommended Mode |
|----------|------------------|
| Independent tasks in same phase | Parallel |
| Tasks modifying same files | Interactive or Autonomous |
| Large phase with many isolated changes | Parallel |
| Tasks with dependencies between them | Autonomous |
| Test/verification tasks | Parallel (if independent) |

### Key Differences: Inline vs Delegated vs Parallel

| Aspect | Inline | Delegated (`--delegate`) | Parallel (`--parallel`) |
|--------|--------|--------------------------|-------------------------|
| Execution | Same context | Subagent per task | Multiple subagents |
| Concurrency | Sequential | Sequential | Concurrent |
| Context usage | Shared | Fresh per task | Distributed |
| File conflicts | N/A | N/A | Detected and prevented |
| Failure impact | Stops on error | Isolated | Isolated per task |

## Flag Combinations

`--parallel` implies `--delegate` and is combinable with `--auto`:

| Flags | Behavior |
|-------|----------|
| `--delegate --parallel` | Interactive, concurrent subagents |
| `--auto --delegate --parallel` | Autonomous, concurrent subagents |

**Defaults:** Loaded from `[implement]` section in `foundry-mcp.toml`. CLI flags override.

## Enabling Parallel Mode

### Via Command Flag

```bash
/implement --delegate --parallel                # Interactive parallel
/implement --auto --delegate --parallel         # Autonomous parallel
/implement --parallel                           # Implies --delegate
```

### Via TOML Defaults

Set in `foundry-mcp.toml`:
```toml
[implement]
auto = true       # Skip prompts
delegate = true   # Use subagents
parallel = true   # Run concurrently
```

### Via Interactive Selection

When running `/implement` without flags and no TOML defaults:
```
"Select execution mode:"
- "Interactive, inline (default)"
- "Autonomous, inline (--auto)"
- "Interactive, delegated (--delegate)"
- "Autonomous, delegated (--auto --delegate)"
- "Autonomous, parallel (--auto --delegate --parallel)"
```

## Batch Operations API

Parallel mode uses batch-specific MCP actions on the task router.

### prepare-batch

Identify tasks eligible for parallel execution.

```bash
mcp__plugin_foundry_foundry-mcp__task action="prepare-batch" \
  spec_id={spec-id} \
  max_tasks=5
```

**Returns:**
```json
{
  "batch_id": "batch-abc123",
  "tasks": [
    {"task_id": "task-3-1", "file_path": "src/auth.py", "conflicts_with": []},
    {"task_id": "task-3-2", "file_path": "src/utils.py", "conflicts_with": []},
    {"task_id": "task-3-3", "file_path": "src/auth.py", "conflicts_with": ["task-3-1"]}
  ],
  "eligible_tasks": ["task-3-1", "task-3-2"],
  "excluded_tasks": [
    {"task_id": "task-3-3", "reason": "file_conflict", "conflicts_with": ["task-3-1"]}
  ]
}
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `spec_id` | string | Target specification |
| `max_tasks` | int | Maximum tasks to include (default: 5) |
| `phase_id` | string | Limit to specific phase (optional) |

### start-batch

Begin parallel execution of a prepared batch.

```bash
mcp__plugin_foundry_foundry-mcp__task action="start-batch" \
  spec_id={spec-id} \
  batch_id={batch-id}
```

**Returns:**
```json
{
  "batch_id": "batch-abc123",
  "started_at": "2026-01-03T10:00:00Z",
  "tasks_started": 2,
  "subagent_ids": ["agent-001", "agent-002"]
}
```

### complete-batch

Report completion and collect results from all subagents.

```bash
mcp__plugin_foundry_foundry-mcp__task action="complete-batch" \
  spec_id={spec-id} \
  batch_id={batch-id} \
  results='[{"task_id": "task-3-1", "status": "completed", "note": "..."}, ...]'
```

**Returns:**
```json
{
  "batch_id": "batch-abc123",
  "completed_at": "2026-01-03T10:15:00Z",
  "summary": {
    "total": 2,
    "completed": 2,
    "failed": 0,
    "blocked": 0
  }
}
```

### reset-batch

Cancel or reset an in-progress batch (for error recovery).

```bash
mcp__plugin_foundry_foundry-mcp__task action="reset-batch" \
  spec_id={spec-id} \
  batch_id={batch-id} \
  reason="User requested cancellation"
```

**Returns:**
```json
{
  "batch_id": "batch-abc123",
  "reset_at": "2026-01-03T10:05:00Z",
  "tasks_reset": ["task-3-1", "task-3-2"],
  "previous_status": "in_progress"
}
```

## Subagent Delegation

Parallel mode uses Claude Code's Task tool to spawn subagents for each task.

### Delegation Pattern

```
Main Agent (orchestrator)
    |
    +-- prepare-batch -> identify eligible tasks
    |
    +-- For each eligible task:
    |       |
    |       +-- Task(subagent_type="general-purpose")
    |           - Receives: task context, file path, acceptance criteria
    |           - Returns: completion status, changes made, issues found
    |
    +-- complete-batch -> aggregate results
```

### Subagent Invocation

```python
# For each task in batch
Task(
    subagent_type="general-purpose",
    prompt=f"""
    Implement task {task_id} from spec {spec_id}.

    Title: {task_title}
    File: {file_path}

    Instructions:
    {task_description}

    Acceptance Criteria:
    {acceptance_criteria}

    Complete the task and report:
    1. What was implemented
    2. Files modified
    3. Any issues encountered
    """,
    run_in_background=True
)
```

### Collecting Results

```python
# Wait for all subagents to complete
for agent_id in subagent_ids:
    result = TaskOutput(task_id=agent_id, block=True, timeout=300000)
    # Aggregate result into batch completion
```

### Subagent Constraints

| Constraint | Reason |
|------------|--------|
| No spec writes | Only main agent updates spec state |
| No task status changes | Main agent manages lifecycle |
| Read-only spec access | Prevents race conditions |
| Limited MCP actions | Only file operations, not task router |

## Conflict Detection

Parallel mode prevents race conditions by detecting file-path conflicts before execution.

### How Conflicts Are Detected

1. **prepare-batch** extracts `file_path` from each task's metadata
2. Tasks sharing the same `file_path` are marked as conflicting
3. Only one task per unique `file_path` is included in a batch
4. Conflicting tasks are deferred to subsequent batches

### Conflict Types

| Type | Description | Resolution |
|------|-------------|------------|
| **Direct** | Same `file_path` | Run sequentially in separate batches |
| **Directory** | Overlapping directories | Usually allowed (different files) |
| **Import chain** | File A imports B, both modified | Detected via dependency metadata |

### Example: Conflict Resolution

```
Tasks:
- task-3-1: modifies src/auth.py
- task-3-2: modifies src/utils.py
- task-3-3: modifies src/auth.py (conflicts with task-3-1)
- task-3-4: modifies tests/test_auth.py

Batch 1: [task-3-1, task-3-2, task-3-4]  <- 3 parallel
Batch 2: [task-3-3]                       <- after task-3-1 completes
```

### Manual Conflict Override

If you know tasks don't actually conflict (e.g., different functions in same file):

```bash
mcp__plugin_foundry_foundry-mcp__task action="prepare-batch" \
  spec_id={spec-id} \
  ignore_conflicts=true  # Use with caution
```

## Partial Failure Handling

When some tasks in a batch fail, the system handles them gracefully.

### Failure Isolation

Each subagent operates independently. A failure in one does not affect others:

```
Batch with 3 tasks:
- task-3-1: completed successfully
- task-3-2: FAILED (import error)
- task-3-3: completed successfully

Result: 2 completed, 1 failed
```

### Failure Response Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| **continue** | Complete successful tasks, report failures | Default |
| **abort** | Cancel remaining tasks on first failure | Critical dependencies |
| **retry** | Retry failed tasks once | Transient errors |

### Handling Failed Tasks

After `complete-batch`, failed tasks remain in their current state:

```bash
# Check failed tasks
mcp__plugin_foundry_foundry-mcp__task action="query" \
  spec_id={spec-id} \
  status_filter="in_progress"  # Failed tasks stay in_progress

# Options:
# 1. Fix and retry in next batch
# 2. Block the task with reason
# 3. Switch to interactive mode for debugging
```

### Error Aggregation

The `complete-batch` response includes error details:

```json
{
  "summary": {
    "total": 3,
    "completed": 2,
    "failed": 1
  },
  "failures": [
    {
      "task_id": "task-3-2",
      "error_type": "implementation_error",
      "message": "ImportError: module 'xyz' not found",
      "subagent_id": "agent-002"
    }
  ]
}
```

## Auto-Commit in Parallel Mode

In parallel mode, commits happen at batch boundaries rather than per-task.

### Commit Timing

After `complete-batch` returns, check if any task in the batch returned `suggest_commit: true`:

| Git Cadence | Commit Behavior |
|-------------|-----------------|
| `manual` | No commits (user handles) |
| `task` | Single commit after batch with all completed task titles |
| `phase` | Commit only if a phase completed during the batch |

### Commit Message Format

**For task cadence (multiple tasks in batch):**
```
tasks: Complete batch of N tasks

- task: Add input validation
- task: Implement error handling
- task: Update API responses
```

**For phase cadence:**
```
phase: Core API Implementation
```

### Implementation

```python
# After complete-batch returns
results = complete_batch_response["results"]
suggest_commits = [r for r in results if r.get("suggest_commit")]

if suggest_commits:
    # Stage all changes from batch
    git add -A

    if any(r.get("commit_scope") == "phase" for r in suggest_commits):
        # Phase completion takes precedence
        phase_hint = next(r["commit_message_hint"] for r in suggest_commits
                         if r.get("commit_scope") == "phase")
        git commit -m "{phase_hint}"
    else:
        # Aggregate task messages
        task_lines = [r["commit_message_hint"] for r in suggest_commits]
        git commit -m "tasks: Complete batch of {len(task_lines)} tasks\n\n" +
                     "\n".join(f"- {line}" for line in task_lines)
```

### Notes

- Commits are silent (no prompting) per user's cadence configuration
- If batch has mixed success/failure, only completed tasks appear in commit message
- Failed tasks remain uncommitted for retry in next batch

## Best Practices

### Before Starting Parallel Mode

1. **Review task file paths** - Ensure metadata includes `file_path`
2. **Check for hidden dependencies** - Tasks may conflict via imports
3. **Commit current work** - Clean git state for easy rollback
4. **Set reasonable batch size** - 3-5 tasks typical, more for simple changes

### During Parallel Execution

1. **Monitor subagent output** - Use `TaskOutput` with `block=false` for progress
2. **Watch for early failures** - May indicate systemic issues
3. **Don't modify files manually** - Let subagents complete

### After Batch Completion

1. **Review git diff** - Verify changes make sense together
2. **Run tests** - Parallel changes may have subtle interactions
3. **Check for merge conflicts** - Multiple files touching shared code
4. **Address failures** - Before starting next batch

### Anti-patterns

| Anti-pattern | Problem | Solution |
|--------------|---------|----------|
| Ignoring conflicts | Race conditions, data loss | Respect prepare-batch exclusions |
| Too large batches | Context exhaustion, failures | Limit to 5-7 tasks |
| Parallel on dependencies | Incorrect ordering | Use autonomous mode instead |
| No progress monitoring | Silent failures | Check TaskOutput periodically |
| Skipping tests | Hidden integration bugs | Always test after batch |
