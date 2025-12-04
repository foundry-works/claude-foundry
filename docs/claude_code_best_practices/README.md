# Claude Code Plugin Best Practices

> Comprehensive guidance for building production-ready Claude Code plugins.

This documentation uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Quick Links

| Category | Documents | Focus |
|----------|-----------|-------|
| **Start Here** | [00-Choosing Features](./00-choosing-features.md) | When to use what |
| **Core Components** | [01-Commands](./01-commands.md), [02-Agents](./02-agents.md), [03-Skills](./03-skills.md) | Plugin features |
| **Automation** | [04-Hooks](./04-hooks.md), [05-MCP Servers](./05-mcp-servers.md) | Event handling, integrations |
| **Configuration** | [06-Plugin Manifest](./06-plugin-manifest.md), [07-Permissions](./07-permissions.md), [11-CLAUDE.md](./11-claude-md.md) | Setup, security, instructions |
| **Reference** | [08-File Structure](./08-file-structure.md), [09-Frontmatter](./09-frontmatter.md) | Standards, schemas |
| **Integration** | [10-MCP Consumption](./10-mcp-consumption.md) | Using MCP in Claude Code |

---

## Document Index

### Getting Started

| # | Document | Description |
|---|----------|-------------|
| 00 | [Choosing Features](./00-choosing-features.md) | When to use commands vs agents vs skills vs CLAUDE.md |

### Core Plugin Components

| # | Document | Description |
|---|----------|-------------|
| 01 | [Commands](./01-commands.md) | User-invoked slash commands with templating and bash execution |
| 02 | [Agents](./02-agents.md) | Autonomous subagents for complex, isolated task execution |
| 03 | [Skills](./03-skills.md) | Model-invoked capabilities automatically activated by context |

### Automation and Integration

| # | Document | Description |
|---|----------|-------------|
| 04 | [Hooks](./04-hooks.md) | Event-driven automation triggers for lifecycle events |
| 05 | [MCP Servers](./05-mcp-servers.md) | External tool integrations via Model Context Protocol |

### Configuration

| # | Document | Description |
|---|----------|-------------|
| 06 | [Plugin Manifest](./06-plugin-manifest.md) | plugin.json schema and configuration |
| 07 | [Permissions](./07-permissions.md) | Security model for tool and file access |
| 11 | [CLAUDE.md](./11-claude-md.md) | Project and user instructions for Claude |

### Reference

| # | Document | Description |
|---|----------|-------------|
| 08 | [File Structure](./08-file-structure.md) | Directory layout and naming conventions |
| 09 | [Frontmatter](./09-frontmatter.md) | YAML frontmatter schemas for all component types |
| 10 | [MCP Consumption](./10-mcp-consumption.md) | How Claude Code discovers and uses MCP servers |

---

## How to Use This Guide

### For New Plugin Developers

1. **Start with [00-Choosing Features](./00-choosing-features.md)** to understand when to use what
2. Read [06-Plugin Manifest](./06-plugin-manifest.md) for plugin structure
3. Read [08-File Structure](./08-file-structure.md) for directory conventions
4. Choose components based on your needs:
   - User shortcuts → [01-Commands](./01-commands.md)
   - Autonomous tasks → [02-Agents](./02-agents.md)
   - Reusable knowledge → [03-Skills](./03-skills.md)
   - Project standards → [11-CLAUDE.md](./11-claude-md.md)

### For Feature Implementation

| I want to... | Read |
|--------------|------|
| Create user-triggered shortcuts | [01-Commands](./01-commands.md) |
| Build autonomous task handlers | [02-Agents](./02-agents.md) |
| Share reusable expertise | [03-Skills](./03-skills.md) |
| Add event-driven automation | [04-Hooks](./04-hooks.md) |
| Integrate external tools | [05-MCP Servers](./05-mcp-servers.md) |
| Control tool access | [07-Permissions](./07-permissions.md) |
| Set project/user instructions | [11-CLAUDE.md](./11-claude-md.md) |

### For Reference

| I need to check... | Read |
|-------------------|------|
| Frontmatter fields | [09-Frontmatter](./09-frontmatter.md) |
| Directory layout | [08-File Structure](./08-file-structure.md) |
| Plugin.json schema | [06-Plugin Manifest](./06-plugin-manifest.md) |
| MCP integration | [10-MCP Consumption](./10-mcp-consumption.md) |
| CLAUDE.md format | [11-CLAUDE.md](./11-claude-md.md) |

---

## Component Selection Guide

For comprehensive guidance on when to use each feature, see **[00-Choosing Features](./00-choosing-features.md)**.

### Quick Summary

| I want to... | Use |
|--------------|-----|
| Set project standards | CLAUDE.md |
| Create user shortcuts | Command |
| Run complex workflows | Agent |
| Provide reusable expertise | Skill |
| Automate on events | Hook |
| Integrate external tools | MCP Server |

---

## Plugin Structure Overview

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Required
├── commands/                 # Optional
│   └── my-command.md
├── agents/                   # Optional
│   └── my-agent.md
├── skills/                   # Optional
│   └── my-skill/
│       └── SKILL.md
├── hooks/                    # Optional
│   └── hooks.json
├── .mcp.json                 # Optional
├── README.md
└── LICENSE
```

---

## Key Conventions

### Naming

| Type | Convention | Example |
|------|------------|---------|
| Plugin name | kebab-case | `code-review-tools` |
| Commands | kebab-case.md | `deploy-app.md` |
| Agents | kebab-case.md | `code-reviewer.md` |
| Skills | SKILL.md (uppercase) | `SKILL.md` |

### Paths

| Requirement | Details |
|-------------|---------|
| MUST | Use relative paths starting with `./` |
| MUST | Place plugin.json in `.claude-plugin/` |
| SHOULD | Use default directories when possible |

### Security

| Requirement | Details |
|-------------|---------|
| MUST | Never hardcode credentials |
| MUST | Deny access to sensitive files |
| SHOULD | Restrict tool access to minimum required |

---

## Related Documentation

- [Claude Code Official Docs](https://code.claude.com/docs/)
- [MCP Specification](https://modelcontextprotocol.io/)

---

*Last updated: 2025-12*
