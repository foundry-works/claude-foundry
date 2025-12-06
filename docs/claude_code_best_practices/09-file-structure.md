# 9. File Structure

> Directory layout and naming conventions for Claude Code plugins.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Claude Code plugins follow a standardized directory structure that enables:

- Automatic component discovery
- Clear organization of different component types
- Consistent patterns across plugins
- Easy maintenance and navigation

---

## Plugin Directory Structure

### Complete Layout

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Required: plugin manifest
├── commands/                 # Optional: slash commands
│   ├── deploy.md
│   └── review.md
├── agents/                   # Optional: subagents
│   ├── code-reviewer.md
│   └── test-runner.md
├── skills/                   # Optional: skills
│   ├── security-review/
│   │   ├── SKILL.md
│   │   └── reference.md
│   └── code-analysis/
│       └── SKILL.md
├── hooks/                    # Optional: hook configuration
│   └── hooks.json
├── scripts/                  # Optional: utility scripts
│   ├── validate.sh
│   └── format.sh
├── servers/                  # Optional: MCP server executables
│   └── api-server.js
├── .mcp.json                 # Optional: MCP server config
├── README.md                 # Recommended: documentation
├── LICENSE                   # Recommended: license file
└── CHANGELOG.md              # Recommended: version history
```

---

## Required Structure

### plugin.json Location

| Requirement | Details |
|-------------|---------|
| MUST | Create `.claude-plugin/` directory |
| MUST | Place `plugin.json` inside `.claude-plugin/` |
| MUST NOT | Place plugin.json at root level |

```
✓ Correct:
plugin/
├── .claude-plugin/
│   └── plugin.json

✗ Wrong:
plugin/
├── plugin.json
```

---

## Component Directories

### Commands Directory

| Aspect | Details |
|--------|---------|
| Location | `commands/` at plugin root |
| Discovery | Automatic |
| Contents | Markdown files with frontmatter |

```
commands/
├── deploy.md
├── review.md
└── git/
    ├── commit.md
    └── push.md
```

| Requirement | Details |
|-------------|---------|
| MUST | Use `.md` extension |
| SHOULD | Use kebab-case filenames |
| MAY | Use subdirectories for organization |

### Agents Directory

| Aspect | Details |
|--------|---------|
| Location | `agents/` at plugin root |
| Discovery | Automatic |
| Contents | Markdown files with frontmatter |

```
agents/
├── code-reviewer.md
├── test-runner.md
└── doc-writer.md
```

| Requirement | Details |
|-------------|---------|
| MUST | Use `.md` extension |
| MUST | Use lowercase with hyphens only |
| MUST NOT | Use underscores or uppercase |

### Skills Directory

| Aspect | Details |
|--------|---------|
| Location | `skills/` at plugin root |
| Discovery | Automatic |
| Contents | Subdirectories with SKILL.md |

```
skills/
├── security-review/
│   ├── SKILL.md           # Required
│   ├── reference.md       # Optional
│   └── examples.md        # Optional
└── code-analysis/
    └── SKILL.md           # Required
```

| Requirement | Details |
|-------------|---------|
| MUST | Create subdirectory per skill |
| MUST | Include SKILL.md in each skill directory |
| MAY | Include supporting files (reference.md, examples.md) |

### Hooks Directory

| Aspect | Details |
|--------|---------|
| Location | `hooks/` at plugin root |
| Contents | hooks.json configuration |

```
hooks/
└── hooks.json
```

| Requirement | Details |
|-------------|---------|
| SHOULD | Name file `hooks.json` |
| MAY | Reference from plugin.json |

---

## Naming Conventions

### Directory Names

| Type | Convention | Example |
|------|------------|---------|
| Components | Lowercase, plural | `commands/`, `agents/`, `skills/` |
| Skills | Lowercase, hyphens | `security-review/`, `code-analysis/` |
| Scripts | Lowercase | `scripts/` |
| Servers | Lowercase | `servers/` |

### File Names

| Type | Convention | Example |
|------|------------|---------|
| Commands | kebab-case.md | `deploy-app.md` |
| Agents | kebab-case.md | `code-reviewer.md` |
| Skills | SKILL.md (uppercase) | `SKILL.md` |
| Hooks | hooks.json | `hooks.json` |
| Scripts | kebab-case.sh | `validate-input.sh` |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Use kebab-case for command/agent files |
| MUST | Use `SKILL.md` (uppercase) for skill files |
| MUST | Use lowercase for directory names |
| MUST NOT | Use spaces in any names |
| SHOULD | Use descriptive, meaningful names |

---

## Auto-Discovery

### Default Directories

These directories are automatically discovered without explicit declaration in plugin.json:

| Directory | Component Type |
|-----------|----------------|
| `commands/` | Slash commands |
| `agents/` | Subagents |
| `skills/` | Skills |

### Custom Paths

Custom paths supplement (don't replace) defaults:

```json
{
  "commands": ["./commands", "./legacy-commands"],
  "agents": ["./agents", "./specialized-agents"]
}
```

| Requirement | Details |
|-------------|---------|
| MUST | Use relative paths starting with `./` |
| MAY | Specify multiple paths as array |
| MAY | Omit if using default directories |

---

## Supporting Files

### Documentation

| File | Purpose |
|------|---------|
| `README.md` | Plugin documentation |
| `LICENSE` | License information |
| `CHANGELOG.md` | Version history |

| Requirement | Details |
|-------------|---------|
| SHOULD | Include README.md with usage examples |
| SHOULD | Include LICENSE file |
| SHOULD | Include CHANGELOG.md for versioning |

### Scripts

```
scripts/
├── validate.sh
├── format.sh
└── setup.sh
```

| Requirement | Details |
|-------------|---------|
| SHOULD | Place utility scripts in `scripts/` |
| SHOULD | Make scripts executable |
| SHOULD | Use `${CLAUDE_PLUGIN_ROOT}` for paths |

### Servers

```
servers/
├── api-server.js
└── db-server.py
```

| Requirement | Details |
|-------------|---------|
| SHOULD | Place MCP server executables in `servers/` |
| SHOULD | Reference via `${CLAUDE_PLUGIN_ROOT}/servers/` |

---

## Project vs Plugin Structure

### Project Structure (.claude/)

```
project/
├── .claude/
│   ├── settings.json        # Project settings
│   ├── settings.local.json  # Personal settings
│   ├── commands/            # Project commands
│   ├── agents/              # Project agents
│   └── skills/              # Project skills
└── src/
```

### Plugin Structure

```
plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/                 # Plugin commands
├── agents/                   # Plugin agents
└── skills/                   # Plugin skills
```

### Key Differences

| Aspect | Project | Plugin |
|--------|---------|--------|
| Config location | `.claude/` | `.claude-plugin/` |
| Config file | `settings.json` | `plugin.json` |
| Distribution | Local only | Installable |
| Namespace | None | Plugin name prefix |

---

## Examples

### Minimal Plugin

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── commands/
    └── hello.md
```

### Standard Plugin

```
code-review-tools/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── review.md
│   └── analyze.md
├── agents/
│   └── code-reviewer.md
├── skills/
│   └── security-analysis/
│       └── SKILL.md
├── README.md
└── LICENSE
```

### Full Plugin

```
enterprise-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── deploy.md
│   ├── rollback.md
│   └── status.md
├── agents/
│   ├── deployment-agent.md
│   └── monitoring-agent.md
├── skills/
│   ├── infrastructure/
│   │   ├── SKILL.md
│   │   └── reference.md
│   └── compliance/
│       └── SKILL.md
├── hooks/
│   └── hooks.json
├── scripts/
│   ├── validate-deploy.sh
│   └── health-check.sh
├── servers/
│   └── metrics-server.js
├── .mcp.json
├── README.md
├── LICENSE
└── CHANGELOG.md
```

---

## Anti-Patterns

### Don't: Wrong plugin.json Location

```
# Bad
plugin/
├── plugin.json          ✗

# Good
plugin/
├── .claude-plugin/
│   └── plugin.json      ✓
```

### Don't: Mixed Case Names

```
# Bad
commands/
├── DeployApp.md         ✗
├── Code_Review.md       ✗

# Good
commands/
├── deploy-app.md        ✓
├── code-review.md       ✓
```

### Don't: Skill Files at Root

```
# Bad
skills/
├── SKILL.md             ✗ (no subdirectory)

# Good
skills/
└── my-skill/
    └── SKILL.md         ✓
```

### Don't: Components Inside .claude-plugin

```
# Bad
.claude-plugin/
├── plugin.json
├── commands/            ✗
└── agents/              ✗

# Good
.claude-plugin/
└── plugin.json
commands/                ✓ (at plugin root)
agents/                  ✓ (at plugin root)
```

---

## Related Documents

- [02-commands.md](./02-commands.md) - Command file format
- [03-agents.md](./03-agents.md) - Agent file format
- [04-skills.md](./04-skills.md) - Skill directory format
- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [10-frontmatter.md](./10-frontmatter.md) - Frontmatter schemas

---

**Navigation:** [← Previous: Permissions](./08-permissions.md) | [Index](./README.md) | [Next: Frontmatter →](./10-frontmatter.md)
