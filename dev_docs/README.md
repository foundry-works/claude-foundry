# Claude Code Plugin Best Practices

> Comprehensive guidance for building production-ready Claude Code plugins.

This documentation uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Quick Links

| Category | Documents | Focus |
|----------|-----------|-------|
| **Start Here** | [00-Development Guide](./00-development-guide.md), [01-Choosing Features](./01-choosing-features.md) | How to build, when to use what |
| **Core Components** | [02-Commands](./02-commands.md), [03-Agents](./03-agents.md), [04-Skills](./04-skills.md) | Plugin features |
| **Automation** | [05-Hooks](./05-hooks.md), [06-MCP Servers](./06-mcp-servers.md) | Event handling, integrations |
| **Configuration** | [07-Plugin Manifest](./07-plugin-manifest.md), [08-Permissions](./08-permissions.md), [12-CLAUDE.md](./12-claude-md.md) | Setup, security, instructions |
| **Reference** | [09-File Structure](./09-file-structure.md), [10-Frontmatter](./10-frontmatter.md) | Standards, schemas |
| **Integration** | [11-MCP Consumption](./11-mcp-consumption.md) | Using MCP in Claude Code |
| **Operations** | [13-Testing](./13-testing.md), [14-Distribution](./14-distribution.md), [15-Debugging](./15-debugging.md) | Testing, publishing, debugging |
| **Prompt Engineering** | [16-Description Writing](./16-description-writing.md), [17-System Prompts](./17-system-prompts.md), [18-Context Management](./18-context-management.md) | Writing effective prompts |
| **Documentation** | [19-README Best Practices](./19-readme-best-practices.md) | Writing effective READMEs |

---

## Claude Code Development Philosophy

Claude Code plugins combine multiple component types to create layered, trustworthy assistance. Use this doc set with the following mental model:

### Core Pillars

- **Intent first.** Capture enduring guidance (CLAUDE.md), reusable expertise (skills), and safety guardrails (permissions) before building workflows. Claude performs best when expectations are explicit.
- **Compose capabilities.** Combine commands, agents, hooks, and MCP servers so each responsibility lives in the component that manages it best—templates for user intent, agents for isolated execution, hooks for automation, MCP for external data.
- **Assure operations.** Treat permissions, manifest metadata, and testing as first-class citizens so every component is auditable, reproducible, and safe to share.
- **Respect context budgets.** Design for minimal always-on text, keep prompts and outputs summarized, and use progressive disclosure (skills, agents, MCP resources) to avoid blowing past token limits during long sessions.

### Component Purposes at a Glance

| Component | Purpose | Design Notes |
|-----------|---------|--------------|
| **CLAUDE.md** | Always-on project and org memory | Use for standards that must be active in every turn. Keep concise; link to deeper docs. |
| **Skills** | Automatically surfaced expertise | Author descriptions that explain *when* Claude should load the skill. Keep SKILL.md high-level and move details into supporting files. |
| **Commands** | User-triggered templates with predictable arguments | Great for scaffolding workflows and surfacing info exactly when a human asks. Validate arguments and keep allowed-tools tight. |
| **Agents** | Isolated, autonomous task handlers | Specify responsibilities, tools, and output contracts. Use when you need multi-step reasoning or protected context windows. |
| **Hooks** | Event-driven automation and enforcement | Use sparingly for policy enforcement, formatting, or notifications. Keep scripts deterministic and fast. |
| **MCP Servers** | Boundary to external systems, tools, and datasets | Treat as integration layer. Document failure modes, auth, and rate limits so plugins degrade gracefully. |

### Layered Disclosure Framework

Surface information based on who controls the timing:

| Scenario | Recommended Mechanism | Rationale |
|----------|----------------------|-----------|
| Standards must always apply | CLAUDE.md | Persistent, auto-loaded context. |
| Expertise should appear only when relevant | Skill | Claude auto-activates based on task semantics. |
| User wants a guided workflow | Command | Human-in-the-loop trigger with optional bash preflight. |
| Deep work needs focused context | Agent prompt | Inject rich guidance without polluting main conversation. |
| External payload should stream on demand | MCP resource/tool | Fetch only what’s requested, keep heavy data outside base context. |

When in doubt, choose the mechanism that gives the right balance of discoverability, timing control, and operational safety for your audience.

---

## Document Index

### Getting Started

| # | Document | Description |
|---|----------|-------------|
| 00 | [Development Guide](./00-development-guide.md) | End-to-end plugin development methodology with worked example |
| 01 | [Choosing Features](./01-choosing-features.md) | When to use commands vs agents vs skills vs CLAUDE.md |

### Core Plugin Components

| # | Document | Description |
|---|----------|-------------|
| 02 | [Commands](./02-commands.md) | User-invoked slash commands with templating and bash execution |
| 03 | [Agents](./03-agents.md) | Autonomous subagents for complex, isolated task execution |
| 04 | [Skills](./04-skills.md) | Model-invoked capabilities with progressive disclosure and token optimization |

### Automation and Integration

| # | Document | Description |
|---|----------|-------------|
| 05 | [Hooks](./05-hooks.md) | Event-driven automation triggers for lifecycle events |
| 06 | [MCP Servers](./06-mcp-servers.md) | External tool integrations via Model Context Protocol |

### Configuration

| # | Document | Description |
|---|----------|-------------|
| 07 | [Plugin Manifest](./07-plugin-manifest.md) | plugin.json schema and configuration |
| 08 | [Permissions](./08-permissions.md) | Security model for tool and file access |
| 12 | [CLAUDE.md](./12-claude-md.md) | Project and user instructions for Claude |

### Reference

| # | Document | Description |
|---|----------|-------------|
| 09 | [File Structure](./09-file-structure.md) | Directory layout and naming conventions |
| 10 | [Frontmatter](./10-frontmatter.md) | YAML frontmatter schemas for all component types |
| 11 | [MCP Consumption](./11-mcp-consumption.md) | How Claude Code discovers and uses MCP servers |

### Operations

| # | Document | Description |
|---|----------|-------------|
| 13 | [Testing](./13-testing.md) | Plugin component validation and testing |
| 14 | [Distribution](./14-distribution.md) | Marketplace publishing, versioning, and maintenance |
| 15 | [Debugging](./15-debugging.md) | Troubleshooting plugin development issues |

### Prompt Engineering

| # | Document | Description |
|---|----------|-------------|
| 16 | [Description Writing](./16-description-writing.md) | Writing effective descriptions for skills, agents, and commands |
| 17 | [System Prompts](./17-system-prompts.md) | Designing system prompts with the "right altitude" principle |
| 18 | [Context Management](./18-context-management.md) | Token budgets, progressive disclosure, and context optimization |

### Documentation

| # | Document | Description |
|---|----------|-------------|
| 19 | [README Best Practices](./19-readme-best-practices.md) | Patterns and guidance for README structure and style |

---

## How to Use This Guide

### For New Plugin Developers

1. **Read [00-Development Guide](./00-development-guide.md)** for the complete development process
2. Use [01-Choosing Features](./01-choosing-features.md) to select the right features
3. Reference [07-Plugin Manifest](./07-plugin-manifest.md) and [09-File Structure](./09-file-structure.md) for structure
4. Deep-dive into specific features:
   - User shortcuts → [02-Commands](./02-commands.md)
   - Autonomous tasks → [03-Agents](./03-agents.md)
   - Reusable knowledge → [04-Skills](./04-skills.md)
   - Project standards → [12-CLAUDE.md](./12-claude-md.md)

### For Feature Implementation

| I want to... | Read |
|--------------|------|
| Create user-triggered shortcuts | [02-Commands](./02-commands.md) |
| Build autonomous task handlers | [03-Agents](./03-agents.md) |
| Share reusable expertise | [04-Skills](./04-skills.md) |
| Add event-driven automation | [05-Hooks](./05-hooks.md) |
| Integrate external tools | [06-MCP Servers](./06-mcp-servers.md) |
| Control tool access | [08-Permissions](./08-permissions.md) |
| Set project/user instructions | [12-CLAUDE.md](./12-claude-md.md) |
| Test my plugin | [13-Testing](./13-testing.md) |
| Publish to marketplace | [14-Distribution](./14-distribution.md) |
| Debug issues | [15-Debugging](./15-debugging.md) |
| Write effective descriptions | [16-Description Writing](./16-description-writing.md) |
| Design system prompts | [17-System Prompts](./17-system-prompts.md) |
| Manage context and tokens | [18-Context Management](./18-context-management.md) |

### For Reference

| I need to check... | Read |
|-------------------|------|
| Frontmatter fields | [10-Frontmatter](./10-frontmatter.md) |
| Directory layout | [09-File Structure](./09-file-structure.md) |
| Plugin.json schema | [07-Plugin Manifest](./07-plugin-manifest.md) |
| MCP integration | [11-MCP Consumption](./11-mcp-consumption.md) |
| CLAUDE.md format | [12-CLAUDE.md](./12-claude-md.md) |
| Debug techniques | [15-Debugging](./15-debugging.md) |
| Description patterns | [16-Description Writing](./16-description-writing.md) |
| Prompt templates | [17-System Prompts](./17-system-prompts.md) |
| Token budgets | [18-Context Management](./18-context-management.md) |

---

## Component Selection Guide

For comprehensive guidance on when to use each feature, see **[01-Choosing Features](./01-choosing-features.md)**.

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
├── mcp/                      # Optional
│   └── servers.json
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

*Last updated: 2025-12-22*
