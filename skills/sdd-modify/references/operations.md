# Operation Formats

Detailed JSON structures for each modification operation.

## Supported Operations

| Operation | Purpose |
|-----------|---------|
| `update_task` | Modify task title, description, file_path, category |
| `add_verification` | Add verification step to task |
| `update_metadata` | Update task metadata (hours, priority, etc.) |
| `batch_update` | Apply same change to multiple nodes |
| `add_node` | Add new task/subtask/verify node |
| `remove_node` | Remove node (optionally cascading) |
| `move_task` | Move task to different parent or position |
| `move_phase` | Reorder phase within the spec |
| `find_replace` | Find and replace text across spec nodes |

---

## update_task

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

## add_verification

Add a verification step to a task:

```json
{
  "operation": "add_verification",
  "task_id": "task-1-3",
  "verification": {
    "verification_type": "fidelity",
    "mcp_tool": "mcp__plugin_foundry_foundry-mcp__review action=\"fidelity\"",
    "scope": "task",
    "target": "task-1-3",
    "expected": "Implementation matches specification"
  }
}
```

**Verification types:**
- `run-tests` - Automated tests via `mcp__plugin_foundry_foundry-mcp__test action="run"`
- `fidelity` - Implementation-vs-spec comparison via `mcp__plugin_foundry_foundry-mcp__review action="fidelity"`

**Run-tests verification example:**
```json
{
  "operation": "add_verification",
  "task_id": "task-2-1",
  "verification": {
    "verification_type": "run-tests",
    "mcp_tool": "mcp__plugin_foundry_foundry-mcp__test action=\"run\"",
    "expected": "All tests pass"
  }
}
```

---

## update_metadata

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

## batch_update

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

## add_node

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

## remove_node

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

## move_task

Move a task to a different parent or position within the spec:

```json
{
  "operation": "move_task",
  "task_id": "task-2-3",
  "parent": "phase-3",
  "position": 1
}
```

**MCP invocation:**
```bash
mcp__plugin_foundry_foundry-mcp__task action="move" spec_id={spec-id} task_id="task-2-3" parent="phase-3" position=1
```

**Parameters:**
- `task_id` (required) - Task to move
- `parent` (optional) - New parent phase/task ID (omit to keep same parent)
- `position` (optional) - Target position within parent (1-based; omit to move to end)

**Response fields:**
- `old_parent` / `new_parent` - Parent before/after move
- `old_position` / `new_position` - Position before/after move
- `is_reparenting` - Whether task moved to a different parent
- `tasks_in_subtree` - Number of tasks moved (includes descendants)

---

## move_phase

Reorder a phase within the spec:

```json
{
  "operation": "move_phase",
  "phase_id": "phase-2",
  "position": 1
}
```

**MCP invocation:**
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="phase-move" spec_id={spec-id} phase_id="phase-2" position=1
```

**Parameters:**
- `phase_id` (required) - Phase to move
- `position` (required) - Target position (1-based index)

**Response fields:**
- `old_position` / `new_position` - Position before/after move
- `moved` - Whether the phase actually moved (false if already at target)

---

## find_replace

Find and replace text across spec nodes:

```json
{
  "operation": "find_replace",
  "find": "old-pattern",
  "replace": "new-value",
  "scope": "all",
  "use_regex": false,
  "case_sensitive": true
}
```

**MCP invocation:**
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-find-replace" spec_id={spec-id} find="old-pattern" replace="new-value" scope="all"
```

**Parameters:**
- `find` (required) - Text or pattern to find
- `replace` (required) - Replacement text
- `scope` (required) - Where to search: `all`, `descriptions`, `titles`
- `use_regex` (optional) - Treat `find` as regex (default: false)
- `case_sensitive` (optional) - Case-sensitive matching (default: true)

**Response fields:**
- `total_replacements` - Total number of replacements made
- `nodes_affected` - Number of nodes modified
- `changes` - Array of changes with `node_id`, `field`, `old`, `new`

---

# Modification JSON Structure

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
