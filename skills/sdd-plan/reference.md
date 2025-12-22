# SDD-Plan Reference

Detailed workflows, templates, and specifications for the sdd-plan skill.

## Table of Contents

- [Parallel Investigation Strategies](#parallel-investigation-strategies)
- [Phase Plan Template](#phase-plan-template)
- [AI Plan Review](#ai-plan-review)
- [JSON Specification Structure](#json-specification-structure)
- [Task Hierarchy](#task-hierarchy)
- [Codebase Analysis Patterns](#codebase-analysis-patterns)
- [Troubleshooting](#troubleshooting)

---

## Parallel Investigation Strategies

Use Claude Code's built-in subagents to efficiently explore codebases without bloating main context.

### When to Use Parallel Exploration

| Scenario | Agents | Thoroughness |
|----------|--------|--------------|
| Single feature area | 1 | medium |
| Cross-cutting concerns | 2-3 | medium each |
| Full architecture review | 2-3 | very thorough |
| Quick file discovery | 1 | quick |

### Parallel Exploration Pattern

Launch multiple Explore agents in a single message for independent investigations:

```
Agent 1: Use the Explore agent (medium thoroughness) to find:
- All files in the authentication module
- Existing auth patterns and middleware
- Related test files

Agent 2: Use the Explore agent (medium thoroughness) to find:
- Database models and schemas
- Migration patterns
- ORM usage examples

Agent 3: Use the Explore agent (quick thoroughness) to find:
- Configuration files
- Environment variable usage
- Deployment scripts
```

### Investigation Focus Areas

| Focus | What to Look For |
|-------|------------------|
| **Patterns** | Existing implementations of similar features |
| **Structure** | Directory organization, naming conventions |
| **Dependencies** | Import chains, shared utilities |
| **Testing** | Test patterns, fixtures, mocks |
| **Config** | Environment handling, feature flags |

### Benefits of Parallel Investigation

1. **Context isolation** - Each agent uses separate context
2. **Speed** - Haiku model processes quickly
3. **Thoroughness** - Multiple perspectives on codebase
4. **Main context preserved** - Results summarized, not raw file contents

### Anti-Patterns

- **Too many agents** - Maximum 3 per round
- **Overlapping scope** - Each agent should have distinct focus
- **Sequential when parallel possible** - Launch independent searches together
- **Exploring known code** - Use direct Read for files you've already identified

---

## Phase Plan Template

High-level markdown plan created before JSON specification.

### Template Structure

```markdown
# {Feature Name} Implementation Plan

## Objective
{One paragraph describing the core goal}

## Success Criteria
- [ ] {Measurable criterion 1}
- [ ] {Measurable criterion 2}
- [ ] {Measurable criterion 3}

## Constraints
- {Technical constraint}
- {Business constraint}
- {Timeline/resource constraint}

## Phases

### Phase 1: {Phase Name}
**Goal:** {What this phase accomplishes}
**Files:** {Key files affected}
**Tasks:**
- {Task 1 description}
- {Task 2 description}
**Verification:** {How to verify phase completion}

### Phase 2: {Phase Name}
**Goal:** {What this phase accomplishes}
**Files:** {Key files affected}
**Tasks:**
- {Task 1 description}
- {Task 2 description}
**Verification:** {How to verify phase completion}

## Risks
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| {Risk 1} | {L/M/H} | {L/M/H} | {Strategy} |

## Dependencies
- {External dependency 1}
- {Internal dependency 1}

## Open Questions
- {Question needing resolution before implementation}
```

### Creating the Plan

```bash
# Create plan with MCP tool (creates template)
mcp__plugin_foundry_foundry-mcp__plan action="create" name="Feature Name" template="detailed"

# Plan created at: specs/.plans/feature-name.md
```

After creation, read the template and fill in all placeholders with actual content.

### Phase Naming Conventions

| Phase Type | Example Names |
|------------|---------------|
| Setup | "Foundation", "Infrastructure Setup", "Core Types" |
| Implementation | "Core Implementation", "Feature Development", "API Layer" |
| Integration | "Integration", "Wiring", "Connection Layer" |
| Testing | "Testing & Validation", "Quality Assurance" |
| Polish | "Documentation", "Cleanup", "Optimization" |

### Good vs Bad Phase Plans

**Good:**
```markdown
### Phase 1: Authentication Core
**Goal:** Implement JWT token generation and validation
**Files:** src/auth/jwt.ts, src/auth/middleware.ts
**Tasks:**
- Create JWT utility with RS256 signing
- Implement auth middleware for protected routes
- Add token refresh endpoint
**Verification:** Unit tests pass, manual auth flow works
```

**Bad:**
```markdown
### Phase 1: Auth
**Goal:** Do auth stuff
**Tasks:**
- Make it work
**Verification:** It works
```

---

## AI Plan Review

Get AI-powered feedback on markdown plans before converting to JSON specs.

### Running a Review

```bash
# Full comprehensive review
mcp__plugin_foundry_foundry-mcp__plan action="review" plan_path="specs/.plans/feature-name.md" review_type="full"

# Quick review for blockers only
mcp__plugin_foundry_foundry-mcp__plan action="review" plan_path="specs/.plans/feature-name.md" review_type="quick"
```

### Review Types

| Type | Dimensions | Use When |
|------|------------|----------|
| `full` | Completeness, Architecture, Sequencing, Feasibility, Risk, Clarity | Initial review, major changes |
| `quick` | Critical blockers, Key questions | Iteration, minor updates |
| `security` | Auth, Input validation, Data handling, Secrets | Security-sensitive features |
| `feasibility` | Complexity, Dependencies, Unknown risks | Novel or risky implementations |

### Review Output Location

Reviews are saved to: `specs/.plan-reviews/<plan-name>-<review-type>.md`

### Review Output Structure

```markdown
# Plan Review: {Plan Name}

## Summary
{Overall assessment}

## Dimensions

### Completeness
**Score:** {1-5}
**Findings:**
- {Finding 1}
- {Finding 2}
**Recommendations:**
- {Recommendation 1}

### Architecture
**Score:** {1-5}
**Findings:**
- {Finding 1}
**Recommendations:**
- {Recommendation 1}

[... additional dimensions ...]

## Critical Blockers
1. {Blocker requiring resolution before proceeding}

## Questions for Stakeholder
1. {Question needing clarification}

## Verdict
{APPROVED | NEEDS_REVISION | BLOCKED}
```

### Iteration Workflow

```
1. Create plan → plan action="create"
2. Fill in content → Read + Edit
3. Run review → plan action="review"
4. Read feedback → specs/.plan-reviews/
5. Revise plan → Edit based on feedback
6. Re-review → plan action="review"
7. Repeat until APPROVED
8. Convert to spec → authoring action="spec-create"
```

### Common Review Feedback

| Issue | Typical Recommendation |
|-------|------------------------|
| Vague tasks | Add specific acceptance criteria |
| Missing dependencies | Explicit task ordering needed |
| No verification | Add test requirements |
| Scope creep | Split into multiple specs |
| Missing risks | Add risk assessment section |

---

## JSON Specification Structure

The formal specification format used by all SDD tools.

### Top-Level Structure

```json
{
  "spec_id": "feature-name-001",
  "title": "Feature Name Implementation",
  "description": "One paragraph describing the feature",
  "version": "1.0.0",
  "created": "2025-01-15T10:00:00Z",
  "updated": "2025-01-15T10:00:00Z",
  "metadata": {
    "status": "pending",
    "priority": "high",
    "owner": "developer@example.com",
    "tags": ["feature", "auth"],
    "estimated_hours": 40
  },
  "phases": { ... },
  "journal": []
}
```

### Metadata Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | string | Yes | `pending`, `active`, `completed`, `archived` |
| `priority` | string | No | `low`, `medium`, `high`, `critical` |
| `owner` | string | No | Email or identifier of owner |
| `tags` | array | No | Categorization tags |
| `estimated_hours` | number | No | Total estimated hours |

### Phase Structure

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

### Task Structure

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
      "estimated_hours": 2
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

### Getting the Schema

```bash
# Export full JSON schema for reference
mcp__plugin_foundry_foundry-mcp__spec action="schema-export"
```

### Creating a Spec

```bash
# Create from template
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create" name="feature-name" template="medium"

# Templates: minimal, small, medium, large
```

---

## Task Hierarchy

The specification uses a hierarchical task structure for organization.

### Hierarchy Levels

```
Spec
└── Phase (phase-1, phase-2, ...)
    └── Task Group (optional: task-group-1-1, ...)
        └── Task (task-1-1, task-1-2, ...)
            └── Subtask (optional: subtask-1-1-1, ...)
            └── Verify (verify-1-1, ...)
```

### Node Types

| Type | ID Pattern | Purpose |
|------|------------|---------|
| `phase` | `phase-{n}` | Major milestone grouping |
| `task-group` | `task-group-{phase}-{n}` | Optional task grouping |
| `task` | `task-{phase}-{n}` | Actionable work item |
| `subtask` | `subtask-{phase}-{task}-{n}` | Task breakdown |
| `verify` | `verify-{phase}-{n}` | Verification step |

### Task Categories

| Category | Description | Requires file_path |
|----------|-------------|-------------------|
| `implementation` | New code creation | Yes |
| `refactoring` | Code restructuring | Yes |
| `investigation` | Research/exploration | No |
| `configuration` | Config changes | Optional |
| `documentation` | Doc updates | Optional |
| `testing` | Test creation | Yes |

### Verification Types

| Type | Skill Used | Purpose |
|------|------------|---------|
| `run-tests` | `Skill(foundry:run-tests)` | Execute test suite |
| `fidelity` | `Skill(foundry:sdd-fidelity-review)` | Compare implementation to spec |
| `manual` | Human verification | Manual testing checklist |

### Verification Task Structure

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

### Dependencies

#### Hard Dependencies (blocks)

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

#### Soft Dependencies

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

### Dependency Rules

1. **No circular dependencies** - Validation will fail
2. **Cross-phase deps allowed** - But prefer phase ordering
3. **Verify tasks depend on implementation** - Implicit dependency on phase tasks
4. **Blocked tasks** - Cannot start until blocker resolved

---

## Codebase Analysis Patterns

Strategies for understanding existing code before planning.

### MCP Code Router

```bash
# Find a class definition
mcp__plugin_foundry_foundry-mcp__code action="find-class" name="AuthService" scope="src/"

# Find a function
mcp__plugin_foundry_foundry-mcp__code action="find-function" name="validateToken" scope="src/"

# Trace call hierarchy
mcp__plugin_foundry_foundry-mcp__code action="trace-calls" function="login" scope="src/" depth=3

# Analyze impact of changes
mcp__plugin_foundry_foundry-mcp__code action="impact-analysis" target="AuthService" scope="src/"
```

### LSP Analysis (When Available)

```python
# Find all references to a symbol
references = findReferences(file="src/auth/service.py", symbol="AuthService", line=15, character=6)

# Get file structure
symbols = documentSymbol(file="src/auth/service.py")

# Navigate to definition
definition = goToDefinition(file="src/api/routes.py", symbol="authenticate", line=42, character=10)

# Find implementations of interface
implementations = goToImplementation(file="src/auth/base.py", symbol="AuthProvider", line=8, character=6)
```

### Analysis Decision Tree

```
Need to understand code?
    |
    +-- Know exact file/symbol?
    |       |
    |       +-- Yes → LSP tools (findReferences, documentSymbol)
    |       |
    |       +-- No → Explore agent or code router
    |
    +-- Need call hierarchy?
    |       |
    |       +-- LSP available → incomingCalls/outgoingCalls
    |       |
    |       +-- No LSP → code action="trace-calls"
    |
    +-- Need impact analysis?
            |
            +-- code action="impact-analysis"
            +-- Or: findReferences + manual analysis
```

### Combining Approaches

```
1. Explore agent (quick) → Find relevant files
2. documentSymbol → Understand file structure
3. findReferences → Map dependencies
4. code action="impact-analysis" → Assess change scope
5. Read critical files → Deep understanding
```

---

## Troubleshooting

### Validation Errors

**Error:** "Circular dependency detected"

**Resolution:**
1. Check `depends_on` arrays for cycles
2. Use `spec action="validate"` to identify specific tasks
3. Remove or reorder dependencies

**Error:** "Missing file_path for implementation task"

**Resolution:**
1. Add `metadata.file_path` to task
2. Or change `task_category` to `investigation`

**Error:** "Invalid task ID format"

**Resolution:**
- Follow pattern: `task-{phase}-{n}`, `verify-{phase}-{n}`
- Phase number must match parent phase

### Spec Not Found

**Symptoms:** MCP tools can't find spec

**Checks:**
1. Verify file exists: `ls specs/pending/` or `specs/active/`
2. Check spec_id matches filename (without .json)
3. Ensure valid JSON: `spec action="validate"`

### Plan Review Fails

**Symptoms:** `plan action="review"` returns error

**Checks:**
1. Plan file exists in `specs/.plans/`
2. Markdown is well-formed
3. Required sections present (Objective, Phases)

### LSP Not Available

**Symptoms:** LSP tools return errors

**Fallbacks:**
1. Use `code action="find-class"` or `code action="find-function"`
2. Use Explore agent with specific search terms
3. Use Grep for symbol search: `Grep pattern="class AuthService"`

---

## Quick Reference

### Creation Commands

```bash
# Create markdown plan
mcp__plugin_foundry_foundry-mcp__plan action="create" name="Feature Name" template="detailed"

# Review plan
mcp__plugin_foundry_foundry-mcp__plan action="review" plan_path="specs/.plans/feature-name.md" review_type="full"

# Create JSON spec
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create" name="feature-name" template="medium"

# Validate spec
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="{spec-id}"

# Auto-fix validation errors
mcp__plugin_foundry_foundry-mcp__spec action="validate-fix" spec_id="{spec-id}" auto_fix=true
```

### Analysis Commands

```bash
# Find class
mcp__plugin_foundry_foundry-mcp__code action="find-class" name="ClassName" scope="src/"

# Find function
mcp__plugin_foundry_foundry-mcp__code action="find-function" name="functionName" scope="src/"

# Trace calls
mcp__plugin_foundry_foundry-mcp__code action="trace-calls" function="functionName" scope="src/"

# Impact analysis
mcp__plugin_foundry_foundry-mcp__code action="impact-analysis" target="SymbolName" scope="src/"
```

### File Locations

| Artifact | Location |
|----------|----------|
| Markdown plans | `specs/.plans/` |
| Plan reviews | `specs/.plan-reviews/` |
| Pending specs | `specs/pending/` |
| Active specs | `specs/active/` |
| Completed specs | `specs/completed/` |
| Archived specs | `specs/archived/` |
