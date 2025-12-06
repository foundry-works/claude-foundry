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

This skill uses the Foundry MCP server (`foundry-mcp`). Tools follow the `mcp__foundry-mcp__<tool-name>` pattern.

**Test execution:**
- `mcp__foundry-mcp__test-run` - Full test run with options
- `mcp__foundry-mcp__test-run-quick` - Quick run (fail-fast, skip slow)
- `mcp__foundry-mcp__test-run-unit` - Unit tests only
- `mcp__foundry-mcp__test-presets` - List available presets
- `mcp__foundry-mcp__test-discover` - Discover tests without running

**AI consultation:**
- `mcp__foundry-mcp__provider-list` - Check available AI tools
- `mcp__foundry-mcp__provider-execute` - Consult AI for debugging

**Code documentation (optional):**
- `mcp__foundry-mcp__code-find-function` - Find function definitions
- `mcp__foundry-mcp__code-trace-calls` - Trace call graphs

## Core Workflow

```
Run Tests
    |
    v
Pass? --Yes--> Done
    |
    No
    v
Investigate (form hypothesis)
    |
    v
Gather Context (optional: use code docs)
    |
    v
Consult External Tools (mandatory if available)
    |
    v
Fix & Verify
```

**Decision rules:**
- Tests pass? Done. No consultation needed.
- Simple fix (typo/obvious)? Fix, then verify.
- Complex/unclear? Investigate, consult AI, fix, verify.

## Phase 1: Run Tests

**Quick run (stop on first failure):**
```
mcp__foundry-mcp__test-run-quick
```

**Full suite with verbose output:**
```
mcp__foundry-mcp__test-run verbose=true
```

**Specific test:**
```
mcp__foundry-mcp__test-run target="tests/test_module.py::test_function"
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

If code documentation exists:
```
mcp__foundry-mcp__doc-stats
mcp__foundry-mcp__code-find-function name="function_name"
mcp__foundry-mcp__code-trace-calls function_name="function_name"
```

## Phase 4: Consult External Tools

**Check availability:**
```
mcp__foundry-mcp__provider-list
```

**Decision:**
- Tests failed AND tools available: Consult (mandatory)
- No tools available: Skip to Phase 5
- Tests passed: Done

**Consultation:**
```
mcp__foundry-mcp__provider-execute provider_id="gemini" prompt="..."
```

> For tool selection guidance by failure type, see `reference.md#tool-selection`

## Phase 5: Fix & Verify

1. Synthesize findings from investigation + AI recommendations
2. Implement fix using Edit tool
3. Verify with specific test: `mcp__foundry-mcp__test-run target="tests/test_module.py::test_function"`
4. Run full suite: `mcp__foundry-mcp__test-run`

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

## Reference

For detailed troubleshooting, failure categories, and AI tool selection:
- See **[reference.md](./reference.md)**
