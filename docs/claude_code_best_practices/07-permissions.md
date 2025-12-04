# 7. Permissions

> The security model that controls tool access, file operations, and command execution.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Claude Code uses a permission system to control:

- **Tool access** - Which tools Claude can invoke
- **File operations** - Which files can be read or modified
- **Command execution** - Which bash commands can run
- **Network access** - Which domains can be fetched

---

## Rule Types

### Permission Categories

| Type | Behavior | Precedence |
|------|----------|------------|
| `deny` | Block entirely | Highest |
| `ask` | Prompt for approval | Medium |
| `allow` | Grant without prompt | Lowest |

### Processing Order

1. PreToolUse Hook evaluation
2. Deny Rules (evaluated first)
3. Allow Rules
4. Ask Rules
5. Permission Mode Check
6. PostToolUse Hook execution

---

## Configuration Format

### Settings File Structure

```json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Bash(npm run:*)",
      "Edit(/src/**)"
    ],
    "ask": [
      "Bash(git push:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./secrets/**)"
    ]
  },
  "defaultMode": "default"
}
```

### Configuration Locations

| Location | Scope | Precedence |
|----------|-------|------------|
| Enterprise managed | System-wide | Highest |
| CLI arguments | Session | High |
| `.claude/settings.local.json` | Personal project | Medium |
| `.claude/settings.json` | Shared project | Medium |
| `~/.claude/settings.json` | User global | Lowest |

### Enterprise Policy Paths

| Platform | Path |
|----------|------|
| macOS | `/Library/Application Support/ClaudeCode/managed-settings.json` |
| Linux/WSL | `/etc/claude-code/managed-settings.json` |
| Windows | `C:\ProgramData\ClaudeCode\managed-settings.json` |

---

## Pattern Syntax

### File Path Patterns

| Prefix | Scope | Example |
|--------|-------|---------|
| `//path` | Absolute filesystem | `Read(//Users/alice/secrets/**)` |
| `~/path` | Home directory | `Read(~/.zshrc)` |
| `/path` | Settings file location | `Edit(/src/**/*.ts)` |
| `./path` | Current directory | `Read(./config.json)` |

All patterns support gitignore-style globs with `**` for recursive matching.

### Bash Command Patterns

```json
{
  "allow": [
    "Bash(npm run build)",     // Exact match
    "Bash(npm run test:*)",    // Prefix with wildcard
    "Bash(git checkout:*)"     // Git variants
  ]
}
```

| Requirement | Details |
|-------------|---------|
| MUST | Use `:*` suffix for wildcard matching |
| MUST | Match command prefix exactly |
| MAY | Match exact command without wildcard |

**Limitations:**
- Options before command prevent matching
- Redirects and variables prevent matching
- Only end-of-pattern wildcards supported

### WebFetch Patterns

```json
{
  "allow": ["WebFetch(domain:github.com)"],
  "deny": ["WebFetch(domain:internal.example.com)"]
}
```

### MCP Tool Patterns

```json
{
  "allow": [
    "mcp__puppeteer",
    "mcp__puppeteer__navigate",
    "mcp__filesystem__list_directory"
  ]
}
```

| Requirement | Details |
|-------------|---------|
| MUST | List each MCP tool individually |
| MUST NOT | Use wildcards (not supported) |

---

## Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Prompts on first use, remembers choice |
| `plan` | Read-only, no modifications |
| `acceptEdits` | Auto-accepts file edits |
| `bypassPermissions` | Skips all prompts |

### Requirements

| Requirement | Details |
|-------------|---------|
| SHOULD | Use `default` for normal development |
| SHOULD | Use `plan` for design/review phases |
| MAY | Use `bypassPermissions` in sandboxed environments |
| MUST NOT | Use `bypassPermissions` with untrusted code |

---

## Common Patterns

### Development Safe Defaults

```json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Bash(npm:*)",
      "Bash(npm run:*)",
      "Bash(git:*)",
      "Edit(/src/**)"
    ],
    "deny": [
      "Read(./.env*)",
      "Read(./secrets/**)",
      "Bash(curl:*)",
      "Bash(sudo:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(git rebase:*)"
    ]
  }
}
```

### Enterprise Hardened

```json
{
  "permissions": {
    "allow": [
      "Read(/src/**)",
      "Edit(/src/**)"
    ],
    "deny": [
      "Bash(*)",
      "Read(./.env*)",
      "Read(./secrets/**)",
      "Read(~/.ssh/**)"
    ]
  },
  "defaultMode": "plan",
  "disableBypassPermissionsMode": true
}
```

### Sensitive File Protection

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./.aws/**)",
      "Read(~/.ssh/**)",
      "Read(./.git/config)"
    ]
  }
}
```

### Dangerous Command Blocking

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(ssh:*)",
      "Bash(git push --force:*)"
    ]
  }
}
```

---

## Enterprise Controls

### Managed Settings

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Bash(sudo:*)",
      "Bash(curl:*)"
    ]
  },
  "disableBypassPermissionsMode": true,
  "environmentVariables": {
    "CORPORATE_PROXY": "proxy.internal.example.com"
  }
}
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Deploy to system-level paths |
| MUST | Cannot be overridden by users |
| SHOULD | Enforce `disableBypassPermissionsMode` |
| SHOULD | Block risky commands organization-wide |

---

## Hook-Based Permissions

### PreToolUse Hook

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/permission-validator.sh"
          }
        ]
      }
    ]
  }
}
```

### Permission Decision Output

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow|deny|ask",
    "permissionDecisionReason": "explanation"
  }
}
```

### LLM-Based Evaluation

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(rm:*)",
        "hooks": [
          {
            "type": "prompt",
            "command": "Is this file deletion safe? Context: $ARGUMENTS"
          }
        ]
      }
    ]
  }
}
```

---

## Best Practices

### Security

| Requirement | Details |
|-------------|---------|
| MUST | Deny access to `.env` and secrets files |
| MUST | Block dangerous bash commands by default |
| SHOULD | Use `default` mode for normal development |
| SHOULD | Review permissions with `/permissions` command |
| MUST NOT | Use `bypassPermissions` outside sandboxes |

### Granularity

| Requirement | Details |
|-------------|---------|
| SHOULD | Use specific patterns, not broad wildcards |
| SHOULD | Separate allow/deny by concern |
| SHOULD | Use hooks for complex logic |
| MAY | Use LLM evaluation for context-aware decisions |

### Maintenance

| Requirement | Details |
|-------------|---------|
| SHOULD | Document permission rationale |
| SHOULD | Audit permissions regularly |
| SHOULD | Test permission changes in isolation |

---

## Anti-Patterns

### Don't: Overly Permissive

```json
// Bad: too broad
{
  "allow": ["Bash(*)", "Read(**)", "Edit(**)"]
}
```

```json
// Good: specific permissions
{
  "allow": [
    "Bash(npm run:*)",
    "Read(/src/**)",
    "Edit(/src/**/*.ts)"
  ]
}
```

### Don't: Missing Sensitive File Protection

```json
// Bad: no protection
{
  "allow": ["Read(**)"]
}
```

```json
// Good: explicit denials
{
  "allow": ["Read(**)"],
  "deny": ["Read(./.env*)", "Read(./secrets/**)"]
}
```

### Don't: Bypass in Production

```json
// Bad: dangerous in production
{
  "defaultMode": "bypassPermissions"
}
```

```json
// Good: explicit approvals
{
  "defaultMode": "default"
}
```

---

## Debugging

| Command | Purpose |
|---------|---------|
| `/permissions` | View all active rules |
| `/permissions` | Check rule sources |
| `/hooks` | View hook configurations |

---

## Related Documents

- [04-hooks.md](./04-hooks.md) - Hook-based permission control
- [05-mcp-servers.md](./05-mcp-servers.md) - MCP server permissions
- [06-plugin-manifest.md](./06-plugin-manifest.md) - Plugin configuration

---

**Navigation:** [← Previous: Plugin Manifest](./06-plugin-manifest.md) | [Index](./README.md) | [Next: File Structure →](./08-file-structure.md)
