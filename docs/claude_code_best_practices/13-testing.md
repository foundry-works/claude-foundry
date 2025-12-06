# 13. Plugin Testing

> Strategies for validating plugin components during development.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Plugin testing ensures your components work correctly before distribution:

- **Structure validation** - Verify plugin.json and directory layout
- **Component validation** - Test commands, agents, skills, hooks individually
- **Integration testing** - Test complete plugin workflows
- **Regression testing** - Maintain test cases for ongoing development

---

## Structure Validation

### Plugin Directory Validation

Verify your plugin structure is correct:

```bash
# Check plugin.json is valid JSON
cat .claude-plugin/plugin.json | jq .

# Verify required directory exists
ls -la .claude-plugin/plugin.json

# Check component directories
ls -la commands/
ls -la agents/
ls -la skills/
```

### Plugin.json Validation

```bash
# Validate required fields
jq '.name' .claude-plugin/plugin.json

# Check for common issues
jq 'select(.name | test("^[a-z][a-z0-9-]*$") | not)' .claude-plugin/plugin.json
```

| Validation | Check |
|------------|-------|
| Name format | Lowercase, hyphens only |
| Version format | Semantic version (X.Y.Z) |
| Paths | Start with `./`, relative |
| Author | Object with `name` field |

---

## Frontmatter Validation

### Common Frontmatter Issues

| Issue | Example | Fix |
|-------|---------|-----|
| Missing delimiter | No `---` at start | Add opening `---` |
| Unclosed delimiter | Only one `---` | Add closing `---` |
| Invalid YAML | `name: my plugin` (space) | Use quotes or hyphens |
| Missing required field | No `description` | Add description field |

### Validating YAML Frontmatter

```bash
# Extract and validate frontmatter
head -20 commands/my-command.md

# Python validation
python3 -c "
import yaml
content = open('commands/my-command.md').read()
parts = content.split('---')
if len(parts) >= 3:
    yaml.safe_load(parts[1])
    print('Valid frontmatter')
"
```

### Field Validation by Type

**Commands:**
```yaml
---
name: my-command          # MUST: lowercase, hyphens
description: What it does # MUST: clear purpose
allowed-tools: Read, Grep # MAY: tool restrictions
---
```

**Agents:**
```yaml
---
name: my-agent            # MUST: lowercase, hyphens
description: When to use  # MUST: trigger context
tools: Read, Grep         # SHOULD: minimal tools
model: sonnet             # MAY: model override
---
```

**Skills:**
```yaml
---
name: my-skill            # MUST: lowercase, hyphens, max 64 chars
description: What + when  # MUST: max 1024 chars, third person
allowed-tools: Read       # MAY: tool restrictions
---
```

---

## Component Testing

### Testing Commands

**Validation checklist:**
- [ ] Command appears in `/` menu
- [ ] Description is clear and accurate
- [ ] Arguments parsed correctly (`$ARGUMENTS`, `$1`, `$2`)
- [ ] Tool restrictions enforced (`allowed-tools`)
- [ ] File references work (`@path/to/file`)
- [ ] Bash pre-execution works (if using `!` prefix)

**Manual test procedure:**
1. Load plugin in Claude Code
2. Type `/` and verify command appears
3. Invoke command with test arguments
4. Verify output matches expectations
5. Test edge cases (no args, invalid args)

### Testing Agents

**Validation checklist:**
- [ ] Agent appears in `/agents` list
- [ ] Description triggers correct automatic selection
- [ ] Tool restrictions enforced
- [ ] System prompt produces expected behavior
- [ ] Output format matches specification
- [ ] Skills auto-load correctly (if specified)

**Manual test procedure:**
1. Run `/agents` to verify registration
2. Request task matching agent description
3. Verify Claude delegates to agent
4. Check agent uses only allowed tools
5. Verify output format

### Testing Skills

**Validation checklist:**
- [ ] Skill directory has SKILL.md
- [ ] Activation triggers match description
- [ ] Progressive disclosure works (reference.md, examples.md load when needed)
- [ ] Tool restrictions enforced
- [ ] No conflicts with other skills

**Manual test procedure:**
1. Verify skill directory structure
2. Perform task that should trigger skill
3. Check if skill activates (verbose mode helps)
4. Verify supporting files load appropriately
5. Test with conflicting task descriptions

### Testing Hooks

**Validation checklist:**
- [ ] hooks.json is valid JSON
- [ ] Matcher patterns match intended tools
- [ ] Exit codes produce expected behavior
- [ ] JSON output parsed correctly
- [ ] Scripts have execute permissions
- [ ] Timeout handling works

**Testing hook scripts independently:**
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

# Check exit code
echo "Exit code: $?"
# 0 = success, 2 = block
```

### Testing MCP Servers

**Validation checklist:**
- [ ] Server starts without errors
- [ ] Tools are exposed correctly
- [ ] Authentication works (if configured)
- [ ] Environment variables resolve
- [ ] Graceful error handling

**Testing server directly:**
```bash
# Test STDIO server initialization
echo '{"jsonrpc":"2.0","method":"initialize","id":1}' | \
  node ./servers/my-server.js

# Check server logs
cat /tmp/mcp-server.log
```

---

## Integration Testing

### End-to-End Test Scenarios

Document complete workflows that exercise multiple components:

```markdown
## Test Scenario: Code Review Flow

**Components involved:**
- /plugin:review command
- code-reviewer agent
- security-scan skill
- format-output hook

**Steps:**
1. User invokes /plugin:review src/main.py
2. Command delegates to code-reviewer agent
3. Agent analyzes code, security-scan skill activates
4. PostToolUse hook formats output
5. Results returned to user

**Expected result:**
Formatted review with security findings in markdown table

**Pass criteria:**
- [ ] All components activate
- [ ] Output format correct
- [ ] No errors in verbose mode
```

### Test Cases Document

Maintain a test cases document:

```markdown
# Plugin Test Cases

## TC-001: Basic command invocation
- **Input:** /my-plugin:hello
- **Expected:** "Hello, world!" response
- **Status:** [ ] Pass / [ ] Fail

## TC-002: Agent delegation
- **Input:** "Use my-agent to analyze main.py"
- **Expected:** Agent executes with Read, Grep tools only
- **Status:** [ ] Pass / [ ] Fail

## TC-003: Skill activation
- **Input:** "Review this Python file for security issues"
- **Expected:** security-review skill activates
- **Status:** [ ] Pass / [ ] Fail

## TC-004: Hook blocking
- **Input:** Attempt to write to .env file
- **Expected:** PreToolUse hook blocks with message
- **Status:** [ ] Pass / [ ] Fail
```

---

## Test Fixtures

### Fixture Directory Structure

```
tests/
├── fixtures/
│   ├── sample-code.py       # Code for review tests
│   ├── sample-data.json     # Data for processing tests
│   └── sample-config.yaml   # Config for command tests
├── expected/
│   ├── review-output.md     # Expected review format
│   └── processed-data.json  # Expected processing result
├── hooks/
│   ├── pretooluse-input.json
│   └── posttooluse-input.json
└── test-cases.md            # Test documentation
```

### Hook Test Fixtures

Create standard test inputs for hooks:

```json
// tests/hooks/pretooluse-bash.json
{
  "session_id": "test-session",
  "transcript_path": "/tmp/transcript.jsonl",
  "cwd": "/project",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  }
}
```

---

## Validation Script

Create a validation script for your plugin:

```bash
#!/bin/bash
# validate-plugin.sh

set -e
PLUGIN_DIR="${1:-.}"

echo "Validating plugin in $PLUGIN_DIR"

# 1. Check plugin.json exists and is valid
echo "1. Checking plugin.json..."
jq empty "$PLUGIN_DIR/.claude-plugin/plugin.json" || {
  echo "ERROR: Invalid plugin.json"
  exit 1
}

# 2. Validate plugin name format
NAME=$(jq -r '.name' "$PLUGIN_DIR/.claude-plugin/plugin.json")
if [[ ! "$NAME" =~ ^[a-z][a-z0-9-]*$ ]]; then
  echo "ERROR: Invalid plugin name: $NAME"
  exit 1
fi

# 3. Check frontmatter in commands
echo "2. Checking command frontmatter..."
for f in "$PLUGIN_DIR"/commands/*.md 2>/dev/null; do
  [ -f "$f" ] || continue
  if ! head -1 "$f" | grep -q "^---$"; then
    echo "ERROR: Missing frontmatter in $f"
    exit 1
  fi
done

# 4. Check frontmatter in agents
echo "3. Checking agent frontmatter..."
for f in "$PLUGIN_DIR"/agents/*.md 2>/dev/null; do
  [ -f "$f" ] || continue
  if ! head -1 "$f" | grep -q "^---$"; then
    echo "ERROR: Missing frontmatter in $f"
    exit 1
  fi
done

# 5. Check skill directories have SKILL.md
echo "4. Checking skill structure..."
for d in "$PLUGIN_DIR"/skills/*/ 2>/dev/null; do
  [ -d "$d" ] || continue
  if [ ! -f "$d/SKILL.md" ]; then
    echo "ERROR: Missing SKILL.md in $d"
    exit 1
  fi
done

# 6. Validate hooks.json if present
echo "5. Checking hooks.json..."
if [ -f "$PLUGIN_DIR/hooks/hooks.json" ]; then
  jq empty "$PLUGIN_DIR/hooks/hooks.json" || {
    echo "ERROR: Invalid hooks.json"
    exit 1
  }
fi

# 7. Check hook scripts are executable
echo "6. Checking hook scripts..."
for script in "$PLUGIN_DIR"/hooks/scripts/*.sh 2>/dev/null; do
  [ -f "$script" ] || continue
  if [ ! -x "$script" ]; then
    echo "WARNING: $script is not executable"
  fi
  # Syntax check
  bash -n "$script" || {
    echo "ERROR: Syntax error in $script"
    exit 1
  }
done

echo "✓ All validations passed"
```

---

## Common Testing Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Command not appearing | Invalid frontmatter | Validate YAML syntax |
| Command not appearing | Wrong file location | Check commands/ directory |
| Agent not triggering | Vague description | Add specific trigger contexts |
| Agent wrong tools | Typo in tools field | Check exact tool names |
| Skill not loading | Wrong directory structure | SKILL.md must be in subdirectory |
| Skill not activating | Description too vague | Add specific trigger keywords |
| Hook not executing | Matcher not matching | Test regex pattern separately |
| Hook blocking unexpectedly | Wrong exit code | Use exit 0 for allow, exit 2 for block |
| MCP server failing | Missing dependencies | Check server requirements |

---

## Best Practices

| Requirement | Details |
|-------------|---------|
| MUST | Validate plugin.json before distribution |
| MUST | Test all components manually |
| SHOULD | Use verbose mode during development |
| SHOULD | Maintain test cases document |
| SHOULD | Create validation script |
| SHOULD | Test hook scripts independently |
| MAY | Create integration test scenarios |

---

## Related Documents

- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [09-file-structure.md](./09-file-structure.md) - Directory layout
- [10-frontmatter.md](./10-frontmatter.md) - Frontmatter schemas
- [15-debugging.md](./15-debugging.md) - Debug tools and troubleshooting

---

**Navigation:** [← Previous: CLAUDE.md](./12-claude-md.md) | [Index](./README.md) | [Next: Distribution →](./14-distribution.md)
