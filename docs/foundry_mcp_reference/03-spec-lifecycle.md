# Spec Lifecycle

> Understanding spec states, transitions, and the JSON schema structure.

---

## Spec States Overview

Specifications progress through four states during their lifecycle:

```
┌─────────┐    ┌─────────┐    ┌───────────┐    ┌──────────┐
│ pending │───►│ active  │───►│ completed │───►│ archived │
└─────────┘    └─────────┘    └───────────┘    └──────────┘
```

| State | Description | Location |
|-------|-------------|----------|
| **pending** | Planning phase; not yet ready for implementation | `specs/pending/` |
| **active** | Implementation in progress | `specs/active/` |
| **completed** | All tasks done; verification passed | `specs/completed/` |
| **archived** | Historical reference | `specs/archived/` |

---

## State Transition Diagram

```
                    ┌───────────────────────┐
                    │                       │
                    ▼                       │
              ┌─────────┐                   │
              │ pending │                   │
              └────┬────┘                   │
                   │                        │
        spec-lifecycle-activate             │
                   │                        │
                   ▼                        │
              ┌─────────┐                   │
              │ active  │◄──────────────────┘
              └────┬────┘       (reactivate if needed)
                   │
        spec-lifecycle-complete
                   │
                   ▼
            ┌───────────┐
            │ completed │
            └─────┬─────┘
                  │
       spec-lifecycle-archive
                  │
                  ▼
            ┌──────────┐
            │ archived │
            └──────────┘
```

---

## Lifecycle Operations

### Activating a Spec

Move from `pending` to `active` when planning is complete:

```
mcp__foundry-mcp__spec-lifecycle-activate with spec_id="my-feature-2025-12-04"
```

**Prerequisites:**
- Spec MUST pass validation (`spec-validate`)
- All required fields MUST be populated
- At least one phase with tasks SHOULD exist

**Effects:**
- Spec moves from `specs/pending/` to `specs/active/`
- `activated_at` timestamp is set
- Tasks become actionable for `task-next`

### Completing a Spec

Move from `active` to `completed` when all work is done:

```
mcp__foundry-mcp__spec-lifecycle-complete with spec_id="my-feature-2025-12-04"
  completion_notes="Feature implemented and verified"
```

**Prerequisites:**
- All tasks SHOULD be in `completed` status
- Verification steps SHOULD pass
- Journal entries SHOULD document key decisions

**Effects:**
- Spec moves from `specs/active/` to `specs/completed/`
- `completed_at` timestamp is set
- Spec remains available for reference

### Archiving a Spec

Move from `completed` to `archived` for long-term storage:

```
mcp__foundry-mcp__spec-lifecycle-archive with spec_id="my-feature-2025-12-04"
```

**Effects:**
- Spec moves from `specs/completed/` to `specs/archived/`
- `archived_at` timestamp is set
- Spec preserved for historical reference

### Checking Current State

```
mcp__foundry-mcp__spec-lifecycle-state with spec_id="my-feature-2025-12-04"
```

**Returns:**
```json
{
  "success": true,
  "data": {
    "spec_id": "my-feature-2025-12-04",
    "state": "active",
    "folder": "specs/active/",
    "activated_at": "2025-12-04T10:00:00Z",
    "completed_at": null,
    "archived_at": null
  }
}
```

---

## Spec Directory Structure

```
specs/
├── pending/           # New specs awaiting activation
│   └── my-feature-2025-12-04.json
│
├── active/            # Currently being worked on
│   └── auth-system-2025-12-01.json
│
├── completed/         # Finished specs with journals
│   └── user-api-2025-11-15.json
│
└── archived/          # Historical reference
    └── legacy-migration-2025-10-01.json
```

### Directory Conventions

| Requirement | Details |
|-------------|---------|
| **MUST** | Each spec is a single JSON file |
| **MUST** | Filename matches `spec_id` |
| **SHOULD** | Include date in `spec_id` for uniqueness |
| **SHOULD** | Use kebab-case for `spec_id` |

---

## Spec JSON Schema Overview

### Top-Level Structure

```json
{
  "spec_id": "my-feature-2025-12-04",
  "title": "My Feature Implementation",
  "description": "Implement the user authentication feature",
  "status": "active",
  "created_at": "2025-12-04T09:00:00Z",
  "activated_at": "2025-12-04T10:00:00Z",
  "completed_at": null,
  "archived_at": null,
  "metadata": { ... },
  "hierarchy": { ... },
  "journal": [ ... ]
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `spec_id` | string | Unique identifier |
| `title` | string | Human-readable title |
| `status` | string | Current state |
| `hierarchy` | object | Phases, tasks, subtasks |

### Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Detailed description |
| `metadata` | object | Custom metadata |
| `journal` | array | Decision journal entries |
| `assumptions` | array | Planning assumptions |
| `constraints` | array | Implementation constraints |

### Hierarchy Structure

The `hierarchy` contains phases, which contain tasks:

```json
{
  "hierarchy": {
    "phases": [
      {
        "phase_id": "phase-1",
        "name": "Core Implementation",
        "description": "Implement core authentication logic",
        "order": 1,
        "status": "in_progress",
        "tasks": [
          {
            "task_id": "task-001",
            "description": "Create User model",
            "status": "completed",
            "file_path": "src/models/user.py",
            "depends_on": [],
            "subtasks": [],
            "verification": {
              "type": "auto",
              "command": "pytest tests/models/test_user.py"
            }
          },
          {
            "task_id": "task-002",
            "description": "Implement JWT token generation",
            "status": "pending",
            "file_path": "src/auth/tokens.py",
            "depends_on": ["task-001"],
            "subtasks": []
          }
        ],
        "verification": {
          "type": "fidelity",
          "criteria": ["All auth endpoints respond correctly"]
        }
      }
    ]
  }
}
```

### Task Status Values

| Status | Description |
|--------|-------------|
| `pending` | Not yet started |
| `in_progress` | Currently being worked on |
| `completed` | Finished and verified |
| `blocked` | Cannot proceed due to blocker |
| `skipped` | Intentionally not implemented |

### Dependencies

Tasks can declare dependencies on other tasks:

```json
{
  "task_id": "task-003",
  "depends_on": ["task-001", "task-002"],
  "blocked_by": ["task-external-approval"]
}
```

- `depends_on` — Tasks that must complete first
- `blocked_by` — External blockers (issues, approvals)

### Verification Types

| Type | Description | Example |
|------|-------------|---------|
| `auto` | Automated command | `pytest tests/` |
| `fidelity` | AI comparison to spec | Spec vs implementation review |
| `manual` | Human checklist | Security audit items |

```json
{
  "verification": {
    "type": "auto",
    "command": "pytest tests/auth/ -v",
    "expected_outcome": "All tests pass"
  }
}
```

---

## Journal Structure

Journals track decisions, blockers, and notes:

```json
{
  "journal": [
    {
      "entry_id": "journal-001",
      "timestamp": "2025-12-04T11:30:00Z",
      "entry_type": "decision",
      "task_id": "task-002",
      "content": "Chose JWT over session tokens",
      "rationale": "Better for stateless API architecture",
      "author": "claude"
    },
    {
      "entry_id": "journal-002",
      "timestamp": "2025-12-04T14:00:00Z",
      "entry_type": "blocker",
      "task_id": "task-003",
      "content": "Waiting for security team approval",
      "resolution": null
    }
  ]
}
```

### Journal Entry Types

| Type | Purpose |
|------|---------|
| `decision` | Record architectural/design choices |
| `blocker` | Track impediments |
| `note` | General observations |
| `completion` | Task completion notes |
| `verification` | Verification results |

---

## Validation Requirements

### Structural Validation

Specs are validated against the JSON schema:

```
mcp__foundry-mcp__spec-validate with spec_id="my-feature"
```

Checks include:
- Required fields present
- Valid status values
- Valid task status values
- Unique task IDs within spec
- Valid dependency references

### Integrity Validation

Beyond structure, logical integrity is checked:

| Check | Description |
|-------|-------------|
| **No circular deps** | Tasks cannot depend on themselves |
| **Valid references** | `depends_on` references existing tasks |
| **Status consistency** | Completed phases have all tasks completed |
| **File paths valid** | Referenced files exist (optional) |

### Auto-Fix

Common issues can be auto-fixed:

```
mcp__foundry-mcp__spec-fix with spec_id="my-feature"
```

Auto-fixable issues:
- Missing timestamps (set to now)
- Invalid status values (reset to `pending`)
- Orphaned dependencies (remove invalid refs)

---

## Spec ID Conventions

### Recommended Format

```
{feature-name}-{YYYY-MM-DD}[-{sequence}]
```

**Examples:**
- `user-authentication-2025-12-04`
- `api-v2-migration-2025-12-04-001`
- `bugfix-login-timeout-2025-12-04`

### Best Practices

| Practice | Rationale |
|----------|-----------|
| Include date | Ensures uniqueness, provides context |
| Use kebab-case | Filesystem-safe, readable |
| Be descriptive | Easier to find and reference |
| Keep reasonable length | Under 50 characters |

---

## Working with Specs

### Creating a New Spec

Use the `sdd-plan` skill or directly:

```
mcp__foundry-mcp__spec-create with
  spec_id="my-feature-2025-12-04"
  title="My Feature"
  description="Implement my feature"
  phases=[...]
```

### Querying Specs

```
# List all active specs
mcp__foundry-mcp__spec-list with status="active"

# Find specs matching criteria
mcp__foundry-mcp__spec-find with query="authentication"

# Get full spec hierarchy
mcp__foundry-mcp__spec-get-hierarchy with spec_id="my-feature"
```

### Rendering Specs

```
# Render as markdown
mcp__foundry-mcp__spec-render with spec_id="my-feature"

# Render progress summary
mcp__foundry-mcp__spec-render-progress with spec_id="my-feature"
```

---

## Related Documentation

- **[02-SDD Philosophy](./02-sdd-philosophy.md)** — Why SDD works
- **[04-Tool Reference](./04-tool-reference.md)** — Lifecycle tools
- **[06-Workflows](./06-workflows.md)** — Practical patterns

---

*[Back to Index](./README.md)*
