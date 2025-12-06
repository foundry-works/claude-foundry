---
name: sdd-fidelity-review
description: Review implementation fidelity against specifications by comparing actual code to spec requirements. Identifies deviations, assesses impact, and generates compliance reports for tasks, phases, or entire specs.
---

# Implementation Fidelity Review Skill

## Overview

The `sdd-fidelity-review` skill compares actual implementation against SDD specification requirements to ensure fidelity between plan and code. It identifies deviations, assesses their impact, and generates detailed compliance reports.

## Skill Family

This skill is part of the **Spec-Driven Development** quality assurance family:

```
sdd-plan → sdd-next → Implementation → sdd-update → sdd-fidelity-review (this skill) → run-tests
```

## When to Use This Skill

Use this skill when you need to:
- Verify implementation matches specification requirements
- Identify deviations between plan and actual code
- Assess task or phase completion accuracy
- Review pull requests for spec compliance
- Audit completed work for fidelity

**Do NOT use for:**
- Creating specifications (use `sdd-plan`)
- Finding next tasks (use `sdd-next`)
- Updating task status (use `sdd-update`)
- Running tests (use `run-tests`)

## MCP Tooling

This skill relies entirely on the Foundry MCP server (`foundry-mcp`). Tool names follow the `mcp__foundry-mcp__<tool-name>` pattern.

**Critical Rules:**
- **ALWAYS** use MCP tools for spec operations
- **NEVER** use `Read()` on spec JSON files
- **NEVER** use shell commands (`cat`, `grep`, `jq`) on specs
- **NEVER** create temp scripts or bash loops for task iteration

## Review Types

| Type | Scope | When to Use |
|------|-------|-------------|
| **Phase Review** | 3-10 tasks | Phase completion checkpoints |
| **Task Review** | 1 file | Critical task validation, high-risk implementations |

> For detailed workflow steps per review type, see `reference.md#review-types`

## Essential Commands

**Phase review:**
```bash
mcp__foundry-mcp__review-fidelity <spec-id> --phase <phase-id>
```

**Task review:**
```bash
mcp__foundry-mcp__review-fidelity <spec-id> --task <task-id>
```

**Query tasks before review:**
```bash
mcp__foundry-mcp__task-query <spec-id> --parent <phase-id> --json
```

> For query patterns and anti-patterns, see `reference.md#querying-spec-and-task-data-efficiently`

## Fidelity Assessment Categories

| Category | Meaning |
|----------|---------|
| **Exact Match** | Implementation precisely matches specification |
| **Minor Deviation** | Small differences with no functional impact |
| **Major Deviation** | Significant differences affecting functionality |
| **Missing** | Specified features not implemented |

## Report Structure

Generate reports in this format:

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

**Assessment:** {exact-match|minor-deviation|major-deviation}

**Deviations:**
1. {deviation-description}
   - **Impact:** {low|medium|high}
   - **Recommendation:** {action}

## Recommendations

1. {recommendation-1}
2. {recommendation-2}
```

> For complete template with all sections, see `reference.md#report-structure`

## When to Invoke

1. **Phase completion** (Recommended) - Before moving to next phase
2. **Task completion** - For high-risk tasks (auth, data, APIs)
3. **Spec completion** - Pre-PR audit
4. **PR review** - Compliance checks

## Long-Running Operations

**This skill may take up to 5 minutes.** Always use foreground execution:

```python
Bash(command="mcp__foundry-mcp__review-fidelity ...", timeout=300000)
```

**Never** use `run_in_background=True` with frequent polling.

## Example Invocation

```bash
Skill(foundry:sdd-fidelity-review) "Review phase phase-1 in spec user-auth-001"
```

## Detailed Reference

For comprehensive documentation including:
- Complete workflow steps with examples
- Report structure template
- Integration with SDD workflow
- Query patterns and anti-patterns
- Error handling scenarios
- Best practices (DO/DON'T lists)

See **[reference.md](./reference.md)**
