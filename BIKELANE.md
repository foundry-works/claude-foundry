# Bikelane

Issues to address later - not blocking current work.

---

## MCP Tooling Discoverability

### `phase-remove` not discoverable via MCP introspection

**Issue:** When trying to remove phases from a spec, `task action="remove"` returns an error saying "Only task, subtask, or verify nodes can be removed" - but doesn't mention that `authoring action="phase-remove"` exists.

**Discovery path:** Had to search skill documentation (`skills/sdd-plan/references/phase-authoring.md`) to find the correct action.

**Suggested fix:** Either:
1. Include `phase-remove` hint in the `task action="remove"` error message
2. Ensure `server action="tools"` returns all available actions including `phase-remove`
3. Add phase removal to the `task` router for consistency

**Date:** 2024-12-28
