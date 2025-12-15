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

## Installation

### Interactive install script

Run from command line. Requires `jq`. Prompts for confirmation before each step. Handles both fresh installs and reinstalls.

```bash
bash -c '
confirm() {
  read -p "$1 [y/N] " response
  [[ "$response" =~ ^[Yy]$ ]]
}

echo "=== Claude Foundry Plugin Installer ==="
echo ""

# Step 1: Check for existing installation and offer to clean
if [ -d ~/.claude/plugins/cache/claude-foundry ]; then
  echo "Found existing claude-foundry plugin cache."
  if confirm "Remove ~/.claude/plugins/cache/claude-foundry for clean reinstall?"; then
    rm -rf ~/.claude/plugins/cache/claude-foundry
    echo "✓ Removed claude-foundry plugin cache"
  fi
fi

if [ -d ~/.claude/plugins/marketplaces/claude-foundry ]; then
  echo "Found existing claude-foundry marketplace clone."
  if confirm "Remove ~/.claude/plugins/marketplaces/claude-foundry for clean reinstall?"; then
    rm -rf ~/.claude/plugins/marketplaces/claude-foundry
    echo "✓ Removed claude-foundry marketplace clone"
  fi
fi

# Step 2: Create settings file if needed
if [ ! -f ~/.claude/settings.json ] || [ ! -s ~/.claude/settings.json ]; then
  if confirm "Create ~/.claude/settings.json?"; then
    mkdir -p ~/.claude
    echo "{}" > ~/.claude/settings.json
    echo "✓ Created settings.json"
  else
    echo "Skipped. Cannot continue without settings.json."
    exit 1
  fi
fi

# Step 3: Add marketplace and enable plugin
if confirm "Add claude-foundry marketplace and enable plugin?"; then
  jq ". * {
    \"extraKnownMarketplaces\": {
      \"claude-foundry\": {
        \"source\": {\"source\": \"github\", \"repo\": \"tylerburleigh/claude-foundry\"}
      }
    },
    \"enabledPlugins\": {
      \"foundry@claude-foundry\": true
    }
  }" ~/.claude/settings.json > ~/.claude/settings.json.tmp && \
  mv ~/.claude/settings.json.tmp ~/.claude/settings.json
  echo "✓ Added marketplace and enabled plugin"
else
  echo "Skipped marketplace setup."
  exit 0
fi

echo ""
echo "Done! Restart Claude Code and trust the repository when prompted."
'
```

### Alternative: Install from within Claude Code

If you prefer, you can add the marketplace interactively from inside Claude Code:

```
/plugin marketplace add tylerburleigh/claude-foundry
```

Then run `/plugin` to browse and install available plugins.

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

- `/foundry-setup` - First-time setup and guided tour of plugin features
- `/sdd-next` - Resume or start spec-driven development work

### Agents

Subagents for specialized tasks (spawned via Task tool).

## Configuration

The plugin automatically configures the foundry-mcp MCP server when installed. The server starts automatically when the plugin is enabled - no manual MCP configuration needed.

If the MCP server fails to start, verify that `foundry-mcp` is installed (see Prerequisites above).

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
