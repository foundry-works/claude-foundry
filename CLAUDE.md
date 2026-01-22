# Claude Foundry

Spec-driven development (SDD) toolkit for Claude Code. Plan before code, verify against spec.

## Workflow

```
foundry-spec → foundry-implement → [CODE] → foundry-review → foundry-test → foundry-pr
```

**Supporting skill:** foundry-refactor (LSP-powered refactoring)

## Skill Selection

| When you need to... | Use |
|---------------------|-----|
| Create/review/modify a spec | `foundry-spec` |
| Find next task, implement, track progress | `foundry-implement` |
| Verify implementation matches spec | `foundry-review` |
| Run tests and debug failures | `foundry-test` |
| Create PR with spec context | `foundry-pr` |
| Safe refactoring with LSP | `foundry-refactor` |
| Quick capture ideas/issues | `foundry-note` |
| AI research (chat, consensus, thinkdeep, ideate, deep) | `foundry-research` |

## Key Patterns

**MCP-First**: All skills use `mcp__plugin_foundry_foundry-mcp__*` tools exclusively. No CLI fallbacks.

**Human-in-Loop**: Skills use `AskUserQuestion` for key decisions. Don't assume - ask.

**Task States**: `pending` → `in_progress` → `completed` | `blocked`
- **Sequential mode** (interactive, autonomous): Only one task `in_progress` at a time
- **Parallel mode** (`--parallel`): Multiple tasks `in_progress` during batch execution
- Mark tasks complete immediately after finishing

**Safety**: Use dry-run previews before spec modifications. Backups created automatically.

**Context Flow**: Spec metadata (journals, dependencies, completion notes) passes through the workflow.

**Note (Autonomous Capture)**: Proactively add notes when encountering issues - do NOT prompt the user:
- MCP tool errors or unexpected failures → `[Error]`
- Missing/wished-for tools → `[Feature]`
- Confusing behavior or documentation gaps → `[Docs]`
- Ideas beyond current scope → `[Idea]`

```bash
mcp__plugin_foundry_foundry-mcp__authoring action="intake-add" title="[Type] description" tags='["foundry-feedback"]' source="skill-name"
```

User can invoke `foundry-note` skill for manual capture or list pending items.

## Common Workflows

### Starting New Work
1. Invoke `foundry-spec` - creates spec in `specs/pending/`
2. Review and refine spec (plan-review and modify are built into foundry-spec)
3. Activate spec (moves to `specs/active/`)

### Resuming Active Work
1. Use `foundry-implement` to find next actionable task
2. Implement the task
3. Complete task via `foundry-implement` workflow (tracks progress automatically)
4. Continue until phase complete

### Completing a Phase
1. Run `foundry-review` to verify implementation matches spec
2. Execute `foundry-test` to validate functionality
3. Create PR with `foundry-pr` when ready

## Subagent Usage

Use the **Explore** subagent before skill invocation when you need:
- Codebase context for planning (`foundry-spec`)
- Understanding existing patterns before implementation
- Finding related files for fidelity review

Example: Before `foundry-spec`, launch Explore to understand existing architecture.
