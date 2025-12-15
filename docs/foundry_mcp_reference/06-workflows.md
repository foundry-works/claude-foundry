# Common Workflows

> Practical patterns for using foundry-mcp tools in spec-driven development.

---

## Workflow Overview

| Workflow | Purpose | Key Routers |
|----------|---------|-------------|
| **Planning** | Create and validate specifications | `authoring`, `spec`, `code` |
| **Task Execution** | Find and complete tasks | `task` |
| **Verification** | Validate implementation | `verification`, `review`, `test` |
| **Phase Completion** | Progress through phases | `task`, `lifecycle` |
| **PR Creation** | Generate pull requests | `pr`, `journal` |

---

## Planning Workflow

### Overview

The planning workflow creates a structured specification before any implementation begins.

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│ Check Docs     │───►│ Create Spec    │───►│ Validate       │
│ Availability   │    │ (phases/tasks) │    │ and Save       │
└────────────────┘    └────────────────┘    └────────────────┘
```

### Step 1: Check Documentation Availability

Before planning, verify codebase documentation is available:

```
mcp__plugin_foundry_foundry-mcp__code action="doc-stats"
```

**Response indicates:**
- Whether documentation exists
- How recent it is
- What modules are covered

If documentation is stale or missing, regenerate before planning.

### Step 2: Gather Context

Use code tools to understand the codebase:

```
# Find related classes
mcp__plugin_foundry_foundry-mcp__code action="find-class" symbol="Auth"

# Find related functions
mcp__plugin_foundry_foundry-mcp__code action="find-function" symbol="authenticate"

# Trace dependencies
mcp__plugin_foundry_foundry-mcp__code action="trace-calls" symbol="authenticate"
```

### Step 3: Create the Specification

Using the `sdd-plan` skill is recommended, or create directly:

```
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create"
  name="user-auth-2025-12-04"
  title="User Authentication System"
  description="Implement JWT-based authentication"
```

### Step 4: Validate and Fix

```
# Validate structure
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="user-auth-2025-12-04"

# Auto-fix issues if needed
mcp__plugin_foundry_foundry-mcp__spec action="fix" spec_id="user-auth-2025-12-04"
```

### Step 5: Activate When Ready

```
mcp__plugin_foundry_foundry-mcp__lifecycle action="activate" spec_id="user-auth-2025-12-04"
```

---

## Task Execution Workflow

### Overview

The task execution workflow progresses through implementation systematically.

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│ Find Next Task │───►│ Get Context    │───►│ Implement      │
└────────────────┘    └────────────────┘    └────────────────┘
        ▲                                            │
        │                                            │
        │            ┌────────────────┐              │
        └────────────│ Complete Task  │◄─────────────┘
                     │ + Journal      │
                     └────────────────┘
```

### Step 1: Find Next Actionable Task

```
mcp__plugin_foundry_foundry-mcp__task action="next" spec_id="user-auth-2025-12-04"
```

**Response provides:**
- Task details (description, file path)
- Dependencies status
- Context for implementation

If no task is returned, all available tasks are blocked or completed.

### Step 2: Prepare Task with Full Context

```
mcp__plugin_foundry_foundry-mcp__task action="prepare"
  spec_id="user-auth-2025-12-04"
  task_id="task-001"
```

**Response provides:**
- Complete task definition
- Related code context
- Dependencies and their outputs
- Verification criteria

### Step 3: Implement the Task

Use the context from `task-prepare` to implement the change:
- Follow the task description
- Modify the specified file(s)
- Respect existing patterns
- Document decisions

### Step 4: Complete and Journal

```
mcp__plugin_foundry_foundry-mcp__task action="complete"
  spec_id="user-auth-2025-12-04"
  task_id="task-001"
  completion_note="Implemented User model with email/password fields"
```

Add a journal entry separately:

```
mcp__plugin_foundry_foundry-mcp__journal action="add"
  spec_id="user-auth-2025-12-04"
  task_id="task-001"
  entry_type="completion"
  content="Created User model following existing patterns from Product model"
```

### Step 5: Repeat

Return to Step 1 to find the next task.

---

## Verification Workflow

### Overview

Verification ensures implementation matches specification requirements.

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│ Run Auto       │───►│ Fidelity       │───►│ Manual         │
│ Tests          │    │ Review         │    │ Checklist      │
└────────────────┘    └────────────────┘    └────────────────┘
```

### Automated Verification

Run tests specified in task/phase verification:

```
# Run specific test
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/auth/"

# Quick smoke test
mcp__plugin_foundry_foundry-mcp__test action="run-quick"
```

### Fidelity Review

Compare implementation against spec requirements:

```
mcp__plugin_foundry_foundry-mcp__review action="fidelity"
  spec_id="user-auth-2025-12-04"
  task_id="task-001"
```

**Response shows:**
- Requirements met/not met
- Deviations detected
- Recommendations

### Phase Verification

Check if all phase verification criteria pass:

```
mcp__plugin_foundry_foundry-mcp__verification action="execute"
  spec_id="user-auth-2025-12-04"
  verify_id="phase-1"
```

### Manual Verification

Some verification requires human review. Track completion:

```
mcp__plugin_foundry_foundry-mcp__journal action="add"
  spec_id="user-auth-2025-12-04"
  task_id="task-manual-review"
  entry_type="verification"
  content="Security review completed by @security-team"
```

---

## Phase Completion Workflow

### Overview

Phase completion marks logical milestones in development.

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│ Check All      │───►│ Run Phase      │───►│ Journal &      │
│ Tasks Done     │    │ Verification   │    │ Proceed        │
└────────────────┘    └────────────────┘    └────────────────┘
```

### Step 1: Check Phase Status

```
mcp__plugin_foundry_foundry-mcp__task action="query"
  spec_id="user-auth-2025-12-04"
  parent="phase-1"
```

**Response shows:**
- Tasks completed vs total
- Blocked tasks
- Missing verifications

### Step 2: Run Phase Verification

```
mcp__plugin_foundry_foundry-mcp__verification action="execute"
  spec_id="user-auth-2025-12-04"
  verify_id="phase-1"
```

### Step 3: Journal Phase Completion

```
mcp__plugin_foundry_foundry-mcp__journal action="add"
  spec_id="user-auth-2025-12-04"
  entry_type="completion"
  content="Phase 1 complete: Core auth logic implemented"
```

### Step 4: Continue to Next Phase

Proceed to tasks in the next phase using the task execution workflow.

---

## PR Creation Workflow

### Overview

PR creation generates pull requests with rich context from specs.

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│ Gather PR      │───►│ Generate       │───►│ Create PR      │
│ Context        │    │ Description    │    │ via gh         │
└────────────────┘    └────────────────┘    └────────────────┘
```

### Step 1: Gather Context

```
mcp__plugin_foundry_foundry-mcp__pr action="context" spec_id="user-auth-2025-12-04"
```

**Response provides:**
- Summary of changes
- Completed tasks
- Journal decisions
- Files modified
- Test status

### Step 2: Generate PR Description

The `sdd-pr` skill uses context to create:
- Summary of changes
- Implementation details
- Testing notes
- Decision rationale

### Step 3: Create the PR

```
mcp__plugin_foundry_foundry-mcp__pr action="create"
  spec_id="user-auth-2025-12-04"
  title="feat: Add user authentication system"
```

### Step 4: Complete the Spec

After PR is merged:

```
mcp__plugin_foundry_foundry-mcp__lifecycle action="complete"
  spec_id="user-auth-2025-12-04"
```

---

## Blocker Management

### Recording a Blocker

```
mcp__plugin_foundry_foundry-mcp__task action="block"
  spec_id="user-auth-2025-12-04"
  task_id="task-003"
  reason="Waiting for API endpoint from backend team"
```

### Listing Blocked Tasks

```
mcp__plugin_foundry_foundry-mcp__task action="list-blocked" spec_id="user-auth-2025-12-04"
```

### Unblocking a Task

```
mcp__plugin_foundry_foundry-mcp__task action="unblock"
  spec_id="user-auth-2025-12-04"
  task_id="task-003"
  resolution="API endpoint now available, documented in wiki"
```

---

## Decision Journaling

### Recording Decisions

```
mcp__plugin_foundry_foundry-mcp__journal action="add"
  spec_id="user-auth-2025-12-04"
  task_id="task-002"
  entry_type="decision"
  content="Chose bcrypt over argon2 for password hashing"
```

### Viewing Journal

```
mcp__plugin_foundry_foundry-mcp__journal action="list"
  spec_id="user-auth-2025-12-04"
  limit=10
```

---

## Related Documentation

- **[04-Tool Reference](./04-tool-reference.md)** — Complete tool catalog
- **[08-Integration Patterns](./08-integration-patterns.md)** — How skills use these workflows
- **[03-Spec Lifecycle](./03-spec-lifecycle.md)** — State transitions

---

*[Back to Index](./README.md)*
