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
| `provider` | `list`, `status`, `execute` | List and manage AI providers |

**Critical Rules:**
- **ALWAYS** use MCP tools for spec operations
- **NEVER** use `Read()` on spec JSON files directly
- **NEVER** shell out to `cat`, `grep`, `jq` for spec parsing

## Review Types

| Type | Models | Duration | Focus | Use When |
|------|--------|----------|-------|----------|
| **quick** | 2 | 10-15 min | Completeness, Clarity | Simple specs, time-constrained |
| **full** | 3-4 | 20-30 min | All 6 dimensions | Complex specs, moderate-high risk |
| **security** | 2-3 | 15-20 min | Risk Management | Auth, data handling, compliance |
| **feasibility** | 2-3 | 10-15 min | Estimates, Dependencies | Tight deadlines, uncertain scope |

> For detailed dimension descriptions, see `reference.md#review-dimensions`

## Essential Commands

**Run a review:**
```bash
mcp__plugin_foundry_foundry-mcp__review action="spec-review" spec_id="{spec-id}" review_type="{type}"
```

**Check available tools:**
```bash
mcp__plugin_foundry_foundry-mcp__review action="list-tools"
```

**Check provider status:**
```bash
mcp__plugin_foundry_foundry-mcp__provider action="list"
```

## Core Workflow

### Step 1: Verify Available Tools

```bash
mcp__plugin_foundry_foundry-mcp__review action="list-tools"
```

Check which AI review toolchains are installed:
- Need at least 1 tool for basic review
- 2+ tools recommended for multi-model feedback
- All 3 tools ideal for comprehensive analysis

### Step 2: Select Review Type

Use this decision matrix:

| Condition | Review Type |
|-----------|-------------|
| Task count ≤ 10 AND no auth/payment/PII tasks | `quick` |
| Task count > 10 OR has architectural decisions | `full` |
| Contains auth, payment, encryption, or PII handling | `security` |
| Phase has deadline pressure OR external dependencies | `feasibility` |

**Priority when multiple conditions match:**
1. `security` (always takes precedence for sensitive data)
2. `full` (for complex specs)
3. `feasibility` (for time-constrained work)
4. `quick` (default fallback)

### Step 3: Execute Review

```bash
mcp__plugin_foundry_foundry-mcp__review action="spec-review" spec_id="{spec-id}" review_type="{type}"
```

The tool automatically:
1. Initiates each model review in parallel
2. Collects responses as they complete (60-120s per tool)
3. Handles failures gracefully
4. Parses and synthesizes responses

### Step 4: Interpret Results

**Priority Levels:**
- **CRITICAL**: Security vulnerabilities, blockers → Address immediately
- **HIGH**: Design flaws, missing info → Address before implementation
- **MEDIUM**: Improvements, unclear requirements → Consider addressing
- **LOW**: Nice-to-have enhancements → Note for future

> For consensus interpretation details, see `reference.md#interpreting-consensus`

### Step 5: Synthesize & Report

1. Organize feedback by category and priority
2. Note reviewer consensus and diverse perspectives
3. Identify type of downstream work needed
4. Return report path for full context

### Step 6: Handoff

After completing the review:

1. Present summary to user with finding counts by priority level
2. Provide paths to full reports (markdown + JSON)
3. Ask: "Would you like me to apply any of these recommendations using sdd-modify?"
4. Do NOT automatically proceed to sdd-modify without user approval

## Output Format

Reports are written to `specs/.plan-reviews/{spec-id}-review-{type}.md`:

```markdown
## Feedback Summary

### Risk Flags (CRITICAL)
1. Missing authentication on admin endpoints
   Priority: CRITICAL | Flagged by: gemini, codex
   Impact: Unauthorized access to sensitive operations
   Recommendation: Add JWT validation middleware
```

> For complete output template, see `reference.md#output-format`

## Output Isolation

**Why Isolation Matters:**
Review output can be extensive (hundreds of lines). Writing directly to the conversation wastes tokens.

**Isolation Behavior:**
1. Full results go to files, not conversation
2. Conversation receives summary only (counts + paths)
3. Callers access full data via file paths

**Output Locations:**
- Markdown report: `specs/.plan-reviews/{plan-name}-{review-type}.md`

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

> For full configuration, see `reference.md#configuration`

## Detailed Reference

For comprehensive documentation including:
- Review dimensions (detailed descriptions)
- Feedback categories
- Error handling scenarios
- Model coordination details
- Configuration values
- Best practices (DO/DON'T lists)

See **[reference.md](./reference.md)**
