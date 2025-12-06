---
name: sdd-render
description: Generate stakeholder-friendly documentation from SDD specifications using Foundry MCP. Provides basic Markdown exports, AI-enhanced narratives, and delivery workflows for stakeholders, teammates, and archival systems.
---

# sdd-render (MCP Edition)

## Overview

`Skill(foundry:sdd-render)` converts an SDD specification into polished documentation using Foundry MCP. It never shells out to the old `sdd render` CLI—instead, it orchestrates the canonical MCP tools for deterministic exports and AI-enhanced narratives. Use this skill whenever you need:

- Quick phase summaries for planning reviews
- Executive or client-facing reports
- Human-readable archives of completed specs
- Markdown artifacts stored in `.human-readable/` for future reference

## MCP Tooling

| MCP Tool | Purpose |
| --- | --- |
| `mcp__foundry-mcp__spec-doc` | Deterministic/basic rendering (fast, no AI) |
| `mcp__foundry-mcp__spec-doc-llm` | AI-enhanced narratives with summary/standard/full levels |
| `mcp__foundry-mcp__spec-review-fidelity` | Optional fidelity audit prior to rendering |

The skill routes every request through one of these tools. There is **no CLI fallback**—all parameters are passed as structured MCP inputs.

## Rendering Modes

| Mode | Underlying Tool | Typical Use |
| --- | --- | --- |
| `basic` | `spec-doc` | Internal status snapshots (< 30s) |
| `enhanced:summary` | `spec-doc-llm` + `enhancement_level="summary"` | Executive briefs (~1-2 min) |
| `enhanced:standard` *(default)* | `spec-doc-llm` + `enhancement_level="standard"` | Team reports (~3-5 min) |
| `enhanced:full` | `spec-doc-llm` + `enhancement_level="full"` | Client handoffs / deep dives (up to 8 min) |

Specify the mode in your instruction (e.g., "render spec=foo mode=basic", or "level=full"). If no mode is given, the skill confirms the preference via `AskUserQuestion` before calling MCP.

## Workflow Summary

1. **Pre-checks** – Verify the spec exists, passes gating rules, and (optionally) run `spec-review-fidelity` when policy requires it.
2. **Parameter assembly** – The skill maps your request into the MCP payload (`spec_id`, `output_path`, `include_progress`, `include_journal`, `use_ai`, etc.).
3. **Rendering** – Invoke `spec-doc` (basic) or `spec-doc-llm` (enhanced). The MCP response includes artifact metadata, warnings, and telemetry.
4. **Delivery** – The generated Markdown path plus summary data is surfaced back to you; the skill can also mirror the file into stakeholder folders (e.g., `docs/client/weekly/`).

## Key Parameters

| Parameter | Description |
| --- | --- |
| `spec_id` *(required)* | Target specification ID (e.g., `auth-platform-2025-11-01-001`). |
| `spec_path` | Optional override if you need to render a raw JSON file outside the specs tree. |
| `mode` | `basic` vs `enhanced`. Defaults to enhanced. |
| `enhancement_level` | `summary`, `standard`, or `full` when `mode=enhanced`. |
| `output_path` | Absolute or repo-relative destination. Defaults to `specs/.human-readable/{spec_id}-{mode}.md`. |
| `include_progress` | Adds phase/task progress bars (true by default). |
| `include_journal` | Embeds the latest journal entries (off by default for basic). |
| `use_ai`, `ai_provider`, `ai_timeout` | Advanced overrides for `spec-doc-llm`; omit unless you need explicit provider control. |

## Example Instructions

```
Skill(foundry:sdd-render) "render spec=ml-handoff-2025-08-02-001 level=summary deliver=client"
```
- Confirms the spec, optionally runs fidelity review, then calls `spec-doc-llm` with `enhancement_level="summary"`.
- Writes the artifact to `.human-readable/ml-handoff-2025-08-02-001-summary.md` and mirrors it to your configured client folder.

```
Skill(foundry:sdd-render) "render spec=search-refactor-2025-09-15-001 mode=basic output=/tmp/search-status.md"
```
- Uses `spec-doc` for a fast snapshot and streams Markdown to `/tmp/search-status.md`.

```
Skill(foundry:sdd-render) "render spec=payments-2025-07-10-001 level=full include_journal=true include_progress=true"
```
- Produces a comprehensive report with AI-driven narratives, progress bars, and recent journal excerpts.

## Quality & Safeguards

- **Validation first** – Rendering is blocked if the spec is pending approval, has blockers, or lives outside allowed directories. The MCP error response is surfaced so you can remediate.
- **Optional fidelity hook** – When configured, `spec-review-fidelity` runs before rendering and its verdict is attached to the final packet.
- **Telemetry** – Every response includes duration, warnings, and (for enhanced mode) AI provider usage to support auditing.
- **Fallbacks** – If AI providers are unavailable, the skill falls back to `spec-doc` and notes the change in the warning list.

## Delivery Patterns

| Use Case | Recommended Mode | Notes |
| --- | --- | --- |
| Internal stand-ups | `basic` | Fast, lightweight snapshots |
| Weekly team sync | `enhanced:standard` | Balanced detail + AI narrative |
| Executive brief | `enhanced:summary` | Focus on highlights and risks |
| Client handoff | `enhanced:full` | Most thorough output, includes narrative, risks, and progress |
| Batch archival | `basic` (looped per spec) | Output to `.human-readable/` for traceability |

The skill can batch across multiple specs by issuing sequential MCP calls; simply provide a spec list in your instruction.

## Tips

- Pair renders with `Skill(foundry:sdd-update)` so the generated Markdown is recorded in the latest journal entry.
- Use `summary` level for monthly executive updates and `full` for contractual deliveries.
- Archive outputs alongside the spec file to preserve historical context.

For broader documentation needs, see `Skill(foundry:llm-doc-gen)` (codebase-wide docs) and `Skill(foundry:sdd-plan-review)` / `Skill(foundry:sdd-fidelity-review)` for pre-render quality gates.
