# SDD-Next Reference

Detailed workflows, examples, and edge cases for the sdd-next skill.

## Table of Contents

- [Context Gathering Best Practices](#context-gathering-best-practices)
- [Tool Value Matrix](#tool-value-matrix)
- [Deep Dive Context Structure](#deep-dive-context-structure)
- [Using doc-scope During Implementation](#using-doc-scope-during-implementation)
- [Post-Implementation Checklist](#post-implementation-checklist)
- [Autonomous Mode Workflow](#autonomous-mode-phase-completion)
- [Phase Loop with Human Checkpoints](#phase-loop-with-human-checkpoints)
- [Troubleshooting](#troubleshooting)

---

## Context Gathering Best Practices

**Default workflow (stick to this unless the spec says otherwise)**
1. `mcp__foundry-mcp__task-prepare` supplies the recommended task plus rich context (previous sibling, parent metadata, phase progress, sibling files, recent journal summary, doc excerpts, dependencies).
2. Escalate to `mcp__foundry-mcp__task-info`, `mcp__foundry-mcp__task-query`, or `mcp__foundry-mcp__specs-find` only when the spec explicitly requires data not already present in the preparation payload.
3. After completing a task, call `task-prepare` again to refresh recommendations and context.

**Extended context flags**
- `include_full_journal`: Pull the previous sibling's full journal history for nuanced refactors.
- `include_phase_history`: Needed when drafting retrospectives or phase reports.
- `include_spec_overview`: Emits spec-wide progress without a separate `specs-find` call.
  Combine flags intentionally—each one increases payload size.

**Decision guide**
- Need additional detail beyond `context`?
  - Task metadata or acceptance criteria -> `mcp__foundry-mcp__task-info` for the specific task.
  - Arbitrary journal history -> `mcp__foundry-mcp__journal-get` (optionally scoped via `task_id`).
  - Alternate tasks or backlog exploration -> `mcp__foundry-mcp__task-query` with the relevant parent/status filters.
  - Otherwise, stay within the preparation payload to avoid redundant lookups.

**Anti-patterns to avoid**
- Chaining `task-info`, `journal-get`, and `task-query` "just in case." The base context already includes dependency details (`context.dependencies`).
- Repeatedly calling `specs-find` or `list-phases` after every adjustment—`context.phase` already tracks health; reserve list calls for explicit reporting.
- Inspecting raw spec JSON or launching doc-query before reviewing the preparation payload.

---

## Tool Value Matrix

| MCP Tool | Returns | Use when | Redundant / Notes |
| --- | --- | --- | --- |
| `mcp__foundry-mcp__task-prepare` | Recommended task plus context (siblings, parent, journal summary, dependencies, doc snippets) | **Always** – foundational call for every task | `file_docs` auto-populate when doc-query data exists |
| `mcp__foundry-mcp__task-info` | Raw task metadata from spec | Spec references data absent from preparation context | Often unnecessary unless spec says so |
| `mcp__foundry-mcp__specs-find` | Lifecycle-aware spec list with `progress_percentage` | Status reports or verifying completion prompts | Filter by status to minimize payload |
| `mcp__foundry-mcp__list-phases` | Every phase with completion % | Re-prioritizing phases / presenting alternate scopes | Usually redundant once `context.phase` is populated |
| `mcp__foundry-mcp__journal-get` | Journal entries for any spec/task | Deep retrospectives or decision lookups | Use `include_full_journal` for the previous sibling instead of separate calls |

---

## Deep Dive Context Structure

The `mcp__foundry-mcp__prepare-task` response contains everything you need:
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
mcp__foundry-mcp__doc-scope <file-path> --view implement
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
mcp__foundry-mcp__doc-scope src/services/auth.ts --view implement

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
- [ ] **Context check performed** (two-step pattern)
- [ ] Next recommended task retrieved via `mcp__foundry-mcp__prepare-task {spec-id}` and shared with user
- [ ] Spec context refreshed via `mcp__foundry-mcp__specs-find --status active` for reporting

---

## Autonomous Mode (Phase Completion)

Use this workflow when the configured work mode is **Autonomous Mode** (`"work_mode": "autonomous"` in config). If the user changes the config to Single Task Mode mid-session, switch to Single Task Workflow.

### When to Use

- Config file has `"work_mode": "autonomous"` set
- User wants to complete multiple tasks in current phase without per-task approval
- User has sufficient context headroom (check context before starting)

### Key Characteristics

- **Phase-scoped**: Completes all tasks within current phase only (does not cross phase boundaries)
- **Context-aware**: Checks context after EVERY task, stops if >=85%
- **Defensive stops**: Stops for blocked tasks and plan deviations (requires user approval)
- **No plan approval**: Creates execution plans internally without showing user

### Autonomous Workflow Loop

#### Step 1: Task Execution Loop

For each task in current phase:

1. **Prepare next task:**
   ```bash
   mcp__foundry-mcp__prepare-task {spec-id}
   ```

2. **Check phase complete:** If no more tasks in current phase -> Exit loop

3. **Check for blockers:**
   - If next task blocked: **STOP**
   - Present blocker info via `AskUserQuestion`
   - Options: alternative tasks, resolve blocker, or stop
   - Exit autonomous mode

4. **Create execution plan (silently):**
   - Analyze task metadata from `prepare-task` output
   - Create detailed internal plan (no user approval needed)
   - Include all standard components: prerequisites, steps, success criteria

5. **Mark task in_progress:**
   ```bash
   mcp__foundry-mcp__task-update-status {spec-id} {task-id} in_progress
   ```

6. **Execute implementation** according to internal plan

7. **Handle plan deviations:**
   - If implementation deviates: **STOP**
   - Document deviation
   - Present to user via `AskUserQuestion`
   - Options: revise plan, update spec, explain more, rollback
   - Exit autonomous mode

8. **Mark task complete:**
   ```bash
   Skill(foundry:sdd-update) "Complete task {task-id} in spec {spec-id}. Completion note: [Brief summary of what was accomplished, tests run, verification performed]."
   ```

9. **CRITICAL: Check context usage (REQUIRED):**

   Run two-step pattern as SEPARATE, SEQUENTIAL Bash calls:
   ```bash
   # First call:
   mcp__foundry-mcp__session-generate-marker
   ```
   ```bash
   # Second call (only after first completes):
   mcp__foundry-mcp__session-context --session-marker "SESSION_MARKER_<hash>"
   ```

   **Check ACTUAL context percentage reported, do NOT speculate:**
   - If context ACTUALLY >=85% (as reported by command): **STOP**, exit loop, go to Summary
   - If context <85%: Continue to next iteration

   **CRITICAL:** Do not stop based on predictions like "upcoming work will use context" or "I should have buffer space for the next phase." Only stop when actual usage reaches the threshold. The 85% threshold already provides safety margin.

10. **Check phase completion:**
    - If current phase complete: Exit loop, go to Summary
    - Otherwise: Return to step 1

#### Step 2: Present Summary Report

When autonomous mode exits:

```markdown
## Autonomous Execution Summary

**Mode:** Phase Completion (Autonomous)
**Spec:** {spec-title} ({spec-id})
**Phase:** {phase-title} ({phase-id})

### Tasks Completed
- task-1-1: [title] - Duration: X min
- task-1-2: [title] - Duration: Y min

### Phase Progress
Phase {phase-id}: {completed}/{total} tasks ({percentage}%)
Overall: {total_completed}/{total_tasks} tasks ({overall_percentage}%)

### Context Usage
Current context: {context_percentage}%

### Exit Reason
{One of:}
- Phase Complete
- Context Limit: >=85% threshold
- Blocked Task
- Plan Deviation
- No Actionable Tasks

### Next Steps
{Contextual recommendations based on exit reason}
```

### Autonomous Mode Best Practices

**DO:**
- Check context after EVERY task completion
- Stop immediately when context >=85%
- Base stopping decisions on ACTUAL context percentage, never predictions
- Use the full safety margin (continue until >=85% reported)
- Stop for blocked tasks (don't auto-pivot)
- Stop for plan deviations (don't auto-revise)
- Create detailed internal plans
- Present comprehensive summary at end

**DON'T:**
- Cross phase boundaries
- Skip plan creation (always plan, just don't show)
- Continue past 85% context
- Stop early based on predictions of future context usage
- Auto-resolve blockers
- Auto-revise plans on deviations
- Batch task completions

---

## Phase Loop with Human Checkpoints

### Scope Confirmation

Show `mcp__foundry-mcp__list-phases {spec-id}` with progress. Ask via `AskUserQuestion`:
- Focus on target phase
- Adjust scope
- Revert to single-task mode

### Queue Preparation

Prime backlog:
```bash
mcp__foundry-mcp__task-query {spec-id} --parent {phase-id} --status pending
```

If queue empty or blocked:
```bash
mcp__foundry-mcp__list-blockers {spec-id}
```
Pause for user direction.

### Task Loop

Reuse Single Task Workflow (steps 3.1-3.6) for each pending task.

After each completion:
- Refresh phase context via `mcp__foundry-mcp__list-phases {spec-id}` or `mcp__foundry-mcp__task-query {spec-id} --parent {phase-id}`
- If granted "auto-continue for this phase", note permission but still report blockers immediately

### Phase Wrap-Up

Summarize results:
```bash
mcp__foundry-mcp__list-phases {spec-id}
mcp__foundry-mcp__task-query {spec-id} --parent {phase-id}
```

Present accomplishments, verification outcomes, blockers.

Ask via `AskUserQuestion`: continue to next phase, perform phase review, or stop.

---

## Troubleshooting

### Spec File Not Found / Path Errors

**Cause:** Wrong working directory or relative paths

**Solution:**
- Provide absolute path: `mcp__foundry-mcp__prepare-task {spec-id} --path /absolute/path/to/specs`
- Run `mcp__foundry-mcp__specs-find --status active` to discover available specs

### All Tasks Blocked

**Diagnosis:**
```bash
mcp__foundry-mcp__list-blockers {spec-id}
mcp__foundry-mcp__task-query {spec-id} --status blocked
```

**Solution:** Present alternatives via `AskUserQuestion` or resolve blockers before continuing

### Journal vs Git History

| Use the Spec Journal When... | Use Git History When... |
|----------------------------|------------------------|
| Closing **any** SDD task (journaling is mandatory and captures intent, verification, and follow-ups). | Investigating merge conflicts, bisects, or broader repo archaeology unrelated to a single spec task. |
| You need implementation details, test results, deviations, or next-task hints (`journal.entries[]` already hold this context in structured JSON). | You must inspect low-level commit metadata, e.g., to see who touched a file outside the spec workflow. |
| Preparing status updates: previous sibling journal summaries come bundled in `mcp__foundry-mcp__prepare-task`. | You're debugging historical code paths predating the current spec. |

**Anti-pattern:** Running `git log` / `git show` to understand a recently completed SDD task when the journal already documents the work. That wastes time and risks contradicting the canonical record. Start with the spec journal; escalate to git history only if a fact is missing or you are diagnosing repo-level issues (rebases, conflicts, regressions).

**Journal advantages**
- Full implementation narrative (what changed and why) tied to `task_id`.
- Test and verification results in one place, ready for audits.
- Deviations, blockers, and next-task hints captured while they are fresh.
- Structured JSON makes it trivial for `mcp__foundry-mcp__prepare-task` to surface the latest context without extra commands.
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
Check for active specs (mcp__foundry-mcp__spec-list)
    |
    +-- Specs exist --> Use sdd-next workflow (task-prepare, execute, complete)
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

### Related Agents

| Agent | Use For |
|-------|---------|
| `sdd-planner` | Creating new specifications |
| `sdd-modify` | Applying bulk changes to existing specs |
| `sdd-spec-reviewer` | Multi-model review of specifications |
| `test-runner` | Running and debugging tests |
