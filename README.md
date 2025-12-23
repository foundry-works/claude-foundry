# claude-foundry

A Claude Code plugin for working with [foundry-mcp](https://github.com/tylerburleigh/foundry-mcp).

## Overview

This plugin provides skills, commands, and agents for spec-driven development (SDD) workflows powered by the Foundry MCP server.

## Prerequisites

### Install foundry-mcp (Required)

The plugin requires the `foundry-mcp` Python package to be installed:

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

> **Note:** The MCP server (`foundry-mcp`) is automatically registered when you install the plugin. Do not manually add it with `claude mcp add` as this would create a duplicate.

## Installation

### Install from Claude Code (Recommended)

From within Claude Code, add the marketplace and install the plugin:

```
/plugin marketplace add tylerburleigh/claude-foundry
/plugin install foundry@claude-foundry
```

Restart Claude Code and trust the repository when prompted. The plugin will auto-update when new versions are released.

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

### Commands

Slash commands for common workflows:

- `/setup-cmd` - First-time setup and guided tour of plugin features
- `/next-cmd` - Resume or start spec-driven development work

### Agents

Subagents for specialized tasks (spawned via Task tool).

## Configuration

The plugin requires the foundry-mcp MCP server to be installed and registered (see Prerequisites above). Once configured, the server starts automatically when Claude Code launches.

## Permissions

Claude Code requires explicit permissions for plugin tools. Add the permissions from `docs/examples/settings.local.json.example` to your project's `.claude/settings.local.json`.

**If you don't have a `.claude/settings.local.json` yet:**

```bash
mkdir -p .claude && curl -o .claude/settings.local.json \
  https://raw.githubusercontent.com/tylerburleigh/claude-foundry/main/docs/examples/settings.local.json.example
```

**If you already have one:** Merge the permissions from the sample file into your existing config. See `docs/examples/settings.local.json.example` in this repo for the full list.

### What's included

The sample config allows:

| Category | Permissions |
|----------|-------------|
| **MCP Tools** | All `mcp__plugin_foundry_foundry-mcp__*` tools |
| **Git** | `status`, `diff`, `log`, `rev-parse`, `branch`, `show` |
| **Testing** | `pytest`, `python -m pytest` |
| **Foundry CLI** | `foundry-mcp`, `foundry-cli` |
| **AI Providers** | `codex`, `claude`, `gemini`, `cursor-agent`, `opencode` |
| **File Access** | Write/Edit to `specs/` directories |
| **Denied** | Direct `Read` of spec JSON files (use MCP tools instead) |

### Customizing

Edit `.claude/settings.local.json` to add or remove permissions. See [Claude Code permissions docs](https://docs.anthropic.com/en/docs/claude-code) for the full syntax.

## License

MIT
