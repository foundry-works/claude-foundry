# Troubleshooting

Solutions to common issues with Claude Foundry.

## Quick Diagnostics

Run this first to check your setup:

```
Use foundry-setup --check
```

This verifies:
- foundry-mcp is installed
- Python is available
- Git repository exists
- Permissions are configured
- Workspace directories exist

---

## Installation Issues

### "foundry-mcp not found" or "MCP tool not found"

**Symptoms:**
- Commands fail with MCP-related errors
- `foundry-setup --check` shows foundry-mcp missing

**Solutions:**

1. **Install foundry-mcp:**
   ```bash
   pip install foundry-mcp
   ```

2. **Verify installation:**
   ```bash
   python -m foundry_mcp.server --help
   ```

3. **Check Python path:**
   Make sure the Python that has foundry-mcp is the one Claude Code uses:
   ```bash
   which python
   python -c "import foundry_mcp; print('OK')"
   ```

4. **Restart Claude Code:**
   After installing, restart Claude Code for the MCP server to register.

### "Permission denied" errors

**Symptoms:**
- Tools fail with permission errors
- Can't write to specs directory

**Solutions:**

1. **Re-run setup:**
   ```
   Use foundry-setup to reconfigure permissions
   ```
   This will reconfigure permissions if needed.

2. **Check permissions file exists:**
   ```bash
   cat .claude/settings.local.json
   ```

3. **Verify required permissions:**
   The file should match `templates/settings.local.json.example`. If entries are missing, copy that template or use `foundry-setup` to reconfigure.

---

## Spec Issues

### "No active specs found"

**Symptoms:**
- `foundry-implement` says no specs found
- Can't find your spec

**Solutions:**

1. **Check spec locations:**
   ```bash
   ls specs/pending/
   ls specs/active/
   ```

2. **If spec is in pending:**
   Activate it:
   ```
   Activate spec [spec-id]
   ```

3. **If no specs exist:**
   Create one with `foundry-spec`:
   ```
   I want to build [describe feature]
   ```

4. **List all specs:**
   Ask Claude:
   ```
   List all specs in the project.
   ```

### "Spec validation failed"

**Symptoms:**
- Can't activate spec
- Errors about missing fields

**Solutions:**

1. **Run validation with fix:**
   ```
   Validate and fix the spec.
   ```

2. **Common issues:**
   - Missing `mission` field in metadata
   - Tasks without `file_path` (for implementation tasks)
   - Circular dependencies
   - Missing acceptance criteria

3. **Manual inspection:**
   Ask Claude to show the validation errors:
   ```
   What validation errors does the spec have?
   ```

### Task stuck in wrong state

**Symptoms:**
- Task shows "in_progress" but work is done
- Task shows "completed" but it's not

**Solutions:**

1. **Manually update status:**
   ```
   Mark task [task-id] as completed.
   Mark task [task-id] as pending.
   ```

2. **If blocked incorrectly:**
   ```
   Unblock task [task-id].
   ```

3. **View task details:**
   ```
   Show me the details of task [task-id].
   ```

---

## Implementation Issues

### Context limit warnings

**Symptoms:**
- Warning about context approaching limit
- Claude suggests clearing context

**What it means:**
Claude Code has limited context window. Long sessions can approach this limit.

**Solutions:**

1. **Clear and continue:**
   ```
   /clear
   ```
   Then:
   ```
   Use foundry-implement
   ```
   Claude will reload the spec and continue.

2. **Use delegated mode:**
   Each subagent gets fresh context:
   ```
   Use foundry-implement --delegate
   ```

3. **Break into smaller sessions:**
   Complete one phase, then start new session for next phase.

### "Dependencies not met" but they are

**Symptoms:**
- Task says it depends on incomplete task
- But you know that task is done

**Solutions:**

1. **Check the dependency:**
   ```
   Show me the status of task [dependency-id].
   ```

2. **Mark dependency complete:**
   ```
   Mark task [dependency-id] as completed.
   ```

3. **Remove incorrect dependency:**
   ```
   Remove the dependency of [task-id] on [dependency-id].
   ```

### Implementation doesn't match acceptance criteria

**Symptoms:**
- Task marked complete but criteria not met
- Review shows major deviations

**Solutions:**

1. **Reopen the task:**
   ```
   Mark task [task-id] as in_progress so we can fix it.
   ```

2. **View acceptance criteria:**
   ```
   What are the acceptance criteria for task [task-id]?
   ```

3. **Implement missing criteria:**
   Address each criterion specifically.

---

## Review Issues

### "LSP not available" during review

**Symptoms:**
- Review skips structural checks
- Warning about LSP

**What it means:**
Language Server Protocol provides code intelligence. Some features need it.

**Solutions:**

1. **This is often OK:**
   Reviews can proceed without LSP, just with less precision.

2. **For better reviews:**
   Ensure your language's LSP is configured in Claude Code.

### Review shows many deviations

**Symptoms:**
- Many "Major Deviation" or "Missing" items
- Implementation seems different from spec

**Solutions:**

1. **Intentional changes:**
   If you made deliberate changes, update the spec:
   ```
   Update the spec to reflect the changes we made.
   ```

2. **Accidental gaps:**
   Identify what's missing and implement:
   ```
   What specific items are missing? Let's implement them.
   ```

3. **Spec was wrong:**
   If the spec was unrealistic:
   ```
   Modify the spec to remove [unrealistic requirement].
   ```

---

## Testing Issues

### Tests fail after implementation

**Symptoms:**
- Tests were passing before
- New failures after your changes

**Solutions:**

1. **Use foundry-test skill:**
   ```
   Run the tests and help me debug failures.
   ```

2. **Check test output:**
   Look at specific failure messages.

3. **Common causes:**
   - Forgot to update tests for new behavior
   - Breaking change to existing API
   - Missing mock/fixture updates

### Tests timing out

**Symptoms:**
- Tests hang or timeout
- Performance issues

**Solutions:**

1. **Run specific failing test:**
   ```
   Run just the failing test to see what's happening.
   ```

2. **Check for infinite loops:**
   Review recently changed code for loop issues.

3. **Increase timeout temporarily:**
   For debugging, increase test timeout.

---

## Research Issues

### Deep research timing out

**Symptoms:**
- `foundry-research deep` never returns
- Status check shows still running

**What it means:**
Deep research runs in background and can take 2-5 minutes.

**Solutions:**

1. **Check status:**
   ```
   Use foundry-research to check status of research-[id]
   ```

2. **Wait longer:**
   Complex research takes time. Don't refresh the page.

3. **Try simpler workflow:**
   For faster results:
   ```
   Use foundry-research chat [question]
   ```

### Research gives unhelpful answers

**Symptoms:**
- Answer doesn't address your question
- Too generic or off-topic

**Solutions:**

1. **Be more specific:**
   ```
   Use foundry-research chat: Specifically, how does X handle Y in the context of Z?
   ```

2. **Try different workflow:**
   - `consensus` for design decisions
   - `thinkdeep` for debugging
   - `ideate` for brainstorming

3. **Provide context:**
   ```
   Use foundry-research chat: Given our use of [framework], how should we [question]?
   ```

---

## PR Issues

### PR description missing context

**Symptoms:**
- Generated PR is too brief
- Missing technical details

**Solutions:**

1. **Ensure journal entries exist:**
   Good completion notes create better PRs.

2. **Add context manually:**
   Edit the draft before creating:
   ```
   Edit the PR draft to add [specific context].
   ```

3. **Reference the spec:**
   PRs link to specs for full context.

### Can't push to remote

**Symptoms:**
- PR creation fails at push step
- Git errors about remote

**Solutions:**

1. **Check remote configuration:**
   ```bash
   git remote -v
   ```

2. **Ensure branch exists:**
   ```bash
   git branch -a
   ```

3. **Push manually first:**
   ```bash
   git push -u origin [branch-name]
   ```
   Then create PR:
   ```
   Create the PR now that the branch is pushed.
   ```

---

## Getting More Help

### In Claude Code
Ask Claude directly:
```
I'm having trouble with [issue]. Can you help diagnose?
```

### Capture for later
If something seems like a bug:
```
Use foundry-note to capture: [Bug] Description of what went wrong
```

### Check plugin version
Ensure you're on the latest version:
```
/plugin update foundry@claude-foundry
```

---

## Common Error Messages

| Error | Likely cause | Solution |
|-------|-------------|----------|
| "MCP tool not found" | foundry-mcp not installed | Install with pip |
| "Permission denied" | Missing permissions | Update settings.local.json |
| "No active specs" | Spec not activated | Activate or create spec |
| "Validation failed" | Spec has issues | Run validate and fix |
| "Dependencies not met" | Task depends on incomplete | Complete dependency first |
| "Context limit" | Long session | Clear context and continue |
