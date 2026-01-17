# 2. Slash Commands

> User-invoked prompts that provide templated workflows and quick access to common operations.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Slash commands are markdown files that expand into prompts when users invoke them with `/command-name`. They enable:

- Templated workflows with dynamic arguments
- Pre-execution of bash commands to gather context
- File content inclusion via `@` references
- Restricted tool access for security

Commands can be project-scoped (shared with team) or user-scoped (personal workflows).

---

## File Location and Naming

### Location

| Scope | Location | Visibility |
|-------|----------|------------|
| Project | `.claude/commands/` | Team-shared via git |
| Personal | `~/.claude/commands/` | Private to user |
| Plugin | Declared in `plugin.json` | Distributed with plugin |

### Naming Conventions

| Requirement | Details |
|-------------|---------|
| MUST | Use `.md` extension |
| MUST | Default command name is filename (minus extension) unless frontmatter `name` overrides |
| SHOULD | Use kebab-case for multi-word names |
| MAY | Use subdirectories for organization (does not affect invocation) |

**Examples:**
```
.claude/commands/
├── review-pr.md          → /review-pr
├── git/
│   ├── commit.md         → /commit (not /git/commit)
│   └── stash.md          → /stash
└── testing/
    └── run-tests.md      → /run-tests
```

---

## Frontmatter Schema

Commands use optional YAML frontmatter:

```yaml
---
name: command-name
description: Brief summary for /help and tool discovery
argument-hint: [required-arg] [optional-arg]
allowed-tools: Bash(git status:*), Bash(git commit:*)
model: claude-opus-4-5-20251101
disable-model-invocation: false
---
```

### Field Reference

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `name` | string | MAY | Optional override for command name (defaults to filename) |
| `description` | string | SHOULD | Summary shown in `/help`; enables SlashCommand tool discovery |
| `argument-hint` | string | SHOULD | Documents expected parameters |
| `allowed-tools` | string | MAY | Comma-separated tool whitelist |
| `model` | string | MAY | Override model for this command |
| `disable-model-invocation` | boolean | MAY | Prevent programmatic invocation via SlashCommand tool |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include `description` if command should be discoverable by SlashCommand tool |
| SHOULD | Include `argument-hint` when command accepts arguments |
| SHOULD | Specify `allowed-tools` with precise patterns (not wildcards) |
| MAY | Set `name` to override filename; if set, MUST use kebab-case and SHOULD match the filename |
| MAY | Override `model` for performance-sensitive commands |

---

## Argument Handling

### Variable Substitution

| Variable | Description | Example |
|----------|-------------|---------|
| `$ARGUMENTS` | All arguments as single string | `/review 456 high` → `"456 high"` |
| `$1`, `$2`, `$3`... | Positional arguments | `/review 456 high` → `$1="456"`, `$2="high"` |

### Best Practices

| Requirement | Details |
|-------------|---------|
| MUST | Document expected arguments in `argument-hint` |
| SHOULD | Provide fallback instructions when arguments are optional |
| SHOULD | Validate arguments in bash before processing |

**Example with validation:**
```yaml
---
description: Review a pull request
argument-hint: [pr-number]
allowed-tools: Bash(gh pr view:*)
---

!`[ -z "$1" ] && echo "Error: PR number required" && exit 1`
!`gh pr view $1 --json title,body,files`

Review this PR focusing on code quality and test coverage.
```

---

## Bash Pre-execution

Commands can execute bash before Claude processes the prompt using `!` prefix:

```markdown
---
allowed-tools: Bash(git status:*), Bash(git log:*)
---

!`git status`
!`git log --oneline -5`

Based on the git status and recent commits above, suggest the next action.
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include bash commands in `allowed-tools` to enable execution |
| MUST | Use backticks around command: `!`command`` |
| SHOULD | Use specific command patterns, not wildcards |

---

## File References

Include file contents using `@` prefix:

```markdown
Compare these implementations:
@src/old-version.ts
@src/new-version.ts

Identify breaking changes and suggest migration steps.
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Use relative paths from project root |
| MUST NOT | Use absolute paths (not portable) |
| MAY | Reference multiple files for comparison workflows |

---

## Tool Access Control

The `allowed-tools` field restricts which tools the command can invoke:

```yaml
allowed-tools: Bash(git status:*), Bash(git log:*), Read, Glob
```

### Pattern Syntax

| Pattern | Matches |
|---------|---------|
| `Bash(git status:*)` | Any `git status` command |
| `Bash(npm run:*)` | Any `npm run` command |
| `Read` | All file reads |
| `Glob` | All glob searches |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Specify precise command patterns for bash |
| MUST NOT | Use `Bash(*)` wildcard (security risk) |
| SHOULD | Limit to minimum tools required |

---

## Plugin Commands

Plugin commands are declared in `plugin.json`:

```json
{
  "name": "my-plugin",
  "commands": "./commands/special.md"
}
```

Or multiple paths:

```json
{
  "commands": [
    "./commands/deploy.md",
    "./commands/rollback.md"
  ]
}
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Use relative paths starting with `./` |
| SHOULD | Place commands in `commands/` directory (auto-discovered) |
| MAY | Use namespace prefix in command names (e.g., `my-plugin:deploy`) |

---

## Context Management & Token Hygiene

- **Keep templates lean.** Put only the instructions needed to guide Claude; link or `@` attach larger specs instead of embedding them inline.
- **Validate before dumping.** Run bash pre-checks to scope file lists or diffs so you don’t paste entire repositories into the prompt.
- **Summarize command output.** Use bash to trim logs (e.g., `tail -n 200`) and prefer statistics over raw dumps.
- **Guide follow-up.** Ask Claude to confirm whether more context is needed rather than automatically inlining additional files.

## Examples

### Simple Template Command

```yaml
---
description: Generate unit tests for a file
argument-hint: [filepath]
---

Generate comprehensive unit tests for the file at $1.
Include edge cases and error handling tests.
```

### Context-Gathering Command

```yaml
---
description: Summarize recent git activity
allowed-tools: Bash(git log:*), Bash(git diff:*)
---

!`git log --oneline -10`
!`git diff --stat HEAD~5`

Summarize the recent development activity and identify key changes.
```

### Multi-File Review Command

```yaml
---
description: Review implementation against specification
argument-hint: [spec-file] [impl-file]
allowed-tools: Read
---

@$1
@$2

Compare the implementation against the specification.
Identify any deviations or missing requirements.
```

---

## Anti-Patterns

### Don't: Overly Permissive Tools

```yaml
# Bad: wildcard allows any bash command
allowed-tools: Bash(*)
```

```yaml
# Good: specific commands only
allowed-tools: Bash(git status:*), Bash(git log:*)
```

### Don't: Missing Description

```yaml
# Bad: command not discoverable
---
argument-hint: [file]
---
```

```yaml
# Good: clear description for discovery
---
description: Format code using project conventions
argument-hint: [file]
---
```

### Don't: Hardcoded Paths

```markdown
# Bad: not portable
@/Users/developer/project/src/file.ts
```

```markdown
# Good: relative path
@src/file.ts
```

### Don't: Silent Side Effects

```markdown
# Bad: modifies without warning
!`git add . && git commit -m "Auto-commit"`
```

```markdown
# Good: informative about actions
!`git status`

The above shows uncommitted changes. Should I create a commit with these changes?
```

---

## Commands vs Skills

| Use Case | Commands | Skills |
|----------|----------|--------|
| Simple templates | Yes | No |
| Quick workflows (1-3 steps) | Yes | No |
| Dynamic user arguments | Yes | Limited |
| Complex multi-step workflows | No | Yes |
| State management | No | Yes |
| Multiple resource coordination | Limited | Yes |

---

## Related Documents

- [03-agents.md](./03-agents.md) - Subagents for autonomous task handling
- [04-skills.md](./04-skills.md) - Model-invoked capabilities
- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [10-frontmatter.md](./10-frontmatter.md) - Frontmatter schemas
- [16-description-writing.md](./16-description-writing.md) - Writing effective command descriptions
- [17-system-prompts.md](./17-system-prompts.md) - System prompt design patterns

---

**Navigation:** [← Previous: Choosing Features](./01-choosing-features.md) | [Index](./README.md) | [Next: Agents →](./03-agents.md)
