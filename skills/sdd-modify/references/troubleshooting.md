# Error Handling and Troubleshooting

Common issues, constraints, and resolution strategies.

## Error Handling

### Spec Not Found

**Symptom:** Error mentioning spec ID not found

**Resolution:**
1. Verify spec ID is correct
2. Check spec location: `mcp__plugin_foundry_foundry-mcp__spec action="list"`
3. Ensure spec is in expected folder (active/, pending/, etc.)

### Invalid Modifications File

**Symptom:** JSON parse error or invalid operation

**Resolution:**
1. Validate JSON syntax
2. Check operation names are correct
3. Verify task IDs exist in spec
4. Ensure required fields are present

### Validation Failure After Apply

**Symptom:** "Validation failed, rolling back" message

**What happened:**
- Modifications were applied
- Validation detected issues
- Original spec restored from backup

**Resolution:**
1. Review validation errors
2. Fix modifications to address validation issues
3. Re-apply with corrected modifications

### Review Report Cannot Be Parsed

**Symptom:** `review-parse-feedback` fails

**Resolution:**
1. Verify review file exists and is readable
2. Check review format matches expected structure
3. Try running review again with proper output format

---

## Constraints

- **Do NOT** read spec JSON directly - use MCP tools only
- **Do NOT** apply modifications without previewing first
- **Do NOT** skip validation after apply
- **Do NOT** continue if validation fails - report and stop
- **Return** concise summaries, not verbose logs
- **Document** significant modifications in journal entries

---

## Troubleshooting

### Preview Shows No Changes

**Possible causes:**
- Modifications file is empty or malformed
- Task IDs don't exist in spec
- Values are already set to requested values

**Fix:** Verify modifications JSON structure and task IDs

### Validation Fails Repeatedly

**Possible causes:**
- Modifications create invalid spec structure
- Circular dependencies introduced
- Required fields missing

**Fix:** Review validation errors, adjust modifications accordingly

### Rollback Occurred But Issue Persists

**Possible causes:**
- Backup file was corrupted
- Manual edits made between backup and apply

**Fix:**
1. Check backup file integrity
2. Use older backup if available
3. Regenerate spec from source if necessary

### Review Parsing Produces Empty Modifications

**Possible causes:**
- Review report has no actionable findings
- Review format not recognized

**Fix:**
1. Check review report content
2. Ensure review follows expected markdown structure
3. Consider direct modifications if parsing fails

---

## Rollback Error Scenarios

### Backup Version Not Found

**Symptom:** `spec-rollback` fails with "version not found" error

**Resolution:**
1. List available backups: `mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="{spec-id}"`
2. Verify the timestamp format matches exactly (e.g., `2025-12-27T21-05-01.437327`)
3. Use an available backup timestamp from the history

### Backup File Corrupted

**Symptom:** Rollback fails with JSON parse error or validation failure

**Resolution:**
1. Try an older backup from history
2. If no valid backups exist, manually reconstruct spec:
   - Export current state: `mcp__plugin_foundry_foundry-mcp__spec action="find" spec_id="{spec-id}"`
   - Create new spec with corrected structure using `sdd-plan`
3. Prevent future issues by verifying backups after major changes

### Rollback Conflicts With Current State

**Symptom:** Rollback succeeds but spec validation fails

**Resolution:**
1. The rollback creates a backup before restoring - you can roll back the rollback
2. Review what changed between backup and current state
3. Consider partial restoration: apply individual modifications instead of full rollback

---

## Recovery Patterns

### Failed Task Move Recovery

**Symptom:** Task move partially completed or left spec in inconsistent state

**Recovery steps:**
1. Check current task location: `mcp__plugin_foundry_foundry-mcp__task action="info" spec_id="{spec-id}" task_id="{task-id}"`
2. If task exists but in wrong location, move again with correct parameters
3. If task missing, restore from backup:
   ```bash
   mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="{spec-id}"
   mcp__plugin_foundry_foundry-mcp__authoring action="spec-rollback" spec_id="{spec-id}" version="{timestamp}"
   ```

### Failed Find-Replace Recovery

**Symptom:** Find-replace made unintended changes or partial replacements

**Recovery steps:**
1. Review changes in dry-run output (if saved)
2. Use rollback to restore previous state:
   ```bash
   mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="{spec-id}"
   mcp__plugin_foundry_foundry-mcp__authoring action="spec-rollback" spec_id="{spec-id}" version="{pre-change-timestamp}"
   ```
3. Re-run find-replace with corrected parameters (more specific pattern, different scope)

**Prevention:**
- Always use `dry_run=true` first
- Use `scope="titles"` or `scope="descriptions"` instead of `"all"` when possible
- Use `case_sensitive=true` (default) for precise matching

### Failed Phase Move Recovery

**Symptom:** Phase ordering incorrect after move operation

**Recovery steps:**
1. Check current phase order: `mcp__plugin_foundry_foundry-mcp__task action="query" spec_id="{spec-id}" node_type="phase"`
2. Move phase to correct position: `mcp__plugin_foundry_foundry-mcp__authoring action="phase-move" spec_id="{spec-id}" phase_id="{phase-id}" position={correct-position}`
3. If multiple phases affected, restore from backup and re-plan the reordering

### General Recovery Pattern

When any structural operation fails:

1. **Assess current state** - Use `task action="query"` to understand current structure
2. **Check backup history** - Use `spec action="history"` to find pre-change backup
3. **Decide recovery approach:**
   - Minor issue → Fix with targeted operation
   - Major corruption → Rollback to backup
   - No valid backup → Recreate affected nodes with `sdd-modify`
4. **Validate after recovery** - Use `spec action="validate"` to confirm spec integrity
5. **Document in journal** - Record what happened and how it was resolved
