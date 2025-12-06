# Troubleshooting

> Common issues and solutions for foundry-mcp.

---

## Connection Issues

### Server Not Starting

**Symptoms:**
- `foundry-mcp` command not found
- Server exits immediately
- No output when starting

**Solutions:**

1. **Check installation:**
   ```bash
   pip show foundry-mcp
   # or
   uvx foundry-mcp --version
   ```

2. **Check Python version:**
   ```bash
   python --version  # Must be 3.10+
   ```

3. **Run directly:**
   ```bash
   python -m foundry_mcp.server
   ```

4. **Check for errors:**
   ```bash
   FOUNDRY_MCP_LOG_LEVEL=DEBUG foundry-mcp
   ```

### Claude Code Not Discovering Server

**Symptoms:**
- Server not shown in `/mcp`
- Tools not available
- MCP connection errors

**Solutions:**

1. **Check MCP configuration:**
   ```json
   {
     "mcpServers": {
       "foundry-mcp": {
         "command": "uvx",
         "args": ["foundry-mcp"]
       }
     }
   }
   ```

2. **Restart Claude Code:**
   - Close and reopen VS Code
   - Or run "Developer: Reload Window"

3. **Verify server runs standalone:**
   ```bash
   uvx foundry-mcp
   # Should start without errors
   ```

4. **Check logs:**
   - View Output panel in VS Code
   - Select "MCP" from dropdown

### Transport Errors

**Symptoms:**
- "Connection refused"
- "Transport error"
- Intermittent disconnections

**Solutions:**

1. **Check port conflicts:**
   ```bash
   lsof -i :3000  # Check if port in use
   ```

2. **Verify stdio transport:**
   - MCP uses stdio by default
   - Ensure no output pollution

3. **Check environment:**
   ```bash
   env | grep FOUNDRY
   ```

---

## Spec Issues

### Spec Not Found

**Error:**
```json
{
  "success": false,
  "data": {
    "error_code": "NOT_FOUND",
    "remediation": "Check spec_id exists in specs/ directory"
  }
}
```

**Solutions:**

1. **Verify spec exists:**
   ```bash
   ls specs/active/
   ls specs/pending/
   ```

2. **Check spec_id:**
   - Must match filename (without `.json`)
   - Case-sensitive

3. **Check FOUNDRY_MCP_SPECS_DIR:**
   ```bash
   echo $FOUNDRY_MCP_SPECS_DIR
   ```

### Validation Failures

**Error:**
```json
{
  "data": {
    "valid": false,
    "issues": [
      {"field": "hierarchy.phases[0].tasks", "error": "required"}
    ]
  }
}
```

**Solutions:**

1. **Check required fields:**
   - `spec_id`
   - `title`
   - `hierarchy` with at least one phase

2. **Use auto-fix:**
   ```
   mcp__foundry-mcp__spec-fix with spec_id="my-spec"
   ```

3. **Get schema:**
   ```
   mcp__foundry-mcp__spec-schema
   ```

### Hierarchy Integrity Errors

**Error:**
```json
{
  "data": {
    "error_code": "INTEGRITY_ERROR",
    "details": {
      "issue": "Circular dependency detected",
      "tasks": ["task-001", "task-002"]
    }
  }
}
```

**Solutions:**

1. **Check dependencies:**
   - Tasks cannot depend on themselves
   - No circular chains (A → B → A)

2. **Verify task IDs:**
   - Must be unique within spec
   - References must exist

3. **Review dependency graph:**
   ```
   mcp__foundry-mcp__task-check-deps with spec_id="my-spec" task_id="task-001"
   ```

---

## Task Issues

### All Tasks Blocked

**Symptoms:**
- `task-next` returns no task
- All tasks show as blocked

**Solutions:**

1. **List blocked tasks:**
   ```
   mcp__foundry-mcp__task-list-blocked with spec_id="my-spec"
   ```

2. **Check dependencies:**
   - Ensure dependent tasks are completed
   - Resolve blockers

3. **Unblock manually:**
   ```
   mcp__foundry-mcp__task-unblock with spec_id="my-spec" task_id="task-001"
   ```

### Task Count Mismatches

**Symptoms:**
- Progress shows wrong numbers
- Completed tasks not counted

**Solutions:**

1. **Validate spec:**
   ```
   mcp__foundry-mcp__spec-validate with spec_id="my-spec"
   ```

2. **Reconcile state:**
   ```
   mcp__foundry-mcp__spec-reconcile-state with spec_id="my-spec"
   ```

3. **Check status values:**
   - Must be: `pending`, `in_progress`, `completed`, `blocked`, `skipped`

### Journal Not Recording

**Solutions:**

1. **Check journal enabled:**
   ```toml
   [workflow]
   journal_enabled = true
   ```

2. **Verify spec is writable:**
   ```bash
   ls -la specs/active/my-spec.json
   ```

3. **Manual journal add:**
   ```
   mcp__foundry-mcp__journal-add with
     spec_id="my-spec"
     entry={"entry_type": "note", "content": "Test entry"}
   ```

---

## Testing Issues

### pytest Not Found

**Error:**
```json
{
  "error_code": "PYTEST_NOT_FOUND",
  "remediation": "Install pytest: pip install pytest"
}
```

**Solutions:**

1. **Install pytest:**
   ```bash
   pip install pytest
   ```

2. **Check in virtual environment:**
   ```bash
   which pytest
   pip show pytest
   ```

3. **Verify in MCP environment:**
   - Check `env` in MCP config includes correct PATH

### Fixture Errors

**Error:**
```
fixture 'db_session' not found
```

**Solutions:**

1. **Check conftest.py location:**
   - Must be in `tests/` or test directory
   - Must define the fixture

2. **Verify fixture scope:**
   ```python
   @pytest.fixture(scope="function")  # or session, module
   def db_session():
       ...
   ```

3. **Check imports:**
   ```python
   from conftest import db_session  # Not needed if in same directory
   ```

### Timeout Issues

**Solutions:**

1. **Mark slow tests:**
   ```python
   @pytest.mark.slow
   def test_large_operation():
       ...
   ```

2. **Use quick preset:**
   ```
   mcp__foundry-mcp__test-run with preset="quick"
   ```

3. **Increase timeout:**
   ```toml
   [testing]
   timeout = 300  # 5 minutes
   ```

---

## LLM Integration Issues

### Provider Authentication

**Error:**
```json
{
  "error_code": "PROVIDER_AUTH_FAILED",
  "remediation": "Check API key configuration"
}
```

**Solutions:**

1. **Verify API key:**
   ```bash
   echo $ANTHROPIC_API_KEY  # Should not be empty
   ```

2. **Check key format:**
   - Anthropic: `sk-ant-...`
   - OpenAI: `sk-...`
   - Gemini: `AIza...`

3. **Test directly:**
   ```bash
   curl -H "x-api-key: $ANTHROPIC_API_KEY" \
        https://api.anthropic.com/v1/models
   ```

### Timeout Errors

**Error:**
```json
{
  "error_code": "LLM_TIMEOUT",
  "remediation": "Request timed out, consider using faster model"
}
```

**Solutions:**

1. **Increase timeout:**
   ```toml
   [llm]
   timeout = 60
   ```

2. **Use faster model:**
   ```toml
   [llm]
   model = "claude-3-haiku"  # Faster than Opus
   ```

3. **Check network:**
   ```bash
   ping api.anthropic.com
   ```

### Rate Limiting

**Error:**
```json
{
  "error_code": "RATE_LIMIT_EXCEEDED",
  "meta": {
    "rate_limit": {
      "remaining": 0,
      "reset_at": "2025-12-04T12:05:00Z"
    }
  }
}
```

**Solutions:**

1. **Wait for reset:**
   - Check `reset_at` timestamp
   - Implement exponential backoff

2. **Use fallback provider:**
   ```toml
   [llm]
   provider = "anthropic"
   fallback_provider = "openai"
   ```

3. **Reduce request frequency:**
   - Cache results where possible
   - Batch operations

---

## Performance Issues

### Slow Responses

**Solutions:**

1. **Enable caching:**
   ```toml
   [llm.cache]
   enabled = true
   ```

2. **Use appropriate model:**
   - Simple tasks: Haiku
   - Complex tasks: Sonnet

3. **Check logging level:**
   ```bash
   export FOUNDRY_MCP_LOG_LEVEL=WARNING  # Reduce debug output
   ```

### Context Limits

**Symptoms:**
- Autonomous mode stopping early
- "Context limit reached" warnings

**Solutions:**

1. **Check context usage:**
   ```
   mcp__foundry-mcp__session-context
   ```

2. **Adjust threshold:**
   ```toml
   [workflow]
   context_limit_percent = 90  # Increase from default 85
   ```

3. **Break into smaller sessions:**
   - Complete fewer tasks per session
   - Use progressive disclosure

### Large Spec Handling

**Solutions:**

1. **Use pagination:**
   - Large task lists return paginated
   - Follow cursor to get more

2. **Query specific phases:**
   ```
   mcp__foundry-mcp__task-query with spec_id="my-spec" phase_id="phase-1"
   ```

3. **Use summary views:**
   ```
   mcp__foundry-mcp__spec-render-progress with spec_id="my-spec"
   ```

---

## Error Code Reference

### Quick Lookup

| Error Code | Category | Typical Cause |
|------------|----------|---------------|
| `VALIDATION_ERROR` | validation | Invalid input |
| `NOT_FOUND` | not_found | Resource missing |
| `ALREADY_EXISTS` | conflict | Duplicate resource |
| `INVALID_STATE` | conflict | Wrong lifecycle state |
| `DEPENDENCY_ERROR` | validation | Unmet dependencies |
| `RATE_LIMIT_EXCEEDED` | rate_limit | Too many requests |
| `FEATURE_DISABLED` | feature_flag | Feature not enabled |
| `PROVIDER_AUTH_FAILED` | authentication | Bad API key |
| `PROVIDER_UNAVAILABLE` | unavailable | Provider down |
| `LLM_TIMEOUT` | timeout | Request too slow |
| `INTERNAL_ERROR` | internal | Server error |

---

## Debug Mode

### Enabling Debug Logging

```bash
export FOUNDRY_MCP_LOG_LEVEL=DEBUG
foundry-mcp
```

### Checking Configuration

```
mcp__foundry-mcp__env-verify-environment
```

**Shows:**
- Active configuration
- Missing requirements
- Warnings

### Verifying Installation

```
mcp__foundry-mcp__sdd-verify-toolchain
```

**Checks:**
- Python version
- Required packages
- Git availability
- pytest installation

---

## Getting Help

### Collecting Diagnostic Info

```bash
# Version info
foundry-mcp --version
python --version
pip show foundry-mcp

# Configuration
cat foundry-mcp.toml
echo $FOUNDRY_MCP_SPECS_DIR

# Logs
FOUNDRY_MCP_LOG_LEVEL=DEBUG foundry-mcp 2>&1 | head -100
```

### Reporting Issues

When reporting issues, include:

1. **Error message** — Full error output
2. **Command/tool** — What you were trying to do
3. **Configuration** — Relevant settings
4. **Version info** — foundry-mcp and Python versions
5. **Reproduction steps** — How to trigger the issue

### Resources

- **GitHub Issues:** [foundry-mcp/issues](https://github.com/tylerburleigh/foundry-mcp/issues)
- **Documentation:** This guide and related docs
- **Logs:** Enable DEBUG logging for detailed info

---

## Related Documentation

- **[07-Configuration](./07-configuration.md)** — Setup issues
- **[09-LLM Providers](./09-llm-providers.md)** — Provider issues
- **[00-Quickstart](./00-quickstart.md)** — Installation verification

---

*[Back to Index](./README.md)*
