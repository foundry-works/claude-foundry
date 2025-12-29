---
name: sdd-plan
description: Plan-first development methodology that creates detailed specifications before coding. Creates structured plans with phases, file-level details, and verification steps. Includes automatic AI review, modification application, and validation.
---

# Spec-Driven Development: Planning Skill

## Overview

`Skill(foundry:sdd-plan)` creates detailed JSON specifications before any code is written. It analyzes the codebase, designs a phased implementation approach, produces structured task hierarchies, and runs automatic quality checks including AI review.

**Core capabilities:**
- Analyze codebase structure using Explore agents and LSP
- Design phased implementation approaches
- Create JSON specifications with task hierarchies
- Define verification steps for each phase
- **Automatic AI review** of specifications
- **Apply review feedback** as modifications
- **Validate and auto-fix** specifications

## Integrated Workflow

This skill combines planning, review, modification, and validation into a unified flow:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  1. Analyze     2. Create Spec    3. AI Review    4. Apply     5. Validate  │
│  ───────────    ────────────      ───────────     ──────────   ──────────   │
│  Explore/LSP →  spec-create   →   spec-review →  apply-plan → validate-fix │
│  (understand)   (scaffold)        (auto)         (if needed)  (auto-fix)   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Flow

> `[x?]`=decision · `(GATE)`=user approval · `→`=sequence · `↻`=loop

```
- **Entry** → UnderstandIntent → Analyze[Explore|LSP]
  - `authoring action="spec-create"` → scaffold spec
  - `authoring action="phase-add-bulk"` ↻ per phase
  - `authoring action="spec-update-frontmatter"` → mission/metadata
- **Review** → `review action="spec-review"` (automatic)
  - [findings?] → `review action="parse-feedback"`
  - (GATE: approve modifications) → `spec action="apply-plan"`
- **Validate** → `spec action="validate"` ↻ [errors?] → `spec action="fix"`
- **Exit** → specs/pending/{spec-id}.json
```

## When to Use This Skill

**Use for:**
- New features or significant functionality additions
- Complex refactoring across multiple files
- API integrations or external service connections
- Architecture changes or system redesigns
- Any task requiring precision and reliability

**Do NOT use for:**
- Simple one-file changes or bug fixes
- Trivial modifications or formatting changes
- Exploratory prototyping or spikes
- Finding next task or tracking progress (use `sdd-implement`)

## MCP Tooling

| Router | Key Actions |
|--------|-------------|
| `authoring` | `spec-create`, `spec-update-frontmatter`, `phase-add-bulk`, `phase-template`, `phase-move`, `phase-update-metadata` |
| `spec` | `validate`, `fix`, `apply-plan`, `completeness-check`, `duplicate-detection`, `stats`, `analyze-deps` |
| `review` | `spec-review`, `parse-feedback`, `list-tools` |
| `task` | `add`, `remove`, `move`, `update-metadata` |

**Critical Rule:** NEVER read spec JSON files directly with `Read()` or shell commands.

## Core Workflow

### Step 1: Understand Intent

Before creating any plan:
- **Core objective**: What is the primary goal?
- **Spec mission**: Single-sentence mission for `metadata.mission`
- **Success criteria**: What defines "done"?
- **Constraints**: What limitations or requirements exist?

### Step 2: Analyze Codebase

Use **Explore subagents** for large codebases (prevents context bloat), or `Glob`/`Grep`/`Read` for targeted lookups.

**LSP-Enhanced Analysis** for refactoring:
- `documentSymbol` - Understand file structure
- `findReferences` - Assess impact (count affected files)
- `goToDefinition` - Navigate to implementations

> See `references/codebase-analysis.md` for detailed patterns.

### Step 3: Create Phase Plan

For complex features, create a markdown phase plan first:

```bash
mcp__plugin_foundry_foundry-mcp__plan action="create" name="Feature Name" template="detailed"
```

Get user approval before detailed spec creation.

> See `references/phase-authoring.md` for templates and bulk macros.

### Step 4: Create JSON Specification

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create" name="feature-name" template="medium"
```

Add phases with tasks:
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="phase-add-bulk" spec_id="{spec-id}" phase='{"title": "Implementation", "description": "Core work"}' tasks='[{"type": "task", "title": "Build core logic", "task_category": "implementation", "file_path": "src/core.py", "acceptance_criteria": ["Workflow works"]}]'
```

Update metadata:
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-update-frontmatter" spec_id="{spec-id}" key="mission" value="Single-sentence objective"
```

> See `references/json-spec.md` and `references/task-hierarchy.md` for structure details.

### Step 5: Run AI Review (Automatic)

After spec creation, AI review runs automatically:

```bash
mcp__plugin_foundry_foundry-mcp__review action="spec-review" spec_id="{spec-id}" review_type="full"
```

**Review types:**
| Type | Models | Focus | Use When |
|------|--------|-------|----------|
| `quick` | 2 | Completeness, Clarity | Simple specs |
| `full` | 3-4 | All 6 dimensions | Complex specs |
| `security` | 2-3 | Risk Management | Auth, data handling |
| `feasibility` | 2-3 | Estimates, Dependencies | Tight deadlines |

> See `references/plan-review-workflow.md` for detailed review workflow.
> See `references/plan-review-dimensions.md` for review dimensions.
> See `references/plan-review-consensus.md` for interpreting results.

### Step 6: Apply Modifications (If Needed)

If review finds issues, parse and apply feedback:

```bash
# Parse review feedback into modifications
mcp__plugin_foundry_foundry-mcp__review action="parse-feedback" spec_id="{spec-id}" review_path="path/to/review.md"

# Preview changes (always preview first!)
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="{spec-id}" modifications_file="suggestions.json" dry_run=true

# Apply changes (backup created automatically)
mcp__plugin_foundry_foundry-mcp__spec action="apply-plan" spec_id="{spec-id}" modifications_file="suggestions.json"
```

> See `references/modification-workflow.md` for detailed workflow.
> See `references/modification-operations.md` for operation formats.

### Step 7: Validate Specification

Validate and auto-fix the specification:

```bash
# Validate
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="{spec-id}"

# Auto-fix common issues
mcp__plugin_foundry_foundry-mcp__spec action="fix" spec_id="{spec-id}"

# Re-validate until clean
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="{spec-id}"
```

**Exit codes:**
- 0: Valid (no errors)
- 1: Warnings only
- 2: Errors (run fix)
- 3: File error

> See `references/validation-workflow.md` for detailed workflow.
> See `references/validation-fixes.md` for fix patterns.
> See `references/validation-issues.md` for issue types.

## Phase-First Authoring

Use this approach for efficient spec creation:

**Step 1: Create spec from template**
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-create" name="my-feature" template="medium"
```

**Step 2: Add phases with bulk macro**
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="phase-add-bulk" spec_id="{spec-id}" phase='{"title": "Phase 1"}' tasks='[...]'
```

**Step 3: Fine-tune tasks**
Use modification operations to adjust individual tasks.

**Step 4: Update frontmatter**
```bash
mcp__plugin_foundry_foundry-mcp__authoring action="spec-update-frontmatter" spec_id="{spec-id}" key="mission" value="..."
```

## File Path Policy

For `implementation` or `refactoring` tasks, set `metadata.file_path` to a **real repo-relative path**. Do not guess or use placeholders. If unclear, use `task_category: "investigation"` first.

## Valid Values Reference

### Spec Templates

| Template | Use Case | Required Fields |
|----------|----------|-----------------|
| `simple` | Small tasks | Basic fields only |
| `medium` | Standard features | `mission`, `description`, `acceptance_criteria`, `task_category` |
| `complex` | Large features | All medium + detailed metadata |
| `security` | Security-sensitive | All complex + security review |

### Task Categories

| Category | Requires `file_path` |
|----------|---------------------|
| `investigation` | No |
| `implementation` | **Yes** |
| `refactoring` | **Yes** |
| `decision` | No |
| `research` | No |

### Node Types

| Node Type | `type` Value | Key Fields |
|-----------|--------------|------------|
| Task | `"task"` | `task_category`, `file_path` |
| Verification | `"verify"` | `verification_type` |

### Verification Types

| Type | Purpose |
|------|---------|
| `run-tests` | Execute test suite |
| `fidelity` | Compare implementation to spec |
| `manual` | Manual testing checklist |

### Task Statuses

| Status | Description |
|--------|-------------|
| `pending` | Not yet started |
| `in_progress` | Currently being worked on |
| `completed` | Finished successfully |
| `blocked` | Cannot proceed |

## Size Guidelines

If >6 phases or >50 tasks, recommend splitting into multiple specs.

## Output Artifacts

1. **JSON spec file** at `specs/pending/{spec-id}.json`
2. **AI review report** at `specs/.plan-reviews/{spec-id}-review-{type}.md`
3. **Validation passed** with no errors

## Detailed Reference

**Planning:**
- Investigation strategies → `references/investigation.md`
- Phase authoring → `references/phase-authoring.md`
- Phase plan template → `references/phase-plan-template.md`
- JSON spec structure → `references/json-spec.md`
- Task hierarchy → `references/task-hierarchy.md`
- Codebase analysis → `references/codebase-analysis.md`

**Review:**
- Review workflow → `references/plan-review-workflow.md`
- Review dimensions → `references/plan-review-dimensions.md`
- Consensus interpretation → `references/plan-review-consensus.md`

**Modification:**
- Modification workflow → `references/modification-workflow.md`
- Operation formats → `references/modification-operations.md`

**Validation:**
- Validation workflow → `references/validation-workflow.md`
- Fix patterns → `references/validation-fixes.md`
- Issue types → `references/validation-issues.md`

**General:**
- AI review → `references/ai-review.md`
- Troubleshooting → `references/troubleshooting.md`
