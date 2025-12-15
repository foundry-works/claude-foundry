# SDD-Fidelity-Review Reference

Detailed workflows, examples, and edge cases for the sdd-fidelity-review skill.

## Table of Contents

- [Long-Running Operations](#long-running-operations)
- [Review Types](#review-types)
- [Querying Spec Data](#querying-spec-and-task-data-efficiently)
- [Workflow Steps](#workflow)
- [Report Structure](#report-structure)
- [Integration with SDD Workflow](#integration-with-sdd-workflow)
- [Fidelity Assessment Categories](#fidelity-assessment)
- [Example Invocations](#example-invocations)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)
- [Subagent Investigation Patterns](#subagent-investigation-patterns)

---

## Long-Running Operations

**This skill may run operations that take up to 5 minutes. Be patient and wait for completion.**

### CRITICAL: Avoid BashOutput Spam
- **ALWAYS use foreground execution with 5-minute timeout:** `Bash(command="...", timeout=300000)`
- **WAIT for the command to complete** - this may take the full 5 minutes
- **NEVER use `run_in_background=True` for test suites, builds, or analysis**
- If you must use background (rare), **wait at least 60 seconds** between BashOutput checks
- **Maximum 3 BashOutput calls per background process** - then kill it or let it finish

### Why?
Polling BashOutput repeatedly creates spam and degrades user experience. Long operations should run in foreground with appropriate timeout, not in background with frequent polling.

### Example (CORRECT):
```python
# Test suite that might take 5 minutes (timeout in milliseconds)
result = Bash(command="pytest src/", timeout=300000)  # Wait up to 5 minutes
# The command will block here until completion - this is correct behavior
```

### Example (WRONG):
```python
# Don't use background + polling
bash_id = Bash(command="pytest", run_in_background=True)
output = BashOutput(bash_id)  # Creates spam!
```

---

## Review Types

### 1. Phase Review
**Scope:** Single phase within specification (typically 3-10 tasks)
**When to use:** Phase completion checkpoints, before moving to next phase
**Output:** Phase-specific fidelity report with per-task breakdown
**Best practice:** Use at phase boundaries to catch drift before starting next phase

### 2. Task Review
**Scope:** Individual task implementation (typically 1 file)
**When to use:** Critical task validation, complex implementation verification
**Output:** Task-specific compliance check with implementation comparison
**Best practice:** Use for high-risk tasks (auth, data handling, API contracts)

**Note:** For full spec reviews, run phase-by-phase reviews for better manageability and quality.

---

## Querying Spec and Task Data Efficiently

Before running a fidelity review, you may need to gather context about the spec, phases, or tasks. Use the following MCP tools instead of writing bash loops:

### Query Multiple Tasks at Once
```bash
# Get all tasks in a phase
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent="phase-1"

# Get tasks by status
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status="completed"

# Combine filters
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent="phase-2" status_filter="pending"
```

### Get Single Task Details
```bash
# Get detailed information about a specific task
mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id="task-1-3"
```

### List Phases
```bash
# See all phases with progress information
mcp__plugin_foundry_foundry-mcp__task action="list" spec_id={spec-id} include_phases=true
```

### DON'T Do This (Inefficient)
```bash
# BAD: Bash loop calling task info repeatedly
for i in 1 2 3 4 5; do
  mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id="task-1-$i"
done

# BAD: Creating temp scripts
cat > /tmp/get_tasks.sh << 'EOF'
...
EOF
```

### DO This Instead
```bash
# GOOD: Single command gets all tasks in phase-1
mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent="phase-1"
```

---

## Workflow

This skill delegates all fidelity review logic to the dedicated `mcp__plugin_foundry_foundry-mcp__review action="fidelity"` MCP tool, which handles spec loading, implementation analysis, AI consultation, and report generation.

### Step 1: Validate Inputs

Ensure the user provides:
- `spec_id`: The specification to review
- Either `task_id` (for task-level review) or `phase_id` (for phase-level review)

### Step 2: Construct MCP Invocation

Build the appropriate `mcp__plugin_foundry_foundry-mcp__review action="fidelity"` command based on review scope:

**For task review:**
```bash
mcp__plugin_foundry_foundry-mcp__review action="fidelity" spec_id={spec-id} task_id={task-id}
```

**For phase review:**
```bash
mcp__plugin_foundry_foundry-mcp__review action="fidelity" spec_id={spec-id} phase_id={phase-id}
```

### Step 3: Execute the MCP Tool

Invoke the MCP tool directly:
```bash
mcp__plugin_foundry_foundry-mcp__review action="fidelity" spec_id={spec-id} task_id={task-id}
```

**CRITICAL:** The MCP tool handles ALL spec file operations. Do NOT:
- Read spec files with Read tool
- Parse specs with Python or jq
- Use cat/head/tail/grep on spec files
- Create temporary bash scripts (e.g., `/tmp/*.sh`)
- Use bash loops to iterate through tasks
- Use grep/sed/awk to parse JSON outputs - all commands return structured JSON

**When you need task/spec context before running fidelity review:**
- Use `mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} parent={phase-id}` to get all tasks in a phase
- Use `mcp__plugin_foundry_foundry-mcp__task action="query" spec_id={spec-id} status={status}` to filter by status
- Use `mcp__plugin_foundry_foundry-mcp__task action="info" spec_id={spec-id} task_id={task-id}` to get a single task's details
- Use `mcp__plugin_foundry_foundry-mcp__task action="list" spec_id={spec-id} include_phases=true` to see all phases

Then execute the fidelity review with the appropriate scope.

Your job is to execute the MCP tool and parse its JSON output.

The MCP tool handles:
- Loading and validating the specification
- Extracting task/phase requirements
- Analyzing implementation files
- Consulting AI tools (gemini, codex, cursor-agent) for deviation analysis
- Detecting consensus across multiple AI perspectives
- Categorizing deviations (exact match, minor, major, missing)
- Assessing impact levels
- Generating structured report

### Step 4: Parse and Present Results

The MCP tool returns JSON output with the structure:
```json
{
  "spec_id": "...",
  "review_type": "task|phase",
  "scope": {"id": "...", "title": "..."},
  "summary": {
    "tasks_reviewed": 0,
    "files_analyzed": 0,
    "deviations_found": 0,
    "fidelity_score": 0
  },
  "findings": [
    {
      "task_id": "...",
      "assessment": "exact_match|minor_deviation|major_deviation|missing",
      "deviations": [...],
      "impact": "low|medium|high",
      "ai_consensus": "..."
    }
  ],
  "recommendations": [...]
}
```

Parse this JSON, surface the structured findings directly to the invoking workflow, and include a link or path to the saved JSON report. The agent's responsibility is to faithfully relay the MCP tool's assessed deviations, recommendations, consensus signals, and metadata—perform validation/formatting as needed, but do not introduce any new analysis beyond what the tool returned. Do not open or read the saved artifact; simply point the caller to its location.

---

## Report Structure

```markdown
# Implementation Fidelity Review

**Spec:** {spec-title} ({spec-id})
**Scope:** {review-scope}
**Date:** {review-date}

## Summary

- **Tasks Reviewed:** {count}
- **Files Analyzed:** {count}
- **Overall Fidelity:** {percentage}%
- **Deviations Found:** {count}

## Fidelity Score

- Exact Matches: {count} tasks
- Minor Deviations: {count} tasks
- Major Deviations: {count} tasks
- Missing Functionality: {count} items

## Detailed Findings

### Task: {task-id} - {task-title}

**Specified:**
- {requirement-1}
- {requirement-2}

**Implemented:**
- {actual-1}
- {actual-2}

**Assessment:** {exact-match|minor-deviation|major-deviation}

**Deviations:**
1. {deviation-description}
   - **Impact:** {low|medium|high}
   - **Recommendation:** {action}

## Recommendations

1. {recommendation-1}
2. {recommendation-2}

## Journal Analysis

**Documented Deviations:**
- {task-id}: {deviation-summary} (from journal on {date})

**Undocumented Deviations:**
- {task-id}: {deviation-summary} (should be journaled)
```

---

## Integration with SDD Workflow

### When to Invoke

Fidelity reviews can be triggered at multiple points in the development workflow:

**1. After task completion (Optional verification):**
   - Optional verification step for critical tasks
   - Ensures task acceptance criteria fully met
   - Particularly useful for high-risk tasks (auth, data handling, API contracts)
   - Can be automated via verification task metadata

**2. Phase completion (Recommended):**
   - Review entire phase before moving to next phase
   - Catch drift early before it compounds
   - Best practice: Use at phase boundaries
   - Prevents accumulated technical debt

**3. Spec completion (Pre-PR audit):**
   - Final comprehensive audit before PR creation
   - Run phase-by-phase reviews for better quality
   - Ensures PR matches spec intent
   - Documents any deviations for PR description

**4. PR review (Automated compliance):**
   - Automated or manual PR compliance checks
   - Verify changes align with original specification
   - Useful for reviewer context and validation

### Review Outcome Handling

The `sdd-fidelity-review` skill hands its synthesized results—JSON findings plus the saved JSON report reference—directly back to the caller. The invoking workflow decides what to do next. Common follow-up actions the main agent may optionally consider include journaling deviations, planning remediation work, running regression tests, or proposing spec updates after stakeholder review. No automatic delegation occurs; the fidelity-review skill's responsibility ends once it delivers the consensus results and report pointer.

### Report Handoff

Fidelity review generates a detailed report comparing implementation against specification:

**Usage Pattern:**
1. Skill executes `mcp__plugin_foundry_foundry-mcp__review action="fidelity"` directly via MCP
2. The tool analyzes implementation, generates JSON output, and saves the JSON consensus report in `.fidelity-reviews/`
3. Skill parses the JSON, presents the summarized findings, and surfaces the stored report path to the caller

---

## Fidelity Assessment

### Exact Match
Implementation precisely matches specification requirements. No deviations detected.

### Minor Deviation
Small differences from spec with no functional impact:
- Different variable names (but consistent with codebase style)
- Minor refactoring for code quality
- Improved error messages
- Additional logging or comments

### Major Deviation
Significant differences affecting functionality or architecture:
- Different API signatures than specified
- Missing required features
- Different data structures
- Changed control flow or logic

### Missing Functionality
Specified features not implemented:
- Required functions missing
- Incomplete implementation
- Skipped acceptance criteria

---

## Example Invocations

**Phase review:**
```bash
Skill(foundry:sdd-fidelity-review) "Review phase phase-1 in spec user-auth-001"
```

**Task-specific review:**
```bash
Skill(foundry:sdd-fidelity-review) "Review task task-2-3 in spec user-auth-001"
```

---

## Error Handling

### Missing Required Information
If invoked without required information, the skill returns a structured error indicating which fields are missing.

### Spec Not Found
If the specified spec file doesn't exist, the skill reports which directories were searched and suggests verification steps.

### No Implementation Found
If the specified files don't exist, the skill warns that the task appears incomplete or the file paths are incorrect.

---

## Best Practices

### DO
- Validate that spec_id and task_id/phase_id are provided
- Present findings clearly with categorized deviations
- Highlight recommendations for remediation
- Note AI consensus from multiple tool perspectives
- Provide context from the fidelity assessment
- Surface the saved JSON report path so the caller can inspect the full consensus artifacts

### DON'T
- Attempt to manually implement review logic (the MCP tool handles it)
- Read spec files directly with Read/Python/jq/Bash (the MCP tool loads them)
- Use grep/sed/awk to parse JSON—the MCP responses are already structured
- Ignore the MCP tool's consensus analysis
- Make up review findings not sourced from the MCP output
- Perform additional analysis beyond the MCP consensus results
- Open the persisted JSON report; reference its filepath instead

---

## Subagent Investigation Patterns

For complex fidelity reviews, leverage Claude Code's built-in subagents to gather context before and investigate findings after the MCP review.

### Built-in Subagents for Fidelity Review

| Subagent | Model | Use Case |
|----------|-------|----------|
| **Explore** | Haiku | Fast file discovery, implementation mapping |
| **general-purpose** | Sonnet | Complex deviation analysis requiring code reading |

### Pre-Review Context Gathering

Before running `mcp__plugin_foundry_foundry-mcp__review action="fidelity"`, gather implementation context:

**Phase review preparation:**
```
Use the Explore agent (medium thoroughness) to find:
- All files matching the phase's task file_path patterns
- Test files corresponding to implementation files
- Configuration or setup files that affect behavior
- Recent git changes in the implementation area
```

**Task review preparation:**
```
Use the Explore agent (quick thoroughness) to find:
- The implementation file and its imports
- Related test file for the task
- Any configuration the implementation depends on
```

### Post-Review Investigation

After receiving fidelity results, investigate deviations:

**For major deviations:**
```
Use the Explore agent (very thorough) to investigate:
- Why the implementation differs from spec
- What constraints led to the deviation
- Whether deviation is documented (comments, commits, journals)
- Impact on dependent files
```

**For missing functionality:**
```
Use the Explore agent (medium thoroughness) to find:
- Whether the feature exists elsewhere (different file/name)
- Related implementations that might explain the gap
- Test coverage for the expected functionality
```

### When to Use Subagents vs Direct Review

**Use Explore subagent when:**
- Phase spans many files you haven't read
- Deviations are unclear and need investigation
- Need to understand implementation context first
- Want to preserve main context for review results

**Skip subagent exploration when:**
- Single task review in familiar code
- Implementation files already in context
- Quick validation of known changes
- Near context limit

### Benefits

| Benefit | Description |
|---------|-------------|
| **Context isolation** | File searches don't bloat main conversation |
| **Faster discovery** | Haiku model finds files quickly |
| **Focused investigation** | Post-review exploration targets specific deviations |
| **Parallel preparation** | Can explore while formulating review strategy |

---

## Success Criteria

A successful fidelity review:
- Compares all specified requirements against implementation
- Identifies and categorizes deviations accurately
- Assesses impact of deviations
- Provides actionable recommendations
- Generates clear, structured report
- Automatically saves to `specs/.fidelity-reviews/` directory
- Documents findings for future reference
