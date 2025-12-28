---
name: run-tests
description: Comprehensive pytest testing and debugging framework. Use when running tests, debugging failures, fixing broken tests, or investigating test errors. Includes systematic investigation workflow with external AI tool consultation and verification strategies.
---

# Test Runner Skill

## Overview

The **Skill(foundry:run-tests)** skill provides systematic pytest testing with a 5-phase investigation workflow. It runs tests, investigates failures, consults external AI tools when available, and guides fix implementation.

**Key capabilities:**
- Run pytest test suites (quick, unit, integration, full)
- Systematic failure categorization and investigation
- External AI consultation for complex failures (mandatory when tools available)
- Hypothesis-driven debugging workflow
- Verification of fixes with regression testing

## MCP Tooling

This skill uses the Foundry MCP server (`foundry-mcp`). Tools use the router+action pattern: `mcp__plugin_foundry_foundry-mcp__<router>` with `action="<action>"`.

**Test execution:**
- `mcp__plugin_foundry_foundry-mcp__test action="run"` - Full test run with options
- `mcp__plugin_foundry_foundry-mcp__test action="run-quick"` - Quick run (fail-fast, skip slow)
- `mcp__plugin_foundry_foundry-mcp__test action="run-unit"` - Unit tests only
- `mcp__plugin_foundry_foundry-mcp__test action="presets"` - List available presets
- `mcp__plugin_foundry_foundry-mcp__test action="discover"` - Discover tests without running

**AI consultation:**
- `mcp__plugin_foundry_foundry-mcp__provider action="list"` - Check available AI tools
- `mcp__plugin_foundry_foundry-mcp__provider action="execute"` - Consult AI for debugging

**Requirement discovery:**
- `mcp__plugin_foundry_foundry-mcp__task action="add-requirement"` - Document discovered requirements when tests reveal spec gaps

## Core Workflow

> `[x?]`=decision · `(GATE)`=user approval · `→`=sequence · `↻`=loop · `§`=section ref

```
- **Entry** → LSP PreFlight
- RunTests → `test action="run"`
  - [pass?] → **Exit**: Done
  - [fail?] → Categorize[Assertion|Exception|Import|Fixture|Timeout|Flaky]
- FormHypothesis → GatherContext[Explore preferred|Glob/Grep]
- [Tools available?] → `provider action="list"`
  - [yes] → Consult (MANDATORY) → `provider action="execute"`
  - [no] → skip
- ImplementFix → VerifySpecific ↻ [pass?]
- RunFullSuite ↻ [pass?] → **Exit**: Done
```

**Decision rules:**
- Tests pass? Done. No consultation needed.
- Simple fix (typo/obvious)? Fix, then verify.
- Complex/unclear? Investigate, consult AI, fix, verify.

## Phase 0: Pre-Flight Diagnostics (LSP-Enhanced)

Before running tests, optionally use LSP to catch import issues early.

**When to use:**
- Full suite on unfamiliar code
- Tests failing with import/resolution errors
- After major refactoring

**When to skip:**
- Quick test run after small change
- Tests already known to work

**Key LSP operations:**
- `documentSymbol()` - Get test file structure
- `goToDefinition()` - Verify imports resolve

If LSP unavailable, skip to Phase 1. Import errors will surface as test failures.

> For pseudocode examples and report format, see `references/pre-flight.md`

## Phase 1: Run Tests

**Quick run (stop on first failure):**
```
mcp__plugin_foundry_foundry-mcp__test action="run-quick"
```

**Full suite with verbose output:**
```
mcp__plugin_foundry_foundry-mcp__test action="run" verbose=true
```

**Specific test:**
```
mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/test_module.py::test_function"
```

## Phase 2: Investigate Failures

**Categorize the failure:**

| Category | Description |
|----------|-------------|
| Assertion | Expected vs actual mismatch |
| Exception | Runtime errors (AttributeError, KeyError) |
| Import | Missing dependencies or module issues |
| Fixture | Fixture or configuration issues |
| Timeout | Performance or hanging issues |
| Flaky | Non-deterministic failures |

**Extract key information:**
- Test file and function name
- Line number where failure occurred
- Error type and message
- Full stack trace

**Form hypothesis:** What's causing the failure?

## Phase 3: Gather Code Context

Use **Explore subagents** (preferred) for code context, or `Glob`, `Grep`, and `Read` for targeted lookups.

**Subagent selection:**
- **Explore (quick)** - Find related test files
- **Explore (medium)** - Understand module dependencies
- **Explore (very thorough)** - Multi-file state/fixture investigation
- **general-purpose** - Complex debugging across packages

> For detailed subagent patterns (including flaky test investigation), see `references/subagent-patterns.md`

## Phase 4: Consult External Tools

**Check availability:**
```
mcp__plugin_foundry_foundry-mcp__provider action="list"
```

**Decision:**
- Tests failed AND tools available: Consult (mandatory)
- No tools available: Skip to Phase 5
- Tests passed: Done

**Consultation:**
```
mcp__plugin_foundry_foundry-mcp__provider action="execute" provider_id="gemini" prompt="..."
```

> For tool selection guidance by failure type, see `references/tool-selection.md`

## Phase 5: Fix & Verify

1. Synthesize findings from investigation + AI recommendations
2. Implement fix using Edit tool
3. Verify with specific test: `mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/test_module.py::test_function"`
4. Run full suite: `mcp__plugin_foundry_foundry-mcp__test action="run"`

### Requirement Discovery

When debugging reveals missing spec requirements (e.g., edge cases, validation rules), document them:

```bash
mcp__plugin_foundry_foundry-mcp__task action="add-requirement" spec_id={spec-id} task_id={task-id} requirement="Handle empty input array gracefully"
```

**When to use:**
- Test failure reveals undocumented edge case
- Fix requires behavior not in acceptance criteria
- Investigation uncovers missing validation rule

This keeps the spec updated without interrupting the debugging workflow.

## Test Presets

| Preset | Behavior |
|--------|----------|
| quick | Stop on first failure, exclude slow tests |
| unit | Unit tests only |
| integration | Integration tests only |
| full | Complete suite with verbose output |

## Important: Long-Running Operations

**Always use foreground execution with timeout:**
```python
Bash(command="pytest src/", timeout=300000)  # Wait up to 5 minutes
```

**Never poll background processes** - this creates spam in the conversation.

## Detailed Reference

For comprehensive documentation including:
- Tool selection → `references/tool-selection.md`
- Pre-flight diagnostics → `references/pre-flight.md`
- Failure categories → `references/failure-categories.md`
- Investigation patterns → `references/investigation.md`
- Subagent patterns → `references/subagent-patterns.md`
- Common fixes → `references/common-fixes.md`
- Troubleshooting → `references/troubleshooting.md`
