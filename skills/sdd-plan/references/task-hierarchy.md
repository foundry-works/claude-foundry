# Task Hierarchy

The specification uses a hierarchical task structure for organization.

## Size Guidelines

| Complexity | Phases | Tasks | Verification Coverage |
|------------|--------|-------|----------------------|
| Short (<5 files) | 1-2 | 3-8 | 20% minimum |
| Medium (5-15 files) | 2-4 | 10-25 | 30-40% |
| Large (>15 files) | 4-6 | 25-50 | 40-50% |

**Splitting recommendation:** If >6 phases or >50 tasks, split into multiple specs.

## Hierarchy Levels

```
Spec
└── Phase (phase-1, phase-2, ...)
    └── Task Group (optional: task-group-1-1, ...)
        └── Task (task-1-1, task-1-2, ...)
            └── Subtask (optional: subtask-1-1-1, ...)
            └── Verify (verify-1-1, ...)
```

## Node Types

| Type | ID Pattern | Purpose |
|------|------------|---------|
| `phase` | `phase-{n}` | Major milestone grouping |
| `task-group` | `task-group-{phase}-{n}` | Optional task grouping |
| `task` | `task-{phase}-{n}` | Actionable work item |
| `subtask` | `subtask-{phase}-{task}-{n}` | Task breakdown |
| `verify` | `verify-{phase}-{n}` | Verification step |

## Task Categories

| Category | Description | Requires file_path |
|----------|-------------|-------------------|
| `investigation` | Research/exploration | No |
| `implementation` | New code creation | Yes |
| `refactoring` | Code restructuring | Yes |
| `decision` | Architecture/design decisions | No |
| `research` | External research/learning | No |

## Required Task Fields (Complex)

For `template="complex"` and `template="security"` specs, every `type: "task"` must include:
- `description`
- `acceptance_criteria` (array with at least one item)
- `metadata.task_category`
- `metadata.file_path` when `task_category` is `implementation` or `refactoring`

## Verification Types

| Type | Skill Used | Purpose |
|------|------------|---------|
| `run-tests` | `Skill(foundry:run-tests)` | Execute test suite |
| `fidelity` | `Skill(foundry:sdd-fidelity-review)` | Compare implementation to spec |
| `manual` | Human verification | Manual testing checklist |

## Verification Task Structure

```json
{
  "verify-1-1": {
    "title": "Verify Phase 1 Implementation",
    "type": "verify",
    "status": "pending",
    "metadata": {
      "verification_type": "fidelity",
      "scope": "phase",
      "target": "phase-1"
    }
  }
}
```

## Dependencies

### Hard Dependencies (blocks)

```json
{
  "task-2-1": {
    "depends_on": ["task-1-1", "task-1-2"],
    "metadata": {
      "blocks": ["task-2-2", "task-2-3"]
    }
  }
}
```

### Soft Dependencies

```json
{
  "task-2-1": {
    "metadata": {
      "soft_depends_on": ["task-1-3"],
      "notes": "Can start without 1-3 but may need revision"
    }
  }
}
```

## Dependency Rules

1. **No circular dependencies** - Validation will fail
2. **Cross-phase deps allowed** - But prefer phase ordering
3. **Verify tasks depend on implementation** - Implicit dependency on phase tasks
4. **Blocked tasks** - Cannot start until blocker resolved
