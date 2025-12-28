# Issue to Fix Mapping

## Auto-Fixable Issues

| Issue | Fix Action | Notes |
|-------|------------|-------|
| Incorrect task counts | Recalculates from hierarchy | Safe, no data loss |
| Missing metadata blocks | Adds empty metadata | Adds `{}` where missing |
| Orphaned nodes | Reconnects to parent | Uses position hints |
| Malformed timestamps | Converts to ISO 8601 | Preserves date/time info |
| Invalid status values | Resets to `pending` | Logs original value |
| Parent/child mismatches | Corrects references | Bi-directional sync |
| Missing required fields | Adds with defaults | Uses schema defaults |

## Manual-Only Issues

| Issue | Why Manual | Resolution |
|-------|-----------|------------|
| Circular dependencies | Requires intent understanding | Remove weakest edge |
| Orphaned dep references | Typo vs intentional | Fix typo or remove ref |
| Semantic inconsistencies | Business logic | Review spec intent |
| Custom metadata errors | Unknown structure | Requires context |
| Duplicate task titles | May be intentional | Merge or differentiate |
| Similar descriptions | Context-dependent | Review scope overlap |
| Overlapping criteria | Verification design | Narrow scope or consolidate |

## Duplicate Detection

Detected via `mcp__plugin_foundry_foundry-mcp__spec action="duplicate-detection"`.

### Issue Types

| Type | Detection | Severity |
|------|-----------|----------|
| Exact title match | Title string equality across phases | High |
| Similar description | >80% fuzzy match on descriptions | Medium |
| Overlapping criteria | Shared acceptance criteria text | Low |
| Redundant verification | Multiple verify tasks with same scope | Medium |

### Resolution Patterns

**Exact Title Matches:**
```
Issue: task-2-3 and task-4-1 both titled "Add user validation"
Resolution options:
1. Merge: Combine into single task if truly duplicate
2. Differentiate: Rename to "Add user validation (API)" and "Add user validation (UI)"
3. Keep: If intentionally same work in different phases (rare)
```

**Similar Descriptions:**
```
Issue: 85% match between task-1-5 and task-3-2
Resolution options:
1. Clarify scope: Add distinguishing context to each description
2. Merge: If covering same functionality
3. Accept: If similar but different phases/contexts
```

**Overlapping Acceptance Criteria:**
```
Issue: verify-2-1 and verify-3-1 both check "all tests pass"
Resolution options:
1. Scope: "All phase-2 tests pass" vs "All phase-3 tests pass"
2. Consolidate: Single verify task covering both phases
3. Remove: If redundant verification
```

### When to Ignore

Duplicates may be intentional when:
- Same validation in different phases (staged rollout)
- Parallel implementations for comparison
- Template-based spec generation
