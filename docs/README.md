# Claude Foundry Documentation

Welcome to Claude Foundry, a Claude Code plugin for **Spec-Driven Development (SDD)**.

Instead of diving straight into code, SDD helps you plan first, implement systematically, and verify your work matches your intentions. Claude Foundry automates this workflow with AI-powered planning, progress tracking, and quality verification.

## Guides

| Guide | Description |
|-------|-------------|
| [Quick Start](01-quick-start.md) | Get started in 5 minutes |
| [Core Concepts](02-core-concepts.md) | Understand specs, phases, tasks, and the SDD philosophy |
| [Workflow Guide](03-workflow-guide.md) | The complete SDD workflow explained step-by-step |
| [Tutorial](04-tutorial.md) | Build a feature from start to finish |
| [Command Reference](05-command-reference.md) | All commands and skills with options |
| [Troubleshooting](06-troubleshooting.md) | Common issues and solutions |

## Prerequisites

Before using Claude Foundry, ensure you have:

- [ ] **Claude Code** installed and running
- [ ] **foundry-mcp** Python package installed:
  ```bash
  pip install foundry-mcp
  ```
- [ ] **Plugin installed** via Claude Code:
  ```
  /plugin marketplace add foundry-works/claude-foundry
  /plugin install foundry@claude-foundry
  ```
- [ ] **Permissions configured** in your project (see [Quick Start](01-quick-start.md))

## The SDD Workflow at a Glance

```
foundry-spec  →  foundry-implement  →  foundry-review  →  foundry-test  →  foundry-pr
      │                 │                    │                 │               │
    Plan            Code it             Verify it          Test it         Ship it
```

Each step is a skill that guides you through the process with AI assistance and human approval gates.

## Getting Help

- **In Claude Code**: Ask Claude questions about workflows or skills
- **Quick capture**: Use `foundry-note` to capture ideas or issues for later
- **AI research**: Use `foundry-research` for complex investigations

Ready to start? Head to the [Quick Start Guide](01-quick-start.md).
