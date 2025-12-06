# SDD-Modify Reference

Detailed workflows, examples, and edge cases for the sdd-modify skill.

## Table of Contents

- [Workflow Details](#workflow-details)
  - [Parse Review Feedback](#workflow-1-parse-review-feedback)
  - [Preview Modifications](#workflow-2-preview-modifications)
  - [Apply Modifications](#workflow-3-apply-modifications)
  - [Direct Modifications](#workflow-4-direct-modifications-without-review-parsing)
- [Operation Formats](#operation-formats)
- [Modification JSON Structure](#modification-json-structure)
- [Review Source Workflows](#review-source-workflows)
- [Error Handling](#error-handling)
- [Constraints](#constraints)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Command Reference](#command-reference)

---

## Workflow Details

### Workflow 1: Parse Review Feedback

When starting from a review report (markdown output from sdd-plan-review or sdd-fidelity-review), convert it to structured modifications:

```bash
mcp__foundry-mcp__review-parse-feedback spec_id="my-spec-001" review_path="reports/plan-review.md"
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

### Workflow 2: Preview Modifications

**Always preview before applying.** Use dry-run mode to see what will change:

```bash
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec-001" modifications_file="suggestions.json" dry_run=true
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

### Workflow 3: Apply Modifications

After confirming the preview, apply changes:

```bash
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec-001" modifications_file="suggestions.json"
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

### Workflow 4: Direct Modifications (without review parsing)

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
        "mcp_tool": "mcp__foundry-mcp__test-run",
        "expected": "All tests pass"
      }
    }
  ]
}
```

**Step 2:** Preview and apply:

```bash
# Preview
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec-001" modifications_file="modifications.json" dry_run=true

# Apply
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec-001" modifications_file="modifications.json"
```

**When to use:**
- Bulk updates to multiple tasks
- Programmatic spec modifications
- Migrating specs with automated transformations
- When you know exactly what changes to make

---

## Operation Formats

### update_task

Modify task fields (title, details, file_path, category):

```json
{
  "operation": "update_task",
  "task_id": "task-2-1",
  "field": "details",
  "value": "Updated task description text"
}
```

**Supported fields:**
- `title` - Task title
- `details` - Task description/details
- `file_path` - Associated file path
- `category` - Category (implementation, testing, etc.)

---

### add_verification

Add a verification step to a task:

```json
{
  "operation": "add_verification",
  "task_id": "task-1-3",
  "verification": {
    "verification_type": "fidelity",
    "mcp_tool": "mcp__foundry-mcp__spec-review-fidelity",
    "scope": "task",
    "target": "task-1-3",
    "expected": "Implementation matches specification"
  }
}
```

**Verification types:**
- `run-tests` - Automated tests via `mcp__foundry-mcp__test-run`
- `fidelity` - Implementation-vs-spec comparison via `mcp__foundry-mcp__spec-review-fidelity`

**Run-tests verification example:**
```json
{
  "operation": "add_verification",
  "task_id": "task-2-1",
  "verification": {
    "verification_type": "run-tests",
    "mcp_tool": "mcp__foundry-mcp__test-run",
    "expected": "All tests pass"
  }
}
```

---

### update_metadata

Update task metadata (hours, priority, etc.):

```json
{
  "operation": "update_metadata",
  "task_id": "task-3-2",
  "metadata": {
    "estimated_hours": 6,
    "priority": "high",
    "complexity": "medium"
  }
}
```

---

### batch_update

Apply the same change to multiple nodes:

```json
{
  "operation": "batch_update",
  "task_ids": ["task-1-1", "task-1-2", "task-1-3"],
  "field": "category",
  "value": "implementation"
}
```

---

### add_node

Add a new task, subtask, or verify node:

```json
{
  "operation": "add_node",
  "parent_id": "phase-2",
  "node_type": "task",
  "node": {
    "title": "Implement rate limiting",
    "description": "Add rate limiting middleware to API endpoints",
    "estimated_hours": 4
  }
}
```

**Node types:** `task`, `subtask`, `verify`

---

### remove_node

Remove a node (optionally cascading to children):

```json
{
  "operation": "remove_node",
  "task_id": "task-4-3",
  "cascade": false
}
```

**Cascade options:**
- `false` (default) - Only remove the specified node; fails if it has children
- `true` - Remove node and all descendants

---

## Modification JSON Structure

Complete structure of a modifications file:

```json
{
  "spec_id": "my-spec-001",
  "source": "reports/review.md",
  "generated": "2025-10-18T14:30:00Z",
  "modifications": [
    {
      "operation": "update_task",
      "task_id": "task-2-1",
      "field": "description",
      "value": "New value",
      "reason": "Optional explanation"
    }
  ],
  "metadata": {
    "review_type": "plan",
    "reviewer": "sdd-plan-review"
  }
}
```

**Required fields:**
- `spec_id` - Target specification ID
- `modifications` - Array of modification operations

**Optional fields:**
- `source` - Where modifications originated
- `generated` - Timestamp of generation
- `metadata` - Additional context

---

## Review Source Workflows

### From sdd-plan-review

After plan quality review:

```bash
# Review generates report
# ... (sdd-plan-review output) ...

# Parse into modifications
mcp__foundry-mcp__review-parse-feedback spec_id="my-spec" review_path="reports/plan-review.md"

# Preview and apply
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec" modifications_file="reports/plan-review.md.suggestions.json" dry_run=true
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec" modifications_file="reports/plan-review.md.suggestions.json"
```

### From sdd-fidelity-review

After implementation fidelity check:

```bash
# Fidelity review generates report
# ... (sdd-fidelity-review output) ...

# Parse into modifications
mcp__foundry-mcp__review-parse-feedback spec_id="my-spec" review_path="reports/fidelity-review.md"

# Preview and apply
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec" modifications_file="reports/fidelity-review.md.suggestions.json" dry_run=true
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec" modifications_file="reports/fidelity-review.md.suggestions.json"
```

### Direct JSON Modification

When you know the exact changes:

```bash
# Create modifications.json manually
# Preview
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec" modifications_file="modifications.json" dry_run=true

# Apply
mcp__foundry-mcp__spec-apply-plan spec_id="my-spec" modifications_file="modifications.json"
```

---

## Error Handling

### Spec Not Found

**Symptom:** Error mentioning spec ID not found

**Resolution:**
1. Verify spec ID is correct
2. Check spec location: `mcp__foundry-mcp__spec-list`
3. Ensure spec is in expected folder (active/, pending/, etc.)

### Invalid Modifications File

**Symptom:** JSON parse error or invalid operation

**Resolution:**
1. Validate JSON syntax
2. Check operation names are correct
3. Verify task IDs exist in spec
4. Ensure required fields are present

### Validation Failure After Apply

**Symptom:** "Validation failed, rolling back" message

**What happened:**
- Modifications were applied
- Validation detected issues
- Original spec restored from backup

**Resolution:**
1. Review validation errors
2. Fix modifications to address validation issues
3. Re-apply with corrected modifications

### Review Report Cannot Be Parsed

**Symptom:** `review-parse-feedback` fails

**Resolution:**
1. Verify review file exists and is readable
2. Check review format matches expected structure
3. Try running review again with proper output format

---

## Constraints

- **Do NOT** read spec JSON directly - use MCP tools only
- **Do NOT** apply modifications without previewing first
- **Do NOT** skip validation after apply
- **Do NOT** continue if validation fails - report and stop
- **Return** concise summaries, not verbose logs
- **Document** significant modifications in journal entries

---

## Troubleshooting

### Preview Shows No Changes

**Possible causes:**
- Modifications file is empty or malformed
- Task IDs don't exist in spec
- Values are already set to requested values

**Fix:** Verify modifications JSON structure and task IDs

### Validation Fails Repeatedly

**Possible causes:**
- Modifications create invalid spec structure
- Circular dependencies introduced
- Required fields missing

**Fix:** Review validation errors, adjust modifications accordingly

### Rollback Occurred But Issue Persists

**Possible causes:**
- Backup file was corrupted
- Manual edits made between backup and apply

**Fix:**
1. Check backup file integrity
2. Use older backup if available
3. Regenerate spec from source if necessary

### Review Parsing Produces Empty Modifications

**Possible causes:**
- Review report has no actionable findings
- Review format not recognized

**Fix:**
1. Check review report content
2. Ensure review follows expected markdown structure
3. Consider direct modifications if parsing fails

---

## Best Practices

### DO

- **Always preview** before applying modifications
- **Use review-parse-feedback** for review reports
- **Validate** after every modification
- **Document** modifications in journal entries
- **Use batch_update** for multiple similar changes
- **Keep modifications atomic** - one logical change per apply

### DON'T

- **Skip** the dry-run preview step
- **Apply** modifications without understanding the scope
- **Continue** after validation failures
- **Manually edit** spec JSON files
- **Make multiple separate** modifications when batch_update would work
- **Ignore** validation warnings

---

## Command Reference

### Core Commands

| Command | Purpose |
|---------|---------|
| `spec-apply-plan` | Apply modifications from JSON file |
| `review-parse-feedback` | Convert review markdown to modifications |
| `spec-validate` | Validate spec structure |
| `spec-validate-fix` | Auto-fix validation issues |

### Supporting Commands

| Command | Purpose |
|---------|---------|
| `spec-get` | Retrieve spec data |
| `spec-get-hierarchy` | Get spec structure |
| `spec-list` | List available specs |
| `task-add` | Add new task node |
| `task-remove` | Remove task node |
| `task-update-metadata` | Update task metadata |
| `spec-update-frontmatter` | Update spec metadata |
| `verification-add` | Add verification result |
| `journal-add` | Add journal entry |

### Common Flags

| Flag | Description |
|------|-------------|
| `dry_run=true` | Preview without applying |
| `cascade=true` | Include child nodes in removal |
| `auto_fix=true` | Automatically fix validation issues |
