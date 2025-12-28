# History Tracking

Track spec modifications, compare versions, and make informed rollback decisions.

## Viewing Spec History

View the modification history for any spec:

```bash
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="{spec-id}"
```

### Filtering Options

```bash
# Limit entries
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" limit=10

# Filter by date range
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" since="2025-12-01" until="2025-12-15"

# Filter by entry type
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" entry_type="status_change"
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" entry_type="modification"
mcp__plugin_foundry_foundry-mcp__spec action="history" spec_id="my-spec" entry_type="backup"
```

### Entry Types

| Type | Description | Typical Content |
|------|-------------|-----------------|
| `status_change` | Task status updates | Task ID, old/new status, timestamp |
| `modification` | Structural changes | Added/removed tasks, phase updates |
| `validation` | Fix applications | Issues fixed, remaining problems |
| `backup` | Backup creation | Backup path, trigger reason |

---

## Diff Comparison Patterns

Compare current spec state against previous versions.

### Compare Against Backup

```bash
# Compare with most recent backup
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="my-spec" compare_to="specs/.backups/my-spec-latest.json"

# Compare with specific backup
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="my-spec" compare_to="specs/.backups/my-spec-2025-12-27T10-30-00.json"
```

### Reading Diff Output

```
📊 Spec Diff: my-spec-001

Tasks:
  + task-2-5: New task (added)
  ~ task-1-3: Description changed
  - task-3-1: Removed

Legend:
  + = Added
  ~ = Modified
  - = Removed
```

### Common Diff Scenarios

**After bulk modification:**
```bash
# See what sdd-modify changed
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="my-spec" compare_to="specs/.backups/my-spec-pre-modify.json"
```

**Before activating pending spec:**
```bash
# Compare pending vs template/original
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="my-spec" compare_to="specs/templates/feature-template.json"
```

**After validation fix:**
```bash
# See what auto-fix changed
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="my-spec" compare_to="specs/.backups/my-spec-pre-fix.json"
```

---

## Rollback Decision Workflow

When something goes wrong, use this workflow to decide whether to rollback.

### Step 1: Assess the Damage

```bash
# Check current validation state
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="my-spec"

# Compare against backup to see changes
mcp__plugin_foundry_foundry-mcp__spec action="diff" spec_id="my-spec" compare_to="specs/.backups/my-spec-{timestamp}.json"
```

### Step 2: Decision Matrix

| Situation | Recommendation | Action |
|-----------|---------------|--------|
| Few changes, easily fixed | Forward-fix | Manually correct issues |
| Many changes, spec broken | Rollback | Use sdd-modify rollback |
| Partial good changes | Selective | Rollback then re-apply good changes |
| Validation errors only | Forward-fix | Run additional fix passes |

### Step 3: Execute Decision

**Forward-fix path:**
```bash
# Continue fixing
mcp__plugin_foundry_foundry-mcp__spec action="fix" spec_id="my-spec"
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="my-spec"
```

**Rollback path:**
```bash
# Use sdd-modify for rollback (see Integration section below)
Skill(foundry:sdd-modify) "Rollback spec my-spec to backup specs/.backups/my-spec-{timestamp}.json"
```

### Rollback Indicators

**Consider rollback when:**
- Validation errors increased significantly after modification
- Critical structure broken (phases missing, hierarchy corrupted)
- Unintended bulk changes applied
- Progress tracking data lost

**Avoid rollback when:**
- Only a few minor issues to fix
- Changes were intentional, just need adjustment
- Forward progress would be lost

---

## Integration with sdd-modify Rollback

The sdd-validate skill works with sdd-modify for rollback operations.

### Workflow Integration

```
sdd-validate (assess)  →  Decision  →  sdd-modify (rollback)
       ↓                                      ↓
   history/diff           ←           backup restored
       ↓
   verify restored
```

### Using sdd-modify for Rollback

```bash
# Invoke sdd-modify rollback capability
Skill(foundry:sdd-modify) "Rollback spec {spec-id} to backup at {backup-path}"
```

**sdd-modify handles:**
- Backup file validation
- Safe file replacement
- Post-rollback validation
- Journal entry creation

### Backup Locations

Automatic backups are stored in `specs/.backups/`:

```
specs/.backups/
├── my-spec-2025-12-27T10-30-00.json    # Before modification
├── my-spec-2025-12-27T14-15-00.json    # Before fix
└── my-spec-latest.json                  # Most recent backup
```

### Post-Rollback Verification

After any rollback, always verify:

```bash
# Validate restored spec
mcp__plugin_foundry_foundry-mcp__spec action="validate" spec_id="my-spec"

# Confirm expected state
mcp__plugin_foundry_foundry-mcp__spec action="stats" spec_id="my-spec"
```
