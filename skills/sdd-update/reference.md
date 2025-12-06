# SDD-Update Reference

Detailed workflows, examples, and edge cases for the sdd-update skill.

## Table of Contents

- [Workflow Details](#workflow-details)
  - [Starting a Task](#workflow-1-starting-a-task)
  - [Tracking Progress](#workflow-2-tracking-progress)
  - [Handling Blockers](#workflow-3-handling-blockers)
  - [Verification Results](#workflow-4-adding-verification-results)
  - [Completing Tasks](#workflow-5-completing-tasks)
  - [Moving Specs](#workflow-6-moving-specs-between-folders)
  - [Git Commit Integration](#workflow-7-git-commit-integration)
  - [Git Push & PR Handoff](#workflow-8-git-push--pr-handoff)
- [Common MCP Patterns](#common-mcp-patterns)
- [JSON Structure Reference](#json-structure-reference)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)
- [Common Mistakes](#common-mistakes)
- [Command Reference](#command-reference)
- [Systematic Spec Modification](#systematic-spec-modification)

---

## Workflow Details

### Workflow 1: Starting a Task

Mark a task as in_progress when you begin work:

```bash
mcp__foundry-mcp__task-update-status {spec-id} {task-id} in_progress
```

The MCP tool automatically records the start timestamp for tracking purposes.

---

### Workflow 2: Tracking Progress

#### Add Journal Entries

Document decisions, deviations, or important notes:

```bash
# Document a decision
mcp__foundry-mcp__journal add {spec-id} --title "Decision Title" --content "Explanation of decision and rationale" --task-id {task-id} --entry-type decision

# Document a deviation from the plan
mcp__foundry-mcp__journal add {spec-id} --title "Deviation: Changed Approach" --content "Created separate service file instead of modifying existing. Improves separation of concerns." --task-id {task-id} --entry-type deviation

# Document task completion (use status_change, NOT completion)
mcp__foundry-mcp__journal add {spec-id} --title "Task Completed: Implement Auth" --content "Successfully implemented authentication with JWT tokens. All tests passing." --task-id {task-id} --entry-type status_change

# Document a note
mcp__foundry-mcp__journal add {spec-id} --title "Implementation Note" --content "Using Redis for session storage as discussed." --task-id {task-id} --entry-type note
```

**Entry types:** `decision`, `deviation`, `blocker`, `note`, `status_change`

---

### Workflow 3: Handling Blockers

#### Mark Task as Blocked

When a task cannot proceed:

```bash
mcp__foundry-mcp__block-task {spec-id} {task-id} --reason "Description of blocker" --type {type} --ticket "TICKET-123"
```

**Blocker types:**
- `dependency` - Waiting on external dependency
- `technical` - Technical issue blocking progress
- `resource` - Resource unavailability
- `decision` - Awaiting architectural/product decision

#### Unblock Task

When blocker is resolved:

```bash
mcp__foundry-mcp__task-unblock {spec-id} {task-id} --resolution "Description of how it was resolved"
```

#### List All Blockers

```bash
mcp__foundry-mcp__list-blockers {spec-id}
```

---

### Workflow 4: Adding Verification Results

#### Manual Verification Recording

Document verification results:

```bash
# Verification passed
mcp__foundry-mcp__add-verification {spec-id} {verify-id} PASSED --command "npm test" --output "All tests passed" --notes "Optional notes"

# Verification failed
mcp__foundry-mcp__add-verification {spec-id} {verify-id} FAILED --command "npm test" --output "3 tests failed" --issues "List of issues found"

# Partial success
mcp__foundry-mcp__add-verification {spec-id} {verify-id} PARTIAL --notes "Most checks passed, minor issues remain"
```

#### Automatic Verification Execution

If verification tasks have metadata specifying how to execute them, run automatically:

```bash
# Execute verification based on metadata
mcp__foundry-mcp__execute-verify {spec-id} {verify-id}

# Execute and automatically record result
mcp__foundry-mcp__execute-verify {spec-id} {verify-id} --record
```

**Requirements:** Verification task must have `skill` or `command` in its metadata.

#### Verify on Task Completion

Automatically run verifications when marking a task complete:

```bash
mcp__foundry-mcp__task-update-status {spec-id} {task-id} completed --verify
```

The `--verify` flag runs all associated verify tasks. If any fail, the task reverts to `in_progress`.

#### Configurable Failure Handling

Verification tasks can specify custom failure behavior via `on_failure` metadata:

```json
{
  "verify-1-1": {
    "metadata": {
      "on_failure": {
        "consult": true,
        "revert_status": "in_progress",
        "max_retries": 2,
        "continue_on_failure": false
      }
    }
  }
}
```

**on_failure fields:**
- `consult` (boolean) - Recommend AI consultation for debugging
- `revert_status` (string) - Status to revert parent task to on failure
- `max_retries` (integer) - Number of automatic retry attempts (0-5)
- `continue_on_failure` (boolean) - Continue with other verifications if this fails

---

### Workflow 5: Completing Tasks

#### Complete a Task (Recommended: Atomic Status + Journal)

When finishing a task, use `complete-task` to atomically mark it complete AND create a journal entry:

```bash
# Complete with automatic journal entry
mcp__foundry-mcp__complete-task {spec-id} {task-id} --journal-content "Successfully implemented JWT authentication with token refresh. All tests passing including edge cases for expired tokens."

# Customize the journal entry
mcp__foundry-mcp__complete-task {spec-id} {task-id} --journal-title "Task Completed: Authentication Implementation" --journal-content "Detailed description of what was accomplished..." --entry-type status_change

# Add a brief status note
mcp__foundry-mcp__complete-task {spec-id} {task-id} --note "All tests passing" --journal-content "Implemented authentication successfully."
```

**What `complete-task` does automatically:**
1. Updates task status to `completed`
2. Records completion timestamp
3. Creates a journal entry documenting the completion
4. Clears the `needs_journaling` flag
5. Syncs metadata and recalculates progress
6. Automatically journals parent nodes (phases, groups) that auto-complete

#### Parent Node Journaling

When completing a task causes parent nodes (phases or task groups) to auto-complete, `complete-task` automatically creates journal entries for those parents:

- **Automatic detection**: The system detects when all child tasks in a phase/group are completed
- **Automatic journaling**: Creates journal entries like "Phase Completed: Phase 1" for each auto-completed parent
- **No manual action needed**: You don't need to manually journal parent completions
- **Hierarchical**: Works for multiple levels (e.g., completing a task can journal both its group AND its phase)

Example output:
```bash
$ mcp__foundry-mcp__complete-task my-spec-001 task-1-2 --journal-content "Completed final task"
✓ Task marked complete
✓ Journal entry added
✓ Auto-journaled 2 parent node(s): group-1, phase-1
```

#### Alternative: Status-Only Update (Not Recommended for Completion)

If you need to mark a task completed without journaling (rare), use:

```bash
mcp__foundry-mcp__task-update-status {spec-id} {task-id} completed --note "Brief completion note"
```

**Warning:** This sets `needs_journaling=True` and requires a follow-up `add-journal` call. Use `complete-task` instead to avoid forgetting to journal.

#### Complete a Spec

When all phases are verified and complete:

```bash
# Check if ready to complete
mcp__foundry-mcp__check-complete {spec-id}

# Complete spec (updates metadata, regenerates docs, moves to completed/)
mcp__foundry-mcp__complete-spec {spec-id}

# Skip documentation regeneration
mcp__foundry-mcp__complete-spec {spec-id} --skip-doc-regen
```

---

### Workflow 6: Moving Specs Between Folders

#### Activate from Backlog

Move a spec from pending/ to active/ when ready to start work:

```bash
mcp__foundry-mcp__activate-spec {spec-id}
```

This updates metadata status to "active" and makes the spec visible to sdd-next.

#### Move to Completed

Use `complete-spec` (see Workflow 5) to properly complete and move a spec.

#### Archive Superseded Specs

Move specs that are no longer relevant:

```bash
mcp__foundry-mcp__move-spec {spec-id} archived
```

---

### Workflow 7: Git Commit Integration

When git integration is enabled via the `[git]` block in `foundry-mcp.toml` (or the corresponding `FOUNDRY_MCP_GIT_*` environment variables), the sdd-update skill can automatically create commits after task completion based on the configured commit cadence.

#### Commit Cadence Configuration

The commit cadence determines when to offer automatic commits:

- **task**: Commit after each task completion (frequent commits, granular history)
- **phase**: Commit after each phase completion (fewer commits, milestone-based)
- **manual**: Never auto-commit (user manages commits manually)

Set `commit_cadence` inside the `[git]` section of `foundry-mcp.toml` (or expose it via `FOUNDRY_MCP_GIT_COMMIT_CADENCE`). The MCP server shares that value across all connected clients so they make the same commit decisions.

#### Commit Workflow Steps

When completing a task and git integration is enabled, the workflow follows these steps:

**1. Check Commit Cadence**

First, confirm whether automatic commits are enabled for the current event type. `sdd-update` reads the MCP configuration (`foundry-mcp.toml` → `[git]`) at runtime, so you only need to inspect that file (or use your preferred helper command) to see whether the cadence is set to `task`, `phase`, or `manual`.

**2. Check for Changes**

Before offering a commit, verify there are uncommitted changes:

```bash
# Check for changes (run in repo root directory)
git status --porcelain

# If output is empty, skip commit offer (nothing to commit)
# If output has content, proceed with commit workflow
```

**3. Generate Commit Message**

Create a structured commit message from task metadata using the pattern `{task-id}: {task-title}` (for example, `task-2-3: Implement JWT verification middleware`). The MCP helper applies this format automatically when generating commits.

**4. Preview and Stage Changes (Two-Step Workflow)**

The workflow now supports **agent-controlled file staging** with two approaches:

**Option A: Show Preview (Default - Recommended)**

When `file_staging.show_before_commit = true` (default), the agent sees uncommitted files and can selectively stage:

```bash
# Step 1: Preview uncommitted files (automatic via show_commit_preview_and_wait)
# Shows: modified, untracked, and staged files
mcp__foundry-mcp__complete-task SPEC_ID TASK_ID

# Step 2: Agent stages only task-related files
git add specs/active/spec.json
git add src/feature/implementation.py
git add tests/test_feature.py
# (Deliberately skip unrelated files like debug scripts, personal notes)

# Step 3: Create commit with staged files only
mcp__foundry-mcp__create-task-commit SPEC_ID TASK_ID
```

**Benefits:**
- Agent controls what files are committed
- Unrelated files protected from accidental commits
- Clean, focused task commits

**Option B: Auto-Stage All (Backward Compatible)**

When `file_staging.show_before_commit = false`, the old behavior is preserved:

```bash
# Automatically stages all files and commits (old behavior)
git add --all
git commit -m "{task-id}: {task-title}"
```

**Configuration:**

File staging behavior is controlled in `foundry-mcp.toml`:

```toml
[git]
enabled = true
auto_commit = true
commit_cadence = "task"
show_before_commit = true  # false = auto-stage all (backward compatible)
```

**Command Reference:**

```bash
# Complete task (shows preview if enabled)
mcp__foundry-mcp__complete-task SPEC_ID TASK_ID

# Create commit from staged files
mcp__foundry-mcp__create-task-commit SPEC_ID TASK_ID

# Example workflow:
mcp__foundry-mcp__complete-task user-auth-001 task-1-2
# (Review preview, stage desired files)
git add specs/active/user-auth-001.json src/auth/service.py
mcp__foundry-mcp__create-task-commit user-auth-001 task-1-2
```

**All git commands use `cwd=repo_root`** obtained from `find_git_root()` to ensure they run in the correct repository directory.

**5. Record Commit Metadata**

After successful commit, capture the commit SHA and update spec metadata:

```bash
# Get the commit SHA
git rev-parse HEAD

# Example output: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0
```

#### Error Handling

Git operations should be non-blocking - failures should not prevent task completion:

**Error handling principles:**

- Check `returncode` for all git commands
- Log warnings for failures but continue execution
- Git failures do NOT prevent task completion
- User can manually commit if automatic commit fails
- Provide clear error messages in logs for debugging

#### Repository Root Detection

All git commands must run in the repository root directory:

- `sdd-update` automatically discovers the repo root (similar to running `git rev-parse --show-toplevel` from the spec directory).
- If no git repository is detected, the commit workflow is skipped and the task still completes.
- Manual workflows should `cd` to the project root (the directory containing `.git/`) before running git commands.

#### Complete Example Workflow

1. Complete the task with `mcp__foundry-mcp__complete-task SPEC_ID TASK_ID`.
2. If git integration is enabled and cadence matches the event, `sdd-update` checks for uncommitted changes.
3. Stage files either via the preview workflow (recommended) or `git add --all` when auto-staging is enabled.
4. Run `mcp__foundry-mcp__create-task-commit SPEC_ID TASK_ID` to generate the commit; the command formats the message and records the SHA automatically.
5. If no changes are staged or the git command fails, the tool logs a warning and the task completion still succeeds.

#### Git Configuration File

Git integration is configured via the `[git]` section in `foundry-mcp.toml` (or the `FOUNDRY_MCP_GIT_*` environment variables):

```toml
[git]
enabled = true
auto_branch = true
auto_commit = true
auto_push = false
auto_pr = false
commit_cadence = "task"
```

**Settings:**
- `enabled`: Enable/disable all git integration features
- `auto_commit`: Offer automatic commits when the cadence matches the current event
- `commit_cadence`: When to commit - `"task"`, `"phase"`, or `"manual"`

---

### Workflow 8: Git Push & PR Handoff

When a spec is completed, the workflow can automatically push commits to the remote repository. Pull request creation is handed off to the dedicated PR workflow.

#### Push-Only Scope (PRs Deferred)

- `Skill(foundry:sdd-update)` handles local commits and optional pushes during completion workflows, but it does **not** create pull requests.
- Pull requests are beyond the scope of your responsibilities.
- If push automation is disabled or fails, finish the spec update, run `git push -u origin <branch>` manually, and then involve the PR skill/agent.

---

## Common MCP Patterns

### Preview Changes

Use `--dry-run` to preview changes before applying:

```bash
mcp__foundry-mcp__task-update-status {spec-id} {task-id} completed --dry-run
mcp__foundry-mcp__block-task {spec-id} {task-id} --reason "Test" --dry-run
```

### Query Spec State

```bash
# Get overall progress
mcp__foundry-mcp__status-report {spec-id}

# List all phases with progress
mcp__foundry-mcp__list-phases {spec-id}

# Find tasks by status
mcp__foundry-mcp__task-query {spec-id} --status pending
mcp__foundry-mcp__task-query {spec-id} --status blocked

# Find tasks by type
mcp__foundry-mcp__task-query {spec-id} --type verify

# Get specific task details
mcp__foundry-mcp__task-info {spec-id} {task-id}
```

### Update Metadata

```bash
# Update spec metadata fields
mcp__foundry-mcp__update-frontmatter {spec-id} status "active"
mcp__foundry-mcp__update-frontmatter {spec-id} owner "user@example.com"
mcp__foundry-mcp__update-frontmatter {spec-id} priority "high"

# Sync metadata with hierarchy state
mcp__foundry-mcp__sync-metadata {spec-id}
```

### Update Task Metadata

Update metadata fields for individual tasks using `update-task-metadata`:

```bash
# Update predefined metadata fields using individual flags
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} --file-path "src/auth.py"
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} --description "Updated task description"
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} --task-category "implementation"
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} --actual-hours 2.5

# Update custom metadata fields using JSON
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} \
  --metadata '{"focus_areas": ["performance", "security"], "priority": "high"}'

# Combine individual flags with custom JSON (flags take precedence)
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} \
  --file-path "src/middleware.py" \
  --metadata '{"focus_areas": ["authentication"], "complexity": "medium"}'

# Complex nested metadata structures
mcp__foundry-mcp__update-task-metadata {spec-id} {task-id} \
  --metadata '{
    "focus_areas": ["error handling", "edge cases"],
    "blockers": ["clarification needed", "dependency X"],
    "implementation_notes": {
      "approach": "incremental",
      "testing_strategy": "unit + integration"
    }
  }'
```

**Available individual flags:**
- `--file-path` - File path associated with this task
- `--description` - Task description
- `--task-category` - Category (e.g., implementation, testing, documentation)
- `--actual-hours` - Actual hours spent on task
- `--status-note` - Status note or completion note
- `--verification-type` - Verification type (run-tests, fidelity)
- `--command` - Command executed

**Custom metadata with --metadata:**
- Accepts JSON object with any custom fields
- Useful for tracking focus areas, priorities, blockers, complexity, etc.
- Merges with individual flags (individual flags take precedence)
- Supports nested structures and arrays

**Common use cases:**
```bash
# Track focus areas for investigation tasks
mcp__foundry-mcp__update-task-metadata {spec-id} task-1-1 \
  --metadata '{"focus_areas": ["code-doc structure", "skill patterns"]}'

# Document blockers and complexity
mcp__foundry-mcp__update-task-metadata {spec-id} task-2-3 \
  --metadata '{"blockers": ["API design unclear"], "complexity": "high"}'

# Track implementation approach
mcp__foundry-mcp__update-task-metadata {spec-id} task-3-2 \
  --metadata '{
    "approach": "refactor existing code",
    "estimated_subtasks": 4,
    "dependencies": ["task-3-1"]
  }'
```

### Validation

For comprehensive spec validation, use the sdd-validate skill:

```bash
Skill(foundry:sdd-validate) "Validate specs/active/{spec-id}.json"
```

For deep audits:

```bash
mcp__foundry-mcp__audit-spec {spec-id}
```

---

## JSON Structure Reference

### Task Status Values

- `pending` - Not yet started
- `in_progress` - Currently being worked on
- `completed` - Successfully finished
- `blocked` - Cannot proceed due to dependencies or issues

### Journal Entry Structure

Journal entries are stored in a top-level `journal` array:

```json
{
  "journal": [
    {
      "timestamp": "2025-10-18T14:30:00Z",
      "entry_type": "decision",
      "title": "Brief title",
      "task_id": "task-1-2",
      "author": "claude-sonnet-4.5",
      "content": "Detailed explanation",
      "metadata": {}
    }
  ]
}
```

### Verification Result Structure

Stored in verification task metadata:

```json
{
  "verify-1-1": {
    "metadata": {
      "verification_result": {
        "date": "2025-10-18T16:45:00Z",
        "status": "PASSED",
        "output": "Command output",
        "notes": "Additional context"
      }
    }
  }
}
```

### Folder Structure

```
specs/
├── pending/      # Backlog - planned but not activated
├── active/       # Currently being implemented
├── completed/    # Finished and verified
└── archived/     # Old or superseded
```

---

## Configuration Reference

### Fidelity Review Configuration

The optional pre-completion fidelity review behavior can be configured via `.claude/sdd_config.json`:

```json
{
  "fidelity_review": {
    "enabled": true,
    "on_task_complete": "prompt",
    "on_phase_complete": "always",
    "skip_categories": ["investigation", "research"],
    "min_task_complexity": "medium"
  }
}
```

**Configuration options:**

- `enabled` (boolean, default: `true`) - Master switch for fidelity review features
- `on_task_complete` (string, default: `"prompt"`) - When to offer fidelity review for task completion:
  - `"always"` - Automatically run fidelity review before marking any task complete
  - `"prompt"` - Ask user if they want to run fidelity review (recommended)
  - `"never"` - Skip automatic prompts, only use manual invocation or verification tasks

- `on_phase_complete` (string, default: `"always"`) - When to offer fidelity review for phase completion:
  - `"always"` - Automatically run phase-level fidelity review when all tasks in phase complete
  - `"prompt"` - Ask user if they want to run phase review
  - `"never"` - Skip automatic phase reviews

- `skip_categories` (array, default: `[]`) - Task categories that don't require fidelity review:
  - Common values: `["investigation", "research", "decision"]`
  - Tasks with these categories will skip automatic review prompts

- `min_task_complexity` (string, default: `"low"`) - Minimum task complexity for automatic review:
  - `"low"` - Review all tasks (most thorough)
  - `"medium"` - Only review medium/high complexity tasks
  - `"high"` - Only review high complexity tasks (least intrusive)

**When fidelity review is triggered:**

Based on configuration, when completing a task via `sdd-update`, the system will:

1. Check if task category is in `skip_categories` → skip if true
2. Check task complexity against `min_task_complexity` → skip if below threshold
3. Check `on_task_complete` setting:
   - `"always"` → Automatically invoke fidelity review via `Skill(foundry:sdd-fidelity-review)`
   - `"prompt"` → Ask user: "Run fidelity review before completing?"
   - `"never"` → Skip (user can still manually invoke)

**Note:** Verification tasks with `verification_type: "fidelity"` always run regardless of configuration.

---

## Troubleshooting

### Spec File Corruption

**Recovery:**
1. Check for backup: `specs/active/{spec-id}.json.backup`
2. If no backup, regenerate from original spec
3. Manually mark completed tasks based on journal entries
4. Validate repaired file

### Orphaned Tasks

**Resolution:**
1. If task in file but not in spec: Check if spec was updated; remove if confirmed deleted
2. If task in spec but not in file: Regenerate spec file using sdd-plan
3. Always preserve completed task history even if spec changed

### Merge Conflicts

**When:** Multiple tools update state simultaneously

**Resolution:**
1. Load both versions
2. Identify conflicting nodes
3. Choose most recent update (check timestamps)
4. Recalculate progress from leaf nodes up
5. Validate merged state

---

## Common Mistakes

### Using `--entry-type completion`

**Error:**
```bash
mcp__foundry-mcp__journal add: error: argument --entry-type: invalid choice: 'completion'
Exit code: 2
```

**Cause:** Confusing the `bulk-journal --template` option with `add-journal --entry-type`

**Fix:** Use `--entry-type status_change` instead:

```bash
# WRONG - "completion" is not a valid entry type
mcp__foundry-mcp__journal add {spec-id} --task-id {task-id} --entry-type completion --title "..." --content "..."

# CORRECT - Use "status_change" for task completion entries
mcp__foundry-mcp__journal add {spec-id} --task-id {task-id} --entry-type status_change --title "Task Completed" --content "..."

# ALTERNATIVE - Use bulk-journal with completion template
mcp__foundry-mcp__bulk-journal {spec-id} --template completion
```

**Why this happens:** The `bulk-journal` command has a `--template` parameter that accepts `completion` as a value for batch journaling. However, `add-journal` has an `--entry-type` parameter with different valid values. These are two separate parameters for different purposes:
- `bulk-journal --template completion` - Batch journal multiple completed tasks using a template
- `journal add --entry-type status_change` - Add individual journal entry about task status changes

### Reading Spec Files Directly

**Error:** Using Read tool, cat, grep, or jq on spec files

**Fix:** Always use the standardized MCP tools:

```bash
# WRONG - Wastes context tokens and bypasses validation
Read("specs/active/my-spec.json")
cat specs/active/my-spec.json

# CORRECT - Use the MCP tool for structured access
mcp__foundry-mcp__status-report {spec-id}
mcp__foundry-mcp__task-info {spec-id} {task-id}
mcp__foundry-mcp__task-query {spec-id} --status pending
```

---

## Command Reference

### Status Management
- `update-status` - Change task status
- `block-task` - Mark task as blocked with reason
- `unblock-task` - Unblock a task with resolution

### Documentation
- `journal add` - Add journal entry to spec
- `bulk-journal` - Add entries for multiple completed tasks
- `check-journaling` - Detect tasks without journal entries
- `add-verification` - Document verification results
- `execute-verify` - Run verification task automatically

### Lifecycle
- `activate-spec` - Move spec from pending/ to active/
- `move-spec` - Move spec between folders
- `complete-spec` - Mark complete and move to completed/

### Query & Reporting
- `status-report` - Get progress and status summary
- `query-tasks` - Filter tasks by status, type, or parent
- `task-info` - Get detailed task information
- `list-phases` - List all phases with progress
- `list-blockers` - List all blocked tasks
- `check-complete` - Verify if spec/phase is ready to complete

### Metadata
- `update-task-metadata` - Update task metadata fields (individual flags or JSON)
- `update-frontmatter` - Update spec metadata fields
- `sync-metadata` - Synchronize metadata with hierarchy

### Validation
- `validate-spec` - Check spec file consistency
- `audit-spec` - Deep audit of spec file integrity

### Common Flags
- `--dry-run` - Preview changes without saving
- `--verify` - Auto-run verify tasks on completion

---

## Systematic Spec Modification

For **structural modifications** to specs (not progress tracking), use `Skill(foundry:sdd-modify)`.

### What is Systematic Spec Modification?

Systematic modification applies structured changes to specs with:
- Automatic backup before changes
- Validation after changes
- Transaction support with rollback
- Preview before applying (dry-run mode)

### Common Use Cases

**1. Apply Review Feedback**

After running sdd-fidelity-review or sdd-plan-review:

```bash
# Parse review feedback into structured modifications
mcp__foundry-mcp__parse-review my-spec-001 --review reports/review.md --output suggestions.json

# Preview modifications
mcp__foundry-mcp__apply-modifications my-spec-001 --from suggestions.json --dry-run

# Apply modifications
mcp__foundry-mcp__apply-modifications my-spec-001 --from suggestions.json
```

**2. Bulk Modifications**

Apply multiple structural changes at once:

```bash
# Create modifications.json with desired changes
# (update task descriptions, add verifications, correct metadata)

# Apply bulk modifications
mcp__foundry-mcp__apply-modifications my-spec-001 --from modifications.json
```

**3. Update Task Descriptions**

Make task descriptions more specific based on implementation learnings:

```json
{
  "modifications": [
    {
      "operation": "update_task",
      "task_id": "task-2-1",
      "field": "description",
      "value": "Implement OAuth 2.0 authentication with PKCE flow and JWT tokens"
    }
  ]
}
```

### When to Use sdd-modify

Use `Skill(foundry:sdd-modify)` when you need to:
- Apply review feedback from sdd-fidelity-review or sdd-plan-review
- Update task descriptions for clarity (beyond just journaling)
- Add verification steps discovered during implementation
- Make multiple structural changes at once
- Ensure changes are validated and safely applied

### See Also

- **Skill(foundry:sdd-modify)** - Full documentation on systematic spec modification
- **skills/sdd-modify/examples/** - Detailed workflow examples
- **mcp__foundry-mcp__parse-review --help** - Parse review reports into modification format
- **mcp__foundry-mcp__apply-modifications --help** - Apply modifications with validation

---

## Best Practices

### When to Update

- **Update immediately** - Don't wait; update status as work happens
- **Be specific** - Vague notes aren't helpful later
- **Document WHY** - Always explain rationale, not just what changed

### Journaling

- **Link to evidence** - Reference tickets, PRs, discussions
- **Decision rationale** - Explain why decisions were made
- **Use bulk-journal** - Efficiently document multiple completed tasks

### Multi-Tool Coordination

- **Read before write** - Always load latest state before updating
- **Update your tasks only** - Don't modify other tools' work
- **Clear handoffs** - Add journal entry when passing work to another tool

### File Organization

- **Clean transitions** - Move specs promptly when status changes
- **Never rename specs** - Spec file names are based on spec_id
- **Backup before changes** - the MCP tooling handles automatic backups
