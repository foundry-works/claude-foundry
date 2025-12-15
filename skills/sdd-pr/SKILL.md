---
name: sdd-pr
description: AI-powered PR creation after spec completion. Analyzes spec metadata, git diffs, commit history, and journal entries to generate comprehensive PR descriptions with user approval before creation.
---

# Spec-Driven Development: PR Creation Skill

## Overview

The `sdd-pr` skill creates professional, comprehensive pull requests by analyzing your completed specs and git history. Instead of manually writing PR descriptions, this skill uses AI to analyze all available context and generate detailed, well-structured PR descriptions.

## When to Use This Skill

Use `Skill(foundry:sdd-pr)` to:
- **After Spec Completion**: Create PRs when handed off from sdd-update
- **Comprehensive PRs**: Generate detailed PR descriptions
- **Save Time**: Automate writing thorough PR descriptions

**When NOT to use:**
- For quick, trivial changes (use `gh` CLI directly)
- When you need a very specific custom PR format
- For PRs without an associated spec

## MCP Tooling

This skill works exclusively through the Foundry MCP server (`foundry-mcp`). Tools use the router+action pattern: `mcp__plugin_foundry_foundry-mcp__<router>` with `action="<action>"`.

**Key tools:**
- `mcp__plugin_foundry_foundry-mcp__pr action="create"` - Create PR with spec context
- `mcp__plugin_foundry_foundry-mcp__pr action="context"` - Gather PR context from spec
- `mcp__plugin_foundry_foundry-mcp__journal action="list"` - Get journal entries for context

## Core Philosophy

### From Template to Intelligence

Traditional PR creation uses static templates. The sdd-pr skill:

1. **Analyzes Multiple Sources** - Spec metadata, git diffs, commit history, journal entries
2. **Generates Context-Aware Descriptions** - Explains "why" not just "what"
3. **Requires User Approval** - Always shows draft before creation

## Essential Workflow

```
1. Invocation     → Skill(foundry:sdd-pr) "Create PR for spec my-feature-001"
2. Context        → Gathers spec, tasks, commits, journals, diff
3. AI Analysis    → Synthesizes context into coherent description
4. Draft          → Shows PR description for review
5. User Approval  → User reviews and approves/requests changes
6. Creation       → Pushes branch, creates PR, updates spec metadata
```

## Context Sources

| Source | What It Provides |
|--------|------------------|
| **Spec Metadata** | Title, description, objectives |
| **Completed Tasks** | Files modified, changes made |
| **Commit History** | Messages, SHAs, task associations |
| **Journal Entries** | Technical decisions, rationale |
| **Git Diff** | Actual code changes |

> For detailed context gathering, see `reference.md#context-sources`

## PR Structure

Generated PRs follow this template:

```markdown
## Summary
{2-3 sentence overview}

## What Changed

### Key Features
- {feature-1}
- {feature-2}

### Files Modified
- `{file-path}`: {brief description}

## Technical Approach
{Key decisions from journal entries}

## Implementation Details

### Phase 1: {phase-title}
- {task-1}: {status}
- {task-2}: {status}

## Testing
- {verification-step-1}
- {verification-step-2}

## Commits
- {sha}: {task-id}: {message}
```

> For full template with example, see `reference.md#draft-template`

## Quick Start

```bash
# After completing a spec, create PR
Skill(foundry:sdd-pr) "Create PR for spec oauth-feature-2025-11-03-001"

# Agent shows draft, user approves, PR created
```

## Long-Running Operations

**This skill may take up to 5 minutes.** Always use foreground execution:

```python
Bash(command="...", timeout=300000)  # 5 minute timeout
```

**Never** use `run_in_background=True` with frequent polling.

## Detailed Reference

For comprehensive documentation including:
- Complete workflow steps with examples
- Full PR draft template
- Context gathering details
- Best practices for better PRs
- Troubleshooting guide

See **[reference.md](./reference.md)**
