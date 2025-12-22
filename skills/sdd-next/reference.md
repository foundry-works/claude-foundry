# SDD-Next Reference

Detailed workflows, examples, and edge cases for the sdd-next skill.

## Table of Contents

- [Context Gathering Best Practices](#context-gathering-best-practices)
- [Agent Delegation](#agent-delegation)
- [Deep Dive Context Structure](#deep-dive-context-structure)
- [Built-in Subagent Patterns](#built-in-subagent-patterns)
- [Using Doc-Scope During Implementation](#using-doc-scope-during-implementation)
- [Post-Implementation Checklist](#post-implementation-checklist)
- [Troubleshooting](#troubleshooting)

---

## Context Gathering Best Practices

### DO: Use MCP Tools for Spec Data

Always use MCP tools to access specification data:

```bash
# Get next actionable task with full context
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id}

# Query tasks by status or parent
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="pending"
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent="phase-1"

# Get detailed task information
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}
```

### DON'T: Read Spec Files Directly

Never use these approaches for spec data:

```bash
# BAD - Wastes context tokens and bypasses validation
Read("specs/active/my-spec.json")
cat specs/active/my-spec.json
jq '.phases' specs/active/my-spec.json
grep "task-1" specs/active/my-spec.json
```

### Context Efficiency

| Approach | Context Cost | Reliability |
|----------|--------------|-------------|
| MCP `task action="prepare"` | Low | High |
| MCP `task action="info"` | Low | High |
| Read spec file directly | High | Low |
| Bash + jq parsing | High | Medium |

### Anti-Patterns

1. **Reading full spec files** - Inflates context with JSON structure
2. **Bash loops over tasks** - Use `task action="query"` instead
3. **Creating temp scripts** - Unnecessary complexity
4. **Parsing JSON with grep/sed** - Fragile and error-prone

---

## Agent Delegation

### When to Use sdd-planner vs Implement Directly

| Scenario | Recommendation |
|----------|---------------|
| Task has clear, detailed instructions | Implement directly |
| Task requires architectural decisions | Consider sdd-plan first |
| Task is exploratory/investigation | Use Explore subagent |
| Task affects multiple systems | Consider sdd-plan first |
| Task has ambiguous requirements | Ask user via AskUserQuestion |

### Delegation Flow

```
Task Ready
    |
    +-- Clear instructions? --> Implement directly
    |
    +-- Needs architecture? --> Skill(foundry:sdd-plan) for sub-spec
    |
    +-- Needs exploration? --> Use Explore subagent
    |
    +-- Ambiguous? --> AskUserQuestion
```

### NEVER Delegate Back to sdd-next

The anti-recursion rule is critical. If you find yourself about to call `Skill(sdd-next)` from within this skill:

1. **STOP** - This indicates a workflow error
2. **Review** - Check if you're trying to surface the next task
3. **Use MCP directly** - Call `task action="prepare"` instead
4. **Continue workflow** - Proceed to 3.5 (Surface Next Recommendation)

---

## Deep Dive Context Structure

The `task action="prepare"` response provides comprehensive context for task execution.

### Response Structure

```json
{
  "task_data": {
    "id": "task-2-3",
    "title": "Implement JWT verification middleware",
    "type": "task",
    "status": "pending",
    "metadata": {
      "file_path": "src/middleware/auth.ts",
      "estimated_hours": 2,
      "task_category": "implementation"
    },
    "instructions": ["Step 1...", "Step 2..."]
  },
  "dependencies": {
    "can_start": true,
    "blocked_by": [],
    "depends_on": ["task-2-1", "task-2-2"]
  },
  "context": {
    "phase": {
      "id": "phase-2",
      "title": "Authentication Implementation",
      "progress": 0.4
    },
    "parent_task": {
      "id": "task-group-2",
      "title": "Auth Middleware"
    },
    "previous_sibling": {
      "id": "task-2-2",
      "title": "Create token service",
      "status": "completed",
      "summary": "Implemented JWT signing and validation..."
    },
    "sibling_files": [
      "src/services/token.ts",
      "tests/services/token.spec.ts"
    ],
    "journal": [
      {
        "timestamp": "2025-01-15T10:30:00Z",
        "entry_type": "decision",
        "title": "Using RS256 algorithm",
        "content": "Chose RS256 for JWT signing..."
      }
    ],
    "dependencies": {
      "task-2-1": {
        "status": "completed",
        "summary": "Database schema ready"
      }
    }
  }
}
```

### Using Context Fields

| Field | Purpose | Usage |
|-------|---------|-------|
| `task_data.instructions` | Step-by-step guidance | Primary implementation guide |
| `task_data.metadata.file_path` | Target file | Where to implement |
| `context.previous_sibling` | Prior work | Continuity reference |
| `context.sibling_files` | Related files | Files to review |
| `context.journal` | Decisions made | Architectural context |
| `dependencies.can_start` | Readiness check | Verify before starting |

### Fallback: task action="info"

When `task action="prepare"` doesn't provide needed data:

```bash
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}
```

Returns focused task details without full context tree.

---

## Built-in Subagent Patterns

Claude Code provides built-in subagents for efficient exploration without bloating main context.

### Available Subagents

| Subagent | Model | Best For |
|----------|-------|----------|
| **Explore** | Haiku | File discovery, pattern search, codebase questions |
| **general-purpose** | Sonnet | Complex multi-step research, code analysis |
| **Plan** | Sonnet | Architecture design, implementation planning |

### Explore Agent Thoroughness Levels

| Level | Use Case | Example |
|-------|----------|---------|
| `quick` | Known location, simple search | Find a specific file |
| `medium` | Moderate exploration | Find related implementations |
| `very thorough` | Comprehensive analysis | Understand entire subsystem |

### Pre-Implementation Exploration

Before implementing a task, gather context efficiently:

```
Use the Explore agent (medium thoroughness) to find:
- Existing implementations of similar patterns
- Test files for the target module
- Related documentation that may need updates
- Import/export patterns in the target directory
```

### Pattern: Find Related Code

```
Use the Explore agent (quick thoroughness) to find:
- All files importing {module-name}
- Test coverage for {function-name}
- Configuration files affecting {feature}
```

### Pattern: Understand Architecture

```
Use the Explore agent (very thorough) to understand:
- How {subsystem} is structured
- Data flow through {component}
- Integration points with {external-system}
```

### When NOT to Use Subagents

- Single file already in context
- Task specifies exact file paths
- Near context limit (80%+)
- Simple, isolated changes

---

## Using Doc-Scope During Implementation

Doc-scope provides focused documentation views during implementation.

### Available Views

| View | Purpose |
|------|---------|
| `implement` | Implementation guidance, code patterns |
| `test` | Testing requirements, test patterns |
| `review` | Review criteria, acceptance checks |

### Invoking Doc-Scope

```bash
# Get implementation-focused docs for current task
mcp__plugin_foundry_foundry-mcp__doc action="scope" view="implement" task_id={task-id}

# Get testing guidance
mcp__plugin_foundry_foundry-mcp__doc action="scope" view="test" task_id={task-id}
```

### When to Use Doc-Scope

1. **Starting implementation** - Get `implement` view for patterns and constraints
2. **Writing tests** - Get `test` view for test requirements
3. **Self-review** - Get `review` view before marking complete

### Example Workflow

```
1. Task selected via sdd-next
2. Get implementation docs: doc action="scope" view="implement"
3. Implement following guidance
4. Get test docs: doc action="scope" view="test"
5. Write/run tests
6. Get review docs: doc action="scope" view="review"
7. Self-verify against criteria
8. Mark complete
```

---

## Post-Implementation Checklist

Before marking a task complete, verify these items.

### Required Checks

- [ ] Implementation matches task instructions
- [ ] All acceptance criteria met
- [ ] Tests written and passing
- [ ] No regressions introduced
- [ ] Code follows project patterns

### Journal Entry Requirements

Every task completion MUST include a journal entry with:

1. **What was accomplished** - Brief summary of implementation
2. **Tests run and results** - Which tests, pass/fail status
3. **Verification performed** - How you verified correctness
4. **Deviations from plan** - Any changes from original instructions
5. **Files created/modified** - List of affected files

### Example Completion

```bash
Skill(foundry:sdd-update) "Complete task task-2-3 in spec my-spec-001. Completion note: Implemented JWT verification middleware with RS256 support. All 8 unit tests passing. Manual verification: auth flow works in dev. Created src/middleware/auth.ts (120 lines) and tests/middleware/auth.spec.ts (8 tests)."
```

### Never Mark Complete If

- Tests are failing
- Implementation is partial
- Unresolved errors exist
- Required files not found
- Blockers prevent verification

### Blocked Task Handling

If you cannot complete:

```bash
# Mark as blocked with reason
mcp__plugin_foundry_foundry-mcp__task action="block" spec_id={spec-id} task_id={task-id} reason="Missing API endpoint from task-2-1" blocker_type="dependency"

# Present alternatives to user
AskUserQuestion: "Task blocked. Options: 1) Resolve blocker first, 2) Work on different task, 3) Skip and document"
```

---

## Troubleshooting

### Task Not Found

**Error:** MCP tool returns "task not found"

**Causes:**
- Incorrect task_id format
- Task in different spec
- Spec not in active/ folder

**Resolution:**
```bash
# List all tasks to find correct ID
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id}

# Check spec location
mcp__plugin_foundry_foundry-mcp__spec action="find" spec_id={spec-id}
```

### Spec Not Active

**Error:** Spec found in pending/ not active/

**Resolution:**
```bash
# Activate the spec first
mcp__plugin_foundry_foundry-mcp__lifecycle action="activate" spec_id={spec-id}
```

### Circular Dependencies

**Error:** Task blocked by task that depends on it

**Resolution:**
1. Check dependency graph with `task action="info"`
2. Identify circular reference
3. Use `Skill(foundry:sdd-modify)` to fix spec

### Context Limit Warnings

**Symptom:** `[CONTEXT X%]` warnings appearing

**Resolution:**
1. Complete current task
2. Mark task complete with journal
3. Run `/clear`
4. Resume with `/sdd-next`

### SDD Tools in Minimal Mode

**Error:** MCP tools not responding

**Resolution:**
1. Run `/sdd-on`
2. Restart Claude
3. Run `/sdd-next` again

---

## Quick Reference

### Core Commands

```bash
# Task preparation (primary)
mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id}

# Task status update
mcp__plugin_foundry_foundry-mcp__task action="start" spec_id={spec-id} task_id={task-id}

# Task completion (via sdd-update skill)
Skill(foundry:sdd-update) "Complete task {task-id} in spec {spec-id}. Completion note: ..."

# Blocking
mcp__plugin_foundry_foundry-mcp__task action="block" spec_id={spec-id} task_id={task-id} reason="..."

# Unblocking
mcp__plugin_foundry_foundry-mcp__task action="unblock" spec_id={spec-id} task_id={task-id} resolution="..."
```

### Decision Tree

```
Start Task
    |
    +-- Dependencies met?
    |       |
    |       +-- No --> Block or choose different task
    |       |
    |       +-- Yes --> Continue
    |
    +-- Clear instructions?
    |       |
    |       +-- No --> Ask user or explore
    |       |
    |       +-- Yes --> Implement
    |
    +-- Implementation complete?
    |       |
    |       +-- No --> Continue or block
    |       |
    |       +-- Yes --> Run tests
    |
    +-- Tests passing?
    |       |
    |       +-- No --> Fix or block
    |       |
    |       +-- Yes --> Mark complete
    |
    +-- Surface next task
```
