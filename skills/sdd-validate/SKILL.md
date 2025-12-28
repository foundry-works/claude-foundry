---
name: sdd-validate
description: Validate SDD JSON specs, auto-fix common issues, generate detailed reports, and analyze dependencies.
---

# Spec Validation Skill

## Overview

The **Skill(foundry:sdd-validate)** skill provides comprehensive validation for Spec-Driven Development (SDD) JSON specification files. It checks for structural consistency, auto-fixes common issues, generates detailed reports, and analyzes dependencies.

**Key capabilities:**
- Validate JSON spec structure and hierarchy integrity
- Auto-fix 13 common issue types with preview and backup support
- Generate detailed validation reports in Markdown or JSON
- Calculate comprehensive spec statistics (depth, coverage, complexity)
- Analyze dependencies (cycles, orphans, deadlocks, bottlenecks)
- Differentiated exit codes for warnings vs errors
- Draft 07 JSON Schema validation (installs automatically when `jsonschema` is available; run `pip install claude-skills[validation]` to enable)

## Understanding Exit Codes

Exit codes indicate the **state of your spec**, not command success/failure:

- **Exit 0**: ✅ Spec is valid (no errors)
- **Exit 1**: ⚠️  Spec has warnings (usable but improvable)
- **Exit 2**: 🔧 Spec has errors (needs fixing) - **THIS IS EXPECTED FOR NEW SPECS**
- **Exit 3**: ❌ File not found or access error (actual command failure)

**Important:** Exit code 2 means "I found errors in your spec" not "I failed to validate". The validate command succeeded at detecting issues - your spec just needs work. This is the normal starting point for most specs.

## When to Use This Skill

Use `Skill(foundry:sdd-validate)` to:
- Confirm a freshly created spec parses correctly
- Auto-fix common validation issues
- Check for structural errors before running other SDD skills
- Generate validation reports for review or CI/CD
- Analyze spec statistics and dependency issues

## MCP Tooling

- This skill interacts solely with the Foundry MCP server (`foundry-mcp`). Tools use the router+action pattern: `mcp__plugin_foundry_foundry-mcp__<router>` with `action="<action>"` (for example, `mcp__plugin_foundry_foundry-mcp__spec action="validate"`, `mcp__plugin_foundry_foundry-mcp__spec action="fix"`).
- There is no CLI fallback. All instructions correspond to MCP invocations with named parameters.
- Stay inside the repo root and rely on these helpers instead of reading spec JSON directly.

| Router | Key Actions |
|--------|-------------|
| `spec` | `validate`, `fix`, `stats`, `analyze-deps`, `diff`, `history`, `completeness-check`, `duplicate-detection` |

## Core Workflow

> `[x?]`=decision · `(GATE)`=user approval · `→`=sequence · `↻`=loop · `§`=section ref

```
- **Entry** → `spec action="validate"`
  - [exit 0] → **Exit**: valid
  - [exit 3] → **Exit**: file error
  - [exit 1-2] → continue
- [AutoFixable?] → `spec action="fix"` (backup created)
- ReValidate ↻ [errors decreasing?]
  - [yes] → ↻ back to Fix
  - [plateau 2+] → ManualFixes with verbose
- **Exit**: valid
```

**Key concept:** Error count decreasing = keep fixing. Plateau (same count for 2+ passes) = switch to manual fixes.

## Key Concepts

### Always Re-validate After Fixing
Fixing issues often reveals new problems that were previously hidden. Always run `mcp__plugin_foundry_foundry-mcp__spec action="validate"` after `mcp__plugin_foundry_foundry-mcp__spec action="fix"` to see the current state.

### Error Count Progression
Track how error count changes across passes:
- **Decreasing** (88 → 23 → 4) - Continue auto-fixing
- **Plateau** (4 → 4 → 4) - Switch to manual fixes with `--verbose`

### When to Stop Auto-Fixing
Switch to manual intervention when:
- Error count unchanged for 2+ passes
- `mcp__plugin_foundry_foundry-mcp__spec action="fix"` reports "skipped issues requiring manual intervention"
- All remaining issues need context or human judgment

### Input Format: Spec Names vs Paths

All validation commands accept **both** spec names and paths for maximum flexibility:

**Spec name (recommended):**
```bash
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="pomodoro-timer-2025-11-18-001"
```
Automatically searches in `specs/pending/`, `specs/active/`, `specs/completed/`, and `specs/archived/`.

**Relative path:**
```bash
mcp__plugin_foundry_foundry-mcp__spec action="validate" path="specs/pending/pomodoro-timer-2025-11-18-001.json"
mcp__plugin_foundry_foundry-mcp__spec action="validate" path="../other-project/specs/active/my-spec.json"
```

**Absolute path:**
```bash
mcp__plugin_foundry_foundry-mcp__spec action="validate" path="/full/path/to/my-spec.json"
```

**Smart fallback:** If you provide a path that doesn't exist, the command extracts the spec name and searches for it automatically.

## Command Reference

### validate (check)

Validate the JSON spec structure and print a summary.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="{spec-id}"
```

**Exit codes:**
- `0` - Clean validation
- `1` - Warnings only
- `2` - Errors detected (expected when spec has issues)

**Example output:**
```
❌ Validation found 12 errors
   8 auto-fixable, 4 require manual intervention

   Errors:
   - 5 incorrect task count rollups
   - 2 missing metadata blocks
   - 1 orphaned node (task-5-3)
   - 2 circular dependencies
   - 2 parent/child mismatches

   Run 'spec action="fix"' to auto-fix 8 issues
```

### fix

Auto-fix common validation issues with preview and backup support.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="fix" spec_id="{spec-id}" dry_run=true
```

**Parameters:**
- `dry_run=true` - Show what would be fixed without modifying files
- `create_backup=false` - Skip backup creation (use with caution)

**Auto-fixable issues:**
- Incorrect task count rollups
- Missing metadata blocks
- Orphaned nodes
- Parent/child hierarchy mismatches
- Malformed timestamps
- Invalid status/type values
- Bidirectional dependency inconsistencies

**Example:**
```bash
# Preview fixes
mcp__plugin_foundry_foundry-mcp__spec action="fix" spec_id="my-spec" dry_run=true
📋 Would apply 8 fixes:
   - Fix 5 task count rollups
   - Add 2 metadata blocks
   - Reconnect 1 orphaned node

⚠️  Would skip 4 issues requiring manual intervention:
   - task-3-2: Circular dependency
   - task-5-2: Dependency references non-existent task

# Apply fixes
mcp__plugin_foundry_foundry-mcp__spec action="fix" spec_id="my-spec"
✅ Applied 8 fixes
Created backup: my-spec.json.backup

# Re-validate
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="my-spec"
❌ Validation found 4 errors (manual intervention required)
```

### stats

Calculate and display comprehensive spec statistics.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="stats" spec_id="{spec-id}"
```

**Shows:**
- Node, task, phase, and verification counts
- Status breakdown (pending, in_progress, completed, blocked)
- Hierarchy maximum depth
- Average tasks per phase
- Verification coverage percentage
- Overall progress percentage

### analyze-deps

Analyze dependencies for cycles, orphans, deadlocks, and bottlenecks.

**Note:** This command analyzes spec-wide dependency issues. For checking individual task dependencies, use `mcp__plugin_foundry_foundry-mcp__task action="prepare"` from sdd-next which includes dependency details by default in `context.dependencies`.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="analyze-deps" spec_id="{spec-id}" bottleneck_threshold=3
```

**Analyzes:**
- **Cycles** - Circular dependency chains
- **Orphaned** - Tasks referencing missing dependencies
- **Deadlocks** - Tasks blocked by each other
- **Bottlenecks** - Tasks blocking many others

**Example:**
```
⚠️  Dependency Analysis: 3 issues found

Cycles (2):
  1. task-3-2 → task-3-5 → task-3-2
  2. task-4-1 → task-4-3 → task-4-1

Orphaned dependencies (1):
  task-5-2 references non-existent "task-2-9"

Recommendation: Fix circular dependencies first to unblock work
```

### diff

Compare two versions of a spec to see what changed.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="{spec-id}" compare_to="{backup-path-or-version}"
```

**Parameters:**
- `compare_to` - Path to backup file or version identifier to compare against

**Output format:**
```
📊 Spec Diff: my-spec-001

Tasks:
  + task-2-5: New authentication handler (added)
  ~ task-1-3: Description updated
  - task-3-1: Removed (was: Legacy endpoint)

Metadata:
  ~ version: 1.0.0 → 1.1.0
  + owner: Added "team-alpha"

Dependencies:
  + task-2-5 → task-1-2 (new dependency)

Summary: 1 added, 2 modified, 1 removed
```

### history

View modification history for a spec with optional filtering.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="{spec-id}" limit=10
```

**Parameters:**
- `limit` - Maximum entries to return (default: 20)
- `since` - Filter entries after this timestamp (ISO 8601)
- `until` - Filter entries before this timestamp (ISO 8601)
- `entry_type` - Filter by type: `status_change`, `modification`, `validation`, `backup`

**Example:**
```bash
# Recent history
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" limit=5

📜 History: my-spec-001 (5 most recent)

2025-12-28T10:30:00Z [status_change] task-2-1: pending → completed
2025-12-28T09:15:00Z [modification] Added phase-3 with 4 tasks
2025-12-27T16:45:00Z [validation] Fixed 3 orphaned nodes
2025-12-27T14:20:00Z [backup] Created before bulk modification
2025-12-27T10:00:00Z [status_change] Spec activated

# Filter by date range
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" since="2025-12-27" entry_type="status_change"
```

### completeness-check

Analyze spec completeness and quality with scoring.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="completeness-check" spec_id="{spec-id}"
```

**Scoring criteria:**
- **Structure (25%)**: All tasks have parents, no orphans, valid hierarchy
- **Metadata (25%)**: Descriptions, estimates, acceptance criteria present
- **Dependencies (25%)**: Dependencies declared, no cycles, proper ordering
- **Verification (25%)**: Verify tasks present, coverage adequate

**Example output:**
```
📋 Completeness Check: my-spec-001

Overall Score: 78/100 (Good)

Structure:     ████████░░  22/25  (1 orphaned task)
Metadata:      ██████████  25/25  (all tasks have descriptions)
Dependencies:  ██████░░░░  16/25  (3 tasks missing dependencies)
Verification:  ███████░░░  15/25  (2 phases lack verify tasks)

Recommendations:
- Add verify task to phase-2 and phase-4
- Declare dependencies for task-3-1, task-3-2, task-4-5
- Reconnect orphaned task-2-7 to parent phase
```

### duplicate-detection

Detect duplicate or near-duplicate tasks within a spec.

```bash
mcp__plugin_foundry_foundry-mcp__spec action="duplicate-detection" spec_id="{spec-id}"
```

**Detects:**
- Exact title matches across different phases
- Similar descriptions (fuzzy matching)
- Overlapping acceptance criteria
- Redundant verification tasks

**Example output:**
```
🔍 Duplicate Detection: my-spec-001

Found 3 potential duplicates:

1. EXACT MATCH (titles):
   - task-2-3: "Add user validation"
   - task-4-1: "Add user validation"
   Resolution: Merge into single task or differentiate titles

2. SIMILAR (85% description match):
   - task-1-5: "Implement error handling for API calls"
   - task-3-2: "Add error handling to API endpoints"
   Resolution: Review if these cover different scopes

3. OVERLAPPING CRITERIA:
   - verify-2-1 and verify-3-1 both check "all tests pass"
   Resolution: Consider consolidating verification or specifying scope

No action required for 45 other tasks (unique)
```

**Resolution guidance:**
- **Exact matches**: Usually indicates copy-paste error; merge or rename
- **Similar descriptions**: May be intentional (different phases); add clarifying context
- **Overlapping criteria**: Narrow scope of each verification task

## Common Patterns

> For detailed issue-to-fix mapping, see `references/issue-mapping.md`

### Issue → Fix Mapping

- **Incorrect task counts** → Auto-fixed by recalculating from hierarchy
- **Missing metadata** → Auto-fixed by adding empty metadata blocks
- **Orphaned nodes** → Auto-fixed by reconnecting to parent
- **Circular dependencies** → Requires manual fix (remove one edge)
- **Invalid timestamps** → Auto-fixed to ISO 8601 format
- **Parent/child mismatches** → Auto-fixed by correcting hierarchy references

### Typical Fix Cycles

**Most specs (2-3 passes):**
```bash
Pass 1: 47 errors → fix → 8 errors
Pass 2: 8 errors → fix → 2 errors
Pass 3: 2 errors → fix → 0 errors ✅
```

**Complex specs with circular deps (4-5 passes):**
```bash
Pass 1: 88 errors → fix → 23 errors
Pass 2: 23 errors → fix → 4 errors
Pass 3: 4 errors → fix → 4 errors (plateau)
→ Manual: Remove circular dependencies
Pass 4: Validate → 0 errors ✅
```

## Troubleshooting

For detailed troubleshooting, see **[references/troubleshooting.md](./references/troubleshooting.md)**.

**Quick tips:**
- Re-validate after every fix pass
- Plateau (same error count 2+ passes) = switch to manual fixes
- 2-3 passes is normal; 5+ suggests circular dependencies
- Use `mcp__plugin_foundry_foundry-mcp__spec action="schema-export"` to understand valid spec structure

## Advanced Usage

### Global Flags

Available on all commands:
- `--quiet` / `-q` - Suppress progress messages
- `--verbose` / `-v` - Show detailed information

## Detailed Reference

For comprehensive documentation including:
- Issue-to-fix mapping → `references/issue-mapping.md`
- Manual fix patterns → `references/manual-fixes.md`
- Troubleshooting → `references/troubleshooting.md`
- Edge cases → `references/edge-cases.md`
