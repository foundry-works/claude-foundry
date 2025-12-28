# Progress Tracking and Verification

Journaling progress and recording verification results.

## Tracking Progress

### Add Journal Entries

Document decisions, deviations, or important notes:

```bash
# Document a decision
mcp__plugin_foundry_foundry-mcp__journal action="add" spec_id={spec-id} title="Decision Title" content="Explanation of decision and rationale" task_id={task-id} entry_type="decision"

# Document a deviation from the plan
mcp__plugin_foundry_foundry-mcp__journal action="add" spec_id={spec-id} title="Deviation: Changed Approach" content="Created separate service file instead of modifying existing. Improves separation of concerns." task_id={task-id} entry_type="deviation"

# Document task completion (use status_change, NOT completion)
mcp__plugin_foundry_foundry-mcp__journal action="add" spec_id={spec-id} title="Task Completed: Implement Auth" content="Successfully implemented authentication with JWT tokens. All tests passing." task_id={task-id} entry_type="status_change"

# Document a note
mcp__plugin_foundry_foundry-mcp__journal action="add" spec_id={spec-id} title="Implementation Note" content="Using Redis for session storage as discussed." task_id={task-id} entry_type="note"
```

**Entry types:** `decision`, `deviation`, `blocker`, `note`, `status_change`

### Update Phase Estimates

When actual work reveals that phase estimates need adjustment (e.g., a phase is taking longer or shorter than expected), update the phase metadata:

```bash
# Update estimated hours for a phase
mcp__plugin_foundry_foundry-mcp__authoring action="phase-update-metadata" spec_id={spec-id} phase_id="phase-2" estimated_hours=12.5

# Update phase description and purpose
mcp__plugin_foundry_foundry-mcp__authoring action="phase-update-metadata" spec_id={spec-id} phase_id="phase-2" description="Updated scope" purpose="Refined objective"

# Preview changes before applying
mcp__plugin_foundry_foundry-mcp__authoring action="phase-update-metadata" spec_id={spec-id} phase_id="phase-2" estimated_hours=8.0 dry_run=true
```

**Use cases:**
- Task completion reveals more/less work than estimated
- Scope changes during implementation
- Post-phase retrospective adjustments

---

## Adding Verification Results

### Manual Verification Recording

Document verification results:

```bash
# Verification passed
mcp__plugin_foundry_foundry-mcp__verification action="add" spec_id={spec-id} verify_id={verify-id} result="PASSED" command="npm test" output="All tests passed" notes="Optional notes"

# Verification failed
mcp__plugin_foundry_foundry-mcp__verification action="add" spec_id={spec-id} verify_id={verify-id} result="FAILED" command="npm test" output="3 tests failed" issues="List of issues found"

# Partial success
mcp__plugin_foundry_foundry-mcp__verification action="add" spec_id={spec-id} verify_id={verify-id} result="PARTIAL" notes="Most checks passed, minor issues remain"
```

### Automatic Verification Execution

If verification tasks have metadata specifying how to execute them, run automatically:

```bash
# Execute verification based on metadata
mcp__plugin_foundry_foundry-mcp__verification action="execute" spec_id={spec-id} verify_id={verify-id}

# Execute and automatically record result
mcp__plugin_foundry_foundry-mcp__verification action="execute" spec_id={spec-id} verify_id={verify-id} record=true
```

**Requirements:** Verification task must have `skill` or `command` in its metadata.

### Verify on Task Completion

Automatically run verifications when marking a task complete via `task action="complete"`.

The `--verify` flag runs all associated verify tasks. If any fail, the task reverts to `in_progress`.

### Configurable Failure Handling

Verification tasks can specify custom failure behavior via `on_failure` metadata:

```json
{
  "verify-1-1": {
    "metadata": {
      "on_failure": {
        "consult": true,
        "revert_status": "in_progress",
        "max_retries": 2,
        "continue_on_failure": false
      }
    }
  }
}
```

**on_failure fields:**
- `consult` (boolean) - Recommend AI consultation for debugging
- `revert_status` (string) - Status to revert parent task to on failure
- `max_retries` (integer) - Number of automatic retry attempts (0-5)
- `continue_on_failure` (boolean) - Continue with other verifications if this fails
