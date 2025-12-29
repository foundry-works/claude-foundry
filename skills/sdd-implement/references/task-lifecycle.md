# Task Lifecycle

This reference describes task status transitions and lifecycle management.

## Task States

| Status | Description |
|--------|-------------|
| `pending` | Not yet started |
| `in_progress` | Currently being worked on |
| `completed` | Successfully finished |
| `blocked` | Cannot proceed due to dependencies or issues |

## State Transitions

```
pending → in_progress → completed
    ↓          ↓
    └──→ blocked ──→ in_progress
```

## Starting a Task

Mark a task as in_progress when you begin work:

```bash
mcp__plugin_foundry_foundry-mcp__task action="start" spec_id={spec-id} task_id={task-id}
```

The MCP tool automatically records the start timestamp.

## Completing a Task

Use `task action="complete"` to atomically mark complete AND create a journal entry:

```bash
mcp__plugin_foundry_foundry-mcp__task action="complete" spec_id={spec-id} task_id={task-id} completion_note="Summary of what was accomplished"
```

**What `task action="complete"` does automatically:**
1. Updates task status to `completed`
2. Records completion timestamp
3. Creates a journal entry documenting the completion
4. Clears the `needs_journaling` flag
5. Syncs metadata and recalculates progress
6. Automatically journals parent nodes (phases, groups) that auto-complete

### Parent Node Journaling

When completing a task causes parent nodes to auto-complete:
- **Automatic detection**: System detects when all child tasks are completed
- **Automatic journaling**: Creates journal entries for each auto-completed parent
- **No manual action needed**: You don't need to manually journal parent completions
- **Hierarchical**: Works for multiple levels

## Handling Blockers

### Mark Task as Blocked

When a task cannot proceed:

```bash
mcp__plugin_foundry_foundry-mcp__task action="block" spec_id={spec-id} task_id={task-id} reason="Description of blocker" blocker_type={type} ticket="TICKET-123"
```

**Blocker types:**
- `dependency` - Waiting on external dependency
- `technical` - Technical issue blocking progress
- `resource` - Resource unavailability
- `decision` - Awaiting architectural/product decision

### Unblock Task

When blocker is resolved:

```bash
mcp__plugin_foundry_foundry-mcp__task action="unblock" spec_id={spec-id} task_id={task-id} resolution="Description of how it was resolved"
```

### List All Blockers

```bash
mcp__plugin_foundry_foundry-mcp__task action="list-blocked" spec_id={spec-id}
```

## Status-Only Update (Not Recommended)

If you need to mark a task completed without journaling (rare):

```bash
mcp__plugin_foundry_foundry-mcp__task action="update-status" spec_id={spec-id} task_id={task-id} status="completed" note="Brief completion note"
```

**Warning:** Use `task action="complete"` instead to ensure proper journaling.

## Best Practices

1. **Update immediately** - Don't batch updates
2. **Be specific** - Vague notes aren't helpful later
3. **Document WHY** - Rationale, not just what changed
4. **Use complete** - Ensures proper journaling
