---
name: sdd-update
description: Progress tracking for spec-driven development. Use to update task status, track progress, journal decisions, move specs between folders, and maintain spec files. Handles the administrative/clerical aspects of specification documents during development.
---

# SDD Update Skill

## When to Use

- Complete tasks (atomically marks complete AND journals)
- Mark tasks as in_progress or blocked
- Document decisions/deviations via journal entries
- Move specs between lifecycle folders
- Update spec metadata

**Do NOT use for:**
- Creating specs → use `sdd-plan`
- Finding next task → use `sdd-next`
- Structural changes → use `sdd-modify`
- Writing code or running tests

> For comparison with sdd-modify, see `references/comparison.md`

## MCP Tooling

| Router | Key Actions |
|--------|-------------|
| `task` | `complete`, `start`, `block`, `unblock`, `query`, `add-dependency`, `remove-dependency`, `add-requirement` |
| `journal` | `add`, `list` |
| `lifecycle` | `activate`, `move`, `complete` |

**Critical Rules:**
- ALWAYS use MCP tools for spec operations
- NEVER use `Read()` on spec JSON files
- NEVER shell out to `cat`/`grep`/`jq` for specs

## Essential Command

```bash
# Complete a task (recommended - atomic status + journal)
mcp__plugin_foundry_foundry-mcp__task action="complete" spec_id={spec-id} task_id={task-id} journal_entry="..."
```

> For workflow details, see `references/workflow-tasks.md`, `references/workflow-tracking.md`, `references/workflow-git.md`

## Core Workflow

> `[x?]`=decision · `(GATE)`=user approval · `→`=sequence · `↻`=loop · `§`=section ref

```
- **Entry** → [Action?]
  - [start] → `task action="start"` → in_progress
  - [progress] → `journal action="add"` (document WHY)
  - [complete] → `task action="complete"` + journal_entry
  - [block] → `task action="block"` + reason
- [Lifecycle?] → `lifecycle action="activate|move|complete"`
- **Exit**
```

## Key Principles

1. **Update immediately** - Don't batch updates
2. **Be specific** - Vague notes aren't helpful later
3. **Document WHY** - Rationale, not just what changed
4. **Use complete** - Ensures proper journaling

## Detailed Reference

- Task states & folder structure → `references/structure.md`
- sdd-update vs sdd-modify → `references/comparison.md`
- Task lifecycle (start/block/complete) → `references/workflow-tasks.md`
- Progress tracking & verification → `references/workflow-tracking.md`
- Spec lifecycle (activate/move) → `references/workflow-lifecycle.md`
- Git integration → `references/workflow-git.md`
- MCP patterns → `references/mcp-patterns.md`
- Troubleshooting → `references/troubleshooting.md`
- Command reference → `references/command-reference.md`

See **[references/](./references/)**
