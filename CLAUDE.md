# Claude Foundry

Spec-driven development (SDD) toolkit for Claude Code. Plan before code, verify against spec.

## Workflow

```
sdd-plan → sdd-plan-review → sdd-modify → sdd-next → [IMPLEMENT] → sdd-update → sdd-fidelity-review → run-tests → sdd-pr
                                                                                                              ↓
                                                                                                         sdd-render
```

**Supporting skills:** sdd-validate (spec validation), doc-query (codebase queries)

## Skill Selection

| When you need to... | Use |
|---------------------|-----|
| Create a spec for new work | `sdd-plan` |
| Get AI feedback on a spec | `sdd-plan-review` |
| Apply review feedback to spec | `sdd-modify` |
| Find the next task to implement | `sdd-next` |
| Mark tasks complete/track progress | `sdd-update` |
| Verify implementation matches spec | `sdd-fidelity-review` |
| Run tests and debug failures | `run-tests` |
| Create PR with spec context | `sdd-pr` |
| Validate spec JSON structure | `sdd-validate` |
| Query codebase documentation | `doc-query` |
| Generate stakeholder documentation | `sdd-render` |

## Key Patterns

**MCP-First**: All skills use `mcp__plugin_foundry_foundry-mcp__*` tools exclusively. No CLI fallbacks.

**Human-in-Loop**: Skills use `AskUserQuestion` for key decisions. Don't assume - ask.

**Task States**: `pending` → `in_progress` → `completed` | `blocked`
- Only one task should be `in_progress` at a time
- Mark tasks complete immediately after finishing

**Safety**: Use dry-run previews before spec modifications. Backups created automatically.

**Context Flow**: Spec metadata (journals, dependencies, completion notes) passes through the workflow.

## Common Workflows

### Starting New Work
1. Invoke `sdd-plan` - creates spec in `specs/pending/`
2. Run `sdd-plan-review` for AI feedback
3. Apply fixes with `sdd-modify`
4. Activate spec (moves to `specs/active/`)

### Resuming Active Work
1. Use `sdd-next` to find next actionable task
2. Implement the task
3. Update status with `sdd-update` (uses `task action="complete"`)
4. Continue until phase complete

### Completing a Phase
1. Run `sdd-fidelity-review` to verify implementation
2. Execute `run-tests` to validate functionality
3. Create PR with `sdd-pr` when ready

## Subagent Usage

Use the **Explore** subagent before skill invocation when you need:
- Codebase context for planning (`sdd-plan`)
- Understanding existing patterns before implementation
- Finding related files for fidelity review

Example: Before `sdd-plan`, launch Explore to understand existing architecture.
