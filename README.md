# claude-foundry

A Claude Code plugin for working with [foundry-mcp](https://github.com/tylerburleigh/foundry-mcp).

## Overview

This plugin provides skills, commands, and agents for spec-driven development (SDD) workflows powered by the Foundry MCP server.

## Installation

Add to your Claude Code plugins:

```bash
claude plugins add tylerburleigh/claude-foundry
```

## Plugin Structure

```
claude-foundry/
├── .claude-plugin/
│   └── marketplace.json    # Plugin registration
├── agents/                 # Subagent definitions
├── commands/               # Slash commands (/foundry:*)
├── hooks/                  # Pre/post tool use hooks
│   └── hooks.json
├── skills/                 # Skills (foundry:*)
└── README.md
```

## Available Components

### Skills (`foundry:*`)

Skills are invoked via `Skill(foundry:skill-name)`:

- `foundry:sdd-plan` - Create detailed specifications before coding
- `foundry:sdd-next` - Find and prepare the next actionable task
- `foundry:sdd-update` - Track progress and update task status
- `foundry:run-tests` - Run tests with debugging support
- `foundry:sdd-validate` - Validate spec structure and dependencies
- `foundry:sdd-pr` - Create PRs with AI-enhanced descriptions
- `foundry:sdd-review` - Review specs for quality and issues

### Commands (`/foundry:*`)

Slash commands for common workflows:

- `/foundry:begin` - Resume or start spec-driven development work
- `/foundry:setup` - First-time project setup

### Agents

Subagents for specialized tasks (spawned via Task tool).

## Configuration

The plugin uses the foundry-mcp server for backend operations. Ensure the MCP server is configured in your Claude Code settings.

## License

MIT
