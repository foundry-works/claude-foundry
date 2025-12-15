# SDD-Next Reference

Detailed workflows, examples, and edge cases for the sdd-next skill.

## Table of Contents

- [Context Gathering Best Practices](#context-gathering-best-practices)
- [Tool Value Matrix](#tool-value-matrix)
- [Deep Dive Context Structure](#deep-dive-context-structure)
- [Using doc-scope During Implementation](#using-doc-scope-during-implementation)
- [Post-Implementation Checklist](#post-implementation-checklist)
- [Phase Loop with Human Checkpoints](#phase-loop-with-human-checkpoints)
- [Troubleshooting](#troubleshooting)
- [Agent Delegation](#agent-delegation)
- [Built-in Subagent Patterns](#built-in-subagent-patterns)

---

## Context Gathering Best Practices

**Default workflow (stick to this unless the spec says otherwise)**
1. `mcp__plugin_foundry_foundry-mcp__task action="prepare"` supplies the recommended task plus rich context (previous sibling, parent metadata, phase progress, sibling files, recent journal summary, doc excerpts, dependencies).
2. Escalate to `mcp__plugin_foundry_foundry-mcp__task action="info"`, `mcp__plugin_foundry_foundry-mcp__task action="query"`, or `mcp__plugin_foundry_foundry-mcp__spec action="find"` only when the spec explicitly requires data not already present in the preparation payload.
3. After completing a task, call `task action="prepare"` again to refresh recommendations and context.

**Extended context flags**
- `include_full_journal`: Pull the previous sibling's full journal history for nuanced refactors.
- `include_phase_history`: Needed when drafting retrospectives or phase reports.
- `include_spec_overview`: Emits spec-wide progress without a separate `specs-find` call.
  Combine flags intentionally—each one increases payload size.

**Decision guide**
- Need additional detail beyond `context`?
  - Task metadata or acceptance criteria -> `mcp__plugin_foundry_foundry-mcp__task action="info"` for the specific task.
  - Arbitrary journal history -> `mcp__plugin_foundry_foundry-mcp__journal action="list"` (optionally scoped via `task_id`).
  - Alternate tasks or backlog exploration -> `mcp__plugin_foundry_foundry-mcp__task action="query"` with the relevant parent/status filters.
  - Otherwise, stay within the preparation payload to avoid redundant lookups.

**Anti-patterns to avoid**
- Chaining `task-info`, `journal-get`, and `task-query` "just in case." The base context already includes dependency details (`context.dependencies`).
- Repeatedly calling `specs-find` or `list-phases` after every adjustment—`context.phase` already tracks health; reserve list calls for explicit reporting.
- Inspecting raw spec JSON or launching doc-query before reviewing the preparation payload.

---

## Tool Value Matrix

| MCP Tool | Returns | Use when | Redundant / Notes |
| --- | --- | --- | --- |
| `task action="prepare"` | Recommended task plus context (siblings, parent, journal summary, dependencies, doc snippets) | **Always** – foundational call for every task | `file_docs` auto-populate when doc-query data exists |
| `task action="info"` | Raw task metadata from spec | Spec references data absent from preparation context | Often unnecessary unless spec says so |
| `spec action="find"` | Lifecycle-aware spec list with `progress_percentage` | Status reports or verifying completion prompts | Filter by status to minimize payload |
| `spec action="list"` | All specs matching status filter | Browsing available specs | Use status filter to minimize payload |
| `journal action="list"` | Journal entries for any spec/task | Deep retrospectives or decision lookups | Use `include_full_journal` for the previous sibling instead of separate calls |

---

## Deep Dive Context Structure

The `mcp__plugin_foundry_foundry-mcp__task action="prepare"` response contains everything you need:
- `task_data` -> title, metadata, instructions pulled from the spec
- `dependencies` -> top-level blocking status (can_start, blocked_by list)
- `context` -> stitched data from the previous sibling, parent task, current phase, sibling files, task journal, AND detailed dependency information

Typical `context` fields:

```json
"context": {
  "previous_sibling": {
    "task_id": "task-3-1-2",
    "title": "Tighten plan creation language",
    "summary": "Updated scope guardrails for Section 3.3"
  },
  "parent_task": {
    "task_id": "task-3-1",
    "title": "Polish the planning workflow",
    "position_label": "Phase 3 - Task 1"
  },
  "phase": {
    "name": "Implementation",
    "percentage": 58,
    "blockers": []
  },
  "sibling_files": [
    {"path": "skills/sdd-next/SKILL.md", "reason": "Touched by previous sibling"}
  ],
  "task_journal": {
    "entry_count": 0,
    "entries": []
  },
  "dependencies": {
    "blocking": [],
    "blocked_by_details": [
      {
        "id": "task-2-3",
        "title": "Update context gathering",
        "status": "in_progress",
        "file_path": "src/context.py"
      }
    ],
    "soft_depends": []
  }
}
```

**Field usage:**
- `context.previous_sibling`: Reference recent work for continuity or reuse its journal summary when explaining why the new task matters.
- `context.parent_task`: Verify how this subtask fits into the backlog; use `position_label` to show progress.
- `context.phase`: Surface phase health (`percentage`, `blockers`) without calling `specs-find`.
- `context.sibling_files`: Prime file navigation by reviewing whatever the spec already touched.
- `context.task_journal`: Access journal entries for this task showing decision history.
- `context.dependencies`: Detailed dependency information with task titles, statuses, and file paths—eliminates the need for separate dependency-check commands in 95% of cases.

---

## Using doc-scope During Implementation

When documentation is available and you need detailed implementation context for a file, use:
```bash
mcp__plugin_foundry_foundry-mcp__code action="scope" path=<file-path> view="implement"
```

This provides implementation-focused context including:
- Detailed function signatures and parameters
- Full implementation logic and patterns
- Code examples from the actual file
- Dependencies and imports
- Usage examples and patterns

**When to use `scope --view implement`:**
- **Starting implementation of a task** - Get comprehensive context before writing code
- **Understanding existing patterns** - See how current code works before extending it
- **Refactoring tasks** - Review full implementation details before restructuring
- **Complex file modifications** - Need deep understanding of current implementation
- **Following established patterns** - Extract patterns from existing code to maintain consistency

**When NOT to use `scope --view implement`:**
- **During planning phase** - Use `scope --plan` instead (lighter context)
- **Quick edits or trivial changes** - Direct file reading may be faster
- **Documentation unavailable** - Fall back to Read tool
- **Context limits approaching** - Avoid heavy payloads near 85% threshold

**Example workflow:**
```bash
# 1. Task started, need implementation context
mcp__plugin_foundry_foundry-mcp__code action="scope" path="src/services/auth.ts" view="implement"

# 2. Review detailed implementation patterns and signatures
# (command returns comprehensive implementation context)

# 3. Implement changes following discovered patterns
# 4. Mark task complete with journal entry
```

**Optimization tips:**
- Use `scope --view implement` at task start, not repeatedly during coding
- Cache insights from output rather than re-running
- Switch to targeted `Read` calls for specific line ranges if context is tight

---

## Post-Implementation Checklist

After completing a task:

- [ ] Task status updated (`in_progress` -> `completed`) with journal entry
- [ ] Follow-up commands or monitoring notes captured in journal
- [ ] Blockers or deviations surfaced to user; next steps agreed
- [ ] Next recommended task retrieved via `mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id}` and shared with user
- [ ] Spec context refreshed via `mcp__plugin_foundry_foundry-mcp__spec action="find" status="active"` for reporting

---

## Phase Loop with Human Checkpoints

### Scope Confirmation

Show phase progress via `mcp__plugin_foundry_foundry-mcp__spec action="get" spec_id={spec-id}`. Ask via `AskUserQuestion`:
- Focus on target phase
- Adjust scope

### Queue Preparation

Prime backlog:
```bash
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent={phase-id} status="pending"
```

If queue empty or blocked:
```bash
mcp__plugin_foundry_foundry-mcp__task action="list-blocked" spec_id={spec-id}
```
Pause for user direction.

### Task Loop

Reuse Task Workflow (steps 3.1-3.5) for each pending task.

After each completion:
- Refresh phase context via `mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent={phase-id}`
- Report blockers immediately

### Phase Wrap-Up

Summarize results:
```bash
mcp__plugin_foundry_foundry-mcp__spec action="get" spec_id={spec-id}
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent={phase-id}
```

Present accomplishments, verification outcomes, blockers.

Ask via `AskUserQuestion`: continue to next phase, perform phase review, or stop.

---

## Troubleshooting

### Spec File Not Found / Path Errors

**Cause:** Wrong working directory or relative paths

**Solution:**
- Provide absolute path: `mcp__plugin_foundry_foundry-mcp__task action="prepare" spec_id={spec-id} path="/absolute/path/to/specs"`
- Run `mcp__plugin_foundry_foundry-mcp__spec action="find" status="active"` to discover available specs

### All Tasks Blocked

**Diagnosis:**
```bash
mcp__plugin_foundry_foundry-mcp__task action="list-blocked" spec_id={spec-id}
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="blocked"
```

**Solution:** Present alternatives via `AskUserQuestion` or resolve blockers before continuing

### Journal vs Git History

| Use the Spec Journal When... | Use Git History When... |
|----------------------------|------------------------|
| Closing **any** SDD task (journaling is mandatory and captures intent, verification, and follow-ups). | Investigating merge conflicts, bisects, or broader repo archaeology unrelated to a single spec task. |
| You need implementation details, test results, deviations, or next-task hints (`journal.entries[]` already hold this context in structured JSON). | You must inspect low-level commit metadata, e.g., to see who touched a file outside the spec workflow. |
| Preparing status updates: previous sibling journal summaries come bundled in `task action="prepare"`. | You're debugging historical code paths predating the current spec. |

**Anti-pattern:** Running `git log` / `git show` to understand a recently completed SDD task when the journal already documents the work. That wastes time and risks contradicting the canonical record. Start with the spec journal; escalate to git history only if a fact is missing or you are diagnosing repo-level issues (rebases, conflicts, regressions).

**Journal advantages**
- Full implementation narrative (what changed and why) tied to `task_id`.
- Test and verification results in one place, ready for audits.
- Deviations, blockers, and next-task hints captured while they are fresh.
- Structured JSON makes it trivial for `task action="prepare"` to surface the latest context without extra commands.
- Archives context even if commits are squashed or rebased later.

---

## Context Fields Quick Reference

| Field | Contains | Use For |
|-------|----------|---------|
| `context.previous_sibling` | Recent completed task | Continuity, referencing prior work |
| `context.parent_task` | Parent task/phase info | Understanding position in hierarchy |
| `context.phase` | Phase progress, blockers | Health checks without extra calls |
| `context.sibling_files` | Files touched by siblings | Prime file navigation |
| `context.task_journal` | Task decision history | Understanding prior decisions |
| `context.dependencies` | Full dependency graph | Blocking/blocked status with details |

---

## Standard AskUserQuestion Options

When presenting choices to users, use structured `AskUserQuestion` with these standard patterns:

**Task approval:**
```
AskUserQuestion:
  - "Approve & Start" - Begin implementation
  - "View Details" - Show more context
  - "Different Task" - Browse alternatives
  - "Stop" - End session
```

---

## Agent Delegation

This skill handles task execution within existing specs. For spec creation or complex planning, delegate to specialized agents.

### When to Delegate

| Scenario | Action |
|----------|--------|
| No active specs exist | Delegate to sdd-planner for spec creation |
| User requests new feature/refactor | Delegate to sdd-planner |
| Multi-phase planning needed | Delegate to sdd-planner |
| Spec modification required | Use sdd-modify agent instead |

### Decision Flow

```
User Request
    |
    v
Check for active specs (mcp__plugin_foundry_foundry-mcp__spec action="list")
    |
    +-- Specs exist --> Use sdd-next workflow (task action="prepare", execute, complete)
    |
    +-- No specs --> Offer options via AskUserQuestion:
                      - "Create New Spec" --> Delegate to sdd-planner
                      - "View Pending Specs" --> Check pending folder
                      - "View Completed" --> Check completed/archived
```

### Invoking the Planner Agent

```
Task(
  subagent_type: "general-purpose",
  prompt: "Create a specification for: [user's request]. Follow the sdd-planner agent workflow in agents/sdd-planner.md.",
  description: "Create new spec"
)
```

### Built-in vs Custom Agents

| Need | Use |
|------|-----|
| Codebase exploration | **Explore** (built-in) |
| Complex investigation | **general-purpose** (built-in) |
| Spec modification | sdd-modify (custom) |
| Test execution | run-tests (custom) |
| Plan creation | sdd-planner (custom) |

### Related Agents

| Agent | Use For |
|-------|---------|
| `sdd-planner` | Creating new specifications |
| `sdd-modify` | Applying bulk changes to existing specs |
| `sdd-spec-reviewer` | Multi-model review of specifications |
| `test-runner` | Running and debugging tests |

---

## Built-in Subagent Patterns

Claude Code provides built-in subagents that complement custom agent delegation. Use these for efficient codebase exploration without bloating main conversation context.

### Explore Subagent

**Purpose:** Fast, read-only codebase exploration using Haiku model

**Tools available:** Glob, Grep, Read, Bash (read-only commands only)

| Task Phase | When to Use Explore |
|------------|---------------------|
| Pre-implementation | Find existing patterns, test files, related code |
| Debugging | Trace error sources, find similar issues |
| Verification | Locate all affected files before verification |
| Plan deviation | Investigate unexpected patterns |

**Invocation example:**
```
Use the Explore agent with "medium" thoroughness to find all files
that import the module being modified
```

### Thoroughness Levels

| Level | Use Case | Trade-off |
|-------|----------|-----------|
| **quick** | Known file patterns, targeted lookup | Fast but may miss edge cases |
| **medium** | General exploration (default) | Good balance of speed and coverage |
| **very thorough** | Security audits, unfamiliar code | Slower but comprehensive |

### General-Purpose Subagent

**Purpose:** Complex multi-step tasks requiring full tool access

**Model:** Sonnet (capable reasoning)

**Tools available:** All tools

**When to use:**
- Multi-file investigation that may require modifications
- Complex debugging requiring code changes
- Tasks with multiple dependent steps

### When NOT to Use Subagents

- **Simple single-file tasks** - Use direct tools instead
- **Known file locations** - Direct Read is faster
- **Spec reading** - Always use MCP tools per skill rules (never direct file access)
- **Near context limit** - Subagent results still consume context when returned

### Benefits of Subagent Delegation

| Benefit | Description |
|---------|-------------|
| **Context isolation** | Search results don't bloat main conversation |
| **Speed** | Explore uses Haiku for fast searches |
| **Focus** | Subagent returns only relevant findings |
| **Parallelization** | Multiple Explore agents can run concurrently |
