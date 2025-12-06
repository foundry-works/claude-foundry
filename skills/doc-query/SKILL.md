---
name: doc-query
description: Targeted query capabilities for machine-readable codebase documentation with cross-reference tracking, call graph analysis, and workflow automation. Enables fast lookups of classes, functions, dependencies, and function relationships without parsing source code.
---

# doc-query: Codebase Documentation Query System

## Overview

`Skill(foundry:doc-query)` builds structured context from the codebase documentation artifacts produced by `mcp__foundry-mcp__llm-doc-generate`. Instead of parsing source files directly, the skill queries the generated JSON corpus (`codebase.json`) plus the sharded Markdown set under `docs/` to answer questions about classes, functions, dependencies, call graphs, and refactoring candidates. The skill never shells out to the legacy `sdd doc …` CLI—every operation flows through the Foundry MCP server.

**Core capabilities**
- Entity lookup for classes, functions, and modules
- Incoming/outgoing call analysis with depth controls
- Blast-radius and dependency impact assessments
- Complexity and refactoring heuristics
- Feature- and data-oriented exploration workflows (scope, trace-entry, trace-data, impact, refactor)
- Automatic documentation staleness detection and refresh

Use the skill whenever you need a fast, structured understanding of an unfamiliar feature, want to estimate the impact of a change, or need to surface complexity/risk hot spots before touching code.

## MCP Tooling

Doc-query calls the following MCP tools (canonical names shown as `mcp__foundry-mcp__<tool>`):

| Tool | Purpose |
| --- | --- |
| `code-find-class` | Locate class definitions by exact or fuzzy name |
| `code-find-function` | Locate function definitions and metadata |
| `code-get-callers` | List functions that call a given function |
| `code-get-callees` | List functions invoked by a given function |
| `code-trace-calls` | Build configurable call graphs (upstream/downstream/both) |
| `code-impact-analysis` | Perform dependency-based blast-radius assessments |
| `doc-stats` | Inspect documentation freshness, entity counts, and complexity metrics |

Higher-level workflows (scope, trace-entry, trace-data, refactor-candidates, etc.) orchestrate combinations of these base tools; you do **not** need to invoke them directly—simply describe the workflow you want when calling the skill and it issues the necessary MCP requests.

## Quick Start

1. **Confirm documentation exists** – run `mcp__foundry-mcp__doc-stats`. If the response warns about missing data, generate docs via `mcp__foundry-mcp__llm-doc-generate` first.
2. **Scope your module** – instruct the skill with something like `"scope module=src/auth/login.py mode=plan"`. The skill gathers summaries, dependencies, and complexity stats by chaining `code-find-*`, `code-trace-calls`, and `code-impact-analysis`.
3. **Explore execution or data flow** – use the skill’s `trace-entry` or `trace-data` workflows (e.g., `"trace entry function=process_request depth=3"`). The skill builds a call tree via `code-trace-calls` plus targeted caller/callee lookups.
4. **Estimate risk** – ask for `impact entity=UserService depth=2`. Behind the scenes the skill runs `code-impact-analysis`, call graph expansion, and dependency checks.
5. **Dive deeper manually** – if you need raw lookups, request them explicitly ("find function calculate_score" → `code-find-function`, "callers calculate_score" → `code-get-callers`, etc.).

## When to Use

✅ **Use doc-query to:**
- Gather context before implementing or refactoring a feature
- Locate relevant code quickly during bug triage
- Understand how data moves through the system
- Prioritize refactoring work based on complexity and usage
- Estimate the blast radius of an upcoming change
- Build call graphs or documentation excerpts for design reviews

❌ **Avoid doc-query when:**
- Documentation has never been generated (run `mcp__foundry-mcp__llm-doc-generate` first)
- You must inspect raw source code (use `Read`/`Explore` tools)
- You require runtime behavior or profiler data (use debugging tools instead)

## Documentation Requirements & Auto-Refresh

- The skill expects `codebase.json` plus the sharded Markdown docs under `docs/`.
- On every query, doc-query compares the documentation timestamp with the latest source commit. If stale, it transparently triggers a regeneration via `mcp__foundry-mcp__llm-doc-generate` (unless you explicitly disable refresh in your instructions).
- Use these flags in your prompt when needed:
  - `skip_refresh=true` – still checks staleness but only warns
  - `no_staleness_check=true` – bypasses the check entirely for maximum speed

You can always inspect the current state via `mcp__foundry-mcp__llm-doc-status` and clear cached artifacts with `mcp__foundry-mcp__llm-doc-cache`.

## Automated Workflows

The skill exposes several named workflows. Simply describe the workflow you want; no CLI syntax is required.

### Scope
Produces a tailored briefing for a module or feature area (planning vs implementation views). Internally calls `code-find-*`, `doc-stats`, and targeted dependency queries.

```
Skill(foundry:doc-query) "scope module=src/auth/login.py mode=implement"
```

### Trace Entry
Builds an end-to-end call chain from an entry function or endpoint, highlighting architectural layers and complexity hot spots (`code-trace-calls` + `code-get-callers/callees`).

### Trace Data
Follows a class or DTO across its lifecycle (create/read/update/delete touch points) using entity lookups and call graphs.

### Impact
Estimates blast radius and risk before modifying a function, class, or module. Combines `code-impact-analysis`, dependency traversal, and caller insight.

### Refactor Candidates
Ranks high-complexity, high-dependency targets for refactoring by querying complexity stats plus usage counts.

## Manual Query Reference

When you need granular control, ask the skill to invoke the specific MCP tools:

| Goal | Instruction Example | Underlying Tool |
| --- | --- | --- |
| Find a class | `"find class WizardSession"` | `code-find-class` |
| Find a function | `"find function calculate_score"` | `code-find-function` |
| Describe callers | `"callers calculate_score"` | `code-get-callers` |
| Describe callees | `"callees process_request"` | `code-get-callees` |
| Build a call graph | `"trace calls process_request direction=down depth=3"` | `code-trace-calls` |
| Impact analysis | `"impact entity=UserService depth=2"` | `code-impact-analysis` |
| Documentation stats | `"doc stats"` | `doc-stats` |

The skill supports pagination for find/class/function queries; specify `limit` or ask for "next page" and it will pass the cursor returned by the tool.

## Tool Verification

Before running complex workflows:
1. Call `mcp__foundry-mcp__doc-stats` to ensure documentation is loaded (the response includes the detected docs path and freshness timestamp).
2. If stats fail with "Documentation not loaded", regenerate via `mcp__foundry-mcp__llm-doc-generate directory="<repo>" output_dir="./docs" use_ai=false`.
3. Optionally, request `doc stats` again to confirm freshness.

## Examples

### Trace Execution Flow
```
Skill(foundry:doc-query) "trace entry function=run_scoring depth=3"
```
Returns the layered call graph (endpoint → service → repository), complexity scores, and a hotspot summary.

### Impact Assessment
```
Skill(foundry:doc-query) "impact entity=get_session depth=2"
```
Provides direct/indirect dependents, test coverage hints, and risk scoring so you know how risky the change is.

### Refactor Planning
```
Skill(foundry:doc-query) "refactor candidates min_complexity=15 limit=10"
```
Returns a prioritized list of high-value refactors with quick-win vs major-effort tags.

## Output Formats

### Scope Output

```markdown
## Module: src/auth/login.py

**Summary:** Handles user authentication via OAuth 2.0

### Dependencies
- `src/auth/session.py` (session management)
- `src/db/users.py` (user storage)

### Complexity
- Functions: 8
- Cyclomatic: 24 (moderate)
- Risk: Medium

### Entry Points
- `authenticate()` - Main auth entry
- `refresh_token()` - Token refresh
```

### Impact Analysis Output

```markdown
## Impact Analysis: get_session

**Direct Dependents:** 12 functions
**Indirect Dependents:** 34 functions
**Risk Score:** High (8/10)

### Affected Areas
- auth/* (5 functions)
- api/routes.py (4 functions)
- middleware/* (3 functions)

### Recommendations
- Add comprehensive tests before modification
- Consider staged rollout
```

---

Refer to `Skill(foundry:llm-doc-gen)` for generating/updating documentation bundles, and combine doc-query with `Skill(foundry:sdd-next)` or `Skill(foundry:sdd-update)` to feed insights directly into task planning and execution.
