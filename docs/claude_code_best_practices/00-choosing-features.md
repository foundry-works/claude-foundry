# 0. Choosing the Right Feature

> A comprehensive guide to selecting between commands, agents, skills, hooks, CLAUDE.md, and MCP servers.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Claude Code offers multiple ways to extend and customize its behavior. Choosing the right feature depends on:

- **Who triggers it** - User, Claude, or system events
- **What persists** - Session, project, or nothing
- **Where it runs** - Main context, isolated context, or external
- **Why you need it** - Shortcuts, knowledge, automation, or integration

---

## Feature Comparison

### Quick Reference

| Feature | Triggered By | Persistence | Context | Primary Use |
|---------|--------------|-------------|---------|-------------|
| **Commands** | User (`/cmd`) | None | Main | Quick shortcuts, templates |
| **Agents** | Claude delegation | Stateless | Isolated | Complex workflows |
| **Skills** | Auto (context match) | Session | Main | Reusable expertise |
| **Hooks** | System events | N/A | Event payload | Automation |
| **CLAUDE.md** | Always loaded | Persistent | All sessions | Project standards |
| **MCP Servers** | Natural language | Server-dependent | Tools/resources | External integrations |

### Detailed Comparison

| Aspect | Commands | Agents | Skills | Hooks | CLAUDE.md | MCP |
|--------|----------|--------|--------|-------|-----------|-----|
| **Invocation** | Explicit `/cmd` | Claude decides | Auto by context | Event trigger | Always active | Natural language |
| **User control** | Full | Partial | None | None | Setup only | Partial |
| **Isolation** | No | Yes | No | Yes | No | External |
| **Tool access** | Configurable | Configurable | Limited | Full | N/A | Server-defined |
| **File format** | `.md` | `.md` | `SKILL.md` | `.json` | `.md` | `.json` |
| **Arguments** | `$ARGUMENTS` | Prompt | N/A | Event data | N/A | Tool params |
| **Best for** | Shortcuts | Workflows | Knowledge | Automation | Standards | Integration |

---

## Decision Flowcharts

### "I Want Claude to Remember Something"

```
What should Claude remember?
├── Project coding standards, conventions?
│   └── CLAUDE.md (persists across sessions)
├── How to do a specific task?
│   ├── Task is complex with multiple steps?
│   │   └── Skill (auto-activated expertise)
│   └── Task is simple, user-triggered?
│       └── Command (template with instructions)
└── Domain knowledge for analysis?
    └── Skill (reusable across conversations)
```

### "I Want to Automate Something"

```
When should automation run?
├── Before/after tool execution?
│   └── Hook (PreToolUse/PostToolUse)
├── On specific user input patterns?
│   └── Hook (UserPromptSubmit)
├── When Claude finishes a task?
│   └── Hook (Stop/SubagentStop)
├── When user types a command?
│   └── Command (explicit trigger)
└── When context matches a pattern?
    └── Skill (auto-activated)
```

### "I Need Reusable Expertise"

```
How should expertise be used?
├── Claude should auto-detect when relevant?
│   └── Skill
├── User explicitly requests it?
│   └── Command
├── Needs isolated execution context?
│   └── Agent
└── Should always apply to this project?
    └── CLAUDE.md
```

### "I Want to Integrate External Tools"

```
What kind of integration?
├── External API or service?
│   └── MCP Server
├── Local CLI tool?
│   └── MCP Server (stdio) or Command with bash
├── Pre/post processing of Claude's actions?
│   └── Hook
└── Providing context from external source?
    └── MCP Server (resources)
```

---

## Use Case Guide

### Coding Standards & Conventions

| Need | Solution |
|------|----------|
| "Always use 2-space indentation" | CLAUDE.md |
| "Follow our naming conventions" | CLAUDE.md |
| "Use our architectural patterns" | CLAUDE.md |
| "Apply our code review checklist" | Skill or CLAUDE.md |

**Why CLAUDE.md**: Persists across sessions, always loaded, applies to all work.

### Workflow Shortcuts

| Need | Solution |
|------|----------|
| "Quick command to deploy" | Command |
| "Template for creating components" | Command |
| "Shortcut to run tests" | Command |
| "Generate boilerplate with args" | Command |

**Why Command**: User-triggered, supports arguments, can execute bash.

### Complex Task Execution

| Need | Solution |
|------|----------|
| "Review code thoroughly" | Agent |
| "Debug failing tests" | Agent |
| "Refactor a module" | Agent |
| "Research codebase" | Agent (Explore) |

**Why Agent**: Isolated context prevents pollution, autonomous execution.

### Reusable Knowledge

| Need | Solution |
|------|----------|
| "Security review expertise" | Skill |
| "Performance analysis patterns" | Skill |
| "Domain-specific terminology" | Skill or CLAUDE.md |
| "How to use our internal APIs" | Skill |

**Why Skill**: Auto-activated when relevant, progressive disclosure.

### Event-Driven Automation

| Need | Solution |
|------|----------|
| "Format code after edits" | Hook (PostToolUse) |
| "Validate before commits" | Hook (PreToolUse) |
| "Notify when done" | Hook (Stop) |
| "Block dangerous commands" | Hook (PreToolUse) |

**Why Hook**: Triggered by events, can block or modify operations.

### External Integrations

| Need | Solution |
|------|----------|
| "Query our database" | MCP Server |
| "Integrate with GitHub API" | MCP Server |
| "Access file system tools" | MCP Server |
| "Use custom CLI tools" | MCP Server |

**Why MCP**: Standard protocol, external execution, rich tool exposure.

---

## Common Scenarios

### Scenario: Team Coding Standards

**Need**: Ensure all team members follow consistent coding practices.

**Solution**: `./CLAUDE.md` (project-level)

```markdown
# Coding Standards

## Style
- TypeScript strict mode
- 2-space indentation
- No semicolons

## Patterns
- React functional components
- Custom hooks for logic
- Zustand for state
```

**Why not alternatives**:
- Command: Not always active
- Skill: Overkill for static guidelines
- Hook: Can't provide context

---

### Scenario: Code Review Workflow

**Need**: Thorough, consistent code reviews with security focus.

**Solution**: Agent (`agents/code-reviewer.md`)

```yaml
---
name: code-reviewer
description: Reviews code for security, performance, and best practices
tools: Read, Grep, Glob
---
```

**Why not alternatives**:
- Command: Too simple for complex analysis
- Skill: Can't execute autonomously
- CLAUDE.md: Not for workflows

---

### Scenario: Quick Deploy Shortcut

**Need**: One command to deploy to staging.

**Solution**: Command (`commands/deploy.md`)

```yaml
---
description: Deploy to staging environment
argument-hint: [environment]
allowed-tools: Bash(npm run deploy:*)
---

Deploy to $1 environment using our deployment script.
```

**Why not alternatives**:
- Agent: Overkill for simple task
- Skill: Needs explicit user trigger
- Hook: Not user-initiated

---

### Scenario: Security Expertise

**Need**: Apply security knowledge when reviewing any code.

**Solution**: Skill (`skills/security-analysis/SKILL.md`)

```yaml
---
name: security-analysis
description: Analyze code for security vulnerabilities. Use when reviewing code for security issues.
---
```

**Why not alternatives**:
- Command: Needs explicit invocation
- Agent: Not always needed
- CLAUDE.md: Too static for detailed expertise

---

### Scenario: Pre-Commit Validation

**Need**: Automatically validate before git commits.

**Solution**: Hook (PreToolUse on Bash)

```json
{
  "PreToolUse": [{
    "matcher": "Bash",
    "hooks": [{
      "type": "command",
      "command": "./scripts/validate-commit.sh"
    }]
  }]
}
```

**Why not alternatives**:
- Command: Not automatic
- Skill: Can't intercept events
- Agent: Not event-driven

---

### Scenario: Database Integration

**Need**: Query production database from Claude.

**Solution**: MCP Server

```json
{
  "mcpServers": {
    "database": {
      "command": "npx",
      "args": ["@mcp/postgres"],
      "env": {"DATABASE_URL": "${DB_URL}"}
    }
  }
}
```

**Why not alternatives**:
- Command: Can't provide rich tools
- Hook: Not for external services
- Skill: Can't execute queries

---

## Anti-Patterns

### Don't: Use CLAUDE.md for Workflows

```markdown
# Bad: CLAUDE.md with workflow steps
When deploying:
1. Run npm run build
2. Run npm run test
3. Run npm run deploy
```

```yaml
# Good: Command for workflows
---
description: Deploy to production
allowed-tools: Bash(npm run:*)
---
```

### Don't: Use Commands for Always-On Knowledge

```yaml
# Bad: Command for coding standards
---
description: Remember our coding standards
---
Use 2-space indentation...
```

```markdown
# Good: CLAUDE.md for standards
## Coding Standards
- 2-space indentation
```

### Don't: Use Skills for User-Triggered Actions

```yaml
# Bad: Skill that needs explicit trigger
---
name: deploy-helper
description: Helps with deployment
---
```

```yaml
# Good: Command for explicit actions
---
description: Deploy to environment
argument-hint: [env]
---
```

### Don't: Use Agents for Simple Tasks

```yaml
# Bad: Agent for simple task
---
name: formatter
description: Format a single file
---
```

```yaml
# Good: Command for simple tasks
---
description: Format current file
allowed-tools: Bash(prettier:*)
---
```

---

## Summary Matrix

| I want to... | Use |
|--------------|-----|
| Set project coding standards | CLAUDE.md |
| Create a quick shortcut | Command |
| Run complex autonomous workflow | Agent |
| Provide reusable expertise | Skill |
| Automate on events | Hook |
| Integrate external tools | MCP Server |
| Remember across sessions | CLAUDE.md |
| Isolate task execution | Agent |
| Auto-activate by context | Skill |
| Block dangerous operations | Hook |
| Accept user arguments | Command |
| Query external APIs | MCP Server |

---

## Related Documents

- [01-commands.md](./01-commands.md) - Slash commands
- [02-agents.md](./02-agents.md) - Subagents
- [03-skills.md](./03-skills.md) - Skills
- [04-hooks.md](./04-hooks.md) - Hooks
- [05-mcp-servers.md](./05-mcp-servers.md) - MCP servers
- [11-claude-md.md](./11-claude-md.md) - CLAUDE.md

---

**Navigation:** [Index](./README.md) | [Next: Commands →](./01-commands.md)
