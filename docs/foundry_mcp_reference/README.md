# foundry-mcp Reference Documentation

> Comprehensive reference for the foundry-mcp backend that powers claude-foundry's spec-driven development capabilities.

This documentation uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## What is foundry-mcp?

**foundry-mcp** is a Python-based MCP (Model Context Protocol) server and native CLI that enables AI assistants to manage spec-driven development workflows. It provides 40+ tools for:

- **Spec Management** — Create, validate, and lifecycle specifications
- **Task Operations** — Track progress, dependencies, and blockers
- **Code Intelligence** — Query codebase documentation and call graphs
- **Testing** — Run and discover tests with pytest integration
- **LLM Features** — AI-powered reviews, documentation, and PR generation

The claude-foundry plugin interfaces with foundry-mcp through MCP tool calls, providing skills like `sdd-plan`, `sdd-next`, `sdd-update`, `run-tests`, and more.

---

## Quick Links

| Category | Documents | Focus |
|----------|-----------|-------|
| **Start Here** | [00-Quickstart](./00-quickstart.md), [01-Architecture](./01-architecture.md) | Installation, system design |
| **Core Concepts** | [02-SDD Philosophy](./02-sdd-philosophy.md), [03-Spec Lifecycle](./03-spec-lifecycle.md) | Methodology, states |
| **Reference** | [04-Tool Reference](./04-tool-reference.md), [05-Response Contract](./05-response-contract.md) | Tools, API format |
| **Usage** | [06-Workflows](./06-workflows.md), [07-Configuration](./07-configuration.md) | Patterns, setup |
| **Integration** | [08-Integration Patterns](./08-integration-patterns.md) | claude-foundry ↔ foundry-mcp |
| **Advanced** | [09-LLM Providers](./09-llm-providers.md), [10-Testing](./10-testing.md) | AI config, test tools |
| **Operations** | [11-Troubleshooting](./11-troubleshooting.md) | Problem resolution |

---

## Document Index

### Getting Started

| # | Document | Description |
|---|----------|-------------|
| 00 | [Quickstart](./00-quickstart.md) | Installation and first SDD workflow |
| 01 | [Architecture](./01-architecture.md) | System design and layer responsibilities |

### Core Concepts

| # | Document | Description |
|---|----------|-------------|
| 02 | [SDD Philosophy](./02-sdd-philosophy.md) | Spec-driven development methodology |
| 03 | [Spec Lifecycle](./03-spec-lifecycle.md) | Spec states, transitions, and schema |

### Reference

| # | Document | Description |
|---|----------|-------------|
| 04 | [Tool Reference](./04-tool-reference.md) | Complete reference for all 40+ MCP tools |
| 05 | [Response Contract](./05-response-contract.md) | Standardized response envelope (v2) |

### Practical Usage

| # | Document | Description |
|---|----------|-------------|
| 06 | [Workflows](./06-workflows.md) | Common workflow patterns and examples |
| 07 | [Configuration](./07-configuration.md) | Environment variables and TOML config |
| 08 | [Integration Patterns](./08-integration-patterns.md) | How claude-foundry skills use foundry-mcp |

### Advanced Topics

| # | Document | Description |
|---|----------|-------------|
| 09 | [LLM Providers](./09-llm-providers.md) | AI provider configuration and fallbacks |
| 10 | [Testing](./10-testing.md) | Test discovery, execution, and debugging |
| 11 | [Troubleshooting](./11-troubleshooting.md) | Common issues and solutions |

---

## How to Use This Guide

### For New Users

1. **Start with [00-Quickstart](./00-quickstart.md)** to install and verify foundry-mcp
2. **Read [02-SDD Philosophy](./02-sdd-philosophy.md)** to understand the methodology
3. **Explore [04-Tool Reference](./04-tool-reference.md)** for available capabilities
4. **Follow [06-Workflows](./06-workflows.md)** for practical patterns

### For Plugin Developers

1. **Read [08-Integration Patterns](./08-integration-patterns.md)** to understand skill-to-tool mapping
2. **Reference [05-Response Contract](./05-response-contract.md)** for response handling
3. **Check [04-Tool Reference](./04-tool-reference.md)** for tool parameters

### For Troubleshooting

1. **Check [11-Troubleshooting](./11-troubleshooting.md)** for common issues
2. **Review [07-Configuration](./07-configuration.md)** for setup problems
3. **Verify [09-LLM Providers](./09-llm-providers.md)** for AI feature issues

---

## Relationship to claude-foundry Skills

claude-foundry provides skills that orchestrate foundry-mcp tools:

| Skill | Primary Tools Used |
|-------|-------------------|
| `sdd-plan` | `spec-create`, `spec-validate`, `doc-scope`, `doc-stats` |
| `sdd-next` | `task-prepare`, `task-next`, `session-work-mode` |
| `sdd-update` | `task-update-status`, `journal-add`, `task-complete` |
| `sdd-validate` | `spec-validate`, `spec-audit` |
| `sdd-pr` | `pr-context`, `pr-create` |
| `run-tests` | `test-run`, `test-discover`, `test-consult` |
| `doc-query` | `doc-stats`, `doc-scope`, `doc-search` |
| `sdd-fidelity-review` | `fidelity-check`, `verify-check` |

All tools follow the naming pattern `mcp__foundry-mcp__<tool-name>`.

---

## Key Concepts at a Glance

### Spec-Driven Development (SDD)

Documentation-first methodology where machine-readable JSON specifications serve as the single source of truth. Specs define what to build *before* implementation begins.

### Spec Lifecycle

```
pending → active → completed → archived
```

Specs progress through states as work is planned, executed, and completed.

### Response Contract (v2)

All tools return a standardized JSON envelope:

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "meta": { "version": "response-v2", ... }
}
```

### Tool Naming

Tools use kebab-case with domain prefixes:
- `spec-*` — Specification operations
- `task-*` — Task operations
- `doc-*` — Documentation queries
- `test-*` — Testing operations
- `review-*` — Review operations

---

## Related Documentation

### claude-foundry Plugin Docs
- [Claude Code Best Practices](../claude_code_best_practices/README.md) — Plugin development guide
- [MCP Consumption](../claude_code_best_practices/11-mcp-consumption.md) — How Claude Code uses MCP

### External Resources
- [foundry-mcp GitHub](https://github.com/tylerburleigh/foundry-mcp) — Source repository
- [MCP Specification](https://modelcontextprotocol.io/) — Protocol documentation

---

*Last updated: 2025-12*
