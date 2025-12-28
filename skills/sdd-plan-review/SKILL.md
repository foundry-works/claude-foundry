---
name: sdd-plan-review
description: Multi-model consultation for SDD specifications providing structured feedback. Coordinates parallel AI reviewers, synthesizes actionable insights, and categorizes findings by feedback type without modifying specs or executing fixes.
---

# SDD Plan Review Skill

## Overview

The `sdd-plan-review` skill coordinates parallel AI reviewers to provide structured, categorized feedback on SDD specifications. It surfaces consensus findings, diverse perspectives, and prioritized recommendations to inform spec quality decisions.

**Core Philosophy:**
- **Diverse Perspectives Improve Quality**: Multiple AI models catch more issues than single review
- **Feedback, Not Gatekeeping**: Provides actionable feedback, not approval/rejection
- **Advisory Only**: Never modifies specifications - all findings are recommendations

## Skill Family

This skill is part of the **Spec-Driven Development** quality assurance family:

```
sdd-plan → sdd-plan-review (this skill) → sdd-modify → sdd-next → Implementation
```

## When to Use This Skill

Use this skill when you need to:
- Review draft specs before implementation begins
- Get multi-model feedback on complex specifications
- Validate security-critical designs (auth, data, privacy)
- Assess feasibility of estimates and dependencies
- Identify risks in novel architectures

**Do NOT use for:**
- Trivial or low-risk specs (<5 tasks, standard patterns)
- Specs already in implementation (use `sdd-fidelity-review`)
- Editing specifications (use `sdd-modify`)
- Exploratory or prototype work

## MCP Tooling

This skill relies entirely on the Foundry MCP server (`foundry-mcp`). Tools use the router+action pattern: `mcp__plugin_foundry_foundry-mcp__<router>` with `action="<action>"`.

| Router | Actions | Purpose |
|--------|---------|---------|
| `review` | `spec-review`, `list-tools`, `list-plan-tools` | Execute reviews and list toolchains |
| `spec` | `get`, `get-hierarchy`, `stats`, `list` | Get spec content and structure |
| `spec` | `completeness-check`, `duplicate-detection` | Automated quality checks |
| `provider` | `list`, `status`, `execute` | List and manage AI providers |

**Critical Rules:**
- **ALWAYS** use MCP tools for spec operations
- **NEVER** use `Read()` on spec JSON files directly
- **NEVER** shell out to `cat`, `grep`, `jq` for spec parsing

### Automated Quality Checks

Run automated checks before or alongside AI review to catch structural issues:

```bash
# Check spec completeness (metadata, descriptions, estimates)
mcp__plugin_foundry_foundry-mcp__spec action="completeness-check" spec_id="{spec-id}"

# Detect duplicate or overlapping tasks
mcp__plugin_foundry_foundry-mcp__spec action="duplicate-detection" spec_id="{spec-id}"
```

**Integration with review types:**

| Review Type | Recommended Quality Checks |
|-------------|---------------------------|
| `quick` | `completeness-check` (fast structural validation) |
| `full` | Both checks before AI review |
| `security` | `completeness-check` (ensure all security requirements specified) |
| `feasibility` | `duplicate-detection` (avoid double-counting estimates) |

**Workflow integration:**
- Run quality checks **before** AI review to catch obvious issues
- Include findings in review context to focus AI on substantive concerns
- Use `duplicate-detection` when specs have similar task titles across phases

## Review Types

| Type | Models | Duration | Focus | Use When |
|------|--------|----------|-------|----------|
| **quick** | 2 | 10-15 min | Completeness, Clarity | Simple specs, time-constrained |
| **full** | 3-4 | 20-30 min | All 6 dimensions | Complex specs, moderate-high risk |
| **security** | 2-3 | 15-20 min | Risk Management | Auth, data handling, compliance |
| **feasibility** | 2-3 | 10-15 min | Estimates, Dependencies | Tight deadlines, uncertain scope |

> For detailed dimension descriptions, see `references/dimensions.md`

## Essential Command

```bash
mcp__plugin_foundry_foundry-mcp__review action="spec-review" spec_id="{spec-id}" review_type="{type}"
```

Use `action="list-tools"` to check available toolchains before running.

## Core Workflow

> `[x?]`=decision · `(GATE)`=user approval · `→`=sequence · `↻`=loop · `§`=section ref

```
- **Entry** → `review action="list-tools"` → [tools ≥1?]
  - [no] → **Exit**: no toolchains available
- QualityChecks → `spec action="completeness-check"` + `spec action="duplicate-detection"`
  - Surface structural issues before AI review
- [Type: quick|full|security|feasibility?] → see table above
- Execute → `review action="spec-review"` (2-4 models parallel)
- Synthesize → PrioritySort[CRITICAL→LOW]
- Report → summary + file paths
- (GATE: handoff) → [proceed to sdd-modify?] → **Exit**
```

**Quality checks in workflow:**
1. Run `completeness-check` to catch missing metadata, descriptions, or estimates
2. Run `duplicate-detection` to find overlapping tasks before AI wastes time reviewing them
3. Include findings in context so AI focuses on substantive design issues

> For detailed workflow steps, see `references/workflow.md`

## Output

- Reports written to `specs/.plan-reviews/{spec-id}-review-{type}.md`
- Full results go to files; conversation receives summary only

> For output format and isolation details, see `references/output.md`

## Example Invocation

```bash
Skill(foundry:sdd-plan-review) "Review spec user-auth-001 with 'full' review type. Check all 6 dimensions and surface consensus findings."
```

## Key Defaults

| Parameter | Value |
|-----------|-------|
| Min models required | 1 |
| Recommended models | 2+ |
| Timeout per model | 120s |
| Max review time | 300s |

> For full configuration, see `references/configuration.md`

## Detailed Reference

For comprehensive documentation including:
- Detailed workflow → `references/workflow.md`
- Review type selection → `references/review-types.md`
- Review dimensions → `references/dimensions.md`
- Feedback categories → `references/feedback.md`
- Consensus interpretation → `references/consensus.md`
- Output format & isolation → `references/output.md`
- Error handling → `references/errors.md`
- Model coordination → `references/coordination.md`
- Configuration → `references/configuration.md`
- Examples → `references/examples.md`
- Best practices → `references/best-practices.md`
