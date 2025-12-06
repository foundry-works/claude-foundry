# 5. Hooks

> Event-driven automation that executes shell commands or LLM evaluations in response to Claude Code lifecycle events.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Hooks are automation triggers that respond to Claude Code events. They can:

- **Execute shell commands** at specific lifecycle points
- **Block operations** based on validation or security rules
- **Modify inputs** before tool execution
- **Inject context** into conversations
- **Query an LLM** for intelligent decisions

---

## Configuration Format

### Location

| Scope | Location | Priority |
|-------|----------|----------|
| User | `~/.claude/settings.json` | Lowest |
| Project (shared) | `.claude/settings.json` | Higher |
| Project (personal) | `.claude/settings.local.json` | Higher |
| Plugin | `hooks/hooks.json` | Merged |
| Enterprise | `managed-settings.json` | Highest |

### Schema

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/validator.sh",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `matcher` | string | Conditional | Pattern to match tools/events |
| `hooks` | array | MUST | Array of hook actions |
| `type` | string | MUST | `command` or `prompt` |
| `command` | string | MUST (command) | Bash command to execute |
| `timeout` | number | MAY | Timeout in seconds (default: 60) |

### Matcher Patterns

| Pattern | Example | Matches |
|---------|---------|---------|
| Exact | `"Bash"` | Tool named "Bash" |
| Regex | `"Read.*"` | Tools matching regex |
| Wildcard | `"*"` | All tools |
| MCP | `"mcp__server__tool"` | MCP tools |

---

## Hook Events

### Event Reference

| Event | Matcher | Blocking | Purpose |
|-------|---------|----------|---------|
| `PreToolUse` | Yes | Yes | Before tool execution |
| `PostToolUse` | Yes | No | After tool completes |
| `UserPromptSubmit` | No | Yes | Before prompt processing |
| `PermissionRequest` | Yes | Yes | When permission shown |
| `Stop` | No | Yes | Main agent finished |
| `SubagentStop` | No | Yes | Subagent completed |
| `SessionStart` | Yes | No | Session initialization |
| `SessionEnd` | No | No | Session termination |
| `PreCompact` | Yes | No | Before context compaction |
| `Notification` | Yes | No | Alert handling |

### Event Input Payload

All hooks receive JSON on stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/conversation.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse"
}
```

Event-specific fields vary by hook type.

---

## Hook Types

### Command Hooks

Execute bash scripts with full system access:

```json
{
  "type": "command",
  "command": "${CLAUDE_PROJECT_DIR}/scripts/validate.sh",
  "timeout": 30
}
```

### Prompt Hooks

Query Claude Haiku for intelligent decisions:

```json
{
  "type": "prompt",
  "command": "Should this tool call be allowed? Input: $ARGUMENTS"
}
```

Available for: PreToolUse, PermissionRequest, Stop, SubagentStop, UserPromptSubmit

---

## Exit Codes and Output

### Exit Code Behavior

| Exit Code | Behavior | Output Handling |
|-----------|----------|-----------------|
| 0 | Success - execution continues | stdout parsed as JSON for structured control |
| 2 | Blocking error - operation prevented | **Only stderr used as message; JSON in stdout ignored** |
| 1, 3+ | Non-blocking error - logged, continues | stderr displayed in verbose mode only |

> **Important:** When exit code 2 is returned, the hook output is formatted as `[command]: {stderr}`. Any JSON in stdout is completely ignored. Use stderr to provide the blocking reason.

### Structured Output

Hooks can output JSON to control behavior:

```json
{
  "continue": true,
  "stopReason": "message when continue is false",
  "suppressOutput": false,
  "systemMessage": "warning to user"
}
```

### Event-Specific Decisions

**PreToolUse:**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow|deny|ask",
    "permissionDecisionReason": "Explanation for the decision",
    "updatedInput": { "field": "modified_value" },
    "additionalContext": "Context string injected into conversation"
  }
}
```

**PermissionRequest:**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "permissionDecision": "allow|deny",
    "permissionDecisionReason": "Reason for automatic approval/denial",
    "message": "Custom message shown to user"
  }
}
```

**PostToolUse:**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Feedback or context for Claude after tool execution"
  }
}
```

### Tool Input Modification (v2.0.10+)

PreToolUse hooks can modify tool inputs before execution using the `updatedInput` field:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "updatedInput": {
      "command": "npm test --coverage"
    }
  }
}
```

This intercepts the tool call, modifies the JSON input, and lets execution proceed with corrected parameters—avoiding the need to block and force retry.

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `$CLAUDE_PROJECT_DIR` | Project directory (absolute) |
| `$CLAUDE_PLUGIN_ROOT` | Plugin root (for plugin hooks) |
| `$CLAUDE_ENV_FILE` | Environment persistence file (SessionStart) |
| `$CLAUDE_CODE_REMOTE` | Set to `"true"` when running in remote/SSH context; absent for local execution |

---

## Execution Behavior

### Parallelization

- All matching hooks execute in **parallel**
- Identical commands deduplicated
- Execution order non-deterministic

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Handle parallel execution correctly |
| MUST NOT | Depend on hook execution order |
| SHOULD | Keep hooks fast (< 1 second target) |
| MAY | Use timeout for long-running hooks |

---

## Plugin Hooks

### Directory Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── hooks/
    └── hooks.json
```

### Plugin Hook Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Place hooks.json in hooks/ directory |
| MUST | Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths |
| SHOULD | Use `${CLAUDE_PROJECT_DIR}` for project-relative paths |

---

## Common Patterns

### Code Formatting (PostToolUse)

```bash
#!/bin/bash
# Format TypeScript files after write
prettier --write "$1" 2>/dev/null || true
```

### Access Control (PreToolUse)

```bash
#!/bin/bash
set -eu

INPUT=$(cat)
TOOL_INPUT=$(echo "$INPUT" | jq -r '.tool_input // ""')

if [[ "$TOOL_INPUT" =~ "\.env"|".git"|"\.ssh" ]]; then
  jq -n '{
    "continue": false,
    "stopReason": "Sensitive file access blocked"
  }'
  exit 2
fi
```

### Audit Logging (PostToolUse)

```bash
#!/bin/bash
jq -r '"\(.tool_name): \(.tool_input.command // .tool_input.path)"' | \
  tee -a ~/.claude/audit-log.txt
```

### Environment Persistence (SessionStart)

```bash
#!/bin/bash
echo 'export TEAM_CONFIG=/shared/config.json' >> "$CLAUDE_ENV_FILE"
```

### Desktop Notifications (Stop)

```bash
#!/bin/bash
osascript -e 'display notification "Claude finished work" with title "Claude Code"'
```

---

## Security Best Practices

### Input Validation

| Requirement | Details |
|-------------|---------|
| MUST | Validate all hook input data |
| MUST | Check for path traversal (`..` sequences) |
| MUST | Use absolute paths |
| SHOULD | Implement allowlists for sensitive operations |

### Shell Script Safety

| Requirement | Details |
|-------------|---------|
| MUST | Quote all variables: `"$VAR"` |
| SHOULD | Use `set -e` (exit on error) |
| SHOULD | Use `set -u` (error on undefined) |
| MUST NOT | Trust user-provided paths without validation |

### Credential Protection

| Requirement | Details |
|-------------|---------|
| MUST NOT | Log environment variables with credentials |
| MUST NOT | Process `.env`, `.git/`, or credential stores |
| SHOULD | Review what files hooks can access |

---

## Performance Considerations

| Requirement | Details |
|-------------|---------|
| SHOULD | Target < 1 second execution time |
| SHOULD | Cache expensive operations |
| MUST NOT | Make blocking network calls without timeout |
| MAY | Leverage parallel execution for independent checks |

---

## Examples

### Pre-commit Validation Hook

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/validate-commit.sh"
          }
        ]
      }
    ]
  }
}
```

**validate-commit.sh:**
```bash
#!/bin/bash
set -eu

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

# Block force pushes
if [[ "$COMMAND" =~ "git push".*"--force" ]]; then
  jq -n '{
    "continue": false,
    "stopReason": "Force push blocked by policy"
  }'
  exit 2
fi

# Allow other commands
echo '{"continue": true}'
```

### LLM-Based Permission Hook

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "prompt",
            "command": "Evaluate if this file write should be allowed. Context: $ARGUMENTS. Respond with allow or deny."
          }
        ]
      }
    ]
  }
}
```

---

## Anti-Patterns

### Don't: Trust Input Without Validation

```bash
# Bad: no validation
rm -rf $PATH_FROM_INPUT
```

```bash
# Good: validate and quote
if [[ ! "$PATH_FROM_INPUT" =~ ^/allowed/path ]]; then
  exit 2
fi
rm -rf "$PATH_FROM_INPUT"
```

### Don't: Depend on Execution Order

```json
// Bad: assumes hooks run in sequence
{
  "hooks": [
    { "command": "setup.sh" },
    { "command": "needs-setup.sh" }
  ]
}
```

```json
// Good: self-contained hooks
{
  "hooks": [
    { "command": "standalone-check.sh" }
  ]
}
```

### Don't: Block on Network Calls

```bash
# Bad: may hang indefinitely
curl https://external-api.com/validate
```

```bash
# Good: timeout protection
curl --max-time 5 https://external-api.com/validate || true
```

### Don't: Log Sensitive Data

```bash
# Bad: credentials in logs
echo "Processing with API_KEY=$API_KEY"
```

```bash
# Good: sanitized logging
echo "Processing request (credentials redacted)"
```

---

## Debugging

| Tool | Purpose |
|------|---------|
| `/hooks` | View hook configuration |
| `claude --debug` | Launch with debug output for hook execution details |
| Verbose mode (Ctrl+O) | Review hook output, exit codes, and matched matchers |
| `transcript_path` | Check conversation context |
| Manual testing | Test scripts independently with stdin JSON |

### Debug Output

When running with `--debug`, you'll see:
- Hook matcher evaluation
- Command execution status
- stdout/stderr content
- Exit codes and timing

---

## Related Documents

- [02-commands.md](./02-commands.md) - Slash commands
- [03-agents.md](./03-agents.md) - Subagents
- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [08-permissions.md](./08-permissions.md) - Permission model

---

**Navigation:** [← Previous: Skills](./04-skills.md) | [Index](./README.md) | [Next: MCP Servers →](./06-mcp-servers.md)
