# Claude Foundry - Spec-Driven Development for Claude Code

Plan before you code. Verify against the spec. Ship with confidence.

## Overview

Ad-hoc coding with AI assistants often leads to unclear requirements, untracked progress, and implementation drift. Claude Foundry brings structure to AI-assisted development through **spec-driven development (SDD)**: you create a detailed specification first, then implement against it with granular task tracking and AI-powered verification.

**Who it's for:** Developers using Claude Code who want traceable, auditable AI-assisted development with clear requirements and verifiable outcomes.

## Table of Contents

- [Key Features](#key-features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Example Workflow](#example-workflow)
- [How It Works](#how-it-works)
- [Skills Reference](#skills-reference)
- [Commands Reference](#commands-reference)
- [Configuration](#configuration)
- [Scope and Limitations](#scope-and-limitations)
- [Documentation](#documentation)
- [Support](#support)
- [License](#license)

## Key Features

- **Spec-Driven Methodology** - Plan before code with mandatory human approval gates. Specs define requirements, phases, tasks, and verification steps.
- **Fidelity Verification** - AI compares implementation to spec requirements. Catches drift between what was planned and what was built.
- **Task Dependencies & Tracking** - Granular task states with automatic next-task recommendations. Know what's done, blocked, or ready.
- **Auditable Decision Journals** - Every task completion logs decisions made. Full traceability of why changes happened.
- **Context Flow to PRs** - PR descriptions generated from spec + journals + git history. Reviewers understand the "why", not just the "what".
- **Multi-Model Research** - Query multiple AI models for consensus, deep research, or structured ideation before planning.

## Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| Python | 3.10+ | Required for foundry-mcp server |
| Claude Code | Latest | The CLI tool from Anthropic |
| foundry-mcp | Latest | MCP server (installed via pip) |

## Installation

### 1. Install foundry-mcp

```bash
pip install foundry-mcp
```

Or with pipx for isolated installation:

```bash
pipx install foundry-mcp
```

Verify installation:

```bash
python -m foundry_mcp.server --help
```

### 2. Install the plugin

From within Claude Code:

```
/plugin marketplace add tylerburleigh/claude-foundry
/plugin install foundry@claude-foundry
```

Restart Claude Code and trust the repository when prompted.

> **Note:** The MCP server is automatically registered when you install the plugin. Do not manually add it with `claude mcp add`.

## Quick Start

1. **Run setup** - Configure permissions and verify installation:
   ```
   /setup
   ```

2. **Research** (optional) - Explore the codebase or research best practices:
   ```
   /research How is authentication currently handled?
   /research deep Current best practices for JWT refresh tokens
   ```

3. **Describe what you want** - Tell Claude what you want to build:
   ```
   I want to add user authentication with JWT tokens.
   ```
   Claude creates a spec with phases, tasks, and verification steps.

4. **Implement** - Work through tasks (verification is built into the spec):
   ```
   /implement
   ```

5. **Ship** - Create a PR when the spec is complete:
   ```
   /sdd-pr
   ```

## Example Workflow

A complete spec-driven development cycle:

```bash
# 1. Research (optional): Understand existing patterns
/research How does the current API handle errors?
# Single-model chat, multi-model consensus, or deep web research

# 2. Describe your feature: Claude creates the spec
"I want to add rate limiting to the API endpoints"
# Claude invokes sdd-plan, asks clarifying questions, creates spec
# Spec saved to specs/pending/, then activated after your approval

# 3. Implement: Work through tasks with /implement
/implement
# Shows next task, you implement it, verification tasks auto-dispatch
# When /implement hits a verify task, it runs sdd-review or tests

# 4. Ship: Create PR when spec is complete
/sdd-pr
# Generates PR from spec metadata, journals, and git history
```

## How It Works

```
/research → describe intent → /implement → (auto-verify) → /sdd-pr
    │             │               │              │             │
    ▼             ▼               ▼              ▼             ▼
 Explore      Claude          Work on       Verify tasks    Create PR
 codebase     creates         tasks via     auto-dispatch   with full
 or web       spec            dependency    to sdd-review   context
                              order         or run-tests
```

| Step | What happens |
|------|--------------|
| **Research** | Optional first step to explore the codebase or research best practices via web search. |
| **Describe intent** | Tell Claude what you want to build. Claude creates a spec with phases, tasks, and verification steps. |
| **Implement** | `/implement` finds next task by dependency order. You implement, it tracks progress. |
| **Auto-verify** | Specs include verify tasks. When `/implement` hits one, it auto-dispatches to `sdd-review` or `run-tests`. |
| **Ship** | `/sdd-pr` creates PR from spec metadata, journals, and commit history. |

## Skills Reference

Skills are invoked by Claude based on your intent. Describe what you want, and Claude selects the appropriate skill.

| Skill | When Claude uses it |
|-------|---------------------|
| `sdd-plan` | You describe a feature or ask to plan something |
| `sdd-implement` | You run `/implement` or ask to work on tasks |
| `sdd-review` | Verify task in spec, or you ask to check implementation |
| `run-tests` | Verify task in spec, or you ask to run/debug tests |
| `sdd-pr` | You ask to create a pull request |
| `sdd-refactor` | You ask to rename, extract, or move code |
| `research` | You run `/research` or ask to investigate something |
| `bikelane` | You run `/bikelane` or ask to capture an idea |

## Commands Reference

| Command | Purpose |
|---------|---------|
| `/setup` | First-time setup and permissions configuration |
| `/implement` | Resume or start spec-driven development |
| `/research` | AI-powered research: chat, consensus, thinkdeep, ideate, deep |
| `/bikelane` | Quick capture for ideas and issues |

## Configuration

The plugin stores configuration in `foundry-mcp.toml`. Key settings:

| Setting | Default | Description |
|---------|---------|-------------|
| `workspace` | Current directory | Working directory for specs |
| `specs_dir` | `specs/` | Where specs are stored |
| `providers` | System default | AI providers for research workflows |

Specs are organized by status:

```
specs/
├── pending/    # Specs in planning phase
├── active/     # Specs being implemented
└── completed/  # Finished specs (archived)
```

## Scope and Limitations

**Best for:**
- Feature development with clear requirements
- Bug fixes that benefit from structured investigation
- Multi-phase projects where progress tracking matters
- Work that needs auditable history (journals, specs)

**Not for:**
- One-line fixes or trivial changes
- Exploratory prototyping without clear goals
- Tasks where overhead exceeds benefit

## Documentation

| Guide | Description |
|-------|-------------|
| [Quick Start](docs/01-quick-start.md) | Get started in 5 minutes |
| [Core Concepts](docs/02-core-concepts.md) | Understand specs, phases, tasks |
| [Workflow Guide](docs/03-workflow-guide.md) | The complete SDD workflow |
| [Tutorial](docs/04-tutorial.md) | Build a feature from start to finish |
| [Command Reference](docs/05-command-reference.md) | All commands and options |
| [Troubleshooting](docs/06-troubleshooting.md) | Common issues and solutions |

## Support

- **Issues:** [GitHub Issues](https://github.com/tylerburleigh/claude-foundry/issues)
- **Bugs:** Open an issue with steps to reproduce
- **Features:** Open an issue describing the use case

## License

[MIT](LICENSE)
