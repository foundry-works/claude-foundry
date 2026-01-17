# 7. Plugin Manifest

> The plugin.json file that defines plugin metadata, components, and configuration.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

The `plugin.json` manifest is the required configuration file for Claude Code plugins. It defines:

- Plugin identity and metadata
- Component paths (commands, agents, skills)
- Hook and MCP server configurations
- Distribution information

> **Beta Status:** Claude Code plugins are in public beta (October 2025). Features, schemas, and best practices may evolve. Check official documentation for the latest specifications.

---

## File Location

```
plugin-root/
├── .claude-plugin/
│   └── plugin.json     ← MUST be here
├── commands/
├── agents/
├── skills/
└── hooks/
```

| Requirement | Details |
|-------------|---------|
| MUST | Place plugin.json inside `.claude-plugin/` directory |
| MUST | Directory named exactly `.claude-plugin` |

---

## Complete Schema

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief explanation of plugin functionality",
  "author": {
    "name": "Your Name",
    "email": "your-email@example.com",
    "url": "https://github.com/username"
  },
  "homepage": "https://github.com/org/plugin",
  "repository": "https://github.com/org/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "commands": "./commands",
  "agents": "./agents",
  "skills": "./skills",
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./mcp/servers.json"
}
```

---

## Field Reference

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique identifier in kebab-case |

### Metadata Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | MUST | Unique identifier (kebab-case) |
| `version` | string | SHOULD | Semantic version (e.g., "2.1.0") |
| `description` | string | SHOULD | Brief functionality explanation |
| `author` | object | MAY | Creator information |
| `homepage` | string | MAY | Documentation URL |
| `repository` | string | MAY | Source code URL |
| `license` | string | SHOULD | SPDX identifier |
| `keywords` | array | MAY | Discovery tags |

### Component Path Fields

| Field | Type | Description |
|-------|------|-------------|
| `commands` | string/array | Slash command paths |
| `agents` | string/array | Subagent paths |
| `skills` | string/array | Skill directory paths |
| `hooks` | string/object | Hook config path or inline |
| `mcpServers` | string/object | MCP config path or inline |

---

## Metadata Details

### Name

| Requirement | Details |
|-------------|---------|
| MUST | Use kebab-case (lowercase with hyphens) |
| MUST | Be unique across installed plugins |

**Examples:**
```
✓ deployment-tools
✓ git-assistant
✓ code-review-bot
✗ DeploymentTools
✗ git_assistant
```

### Version

| Requirement | Details |
|-------------|---------|
| SHOULD | Use semantic versioning (MAJOR.MINOR.PATCH) |
| SHOULD | Increment MAJOR for breaking changes |
| SHOULD | Increment MINOR for new features |
| SHOULD | Increment PATCH for bug fixes |

### Author

The author field MUST be an object (not a string) with the following structure:

```json
{
  "author": {
    "name": "Team Name",
    "email": "team@example.com",
    "url": "https://github.com/team"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | MUST | Creator or team name |
| `email` | SHOULD | Contact email for support |
| `url` | MAY | Homepage or GitHub profile |

> **Note:** Unlike npm's package.json which accepts strings like `"Author Name <email>"`, plugin.json requires the object format.

### License

| Requirement | Details |
|-------------|---------|
| SHOULD | Use SPDX identifiers |
| SHOULD | Include LICENSE file in repo |

Common values: `MIT`, `Apache-2.0`, `GPL-3.0`, `BSD-3-Clause`

---

## Marketplace Fields

For plugins distributed via marketplaces, additional fields can be included:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Plugin description",
  "source": "github:username/repo",
  "category": "development",
  "tags": ["code-review", "testing", "automation"]
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | string | MAY | Repository source (e.g., `github:user/repo`) |
| `category` | string | MAY | Marketplace category for discovery |
| `tags` | array | MAY | Additional tags beyond keywords |

### Categories

Common marketplace categories:

| Category | Use For |
|----------|---------|
| `development` | Code editing, generation, review |
| `devops` | CI/CD, deployment, infrastructure |
| `testing` | Test automation, coverage, QA |
| `documentation` | Doc generation, API docs |
| `security` | Security scanning, vulnerability detection |
| `productivity` | Workflow automation, utilities |

---

## Component Paths

### Path Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Use relative paths starting with `./` |
| MUST | Paths are relative to plugin root |
| MAY | Use array for multiple paths |

### Default Directories

These directories are auto-discovered without explicit declaration:

| Directory | Purpose |
|-----------|---------|
| `./commands/` | Slash commands |
| `./agents/` | Subagents |
| `./skills/` | Skills |

### Commands

```json
{
  "commands": "./commands"
}
```

Or multiple paths:

```json
{
  "commands": ["./commands", "./legacy-commands"]
}
```

### Agents

```json
{
  "agents": "./agents"
}
```

### Skills

```json
{
  "skills": "./skills"
}
```

### Hooks

External file:
```json
{
  "hooks": "./hooks/hooks.json"
}
```

Inline configuration:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{"type": "command", "command": "./scripts/validate.sh"}]
      }
    ]
  }
}
```

### MCP Servers

External file:
```json
{
  "mcpServers": "./mcp/servers.json"
}
```

Inline configuration:
```json
{
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api.js"
    }
  }
}
```

---

## Environment Variables

Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths in:
- Hook configurations
- MCP server settings
- Script references

```json
{
  "mcpServers": {
    "server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/server",
      "env": {
        "CONFIG": "${CLAUDE_PLUGIN_ROOT}/config.json"
      }
    }
  }
}
```

---

## Examples

### Minimal Plugin

```json
{
  "name": "simple-tools"
}
```

### Standard Plugin

```json
{
  "name": "code-review-tools",
  "version": "1.0.0",
  "description": "Automated code review and analysis tools",
  "license": "MIT",
  "keywords": ["code-review", "analysis"]
}
```

### Full Plugin

```json
{
  "name": "enterprise-deployment",
  "version": "2.1.0",
  "description": "Comprehensive deployment and infrastructure management",
  "author": {
    "name": "DevOps Team",
    "email": "devops@company.com",
    "url": "https://github.com/company"
  },
  "homepage": "https://github.com/company/deployment-tools/wiki",
  "repository": "https://github.com/company/deployment-tools",
  "license": "Apache-2.0",
  "keywords": ["deployment", "infrastructure", "devops"],
  "commands": "./commands",
  "agents": ["./agents", "./specialized-agents"],
  "skills": "./skills",
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./mcp/servers.json"
}
```

---

## Best Practices

### Naming

| Requirement | Details |
|-------------|---------|
| MUST | Use kebab-case for plugin name |
| SHOULD | Use descriptive, meaningful names |
| SHOULD | Avoid generic names like "tools" or "utils" |

### Documentation

| Requirement | Details |
|-------------|---------|
| SHOULD | Include README.md with usage examples |
| SHOULD | Include LICENSE file |
| SHOULD | Include CHANGELOG.md for versions |
| SHOULD | Add meaningful keywords |

### Versioning

| Requirement | Details |
|-------------|---------|
| SHOULD | Follow semantic versioning |
| SHOULD | Document breaking changes |
| SHOULD | Provide release notes |

### Paths

| Requirement | Details |
|-------------|---------|
| MUST | Use relative paths with `./` |
| SHOULD | Use default directory names when possible |
| MAY | Supplement defaults with custom paths |

---

## Anti-Patterns

### Don't: Invalid Name Format

```json
// Bad: uppercase and underscore
{
  "name": "My_Plugin"
}
```

```json
// Good: kebab-case
{
  "name": "my-plugin"
}
```

### Don't: Absolute Paths

```json
// Bad: absolute path
{
  "commands": "/Users/dev/plugin/commands"
}
```

```json
// Good: relative path
{
  "commands": "./commands"
}
```

### Don't: Missing plugin.json Location

```
// Bad: plugin.json at root
plugin/
├── plugin.json          ✗
└── commands/
```

```
// Good: inside .claude-plugin/
plugin/
├── .claude-plugin/
│   └── plugin.json      ✓
└── commands/
```

---

## Related Documents

- [02-commands.md](./02-commands.md) - Slash commands
- [03-agents.md](./03-agents.md) - Subagents
- [04-skills.md](./04-skills.md) - Skills
- [05-hooks.md](./05-hooks.md) - Hooks
- [06-mcp-servers.md](./06-mcp-servers.md) - MCP servers
- [09-file-structure.md](./09-file-structure.md) - Directory layout

---

**Navigation:** [← Previous: MCP Servers](./06-mcp-servers.md) | [Index](./README.md) | [Next: Permissions →](./08-permissions.md)
