# Common MCP Patterns

Reusable patterns for interacting with the MCP tools.

## Preview Changes

Use `dry_run=true` to preview changes before applying:

```bash
mcp__plugin_foundry_foundry-mcp__task action="update-status" spec_id={spec-id} task_id={task-id} status="completed" dry_run=true
mcp__plugin_foundry_foundry-mcp__task action="block" spec_id={spec-id} task_id={task-id} reason="Test" dry_run=true
```

## Query Spec State

```bash
# Get spec with progress
mcp__plugin_foundry_foundry-mcp__spec action="get" spec_id={spec-id}

# Find tasks by status
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="pending"
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="blocked"

# Get specific task details
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}
```

## Update Metadata

```bash
# Update spec metadata fields
mcp__plugin_foundry_foundry-mcp__authoring action="update-frontmatter" spec_id={spec-id} key="status" value="active"
mcp__plugin_foundry_foundry-mcp__authoring action="update-frontmatter" spec_id={spec-id} key="owner" value="user@example.com"
```

## Update Task Metadata

Update metadata fields for individual tasks:

```bash
# Update predefined metadata fields
mcp__plugin_foundry_foundry-mcp__task action="update-metadata" spec_id={spec-id} task_id={task-id} file_path="src/auth.py"
mcp__plugin_foundry_foundry-mcp__task action="update-metadata" spec_id={spec-id} task_id={task-id} description="Updated task description"

# Update with custom metadata JSON
mcp__plugin_foundry_foundry-mcp__task action="update-metadata" spec_id={spec-id} task_id={task-id} custom_metadata='{"focus_areas": ["performance", "security"], "priority": "high"}'
```

**Available metadata fields:**
- `file_path` - File path associated with this task
- `description` - Task description
- `task_category` - Category (e.g., implementation, testing, documentation)
- `actual_hours` - Actual hours spent on task
- `status_note` - Status note or completion note
- `verification_type` - Verification type (run-tests, fidelity)
- `command` - Command executed
- `custom_metadata` - JSON object with any custom fields

**Common use cases:**
```bash
# Track focus areas for investigation tasks
mcp__plugin_foundry_foundry-mcp__task action="update-metadata" spec_id={spec-id} task_id="task-1-1" custom_metadata='{"focus_areas": ["code-doc structure", "skill patterns"]}'

# Document blockers and complexity
mcp__plugin_foundry_foundry-mcp__task action="update-metadata" spec_id={spec-id} task_id="task-2-3" custom_metadata='{"blockers": ["API design unclear"], "complexity": "high"}'
```

## Dependency Management

Manage task dependencies discovered during implementation.

### Query Dependencies

```bash
# Check task dependencies via task info
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}

# Find tasks with blocking dependencies
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="blocked"

# Get task preparation context (includes dependency status)
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id} task_id={task-id}
```

### Add Dependencies

When you discover a task depends on another during implementation:

```bash
# Add dependency relationship
mcp__plugin_foundry_foundry-mcp__task action="add-dependency" spec_id={spec-id} task_id="task-2-3" depends_on="task-1-2"
```

### Remove Dependencies

When a dependency is no longer needed:

```bash
# Remove dependency relationship
mcp__plugin_foundry_foundry-mcp__task action="remove-dependency" spec_id={spec-id} task_id="task-2-3" depends_on="task-1-2"
```

### Track Discovered Requirements

When tests or implementation reveal missing requirements:

```bash
# Add requirement to task
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id={spec-id} task_id="task-1-3" requirement="Validate input before processing"
```

### Common Dependency Patterns

```bash
# Pattern: Discover dependency mid-task, add it, then continue
mcp__plugin_foundry_foundry-mcp__task action="add-dependency" spec_id="my-spec" task_id="task-3-1" depends_on="task-2-4"
mcp__plugin_foundry_foundry-mcp__journal action="add" spec_id="my-spec" task_id="task-3-1" content="Added dependency on task-2-4: shared utility needed"

# Pattern: Check if task can start (no blockers)
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id="my-spec" task_id="task-3-1"
# Response includes: dependencies.can_start, dependencies.blocked_by

# Pattern: Track requirements discovered during testing
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id="my-spec" task_id="task-2-1" requirement="Handle empty array edge case"
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id="my-spec" task_id="task-2-1" requirement="Validate maximum input length"
```

## Validation

For comprehensive spec validation, use the sdd-validate skill:

```bash
Skill(foundry:sdd-validate) "Validate specs/active/{spec-id}.json"
```

For statistics and validation:

```bash
mcp__plugin_foundry_foundry-mcp__spec action="stats" spec_id={spec-id}
```
