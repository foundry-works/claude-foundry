# Workflow Details

Step-by-step workflows for applying spec modifications.

## Table of Contents

- [Parse Review Feedback](#workflow-1-parse-review-feedback)
- [Preview Modifications](#workflow-2-preview-modifications)
- [Apply Modifications](#workflow-3-apply-modifications)
- [Direct Modifications](#workflow-4-direct-modifications-without-review-parsing)
- [Task Repositioning](#workflow-5-task-repositioning)
- [Phase Reordering](#workflow-6-phase-reordering)
- [Bulk Find-Replace](#workflow-7-bulk-find-replace)
- [Rollback Recovery](#workflow-8-rollback-recovery)

---

## Workflow 1: Parse Review Feedback

When starting from a review report (markdown output from sdd-plan-review or sdd-fidelity-review), convert it to structured modifications:

```bash
mcp__plugin_foundry_foundry-mcp__review action="parse-feedback" spec_id="my-spec-001" review_path="reports/plan-review.md"
```

**Output:** Creates a modifications JSON file (typically `{review-path}.suggestions.json`)

**Example output structure:**
```json
{
  "spec_id": "my-spec-001",
  "source": "reports/plan-review.md",
  "generated": "2025-10-18T14:30:00Z",
  "modifications": [
    {
      "operation": "update_task",
      "task_id": "task-2-1",
      "field": "description",
      "value": "Implement OAuth 2.0 with PKCE flow",
      "reason": "Plan review suggested more specific description"
    }
  ]
}
```

**When to use:**
- After running `sdd-plan-review` or `sdd-fidelity-review`
- When you have markdown review output that needs conversion
- When review findings need to be applied systematically

---

## Workflow 2: Preview Modifications

**Always preview before applying.** Use dry-run mode to see what will change:

```bash
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="my-spec-001" modifications_file="suggestions.json" dry_run=true
```

**Example preview output:**
```
Dry Run - Modifications Preview
================================

Spec: my-spec-001
Source: suggestions.json

Changes that would be applied:
1. UPDATE task-2-1.description
   Old: "Implement authentication"
   New: "Implement OAuth 2.0 with PKCE flow"

2. ADD verification to task-1-3
   Type: manual
   Description: "Verify token refresh works correctly"

3. UPDATE task-3-2.metadata.estimated_hours
   Old: 4
   New: 6

Summary: 2 task updates, 1 verification addition
```

**Review checklist:**
- [ ] All intended changes are listed
- [ ] No unexpected changes
- [ ] Task IDs are correct
- [ ] Values look correct

---

## Workflow 3: Apply Modifications

After confirming the preview, apply changes:

```bash
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="my-spec-001" modifications_file="suggestions.json"
```

**What happens automatically:**
1. **Backup created** at `specs/.backups/my-spec-001-{timestamp}.json`
2. **Modifications applied** in order
3. **Validation runs** on the modified spec
4. **Rollback occurs** if validation fails (original restored from backup)

**Success output:**
```
Modifications Applied Successfully
==================================

Spec: my-spec-001
Backup: specs/.backups/my-spec-001-20251018-143000.json

Applied:
- Updated 2 task descriptions
- Added 1 verification step
- Modified 1 metadata field

Validation: PASSED
```

---

## Workflow 4: Direct Modifications (without review parsing)

When you have pre-prepared modifications or want to make manual bulk changes:

**Step 1:** Create modifications JSON file directly:

```json
{
  "spec_id": "my-spec-001",
  "modifications": [
    {
      "operation": "update_task",
      "task_id": "task-1-2",
      "field": "details",
      "value": "Updated task description text"
    },
    {
      "operation": "add_verification",
      "task_id": "task-2-1",
      "verification": {
        "verification_type": "run-tests",
        "mcp_tool": "mcp__plugin_foundry_foundry-mcp__test action=\"run\"",
        "expected": "All tests pass"
      }
    }
  ]
}
```

**Step 2:** Preview and apply:

```bash
# Preview
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="my-spec-001" modifications_file="modifications.json" dry_run=true

# Apply
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="my-spec-001" modifications_file="modifications.json"
```

**When to use:**
- Bulk updates to multiple tasks
- Programmatic spec modifications
- Migrating specs with automated transformations
- When you know exactly what changes to make

---

## Workflow 5: Task Repositioning

Move tasks within a phase or between phases to reorganize work order.

**Step 1:** Identify the task and target location:

```bash
# View current phase structure
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id="my-spec-001" parent_filter="phase-2"
```

**Step 2:** Preview the move with dry-run:

```bash
# Move task-2-3 to position 1 within same phase
mcp__plugin_foundry_foundry-mcp__task action="move" spec_id="my-spec-001" task_id="task-2-3" position=1 dry_run=true

# Move task to different phase
mcp__plugin_foundry_foundry-mcp__task action="move" spec_id="my-spec-001" task_id="task-2-3" parent="phase-3" position=1 dry_run=true
```

**Step 3:** Apply the move:

```bash
mcp__plugin_foundry_foundry-mcp__task action="move" spec_id="my-spec-001" task_id="task-2-3" parent="phase-3" position=1
```

**Example output:**
```json
{
  "task_id": "task-2-3",
  "old_parent": "phase-2",
  "new_parent": "phase-3",
  "old_position": 3,
  "new_position": 1,
  "is_reparenting": true,
  "tasks_in_subtree": 1
}
```

**When to use:**
- Reordering tasks based on changing priorities
- Moving related tasks to group them together
- Restructuring phases after scope changes
- Promoting subtasks to top-level tasks

---

## Workflow 6: Phase Reordering

Change the execution order of phases within a specification.

**Step 1:** View current phase order:

```bash
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id="my-spec-001" node_type="phase"
```

**Step 2:** Preview the reorder with dry-run:

```bash
# Move phase-3 to first position
mcp__plugin_foundry_foundry-mcp__authoring action="phase-move" spec_id="my-spec-001" phase_id="phase-3" position=1 dry_run=true
```

**Step 3:** Apply the reorder:

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="phase-move" spec_id="my-spec-001" phase_id="phase-3" position=1
```

**Example output:**
```json
{
  "phase_id": "phase-3",
  "phase_title": "Infrastructure Setup",
  "old_position": 3,
  "new_position": 1,
  "moved": true
}
```

**When to use:**
- Adjusting phase order based on dependency changes
- Moving foundational work earlier in the spec
- Deferring phases that become lower priority
- Reorganizing after review feedback

---

## Workflow 7: Bulk Find-Replace

Replace text across multiple nodes in a specification.

**Step 1:** Preview changes with dry-run:

```bash
# Replace across all text fields
mcp__plugin_foundry_foundry-mcp__authoring action="spec-find-replace" spec_id="my-spec-001" find="old-api" replace="new-api" scope="all" dry_run=true

# Replace only in titles
mcp__plugin_foundry_foundry-mcp__authoring action="spec-find-replace" spec_id="my-spec-001" find="v1" replace="v2" scope="titles" dry_run=true

# Case-insensitive regex replacement
mcp__plugin_foundry_foundry-mcp__authoring action="spec-find-replace" spec_id="my-spec-001" find="API" replace="REST API" scope="descriptions" case_sensitive=false dry_run=true
```

**Step 2:** Review the preview output:

```json
{
  "total_replacements": 6,
  "nodes_affected": 4,
  "changes": [
    {
      "node_id": "task-1-2",
      "field": "title",
      "old": "Implement old-api endpoints",
      "new": "Implement new-api endpoints"
    }
  ]
}
```

**Step 3:** Apply the replacement:

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-find-replace" spec_id="my-spec-001" find="old-api" replace="new-api" scope="all"
```

**Scope options:**
- `all` - Search titles and descriptions
- `titles` - Only search task/phase titles
- `descriptions` - Only search descriptions

**When to use:**
- Renaming technologies or APIs across the spec
- Updating version numbers
- Fixing consistent typos
- Standardizing terminology

---

## Workflow 8: Rollback Recovery

Restore a specification to a previous state using automatic backups.

**Step 1:** List available backups:

```bash
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec-001"
```

**Example output:**
```json
{
  "spec_id": "my-spec-001",
  "entries": [
    {
      "type": "backup",
      "timestamp": "2025-12-27T21-05-01.437327",
      "file_path": "specs/.backups/my-spec-001/2025-12-27T21-05-01.437327.json"
    },
    {
      "type": "backup",
      "timestamp": "2025-12-27T20-54-48.149929",
      "file_path": "specs/.backups/my-spec-001/2025-12-27T20-54-48.149929.json"
    }
  ],
  "backup_count": 10
}
```

**Step 2:** Preview the rollback with dry-run:

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-rollback" spec_id="my-spec-001" version="2025-12-27T20-54-48.149929" dry_run=true
```

**Step 3:** Apply the rollback:

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-rollback" spec_id="my-spec-001" version="2025-12-27T20-54-48.149929"
```

**When to use:**
- Recovering from unintended modifications
- Reverting experimental changes that didn't work out
- Restoring state after failed bulk operations
- Undoing changes when requirements change back

**Recovery notes:**
- Backups are created automatically before each modification
- The version parameter uses the backup timestamp
- Rollback creates a new backup before restoring (safe to undo)
- Use dry-run to verify which version you're restoring
