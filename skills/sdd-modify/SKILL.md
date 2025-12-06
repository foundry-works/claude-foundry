---
name: sdd-modify
description: Apply spec modifications systematically. Use to apply review feedback, bulk modifications, or interactive spec updates with safety checks, validation, and rollback support.
---

# Spec-Driven Development: Modify Skill

## When to Use This Skill

Use `Skill(foundry:sdd-modify)` to:
- **Apply review feedback** from sdd-plan-review or sdd-fidelity-review
- **Bulk modify** task descriptions, metadata, or verification steps
- **Add/remove tasks** or verification nodes
- **Update task metadata** across multiple nodes
- **Interactive spec updates** with preview and validation

**Do NOT use for:**
- Status tracking or task completion (use `sdd-update`)
- Creating specifications (use `sdd-plan`)
- Finding next actionable task (use `sdd-next`)
- Reading spec files directly (use MCP tools)

## Core Philosophy

**Safe Systematic Changes**: Modifications are applied with automatic backup, preview (dry-run), validation, and rollback capabilities. Never apply changes without previewing first. The spec file is the source of truth, and all changes flow through MCP tools that handle validation automatically.

## MCP Tooling

This skill operates entirely through the Foundry MCP server (`foundry-mcp`). Tool names follow the `mcp__foundry-mcp__<tool-name>` pattern.

| Tool | Purpose |
|------|---------|
| `spec-apply-plan` | Apply structured modifications from JSON file |
| `review-parse-feedback` | Convert review markdown to modifications JSON |
| `spec-validate` | Check spec validity after modifications |
| `spec-validate-fix` | Auto-fix common validation issues |
| `spec-get` | Retrieve spec data |
| `spec-get-hierarchy` | Get spec hierarchy structure |
| `spec-list` | List available specs |
| `spec-update-frontmatter` | Update spec-level metadata |
| `task-add` | Add new task/subtask/verify nodes |
| `task-remove` | Remove nodes (optionally cascading) |
| `task-update-metadata` | Update task metadata fields |
| `verification-add` | Add verification results |
| `journal-add` | Document modifications |

**Critical Rules:**
- **ALWAYS** use MCP tools for spec operations
- **NEVER** use `Read()` on spec JSON files
- **NEVER** use shell commands (`cat`, `grep`, `jq`) on specs
- **ALWAYS** preview with dry-run before applying modifications

## Skill Family

This skill is part of the **Spec-Driven Development** workflow:

```
sdd-plan --> sdd-plan-review --> sdd-modify (this skill) --> sdd-next --> Implementation --> sdd-update
                                       ^
             sdd-fidelity-review ------+
```

Use after `sdd-plan-review` or `sdd-fidelity-review` to apply suggested changes.

## sdd-modify vs sdd-update

| Operation | sdd-modify | sdd-update |
|-----------|:----------:|:----------:|
| **Update task descriptions** | Yes | No |
| **Add/remove tasks** | Yes | No |
| **Add verification steps** | Yes | No |
| **Apply review feedback** | Yes | No |
| Mark task completed | No | Yes |
| Update task status | No | Yes |
| Add journal entries | Both | Yes |
| Move spec between folders | No | Yes |

**Key Distinction:**
- **sdd-modify** = Structural changes (tasks, descriptions, verifications)
- **sdd-update** = Status updates (progress, journals, lifecycle)

## Core Workflow

> For detailed workflow steps with examples, see `reference.md#workflow-details`

### Step 1: Understand the Request

Parse the input to identify:
- **Spec ID**: Which specification to modify
- **Source**: Review report path OR modifications JSON path
- **Mode**: Apply vs dry-run preview

### Step 2: Parse Review Feedback (if review report provided)

```bash
mcp__foundry-mcp__review-parse-feedback spec_id="{spec-id}" review_path="{path}"
```

This converts markdown review findings into structured modification JSON.

### Step 3: Preview Modifications (always do this first)

```bash
mcp__foundry-mcp__spec-apply-plan spec_id="{spec-id}" modifications_file="{path}" dry_run=true
```

Review the preview output: tasks to update, verification steps to add, metadata changes, impact summary.

### Step 4: Apply Modifications

After confirming preview looks correct:

```bash
mcp__foundry-mcp__spec-apply-plan spec_id="{spec-id}" modifications_file="{path}"
```

The tool automatically creates backup, applies modifications, validates result, and rolls back if validation fails.

### Step 5: Validate and Report

Confirm spec is valid:

```bash
mcp__foundry-mcp__spec-validate spec_id="{spec-id}"
```

If validation issues found, apply auto-fixes:

```bash
mcp__foundry-mcp__spec-validate-fix spec_id="{spec-id}" auto_fix=true
```

## Supported Operations

| Operation | Purpose |
|-----------|---------|
| `update_task` | Modify task title, description, file_path, category |
| `add_verification` | Add verification step to task |
| `update_metadata` | Update task metadata (hours, priority, etc.) |
| `batch_update` | Apply same change to multiple nodes |
| `add_node` | Add new task/subtask/verify node |
| `remove_node` | Remove node (optionally cascading) |

> For detailed operation formats and examples, see `reference.md#operation-formats`

## Safety Features

- **Automatic backup** before every modification (`specs/.backups/{spec-id}-{timestamp}.json`)
- **Dry-run preview** before applying any changes
- **Validation** after every apply
- **Rollback** if validation fails
- **Idempotent** operations (safe to retry)

> For rollback and recovery procedures, see `reference.md#error-handling`

## Output Format

Return a structured summary:

```markdown
## Modification Summary

**Spec:** {spec-id}
**Source:** {review report or modifications file}

### Changes Applied
- Updated X task descriptions
- Added Y verification steps
- Modified Z metadata fields

### Backup Location
`specs/.backups/{spec-id}-{timestamp}.json`

### Validation Status
{PASSED | FAILED with details}

### Next Steps
- {Contextual recommendations}
```

## Detailed Reference

For comprehensive documentation including:
- Detailed workflow steps with examples
- Operation formats and JSON structures
- Error handling and rollback procedures
- Direct modification patterns (when to skip review parsing)
- Troubleshooting guide

See **[reference.md](./reference.md)**
