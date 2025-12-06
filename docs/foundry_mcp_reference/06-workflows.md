# Common Workflows

> Practical patterns for using foundry-mcp tools in spec-driven development.

---

## Workflow Overview

| Workflow | Purpose | Key Tools |
|----------|---------|-----------|
| **Planning** | Create and validate specifications | `spec-create`, `spec-validate`, `doc-*` |
| **Task Execution** | Find and complete tasks | `task-next`, `task-prepare`, `task-complete` |
| **Verification** | Validate implementation | `verify-check`, `fidelity-check`, `test-run` |
| **Phase Completion** | Progress through phases | `phase-check-complete`, `spec-lifecycle-*` |
| **PR Creation** | Generate pull requests | `pr-context`, `pr-create` |
| **Autonomous Mode** | Continuous task execution | `session-*`, `task-next` |

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
mcp__foundry-mcp__doc-stats with path="src/"
```

**Response indicates:**
- Whether documentation exists
- How recent it is
- What modules are covered

If documentation is stale or missing, regenerate before planning.

### Step 2: Gather Context

Use documentation tools to understand the codebase:

```
# Get module overview
mcp__foundry-mcp__doc-scope with path="src/auth" view="plan"

# Find related implementations
mcp__foundry-mcp__doc-search with query="authentication" scope="src/"

# Understand dependencies
mcp__foundry-mcp__doc-dependencies with path="src/auth/login.py"
```

### Step 3: Create the Specification

Using the `sdd-plan` skill is recommended, or create directly:

```
mcp__foundry-mcp__spec-create with
  spec_id="user-auth-2025-12-04"
  title="User Authentication System"
  description="Implement JWT-based authentication"
  phases=[
    {
      "phase_id": "phase-1",
      "name": "Core Auth Logic",
      "tasks": [
        {
          "task_id": "task-001",
          "description": "Create User model",
          "file_path": "src/models/user.py"
        }
      ]
    }
  ]
```

### Step 4: Validate and Fix

```
# Validate structure
mcp__foundry-mcp__spec-validate with spec_id="user-auth-2025-12-04"

# Auto-fix issues if needed
mcp__foundry-mcp__spec-fix with spec_id="user-auth-2025-12-04"
```

### Step 5: Activate When Ready

```
mcp__foundry-mcp__spec-lifecycle-activate with spec_id="user-auth-2025-12-04"
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
mcp__foundry-mcp__task-next with spec_id="user-auth-2025-12-04"
```

**Response provides:**
- Task details (description, file path)
- Dependencies status
- Context for implementation

If no task is returned, all available tasks are blocked or completed.

### Step 2: Prepare Task with Full Context

```
mcp__foundry-mcp__task-prepare with
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
mcp__foundry-mcp__task-complete with
  spec_id="user-auth-2025-12-04"
  task_id="task-001"
  notes="Implemented User model with email/password fields"
  journal_entry={
    "entry_type": "completion",
    "content": "Created User model following existing patterns from Product model"
  }
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
mcp__foundry-mcp__test-run with path="tests/auth/"

# Quick smoke test
mcp__foundry-mcp__test-run-quick with path="tests/"
```

### Fidelity Review

Compare implementation against spec requirements:

```
mcp__foundry-mcp__fidelity-check with
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
mcp__foundry-mcp__verify-check with
  spec_id="user-auth-2025-12-04"
  phase_id="phase-1"
```

### Manual Verification

Some verification requires human review. Track completion:

```
mcp__foundry-mcp__journal-add with
  spec_id="user-auth-2025-12-04"
  task_id="task-manual-review"
  entry={
    "entry_type": "verification",
    "content": "Security review completed by @security-team",
    "details": { "reviewer": "security-team", "approved": true }
  }
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
mcp__foundry-mcp__phase-check-complete with
  spec_id="user-auth-2025-12-04"
  phase_id="phase-1"
```

**Response shows:**
- Tasks completed vs total
- Blocked tasks
- Missing verifications

### Step 2: Run Phase Verification

```
mcp__foundry-mcp__verify-check with
  spec_id="user-auth-2025-12-04"
  phase_id="phase-1"
```

### Step 3: Journal Phase Completion

```
mcp__foundry-mcp__journal-add with
  spec_id="user-auth-2025-12-04"
  entry={
    "entry_type": "completion",
    "content": "Phase 1 complete: Core auth logic implemented",
    "phase_id": "phase-1"
  }
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
mcp__foundry-mcp__pr-context with spec_id="user-auth-2025-12-04"
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
mcp__foundry-mcp__pr-create with
  spec_id="user-auth-2025-12-04"
  title="feat: Add user authentication system"
  body="[Generated PR description]"
  base_branch="main"
```

### Step 4: Complete the Spec

After PR is merged:

```
mcp__foundry-mcp__spec-lifecycle-complete with
  spec_id="user-auth-2025-12-04"
  completion_notes="Merged in PR #123"
```

---

## Autonomous Mode Workflow

### Overview

Autonomous mode enables continuous task execution without human intervention.

```
┌────────────────┐
│ Configure      │
│ Work Mode      │
└───────┬────────┘
        │
        ▼
┌───────────────────────────────────────────────┐
│              AUTO-EXECUTION LOOP              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐│
│  │Get Next  │───►│Implement │───►│Complete  ││
│  │Task      │    │          │    │& Journal ││
│  └──────────┘    └──────────┘    └──────────┘│
│        ▲                              │       │
│        └──────────────────────────────┘       │
└───────────────────────────────────────────────┘
        │
        ▼ (exit conditions met)
┌────────────────┐
│ Report Status  │
└────────────────┘
```

### Step 1: Configure Work Mode

```
mcp__foundry-mcp__session-work-mode with mode="autonomous"
```

### Step 2: Generate Session Marker

```
mcp__foundry-mcp__session-generate-marker
```

This creates a marker for tracking the session.

### Step 3: Check Context Usage

Monitor context consumption:

```
mcp__foundry-mcp__session-context
```

**Response shows:**
- Context percentage used
- Tokens consumed
- Remaining capacity

### Step 4: Auto-Execution Loop

The autonomous workflow:
1. Calls `task-next` to find work
2. Implements the task
3. Calls `task-complete` with journal
4. Checks context (stop at ~85%)
5. Repeats until exit condition

### Exit Conditions

| Condition | Action |
|-----------|--------|
| No more tasks | Report completion |
| Context > 85% | Stop and report progress |
| Blocked task | Report blocker |
| Error | Stop and report error |

### Step 5: Report Status

At exit, summarize:
- Tasks completed this session
- Remaining tasks
- Blockers encountered
- Next steps

---

## Blocker Management

### Recording a Blocker

```
mcp__foundry-mcp__task-block with
  spec_id="user-auth-2025-12-04"
  task_id="task-003"
  reason="Waiting for API endpoint from backend team"
  metadata={"issue_url": "https://github.com/org/repo/issues/45"}
```

### Listing Blocked Tasks

```
mcp__foundry-mcp__task-list-blocked with spec_id="user-auth-2025-12-04"
```

### Unblocking a Task

```
mcp__foundry-mcp__task-unblock with
  spec_id="user-auth-2025-12-04"
  task_id="task-003"
  resolution="API endpoint now available, documented in wiki"
```

---

## Decision Journaling

### Recording Decisions

```
mcp__foundry-mcp__journal-add with
  spec_id="user-auth-2025-12-04"
  task_id="task-002"
  entry={
    "entry_type": "decision",
    "content": "Chose bcrypt over argon2 for password hashing",
    "rationale": "Better library support, bcrypt is battle-tested",
    "alternatives_considered": ["argon2", "pbkdf2", "scrypt"]
  }
```

### Viewing Journal

```
mcp__foundry-mcp__journal-list with
  spec_id="user-auth-2025-12-04"
  limit=10
```

### Bulk Journaling

```
mcp__foundry-mcp__journal-bulk-add with
  spec_id="user-auth-2025-12-04"
  entries=[
    { "task_id": "task-001", "entry_type": "note", "content": "..." },
    { "task_id": "task-002", "entry_type": "decision", "content": "..." }
  ]
```

---

## Related Documentation

- **[04-Tool Reference](./04-tool-reference.md)** — Complete tool catalog
- **[08-Integration Patterns](./08-integration-patterns.md)** — How skills use these workflows
- **[03-Spec Lifecycle](./03-spec-lifecycle.md)** — State transitions

---

*[Back to Index](./README.md)*
