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

### 3.1 Subagent Guidance (Context Exploration)

For complex failures or unfamiliar code, use Claude Code's built-in subagents:

| Scenario | Subagent | Thoroughness |
|----------|----------|--------------|
| Find related test files | Explore | quick |
| Understand module dependencies | Explore | medium |
| Multi-file state/fixture investigation | Explore | very thorough |
| Complex debugging across packages | general-purpose | N/A |

**Example: Investigating shared state (flaky tests)**
```
Use the Explore agent (very thorough) to find:
- All fixtures in conftest.py files
- Global state or module-level variables
- Files importing the failing module
- Test files that modify shared resources
```

**Benefits:**
- Prevents test output and search results from bloating context
- Haiku model is faster for searching test directories
- Returns focused findings for targeted debugging

> For detailed investigation patterns with subagents, see `reference.md#subagent-investigation-patterns`

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

> For tool selection guidance by failure type, see `reference.md#tool-selection`

## Phase 5: Fix & Verify

1. Synthesize findings from investigation + AI recommendations
2. Implement fix using Edit tool
3. Verify with specific test: `mcp__plugin_foundry_foundry-mcp__test action="run" target="tests/test_module.py::test_function"`
4. Run full suite: `mcp__plugin_foundry_foundry-mcp__test action="run"`

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
