# SDD-Plan Reference

Detailed workflows, examples, and edge cases for the sdd-plan skill.

## Table of Contents

- [Phase Plan Template](#phase-plan-template)
- [AI Plan Review](#ai-plan-review)
- [Task Hierarchy](#task-hierarchy)
- [Task Categories](#task-categories)
- [Verification Types](#verification-types)
- [Dependency Tracking](#dependency-tracking)
- [JSON Specification Structure](#json-specification-structure)
- [Codebase Analysis Patterns](#codebase-analysis-patterns)
- [Parallel Investigation Strategies](#parallel-investigation-strategies)
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

## AI Plan Review

AI-powered review of markdown plans before converting them to formal JSON specifications. This enables iterative refinement with AI feedback to catch issues early.

### Tool Usage

```bash
mcp__plugin_foundry_foundry-mcp__plan action="review" plan_path="./PLAN.md" review_type="full"
```

### Review Types

| Type | Focus | Dimensions |
|------|-------|------------|
| `full` | Comprehensive analysis | Completeness, Architecture, Sequencing, Feasibility, Risk, Clarity |
| `quick` | Critical issues only | Critical Blockers, Questions |
| `security` | Security focus | Auth, Data Protection, Vulnerabilities |
| `feasibility` | Technical risks | Complexity, Dependencies, Constraints |

### Review Output Format

Reviews are written to `./tmp/<plan-name>-review.md` with the following structure:

```markdown
# Review Summary

## Critical Blockers
Issues that MUST be fixed before this becomes a spec.

- **[Completeness]** Missing error handling strategy
  - **Description:** No mention of how errors will be surfaced to users
  - **Impact:** Could lead to poor UX and debugging challenges
  - **Fix:** Add "Error Handling" section specifying error types and UI patterns

## Major Suggestions
Significant improvements to strengthen the plan.

- **[Architecture]** Consider caching layer
  - **Description:** API calls may be expensive at scale
  - **Impact:** Performance degradation as user base grows
  - **Fix:** Add caching consideration to Phase 2

## Minor Suggestions
Smaller refinements.

- **[Clarity]** Phase ordering unclear
  - **Description:** Phase 2 and 3 could potentially run in parallel
  - **Fix:** Add note about parallel vs sequential execution

## Questions
Clarifications needed before proceeding.

- **[Feasibility]** What is the expected data volume?
  - **Context:** Affects database design choices
  - **Needed:** Rough estimates for initial launch and growth projections

## Praise
What the plan does well.

- **[Architecture]** Clear separation of concerns
  - **Why:** Each phase has distinct responsibility and clear boundaries
```

### Iteration Pattern

The review workflow supports iteration:

```
Create Plan → Review → Address Feedback → Re-review → Approve → Create Spec
     ↑                      ↓
     └──────────────────────┘
         (repeat until clean)
```

**Example iteration:**

1. Create initial `PLAN.md`
2. Run `plan-review` with type "full"
3. Review identifies 2 critical blockers, 3 major suggestions
4. Update `PLAN.md` addressing blockers
5. Run `plan-review` again
6. Review shows 0 critical blockers, 1 minor suggestion
7. User approves - proceed to JSON spec creation

### Summary Output

The tool returns a structured summary:

```json
{
  "plan_path": "./PLAN.md",
  "plan_name": "PLAN",
  "review_type": "full",
  "review_path": "./tmp/PLAN-review.md",
  "summary": {
    "critical_blockers": 2,
    "major_suggestions": 3,
    "minor_suggestions": 1,
    "questions": 2,
    "praise": 4
  },
  "inline_summary": "2 critical blocker(s), 3 major suggestion(s), 1 minor suggestion(s), 2 question(s), 4 praise item(s)"
}
```

### When to Use Each Review Type

| Scenario | Recommended Type |
|----------|------------------|
| Initial plan review | `full` |
| Quick sanity check after revisions | `quick` |
| Security-sensitive features | `security` |
| Complex or risky technical work | `feasibility` |
| Time pressure (just need blockers) | `quick` |

### Integration with Workflow

1. After creating phase plan (Step 3)
2. User approves high-level structure
3. Run AI review for comprehensive feedback
4. Iterate on plan addressing issues
5. When review shows no critical blockers
6. Proceed to JSON spec creation (Step 4)

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
| `run-tests` | Automated tests via pytest or similar | `mcp__plugin_foundry_foundry-mcp__test action="run"` |
| `fidelity` | Implementation-vs-spec comparison | `mcp__plugin_foundry_foundry-mcp__review action="fidelity"` |

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
    "mcp_tool": "mcp__plugin_foundry_foundry-mcp__test action=\"run\"",
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
- Avoid circular dependencies (use `mcp__plugin_foundry_foundry-mcp__spec action="detect-cycles"` to check)
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
        "mcp_tool": "mcp__plugin_foundry_foundry-mcp__test action=\"run\"",
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
        "mcp_tool": "mcp__plugin_foundry_foundry-mcp__review action=\"fidelity\"",
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
mcp__plugin_foundry_foundry-mcp__code action="doc-stats"
```

If `classes_count > 0` or `functions_count > 0`, use documentation tools:

```bash
# Find existing implementations
mcp__plugin_foundry_foundry-mcp__code action="find-class" symbol="UserService"
mcp__plugin_foundry_foundry-mcp__code action="find-function" symbol="authenticate"

# Trace dependencies
mcp__plugin_foundry_foundry-mcp__code action="trace-calls" symbol="login"

# Analyze impact of changes
mcp__plugin_foundry_foundry-mcp__code action="impact-analysis" symbol="AuthController"
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

**Fallback with Subagents:**

When documentation is unavailable, use the Explore subagent for efficient fallback:

```
Use the Explore agent (medium thoroughness) to:
1. Map the source directory structure
2. Find files matching feature patterns
3. Identify test file conventions
4. Locate configuration files
```

This keeps exploration context isolated and returns focused results for planning.

### Impact Analysis for Refactoring

Before planning a refactor, always run impact analysis:

```bash
mcp__plugin_foundry_foundry-mcp__code action="impact-analysis" symbol="TargetClass" depth=3
```

This identifies:
- Direct impacts (files that import/use the target)
- Indirect impacts (files affected by direct impacts)
- Impact score (how risky the change is)

---

## Parallel Investigation Strategies

For large codebases, leverage built-in subagents to parallelize exploration during the analysis phase.

### Built-in Subagents for Planning

| Subagent | Model | Purpose |
|----------|-------|---------|
| **Explore** | Haiku | Fast read-only codebase search |
| **general-purpose** | Sonnet | Complex multi-step investigation |

### Exploration Patterns by Planning Phase

**Phase 1: Initial Survey (use Explore - quick)**
```
Find all files matching:
- Source directories structure
- Test file locations
- Configuration files
- Documentation
```

**Phase 2: Pattern Discovery (use Explore - medium)**
```
Investigate existing implementations:
- Similar features in codebase
- Established coding patterns
- Error handling approaches
- Testing conventions
```

**Phase 3: Impact Analysis (use Explore - very thorough)**
```
For refactoring/complex changes:
- All files importing affected modules
- Downstream dependencies
- Test files that may need updates
- Documentation requiring changes
```

### Combining MCP Tools with Subagents

| Analysis Need | Primary Tool | Subagent Backup |
|---------------|--------------|-----------------|
| Class/function lookup | `code-find-*` | Explore with Grep |
| Call graph | `code-trace-calls` | Explore (very thorough) |
| Impact scope | `code-impact-analysis` | general-purpose |
| File structure | `doc-stats` | Explore (quick) |

### When to Use Subagents vs Direct Tools

**Use Explore subagent when:**
- Codebase documentation unavailable
- Need to search multiple areas simultaneously
- Want to preserve main context for spec creation
- Initial exploration of unfamiliar code

**Use direct tools when:**
- Single targeted query
- Documentation exists and is current
- Simple file read needed
- Near context limit (subagent results still consume context)

### Benefits of Subagent Exploration

| Benefit | Description |
|---------|-------------|
| **Context isolation** | Search results don't bloat main conversation |
| **Speed** | Explore uses Haiku for fast searches |
| **Focus** | Subagent returns only relevant findings |
| **Parallelization** | Multiple Explore agents can run concurrently |

---

## Troubleshooting

### Spec Validation Fails

After creating a spec, validation may fail. This is normal for new specs.

**Solution:**
1. Run `mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="{spec-id}"` to see errors
2. Run `mcp__plugin_foundry_foundry-mcp__spec action="validate-fix" spec_id="{spec-id}" auto_fix=true`
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

If `mcp__plugin_foundry_foundry-mcp__spec action="detect-cycles"` finds cycles:

**Solution:**
1. Identify the cycle path (e.g., `task-1-2 -> task-2-3 -> task-1-2`)
2. Determine which dependency is weakest
3. Remove one edge to break the cycle
4. Consider if tasks should be merged or reordered

### Documentation Not Available

If `mcp__plugin_foundry_foundry-mcp__code action="doc-stats"` shows no documentation:

**Solution:**
1. Use `Glob`, `Grep`, and `Read` for analysis
2. Consider generating documentation first:
   ```bash
   mcp__plugin_foundry_foundry-mcp__spec action="doc-llm" directory="." use_ai=true
   ```
3. Re-run `code action="doc-stats"` after generation

### User Doesn't Approve Phase Plan

If user requests changes to the high-level plan:

**Solution:**
1. Note their feedback
2. Revise the phase structure
3. Present updated plan
4. Repeat until approved
5. Only then proceed to detailed planning

### AI Plan Review Not Available

If `mcp__plugin_foundry_foundry-mcp__plan action="review"` returns an error about no AI provider:

**Solution:**
1. Check that an AI provider is configured (GEMINI_API_KEY, OPENAI_API_KEY, or ANTHROPIC_API_KEY)
2. Verify the provider is accessible
3. Use `mcp__plugin_foundry_foundry-mcp__provider action="list"` to see available providers
4. **Do not proceed** to JSON spec creation until AI review is working - this is a required step

### Plan Review Returns Many Critical Blockers

If the AI review identifies many critical blockers:

**Solution:**
1. Prioritize blockers by impact (fix showstoppers first)
2. Group related issues that can be addressed together
3. Address completeness issues before architecture issues
4. Re-run review after each major revision
5. Consider splitting the plan if blockers indicate scope is too large

### Review Takes Too Long

If `plan-review` times out:

**Solution:**
1. Use `--ai-timeout` flag with a longer timeout (e.g., 180 seconds)
2. Split large plans into smaller, focused sections
3. Use `quick` review type instead of `full` for faster feedback
4. Check AI provider status for availability issues
