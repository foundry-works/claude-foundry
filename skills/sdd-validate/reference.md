# SDD-Validate Reference

Detailed troubleshooting, edge cases, and advanced patterns for the sdd-validate skill.

## Table of Contents

- [Issue to Fix Mapping](#issue-to-fix-mapping)
- [Manual Fix Patterns](#manual-fix-patterns)
- [Advanced Troubleshooting](#advanced-troubleshooting)
- [Edge Cases](#edge-cases)

---

## Issue to Fix Mapping

### Auto-Fixable Issues

| Issue | Fix Action | Notes |
|-------|------------|-------|
| Incorrect task counts | Recalculates from hierarchy | Safe, no data loss |
| Missing metadata blocks | Adds empty metadata | Adds `{}` where missing |
| Orphaned nodes | Reconnects to parent | Uses position hints |
| Malformed timestamps | Converts to ISO 8601 | Preserves date/time info |
| Invalid status values | Resets to `pending` | Logs original value |
| Parent/child mismatches | Corrects references | Bi-directional sync |
| Missing required fields | Adds with defaults | Uses schema defaults |

### Manual-Only Issues

| Issue | Why Manual | Resolution |
|-------|-----------|------------|
| Circular dependencies | Requires intent understanding | Remove weakest edge |
| Orphaned dep references | Typo vs intentional | Fix typo or remove ref |
| Semantic inconsistencies | Business logic | Review spec intent |
| Custom metadata errors | Unknown structure | Requires context |

---

## Manual Fix Patterns

### Fixing Circular Dependencies

1. Identify the cycle from validation output
2. Understand the intent of each dependency
3. Remove the dependency that represents "nice to have" vs "must have"
4. Re-validate to confirm cycle is broken

**Example:**
```
Cycle: task-3-2 -> task-3-5 -> task-3-2

Analysis:
- task-3-2 "Create API client" depends on task-3-5 "Define interfaces"
- task-3-5 "Define interfaces" depends on task-3-2 "Create API client"

Resolution: Interfaces should be defined first
Remove: task-3-5's dependency on task-3-2
```

### Fixing Orphaned Dependencies

1. Check if the referenced task ID is a typo
2. If typo: correct the task ID
3. If intentional removal: delete the dependency reference

---

## Advanced Troubleshooting

### Validation Passes But Skill Fails

The spec may be structurally valid but semantically incorrect:
- Check that task IDs follow naming conventions
- Verify dependencies reference actual tasks
- Confirm parent-child relationships are logical

### Large Spec Performance

For specs with 100+ tasks:
- Use `--quiet` to suppress progress output
- Run validation in phases (validate -> fix -> re-validate)
- Consider splitting very large specs into multiple smaller ones

### Schema Validation Errors

If you see "JSON Schema validation failed":
1. Run `mcp__foundry-mcp__spec-schema-export` to see valid structure
2. Compare your spec against the schema
3. Look for required fields that are missing
4. Check enum values match allowed options

---

## Edge Cases

### Empty Spec Files

Validation will fail with "missing required fields". Use `sdd-plan` to create new specs rather than manual creation.

### Backup Files

`mcp__foundry-mcp__validate-fix` creates `.backup` files before modification. To revert:
```bash
cp my-spec.json.backup my-spec.json
```

### CI/CD Integration

Use exit codes for automation:
- Exit 0: Pass
- Exit 1: Warning (may be acceptable)
- Exit 2: Error (fail the build)
- Exit 3: File error (fail the build)
