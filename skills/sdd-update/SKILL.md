---
name: sdd-update
description: Progress tracking for spec-driven development. Use to update task status, track progress, journal decisions, move specs between folders, and maintain spec files. Handles the administrative/clerical aspects of specification documents during development.
---

# Spec-Driven Development: Update Skill

## When to Use This Skill

Use `Skill(foundry:sdd-update)` to:
- **Complete tasks** (atomically marks as completed AND creates journal entry)
- Mark tasks as in_progress or blocked
- Document decisions and deviations in journal entries
- Add verification results to specs
- Move specs between lifecycle folders (pending → active → completed)
- Update spec metadata fields

**Do NOT use for:**
- Creating specifications (use `sdd-plan`)
- Finding what to work on next (use `sdd-next`)
- Writing code or running tests
- Structural spec modifications (use `sdd-modify`)

## Core Philosophy

**Document Reality**: JSON spec files are living documents that evolve during implementation. This skill ensures the spec accurately reflects current progress, decisions, and status. All updates flow through the MCP tools, which handle validation, backups, and progress recalculation automatically.

## MCP Tooling

This skill operates entirely through the Foundry MCP server (`foundry-mcp`). Tools use the router+action pattern: `mcp__plugin_foundry_foundry-mcp__<router>` with `action="<action>"`.

**Critical Rules:**
- **ALWAYS** use MCP tools for spec operations
- **NEVER** use `Read()` on spec JSON files
- **NEVER** use shell commands (`cat`, `grep`, `jq`) on specs
- Spec files are large; direct reading wastes context and bypasses validation

## Skill Family

This skill is part of the **Spec-Driven Development** workflow:

```
sdd-plan → sdd-plan-review → sdd-modify → sdd-next → Implementation → sdd-update (this skill)
```

## sdd-update vs sdd-modify

| Operation | sdd-update | sdd-modify |
|-----------|:----------:|:----------:|
| Mark task completed | Yes | No |
| Update task status | Yes | No |
| Add journal entries | Yes | No |
| Move spec between folders | Yes | No |
| **Update task descriptions** | No | Yes |
| **Add/remove tasks** | No | Yes |
| **Add verification steps** | No | Yes |
| **Apply review feedback** | No | Yes |

**Key Distinction:**
- **sdd-update** = Lightweight metadata updates (status, progress, journals)
- **sdd-modify** = Heavyweight structural changes (tasks, descriptions, verifications)

## Essential Workflows

> For detailed workflow steps with examples, see `reference.md#workflow-details`

### Complete a Task (Recommended)

Use `task action="complete"` to atomically update status AND create a journal entry:

```bash
mcp__plugin_foundry_foundry-mcp__task action="complete" spec_id={spec-id} task_id={task-id} journal_entry="What was accomplished, tests run, verification performed."
```

This automatically:
1. Updates task status to `completed`
2. Records completion timestamp
3. Creates journal entry
4. Clears `needs_journaling` flag
5. Auto-journals parent nodes that complete

### Start a Task

```bash
mcp__plugin_foundry_foundry-mcp__task action="start" spec_id={spec-id} task_id={task-id}
```

### Block/Unblock Tasks

```bash
# Block with reason
mcp__plugin_foundry_foundry-mcp__task action="block" spec_id={spec-id} task_id={task-id} reason="Description" blocker_type="dependency"

# Unblock with resolution
mcp__plugin_foundry_foundry-mcp__task action="unblock" spec_id={spec-id} task_id={task-id} resolution="How resolved"
```

### Add Journal Entries

```bash
mcp__plugin_foundry_foundry-mcp__journal action="add" spec_id={spec-id} title="Title" content="Content" task_id={task-id} entry_type="decision"
```

**Entry types:** `decision`, `deviation`, `blocker`, `note`, `status_change`

### Move Specs

```bash
# Activate from backlog
mcp__plugin_foundry_foundry-mcp__lifecycle action="activate" spec_id={spec-id}

# Archive
mcp__plugin_foundry_foundry-mcp__lifecycle action="move" spec_id={spec-id} to_folder="archived"

# Complete (all tasks done)
mcp__plugin_foundry_foundry-mcp__lifecycle action="complete" spec_id={spec-id}
```

## Task Status Values

- `pending` - Not yet started
- `in_progress` - Currently being worked on
- `completed` - Successfully finished
- `blocked` - Cannot proceed

> For JSON structure details, see `reference.md#json-structure-reference`

## Folder Structure

```
specs/
├── pending/      # Backlog - planned but not activated
├── active/       # Currently being implemented
├── completed/    # Finished and verified
└── archived/     # Old or superseded
```

> For spec lifecycle transitions, see `reference.md#workflow-6-moving-specs-between-folders`

## Key Principles

1. **Update immediately** - Don't wait; update status as work happens
2. **Be specific** - Vague notes aren't helpful later
3. **Document WHY** - Always explain rationale, not just what changed
4. **Use complete-task** - Ensures proper journaling of task completion

## Detailed Reference

For comprehensive documentation including:
- All workflow details with examples
- Git commit integration
- Verification result recording
- Configuration options
- Troubleshooting guide
- Common mistakes and fixes
- Full command reference

See **[reference.md](./reference.md)**
