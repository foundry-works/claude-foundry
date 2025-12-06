# SDD-Plan Reference

Detailed workflows, examples, and edge cases for the sdd-plan skill.

## Table of Contents

- [Phase Plan Template](#phase-plan-template)
- [Task Hierarchy](#task-hierarchy)
- [Task Categories](#task-categories)
- [Verification Types](#verification-types)
- [Dependency Tracking](#dependency-tracking)
- [JSON Specification Structure](#json-specification-structure)
- [Codebase Analysis Patterns](#codebase-analysis-patterns)
- [Troubleshooting](#troubleshooting)

---

## Phase Plan Template

For complex features (3+ phases expected), create a phase-only markdown plan for user approval before detailed planning:

```markdown
# High-Level Plan: [Feature Name]

## Overview
Brief description of what this change accomplishes.

## Proposed Phases

### Phase 1: [Phase Name]
**Purpose**: What this phase accomplishes
**Risk Level**: Low/Medium/High
**Key Deliverables**: List main outputs
**Estimated Files Affected**: N files

### Phase 2: [Phase Name]
**Purpose**: What this phase accomplishes
**Risk Level**: Low/Medium/High
**Key Deliverables**: List main outputs
**Estimated Files Affected**: N files

[Repeat for each phase]

## Implementation Order
1. Phase X (must complete first)
2. Phase Y (depends on X)
3. Phase Z (can run in parallel with Y)

## Open Questions
- Any decisions that need user input
- Architectural choices to confirm
```

**Critical:** Present this to the user and get explicit approval before proceeding to detailed planning.

---

## Task Hierarchy

### Node Types

| Type | ID Pattern | Purpose |
|------|------------|---------|
| **spec** | `spec-root` | Root node for the entire specification |
| **phase** | `phase-N` | Major implementation stage |
| **task** | `task-N-M` | Individual file modification |
| **subtask** | `task-N-M-P` | Specific change within file |
| **verify** | `verify-N-M` | Verification step |

### Hierarchy Structure

```
spec-root
├── phase-1
│   ├── task-1-1
│   │   ├── task-1-1-1 (subtask)
│   │   └── task-1-1-2 (subtask)
│   ├── task-1-2
│   ├── verify-1-1
│   └── verify-1-2
├── phase-2
│   └── ...
```

### Node Properties

Each node contains:
- `type`: Node type (spec, phase, task, subtask, verify)
- `title`: Human-readable name
- `status`: pending, in_progress, completed, blocked
- `parent`: Parent node ID
- `children`: Array of child node IDs
- `total_tasks`: Count of tasks in subtree
- `completed_tasks`: Count of completed tasks
- `metadata`: Additional properties (file_path, details, etc.)
- `dependencies`: Dependency tracking (blocks, blocked_by, depends)

---

## Task Categories

Assign categories to tasks based on their purpose:

| Category | Use When |
|----------|----------|
| `investigation` | Analyzing existing code, understanding patterns |
| `implementation` | Writing new functionality |
| `refactoring` | Improving code structure without changing behavior |
| `decision` | Architectural choices requiring user input |
| `research` | Gathering external information (APIs, libraries) |

**Example:**
```json
{
  "type": "task",
  "title": "Analyze authentication flow",
  "metadata": {
    "category": "investigation",
    "details": "Analyze the existing authentication flow",
    "file_path": "src/auth/login.ts"
  }
}
```

---

## Verification Types

Each verification step should specify its type:

| Type | Description | MCP Tool |
|------|-------------|----------|
| `run-tests` | Automated tests via pytest or similar | `mcp__foundry-mcp__test-run` |
| `fidelity` | Implementation-vs-spec comparison | `mcp__foundry-mcp__spec-review-fidelity` |

**Example verification node:**
```json
{
  "type": "verify",
  "title": "Run tests",
  "status": "pending",
  "parent": "phase-1",
  "children": [],
  "metadata": {
    "verification_type": "run-tests",
    "mcp_tool": "mcp__foundry-mcp__test-run",
    "expected": "All tests pass"
  },
  "dependencies": {
    "blocks": [],
    "blocked_by": [],
    "depends": []
  }
}
```

---

## Dependency Tracking

Dependencies are stored at the node level in a `dependencies` object, NOT in metadata.

### Hard Dependencies (`blocked_by`)

Task cannot start until dependency completes. Use for sequential requirements.

```json
{
  "type": "task",
  "title": "Add API endpoint",
  "dependencies": {
    "blocks": [],
    "blocked_by": ["task-1-2"],
    "depends": []
  }
}
```

### Soft Dependencies (`depends`)

Recommended order but not strictly required. Use when tasks benefit from ordering but can proceed independently.

```json
{
  "type": "task",
  "title": "Update documentation",
  "dependencies": {
    "blocks": [],
    "blocked_by": [],
    "depends": ["task-2-1"]
  }
}
```

### Blocks (`blocks`)

Marks what this task prevents from starting. Inverse of `blocked_by`.

```json
{
  "type": "task",
  "title": "Create database schema",
  "dependencies": {
    "blocks": ["task-1-3", "task-1-4"],
    "blocked_by": [],
    "depends": []
  }
}
```

### Dependency Best Practices

- Prefer `blocked_by` over `blocks` for clarity
- Keep dependency chains short (3-4 hops max)
- Avoid circular dependencies (use `mcp__foundry-mcp__spec-detect-cycles` to check)
- Group independent tasks in parallel branches

---

## JSON Specification Structure

Full specification structure:

```json
{
  "spec_id": "feature-YYYY-MM-DD-001",
  "title": "Feature Name",
  "generated": "2025-01-15T10:30:00Z",
  "last_updated": "2025-01-15T10:30:00Z",
  "status": "pending",
  "progress_percentage": 0,
  "current_phase": "phase-1",
  "metadata": {
    "description": "",
    "objectives": [],
    "complexity": "medium",
    "estimated_hours": 10,
    "category": "implementation"
  },
  "hierarchy": {
    "spec-root": {
      "type": "spec",
      "title": "Feature Name",
      "status": "pending",
      "parent": null,
      "children": ["phase-1"],
      "total_tasks": 3,
      "completed_tasks": 0,
      "metadata": {
        "purpose": "Brief description of the feature",
        "category": "implementation"
      },
      "dependencies": {
        "blocks": [],
        "blocked_by": [],
        "depends": []
      }
    },
    "phase-1": {
      "type": "phase",
      "title": "Core Implementation",
      "status": "pending",
      "parent": "spec-root",
      "children": ["task-1-1", "verify-1-1", "verify-1-2"],
      "total_tasks": 3,
      "completed_tasks": 0,
      "metadata": {
        "purpose": "Core implementation work",
        "estimated_hours": 8
      },
      "dependencies": {
        "blocks": [],
        "blocked_by": [],
        "depends": []
      }
    },
    "task-1-1": {
      "type": "task",
      "title": "Create user model",
      "status": "pending",
      "parent": "phase-1",
      "children": [],
      "total_tasks": 1,
      "completed_tasks": 0,
      "metadata": {
        "details": "Implement the user model with validation",
        "category": "implementation",
        "estimated_hours": 2,
        "file_path": "src/models/user.ts"
      },
      "dependencies": {
        "blocks": [],
        "blocked_by": [],
        "depends": []
      }
    },
    "verify-1-1": {
      "type": "verify",
      "title": "Run tests",
      "status": "pending",
      "parent": "phase-1",
      "children": [],
      "total_tasks": 1,
      "completed_tasks": 0,
      "metadata": {
        "verification_type": "run-tests",
        "mcp_tool": "mcp__foundry-mcp__test-run",
        "expected": "All tests pass"
      },
      "dependencies": {
        "blocks": ["verify-1-2"],
        "blocked_by": [],
        "depends": []
      }
    },
    "verify-1-2": {
      "type": "verify",
      "title": "Fidelity review",
      "status": "pending",
      "parent": "phase-1",
      "children": [],
      "total_tasks": 1,
      "completed_tasks": 0,
      "metadata": {
        "verification_type": "fidelity",
        "mcp_tool": "mcp__foundry-mcp__spec-review-fidelity",
        "scope": "phase",
        "target": "phase-1",
        "expected": "Implementation matches specification"
      },
      "dependencies": {
        "blocks": [],
        "blocked_by": ["verify-1-1"],
        "depends": []
      }
    }
  },
  "journal": []
}
```

### Required Fields

- `spec_id`: Unique identifier (format: `name-YYYY-MM-DD-NNN`)
- `generated`: ISO 8601 timestamp
- `last_updated`: ISO 8601 timestamp
- `hierarchy`: Object containing all nodes keyed by ID

### Spec ID Naming Convention

```
{feature-name}-{YYYY-MM-DD}-{sequence}
```

Examples:
- `user-auth-2025-01-15-001`
- `api-refactor-2025-01-15-002`
- `dark-mode-2025-01-15-001`

---

## Codebase Analysis Patterns

### When Documentation Exists

Always check first:
```bash
mcp__foundry-mcp__doc-stats
```

If `classes_count > 0` or `functions_count > 0`, use documentation tools:

```bash
# Find existing implementations
mcp__foundry-mcp__code-find-class name="UserService"
mcp__foundry-mcp__code-find-function name="authenticate"

# Trace dependencies
mcp__foundry-mcp__code-trace-calls function_name="login" direction="both"

# Analyze impact of changes
mcp__foundry-mcp__code-impact-analysis target="AuthController"
```

### When Documentation Unavailable

Fall back to file operations:

```bash
# Find files by pattern
Glob pattern="**/auth/**/*.ts"

# Search for code patterns
Grep pattern="class.*Service" type="ts"

# Read specific files
Read file_path="src/services/auth.ts"
```

### Impact Analysis for Refactoring

Before planning a refactor, always run impact analysis:

```bash
mcp__foundry-mcp__code-impact-analysis target="TargetClass" max_depth=3
```

This identifies:
- Direct impacts (files that import/use the target)
- Indirect impacts (files affected by direct impacts)
- Impact score (how risky the change is)

---

## Troubleshooting

### Spec Validation Fails

After creating a spec, validation may fail. This is normal for new specs.

**Solution:**
1. Run `mcp__foundry-mcp__spec-validate spec_id="{spec-id}"` to see errors
2. Run `mcp__foundry-mcp__spec-validate-fix spec_id="{spec-id}" auto_fix=true`
3. Re-validate until error count stops decreasing
4. For remaining errors, see `Skill(foundry:sdd-validate)` troubleshooting

### Spec Too Large

If a spec exceeds 6 phases or 50 tasks:

**Solution:**
1. Split by feature boundary (e.g., "auth" vs "permissions")
2. Split by layer (e.g., "backend" vs "frontend")
3. Split by phase (extract later phases into separate spec)
4. Create parent spec that references child specs

### Circular Dependencies Detected

If `mcp__foundry-mcp__spec-detect-cycles` finds cycles:

**Solution:**
1. Identify the cycle path (e.g., `task-1-2 -> task-2-3 -> task-1-2`)
2. Determine which dependency is weakest
3. Remove one edge to break the cycle
4. Consider if tasks should be merged or reordered

### Documentation Not Available

If `mcp__foundry-mcp__doc-stats` shows no documentation:

**Solution:**
1. Use `Glob`, `Grep`, and `Read` for analysis
2. Consider generating documentation first:
   ```bash
   mcp__foundry-mcp__spec-doc-llm directory="." use_ai=true
   ```
3. Re-run `doc-stats` after generation

### User Doesn't Approve Phase Plan

If user requests changes to the high-level plan:

**Solution:**
1. Note their feedback
2. Revise the phase structure
3. Present updated plan
4. Repeat until approved
5. Only then proceed to detailed planning
