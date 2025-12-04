# 9. Frontmatter Schemas

> YAML frontmatter specifications for commands, agents, and skills.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Claude Code uses YAML frontmatter to configure commands, agents, and skills. Frontmatter appears at the top of markdown files, delimited by `---`.

```yaml
---
field: value
another-field: value
---

Content goes here...
```

---

## Command Frontmatter

### Schema

```yaml
---
description: Brief summary for /help and tool discovery
argument-hint: [required-arg] [optional-arg]
allowed-tools: Bash(git status:*), Bash(git commit:*)
model: claude-opus-4-5-20251101
disable-model-invocation: false
---
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `description` | string | SHOULD | Summary shown in `/help`; enables SlashCommand tool discovery |
| `argument-hint` | string | SHOULD | Documents expected parameters (e.g., `[file] [options]`) |
| `allowed-tools` | string | MAY | Comma-separated tool whitelist |
| `model` | string | MAY | Model override (e.g., `claude-opus-4-5-20251101`) |
| `disable-model-invocation` | boolean | MAY | Prevent programmatic invocation (default: `false`) |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include `description` for discoverable commands |
| SHOULD | Include `argument-hint` when command accepts arguments |
| SHOULD | Specify `allowed-tools` with precise patterns |
| MAY | Override `model` for performance-sensitive commands |

### Example

```yaml
---
description: Create a git commit with validation
argument-hint: [message]
allowed-tools: Bash(git status:*), Bash(git commit:*), Bash(git diff:*)
model: claude-sonnet-4-20250514
---

Review the changes and create a commit with the provided message: $ARGUMENTS
```

---

## Agent Frontmatter

### Schema

```yaml
---
name: unique-agent-identifier
description: When and why this agent should be invoked
tools: Read, Grep, Glob
model: sonnet
permissionMode: default
skills: skill1, skill2
---
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | MUST | Unique identifier (lowercase, hyphens only) |
| `description` | string | MUST | When this agent should be invoked |
| `tools` | string | SHOULD | Comma-separated tool whitelist |
| `model` | string | MAY | Model alias: `sonnet`, `opus`, `haiku`, `inherit` |
| `permissionMode` | string | MAY | Permission handling mode |
| `skills` | string | MAY | Pre-loaded skill names |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include `name` with valid characters |
| MUST | Include `description` for automatic invocation |
| SHOULD | Restrict `tools` to minimum required |
| MAY | Override `model` for specific agents |

### Name Constraints

| Requirement | Details |
|-------------|---------|
| MUST | Use lowercase letters only |
| MUST | Use hyphens for word separation |
| MUST NOT | Use underscores or uppercase |
| MUST NOT | Use spaces |

Valid: `code-reviewer`, `test-runner`, `sql-expert`
Invalid: `Code_Reviewer`, `TestRunner`, `code reviewer`

### Example

```yaml
---
name: code-reviewer
description: Reviews code changes for security, performance, and best practices
tools: Read, Grep, Glob
model: sonnet
---

You are an expert code reviewer. For each review:

1. Security: Check for vulnerabilities
2. Performance: Identify bottlenecks
3. Best practices: Verify standards

Output structured findings with severity levels.
```

---

## Skill Frontmatter

### Schema

```yaml
---
name: lowercase-alphanumeric-hyphens
description: What it does and when to use it
allowed-tools: Read, Bash
---
```

### Field Reference

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `name` | string | MUST | Lowercase, numbers, hyphens. Max 64 chars |
| `description` | string | MUST | What + when. Max 1024 chars |
| `allowed-tools` | array | MAY | Tool whitelist without prompts |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include `name` with valid characters |
| MUST | Include `description` with trigger context |
| SHOULD | Explain when/why to use the skill |
| MAY | Specify `allowed-tools` |

### Description Best Practices

| Requirement | Details |
|-------------|---------|
| MUST | Explain what the skill does |
| MUST | Explain when to use it |
| SHOULD | Name specific file types, formats, tools |
| SHOULD | Include trigger contexts |

### Example

```yaml
---
name: pdf-extractor
description: Extract text and tables from PDF files. Use when working with PDF documents, forms, or scanned reports.
allowed-tools: Read, Bash
---

# PDF Extraction

Extract content from PDF files:

1. Identify PDF type (text vs scanned)
2. Select appropriate extraction method
3. Structure output in requested format
```

---

## Comparison Table

| Field | Commands | Agents | Skills |
|-------|----------|--------|--------|
| `name` | - | MUST | MUST |
| `description` | SHOULD | MUST | MUST |
| `tools`/`allowed-tools` | MAY | SHOULD | MAY |
| `model` | MAY | MAY | - |
| `argument-hint` | SHOULD | - | - |
| `disable-model-invocation` | MAY | - | - |
| `permissionMode` | - | MAY | - |
| `skills` | - | MAY | - |

---

## Variable Substitution

### Commands Only

| Variable | Description | Example |
|----------|-------------|---------|
| `$ARGUMENTS` | All arguments as string | `/cmd arg1 arg2` → `"arg1 arg2"` |
| `$1`, `$2`... | Positional arguments | `$1` → `"arg1"` |

### File References

Use `@` prefix to include file contents (commands only):

```markdown
---
description: Review implementation
---

@src/implementation.ts
@docs/spec.md

Compare implementation against specification.
```

---

## Common Patterns

### Command with Arguments

```yaml
---
description: Run tests for a specific module
argument-hint: [module-name] [--verbose]
allowed-tools: Bash(npm run test:*)
---

Run tests for module: $1
Options: $2
```

### Agent with Tool Restrictions

```yaml
---
name: read-only-analyzer
description: Analyze code without making changes
tools: Read, Grep, Glob
---

You can only read and search code, not modify it.
```

### Skill with Trigger Context

```yaml
---
name: sql-optimizer
description: Analyze SQL queries for performance. Use when reviewing database queries or investigating slow queries.
allowed-tools: Read
---

Focus on query performance optimization.
```

---

## Anti-Patterns

### Don't: Missing Required Fields

```yaml
# Bad: agent without name
---
description: Reviews code
tools: Read
---
```

```yaml
# Good: all required fields
---
name: code-reviewer
description: Reviews code for issues
tools: Read
---
```

### Don't: Invalid Name Format

```yaml
# Bad: uppercase and underscores
---
name: Code_Reviewer
---
```

```yaml
# Good: lowercase with hyphens
---
name: code-reviewer
---
```

### Don't: Vague Description

```yaml
# Bad: no trigger context
---
name: helper
description: Helps with things
---
```

```yaml
# Good: specific with trigger
---
name: sql-optimizer
description: Analyze SQL queries for performance issues. Use when reviewing slow database queries.
---
```

### Don't: Overly Permissive Tools

```yaml
# Bad: wildcard
---
allowed-tools: Bash(*)
---
```

```yaml
# Good: specific tools
---
allowed-tools: Bash(npm run test:*), Bash(npm run lint:*)
---
```

---

## Validation Rules

### Name Validation

| Rule | Valid | Invalid |
|------|-------|---------|
| Lowercase only | `code-reviewer` | `Code-Reviewer` |
| Hyphens allowed | `test-runner` | `test_runner` |
| No spaces | `sql-expert` | `sql expert` |
| Alphanumeric | `tool-v2` | `tool@v2` |

### Description Validation

| Rule | Details |
|------|---------|
| Skills | Max 1024 characters |
| Should | Be specific and actionable |
| Should | Include trigger context |

### Tools Validation

| Rule | Details |
|------|---------|
| Format | Comma-separated list |
| Pattern | `ToolName(pattern:*)` for bash |
| Wildcards | Only at end with `:*` |

---

## Related Documents

- [01-commands.md](./01-commands.md) - Command details
- [02-agents.md](./02-agents.md) - Agent details
- [03-skills.md](./03-skills.md) - Skill details
- [08-file-structure.md](./08-file-structure.md) - File organization

---

**Navigation:** [← Previous: File Structure](./08-file-structure.md) | [Index](./README.md) | [Next: MCP Consumption →](./10-mcp-consumption.md)
