# Claude Foundry

Spec-driven development (SDD) toolkit for Claude Code. Plan before code, verify against spec.

## Workflow

```
sdd-plan → sdd-implement → [CODE] → sdd-review → run-tests → sdd-pr
```

**Supporting skill:** sdd-refactor (LSP-powered refactoring)

## Skill Selection

| When you need to... | Use |
|---------------------|-----|
| Create/review/modify a spec | `sdd-plan` |
| Find next task, implement, track progress | `sdd-implement` |
| Verify implementation matches spec | `sdd-review` |
| Run tests and debug failures | `run-tests` |
| Create PR with spec context | `sdd-pr` |
| Safe refactoring with LSP | `sdd-refactor` |

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
2. Review and refine spec (plan-review and modify are built into sdd-plan)
3. Activate spec (moves to `specs/active/`)

### Resuming Active Work
1. Use `sdd-implement` to find next actionable task
2. Implement the task
3. Complete task via `sdd-implement` workflow (tracks progress automatically)
4. Continue until phase complete

### Completing a Phase
1. Run `sdd-review` to verify implementation matches spec
2. Execute `run-tests` to validate functionality
3. Create PR with `sdd-pr` when ready

## Subagent Usage

Use the **Explore** subagent before skill invocation when you need:
- Codebase context for planning (`sdd-plan`)
- Understanding existing patterns before implementation
- Finding related files for fidelity review

Example: Before `sdd-plan`, launch Explore to understand existing architecture.
