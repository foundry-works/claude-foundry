<foundry-instructions>

# Foundry

Spec-driven development (SDD) toolkit. Plan before code, verify against spec.

## Workflow

```
foundry-spec → foundry-implement → [CODE] → foundry-review
```

## Skill Selection

### Core Workflow

| Skill | Invoke When | Skip If |
|-------|-------------|---------|
| `foundry-spec` | New feature, multi-file refactor, API integration, architecture change | Single-file edit, trivial fix, exploratory spike |
| `foundry-implement` | Spec active, need next task, resume work, track progress | No spec, need to plan new work (use spec first) |
| `foundry-review` | Phase complete, before PR, audit task compliance | Finding tasks (use implement) |

### Supporting

| Skill | Invoke When | Skip If |
|-------|-------------|---------|
| `foundry-setup` | First foundry use, setup environment | Already configured |
| `foundry-research` | Need AI consultation, multiple perspectives, investigation | Simple questions, no research needed |

### Research (`foundry-research`)

| Signal | Workflow |
|--------|----------|
| Follow-up, iteration | `chat` |
| Multiple perspectives needed | `consensus` |
| Complex investigation | `thinkdeep` |
| Brainstorming | `ideate` |
| Web research, multiple sources, deep research | `deep research` |

## Key Patterns

**MCP-First**: All skills use Foundry MCP tools exclusively. No CLI fallbacks.

**Human-in-Loop**: Skills prompt for key decisions. Don't assume - ask.

**Task States**: `pending` → `in_progress` → `completed` | `blocked`
- **Sequential mode** (interactive, autonomous): Only one task `in_progress` at a time
- **Parallel mode** (`--parallel`): Multiple tasks `in_progress` during batch execution
- Mark tasks complete immediately after finishing

**Safety**: Use dry-run previews before spec modifications. Backups created automatically.

**Context Flow**: Spec metadata (journals, dependencies, completion notes) passes through the workflow.

**Note (Autonomous Capture)**: Proactively journal observations when encountering issues - do NOT prompt the user:
- MCP tool errors or unexpected failures → `[Error]`
- Missing/wished-for tools → `[Feature]`
- Confusing behavior or documentation gaps → `[Docs]`
- Ideas beyond current scope → `[Idea]`

Use `journal` MCP tool with `action="add"` to capture notes (requires a `spec_id`).

## Common Workflows

### Workflow Selection

| User Signal | Entry Point |
|-------------|-------------|
| "Build X", "Add feature Y" (multi-file) | Starting New Work |
| "Continue", "What's next?", spec exists | Resuming Active Work |
| "Done with phase", "Ready for PR" | Completing a Phase |
| "First time", "Setup foundry" | Run `foundry-setup` |

### Starting New Work
1. Invoke `foundry-spec` - creates spec in `specs/pending/`
   - Plan is linked to spec via `plan_path` / `plan_review_path` for traceability
   - Metadata (constraints, risks, open questions, success criteria) is extracted from the approved plan and populated via authoring actions
   - Phases are created using `phase-add-bulk` with inline tasks
2. Review and refine spec (plan-review and modify are built into foundry-spec)
   - Spec review compares the JSON spec against the source plan to catch translation gaps
3. Activate spec (moves to `specs/active/`)

### Resuming Active Work
1. Use `foundry-implement` to find next actionable task
2. Implement the task
3. Complete task via `foundry-implement` workflow (tracks progress automatically)
4. Continue until phase complete

### Completing a Phase
1. Run `foundry-review` to verify implementation matches spec
2. Run tests directly to validate functionality
3. Create PR with `gh` CLI when ready

## Versioning

When making doc/skill changes that warrant a version bump, update the version in `.claude-plugin/plugin.json`. Commit messages should include the version tag (e.g., `v1.8.0`) and the plugin.json version must match.

## Codebase Exploration

Launch the Explore tool before skill invocation when you need:
- Codebase context for planning (`foundry-spec`)
- Understanding existing patterns before implementation
- Finding related files for fidelity review

Example: Before `foundry-spec`, launch Explore tool to understand existing architecture.

</foundry-instructions>
