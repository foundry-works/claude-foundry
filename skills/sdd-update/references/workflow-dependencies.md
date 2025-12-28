# Dependency Management Workflows

Adding, removing, and managing task dependencies during implementation.

## Adding Dependencies

When you discover a new dependency during implementation, use `add-dependency` to record it without leaving the task context:

```bash
mcp__plugin_foundry_foundry-mcp__task action="add-dependency" spec_id={spec-id} task_id={task-id} depends_on={dependency-task-id}
```

**When to add dependencies:**
- Discovered during implementation that task A requires task B to complete first
- Found shared code that must be implemented before current task
- External blocker identified that maps to another spec task

**Example - discovering a dependency mid-task:**
```bash
# Working on task-2-3, discover it needs task-1-2 first
mcp__plugin_foundry_foundry-mcp__task action="add-dependency" spec_id="my-spec-001" task_id="task-2-3" depends_on="task-1-2"
```

---

## Removing Dependencies

When a dependency is no longer needed (e.g., approach changed, task merged):

```bash
mcp__plugin_foundry_foundry-mcp__task action="remove-dependency" spec_id={spec-id} task_id={task-id} depends_on={dependency-task-id}
```

**When to remove dependencies:**
- Original dependency was incorrectly identified
- Implementation approach changed, eliminating the need
- Dependency task was merged into current task
- Dependency task was removed from spec

**Example - removing an obsolete dependency:**
```bash
# Task-2-3 no longer needs task-1-2 after refactoring approach
mcp__plugin_foundry_foundry-mcp__task action="remove-dependency" spec_id="my-spec-001" task_id="task-2-3" depends_on="task-1-2"
```

---

## Requirement Management

When tests or implementation reveal missing requirements not in the original spec:

```bash
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id={spec-id} task_id={task-id} requirement="Description of discovered requirement"
```

**When to add requirements:**
- Tests reveal edge cases not covered in acceptance criteria
- Implementation uncovers implicit requirements
- User feedback during implementation surfaces new needs
- Integration testing reveals missing validation

**Example - adding discovered requirements:**
```bash
# During implementation, discovered we need input validation
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id="my-spec-001" task_id="task-1-3" requirement="Validate email format before API call"

# Test revealed edge case
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id="my-spec-001" task_id="task-1-3" requirement="Handle empty input array gracefully"
```

**Requirements vs Acceptance Criteria:**
- Requirements are discovered during implementation
- Acceptance criteria are defined during planning
- Use `add-requirement` to track discoveries without modifying the original spec structure
- Requirements are journaled and can inform future spec updates

---

## Circular Dependency Prevention

The MCP tools automatically detect and prevent circular dependencies.

### How It Works

When adding a dependency, the system checks for cycles:

```bash
# This will fail if it creates a cycle
mcp__plugin_foundry_foundry-mcp__task action="add-dependency" spec_id="my-spec-001" task_id="task-1" depends_on="task-3"
```

**Error response if cycle detected:**
```
Error: Circular dependency detected: task-1 -> task-3 -> task-2 -> task-1
```

### Checking Dependencies Before Adding

To preview the dependency graph before adding:

```bash
# Check current dependencies for a task
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}

# Query all blocked tasks to understand dependency chains
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="blocked"
```

### Resolving Circular Dependencies

If you need to restructure dependencies:

1. **Identify the cycle** - Review the error message for the dependency chain
2. **Find the break point** - Determine which dependency is incorrect
3. **Remove the problematic link** - Use `remove-dependency`
4. **Add correct dependency** - Re-add with proper direction

**Example - fixing a cycle:**
```bash
# Remove the dependency causing the cycle
mcp__plugin_foundry_foundry-mcp__task action="remove-dependency" spec_id="my-spec-001" task_id="task-3" depends_on="task-1"

# Add correct dependency in the other direction if needed
mcp__plugin_foundry_foundry-mcp__task action="add-dependency" spec_id="my-spec-001" task_id="task-1" depends_on="task-3"
```

---

## Best Practices

1. **Add dependencies early** - Record them as soon as discovered
2. **Keep dependencies minimal** - Only add true blockers
3. **Document why** - Use journal entries to explain complex dependency decisions
4. **Review before removing** - Ensure the dependency is truly obsolete
5. **Track requirements** - Don't let discovered requirements slip through cracks
