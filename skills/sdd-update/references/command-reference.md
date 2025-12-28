# Command Reference

Quick reference for all MCP tool actions.

## Task Router (`task`)
- `action="update-status"` - Change task status
- `action="block"` - Mark task as blocked with reason
- `action="unblock"` - Unblock a task with resolution
- `action="complete"` - Complete task with journal entry
- `action="start"` - Mark task as in_progress
- `action="query"` - Filter tasks by status, type, or parent
- `action="info"` - Get detailed task information
- `action="list-blocked"` - List all blocked tasks
- `action="update-metadata"` - Update task metadata fields
- `action="add-dependency"` - Add dependency: `depends_on={task-id}`
- `action="remove-dependency"` - Remove dependency: `depends_on={task-id}`
- `action="add-requirement"` - Add discovered requirement: `requirement="..."`

## Journal Router (`journal`)
- `action="add"` - Add journal entry to spec
- `action="list"` - List journal entries

## Lifecycle Router (`lifecycle`)
- `action="activate"` - Move spec from pending/ to active/
- `action="move"` - Move spec between folders
- `action="complete"` - Mark complete and move to completed/

## Spec Router (`spec`)
- `action="get"` - Get spec with progress
- `action="validate"` - Check spec file consistency
- `action="stats"` - Get statistics and validation

## Verification Router (`verification`)
- `action="add"` - Document verification results
- `action="execute"` - Run verification task automatically

## Common Parameters
- `dry_run=true` - Preview changes without saving

---

# Systematic Spec Modification

For **structural modifications** to specs (not progress tracking), use `Skill(foundry:sdd-modify)`.

## What is Systematic Spec Modification?

Systematic modification applies structured changes to specs with:
- Automatic backup before changes
- Validation after changes
- Transaction support with rollback
- Preview before applying (dry-run mode)

## Common Use Cases

**1. Apply Review Feedback**

After running sdd-fidelity-review or sdd-plan-review:

```bash
# Parse review feedback into structured modifications
mcp__plugin_foundry_foundry-mcp__review action="parse-feedback" spec_id="my-spec-001" review_path="reports/review.md"

# Preview modifications
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="my-spec-001" dry_run=true

# Apply modifications
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="my-spec-001"
```

**2. Bulk Modifications**

Apply multiple structural changes at once using `Skill(foundry:sdd-modify)` for guided workflows.

**3. Update Task Descriptions**

Make task descriptions more specific based on implementation learnings:

```json
{
  "modifications": [
    {
      "operation": "update_task",
      "task_id": "task-2-1",
      "field": "description",
      "value": "Implement OAuth 2.0 authentication with PKCE flow and JWT tokens"
    }
  ]
}
```

## When to Use sdd-modify

Use `Skill(foundry:sdd-modify)` when you need to:
- Apply review feedback from sdd-fidelity-review or sdd-plan-review
- Update task descriptions for clarity (beyond just journaling)
- Add verification steps discovered during implementation
- Make multiple structural changes at once
- Ensure changes are validated and safely applied

## See Also

- **Skill(foundry:sdd-modify)** - Full documentation on systematic spec modification
- **skills/sdd-modify/examples/** - Detailed workflow examples
- **`review action="parse-feedback"`** - Parse review reports into modification format
- **`spec action="apply-plan"`** - Apply modifications with validation

---

# Best Practices

## When to Update

- **Update immediately** - Don't wait; update status as work happens
- **Be specific** - Vague notes aren't helpful later
- **Document WHY** - Always explain rationale, not just what changed

## Journaling

- **Link to evidence** - Reference tickets, PRs, discussions
- **Decision rationale** - Explain why decisions were made
- **Use bulk-journal** - Efficiently document multiple completed tasks

## Multi-Tool Coordination

- **Read before write** - Always load latest state before updating
- **Update your tasks only** - Don't modify other tools' work
- **Clear handoffs** - Add journal entry when passing work to another tool

## File Organization

- **Clean transitions** - Move specs promptly when status changes
- **Never rename specs** - Spec file names are based on spec_id
- **Backup before changes** - the MCP tooling handles automatic backups
