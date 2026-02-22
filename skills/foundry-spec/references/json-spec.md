# JSON Specification Structure

The formal specification format used by all SDD tools.

## Top-Level Structure

```json
{
  "spec_id": "feature-name-001",
  "title": "Feature Name Implementation",
  "description": "One paragraph describing the feature",
  "version": "1.0.0",
  "created": "2025-01-15T10:00:00Z",
  "updated": "2025-01-15T10:00:00Z",
  "metadata": {
    "mission": "Ensure every school admin sees accurate trust signals before launch",
    "status": "pending",
    "priority": "high",
    "tags": ["feature", "auth"],
    "plan_path": ".plans/feature-name.md",
    "plan_review_path": ".plan-reviews/feature-name-review.md",
    "constraints": ["Must maintain backward compatibility with v2 API"],
    "risks": [{"description": "OAuth rate limits", "likelihood": "medium", "impact": "high", "mitigation": "Token caching"}],
    "open_questions": ["Which OAuth scopes for admin flow?"],
    "success_criteria": ["All protected endpoints return 401 without valid token"]
  },
  "phases": { ... },
  "journal": []
}
```

## Metadata Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mission` | string | Recommended | Single-sentence mission describing the spec's purpose |
| `status` | string | Yes | `pending`, `active`, `completed`, `archived` |
| `priority` | string | No | `low`, `medium`, `high`, `critical` |
| `tags` | array | No | Categorization tags |
| `plan_path` | string | Required | Path to the source markdown plan (relative to specs dir) |
| `plan_review_path` | string | Required | Path to the plan review output (relative to specs dir) |
| `spec_review_path` | string | No | Auto-populated path to the spec review output (`.spec-reviews/{spec_id}-spec-review.md`) |
| `constraints` | array of strings | No | Technical/business constraints from the plan |
| `risks` | array of objects | No | Risk entries: `{description, likelihood, impact, mitigation}` |
| `open_questions` | array of strings | No | Unresolved questions from planning |
| `success_criteria` | array of strings | No | Measurable success criteria from the plan |

> Mission helps provide context and should be populated before handing off the spec. Use `plan_path` to link the spec to its source plan for traceability and plan-enhanced review. Paths are stored relative to the specs directory (e.g. `.plans/feature.md`). Absolute paths and `specs/`-prefixed paths are automatically normalized at creation time.
 
## Phase Structure


```json
{
  "phases": {
    "phase-1": {
      "title": "Foundation",
      "description": "Set up core infrastructure",
      "order": 1,
      "status": "pending",
      "tasks": { ... }
    },
    "phase-2": {
      "title": "Implementation",
      "description": "Build core functionality",
      "order": 2,
      "status": "pending",
      "depends_on": ["phase-1"],
      "tasks": { ... }
    }
  }
}
```

## Task Structure

```json
{
  "task-1-1": {
    "title": "Create database schema",
    "type": "task",
    "status": "pending",
    "description": "Define tables for user authentication",
    "metadata": {
      "file_path": "src/db/schema.sql",
      "task_category": "implementation",
      "complexity": "medium"
    },
    "instructions": [
      "Create users table with id, email, password_hash",
      "Add sessions table for token storage",
      "Create necessary indexes"
    ],
    "acceptance_criteria": [
      "Schema passes validation",
      "Migrations run without errors"
    ]
  }
}
```

### Task Metadata Fields

| Field | Type | Description |
|-------|------|-------------|
| `file_path` | string | Associated file path (required for `implementation`/`refactoring` tasks) |
| `task_category` | string | `implementation`, `refactoring`, `investigation`, `decision`, `research` |
| `complexity` | string | Task complexity: `low`, `medium`, `high` |

### Required Fields for Implementation Tasks

Best practices for well-structured specs:
- `metadata.mission` at the spec level
- Every `type: "task"` node includes `description`, `acceptance_criteria`, and `metadata.task_category`
- `metadata.file_path` for `implementation` and `refactoring` tasks
- `metadata.complexity` to indicate task difficulty

## Creating a Spec

```bash
# Create from template
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create" name="feature-name" template="empty"

# Templates: empty (blank spec with no phases - use phase templates to add structure)
```
