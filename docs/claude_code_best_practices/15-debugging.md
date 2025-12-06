# 15. Debugging

> Debug flags, troubleshooting techniques, and common issue resolution for Claude Code plugins.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Effective debugging requires understanding:

- **Debug flags** - CLI options for verbose output
- **In-session tools** - Interactive debugging commands
- **Component-specific debugging** - Hooks, skills, agents, MCP
- **Common issues** - Patterns and solutions

---

## Debug Flags

### CLI Debug Options

| Flag | Purpose |
|------|---------|
| `--debug` | Enable full debug output |
| `--verbose` | Show detailed operation logs |
| `--mcp-debug` | Debug MCP server connections |

### Usage Examples

```bash
# Full debug mode
claude --debug

# Verbose output
claude --verbose

# Debug MCP connections
claude --mcp-debug

# Combine flags
claude --debug --mcp-debug --verbose
```

### Debug Output

With `--debug` enabled, you'll see:

```
[DEBUG] Loading plugins from ~/.claude/plugins/
[DEBUG] Found plugin: my-plugin v1.0.0
[DEBUG] Loading commands: 3 found
[DEBUG] Loading agents: 2 found
[DEBUG] Loading skills: 4 found
[DEBUG] Initializing MCP server: plugin-api
[DEBUG] Hook registered: PreToolUse -> validate.sh
```

---

## In-Session Debugging

### Verbose Mode

Toggle verbose mode during a session:

```
Press Ctrl+O to toggle verbose mode
```

Verbose mode shows:
- Hook execution and exit codes
- Tool invocations and results
- Skill activation events
- Context size and token usage

### Useful Commands

| Command | Purpose |
|---------|---------|
| `/plugins` | List installed plugins |
| `/agents` | List available agents |
| `/skills` | List available skills |
| `/hooks` | View hook configuration |
| `/mcp` | View MCP server status |
| `/context` | Show context usage |

### Viewing Component Details

```
# Check if plugin loaded correctly
/plugins

# Verify agent is available
/agents

# Check skill registration
What skills are available?

# View MCP server status
/mcp
```

---

## Hook Debugging

### Debug Hook Execution

1. **Enable verbose mode** (Ctrl+O)
2. **Trigger hook event** (use matching tool)
3. **Observe output:**
   ```
   [HOOK] PreToolUse matching "Bash"
   [HOOK] Executing: /path/to/validate.sh
   [HOOK] Exit code: 0
   [HOOK] Output: {"continue": true}
   ```

### Testing Hooks Independently

```bash
# Create test input
cat > /tmp/hook-test.json << 'EOF'
{
  "session_id": "test-123",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
EOF

# Run hook script
cat /tmp/hook-test.json | ./hooks/scripts/validate.sh
echo "Exit code: $?"
```

### Common Hook Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Hook not executing | Matcher not matching | Check regex pattern |
| Exit code 1 but not blocking | Wrong exit code | Use exit code 2 to block |
| JSON not parsed | Invalid JSON output | Validate JSON structure |
| Script not found | Wrong path | Use `${CLAUDE_PLUGIN_ROOT}` |
| Permission denied | Not executable | `chmod +x script.sh` |

### Hook Debugging Script

```bash
#!/bin/bash
# debug-hook.sh - Wrapper for debugging hooks

# Log input
cat > /tmp/hook-input.json

# Log execution
echo "[$(date)] Hook executed" >> /tmp/hook-debug.log
echo "Input:" >> /tmp/hook-debug.log
cat /tmp/hook-input.json >> /tmp/hook-debug.log

# Run actual hook
cat /tmp/hook-input.json | ./actual-hook.sh
EXIT_CODE=$?

echo "Exit code: $EXIT_CODE" >> /tmp/hook-debug.log
exit $EXIT_CODE
```

---

## Skill Debugging

### Verifying Skill Registration

```bash
# Check skill directory structure
ls -la .claude/skills/my-skill/

# Verify SKILL.md exists
cat .claude/skills/my-skill/SKILL.md

# Validate frontmatter
head -10 .claude/skills/my-skill/SKILL.md
```

### Common Skill Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Skill not appearing | Wrong directory | Check skill location |
| Skill not activating | Vague description | Add specific triggers |
| Wrong skill activating | Overlapping descriptions | Differentiate keywords |
| Supporting files not loading | Wrong reference cues | Check file paths |
| YAML error | Invalid frontmatter | Validate YAML syntax |

### Debugging Activation

Add explicit activation cues to your skill:

```yaml
---
name: my-skill
description: Process Excel files. Use when working with .xlsx, .xls, spreadsheets, or Excel documents.
---

# Debug Activation
If this skill activated, you should see this message.
The trigger was likely: Excel, .xlsx, spreadsheet, or related terms.
```

---

## Agent Debugging

### Verifying Agent Registration

```bash
# Check agent files
ls -la .claude/agents/

# Verify frontmatter
head -20 .claude/agents/my-agent.md
```

### Common Agent Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Agent not appearing | Invalid frontmatter | Check YAML syntax |
| Agent not triggering | Vague description | Add trigger context |
| Wrong tools available | Tool restriction issue | Check `tools` field |
| Model not as expected | `model` field wrong | Verify model value |
| Permission errors | Wrong permissionMode | Check permission setting |

### Testing Agent Invocation

```
# Explicit invocation
Use the my-agent agent to analyze this file

# Check agent received correct context
[In agent response, verify it has expected information]
```

---

## MCP Server Debugging

### Debug MCP Connections

```bash
# Launch with MCP debug
claude --mcp-debug

# Output shows:
# [MCP] Connecting to server: plugin-api
# [MCP] Transport: stdio
# [MCP] Command: node /path/to/server.js
# [MCP] Connected successfully
# [MCP] Available tools: 3
```

### Common MCP Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Server not starting | Command not found | Check server path |
| Connection timeout | Server crash | Check server logs |
| Tools not appearing | Server error | Debug server directly |
| Authentication failed | Wrong credentials | Check env variables |

### Testing MCP Server Directly

```bash
# Test STDIO server
echo '{"jsonrpc":"2.0","method":"initialize","id":1}' | \
  node ./servers/my-server.js

# Check server logs
tail -f /tmp/mcp-server.log
```

### MCP Server Debug Logging

Add debug logging to your server:

```javascript
// In your MCP server
const DEBUG = process.env.DEBUG === 'true';

function log(...args) {
  if (DEBUG) {
    console.error('[MCP Server]', ...args);
  }
}

log('Server starting');
log('Received request:', request);
```

---

## Command Debugging

### Verifying Command Registration

```bash
# List commands
/

# Check command file
cat commands/my-command.md
```

### Common Command Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Command not appearing | Invalid frontmatter | Check YAML |
| Arguments not working | Wrong placeholder | Use $ARGUMENTS or $1 |
| Tool restrictions ignored | Invalid tool names | Check exact tool names |
| File references failing | Wrong @ syntax | Check file paths |

---

## Log Analysis

### Finding Claude Code Logs

```bash
# macOS
ls ~/Library/Application\ Support/Claude/logs/

# Linux
ls ~/.config/claude/logs/

# View recent logs
tail -100 ~/Library/Application\ Support/Claude/logs/claude.log
```

### Filtering Logs

```bash
# Filter for errors
grep -i error claude.log

# Filter for specific plugin
grep "my-plugin" claude.log

# Filter for hooks
grep "\[HOOK\]" claude.log

# Filter for MCP
grep "\[MCP\]" claude.log
```

---

## Debugging Workflow

### Systematic Debug Process

1. **Identify the issue type**
   - Plugin not loading?
   - Component not working?
   - Unexpected behavior?

2. **Enable appropriate debugging**
   ```bash
   claude --debug --verbose
   ```

3. **Reproduce the issue**
   - Document exact steps
   - Note any error messages

4. **Check logs**
   - Review debug output
   - Check application logs

5. **Isolate the problem**
   - Test component independently
   - Remove other plugins

6. **Verify fix**
   - Restart Claude Code
   - Test full workflow

### Quick Debug Checklist

| Check | Command |
|-------|---------|
| Plugin loaded? | `/plugins` |
| Agent available? | `/agents` |
| Skill registered? | Ask about skills |
| Hook configured? | `/hooks` |
| MCP connected? | `/mcp` |
| Context usage? | `/context` |

---

## Performance Debugging

### Context Usage

```
# Check context consumption
/context

# Output:
# Context: 45,000 / 200,000 tokens (22.5%)
# Skills loaded: 3
# MCP tools: 12
```

### Reducing Token Usage

| Issue | Solution |
|-------|----------|
| Skills consuming tokens | Keep SKILL.md under 500 lines |
| Agent output too large | Add output constraints to prompt |
| Too many MCP tools | Disable unused servers |
| Command templates large | Use file references |

---

## Getting Help

### Reporting Issues

When reporting bugs:

1. **Environment info**
   ```bash
   claude --version
   uname -a
   ```

2. **Debug output**
   ```bash
   claude --debug 2>&1 | tee debug.log
   ```

3. **Minimal reproduction**
   - Simplest steps to reproduce
   - Minimal plugin configuration

4. **Expected vs actual**
   - What you expected
   - What actually happened

---

## Related Documents

- [05-hooks.md](./05-hooks.md) - Hook configuration
- [06-mcp-servers.md](./06-mcp-servers.md) - MCP server setup
- [13-testing.md](./13-testing.md) - Testing plugins

---

**Navigation:** [← Previous: Distribution](./14-distribution.md) | [Index](./README.md) | [Next: Description Writing →](./16-description-writing.md)
