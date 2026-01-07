# Changelog

All notable changes to claude-foundry will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.6.6] - 2026-01-07

### Documentation

- **Document assumption-add action in sdd-plan**: Added missing `assumption-add` and `assumption-list` actions
  - Added to MCP Tooling table in authoring router
  - Added usage examples showing how to add and list assumptions
  - Prevents Claude from misusing `spec-update-frontmatter` for assumptions

## [1.6.5] - 2026-01-07

### Changed

- **Deep research polling strategy**: Improved polling approach for Claude's async limitations
  - Changed from timed polling (every 15 seconds) to maximum 5 status checks
  - Added progress communication guidelines between checks
  - Added stall detection logic (5 min elapsed + no progress change)
  - Added user options at final check (keep waiting, background, cancel)

### Documentation

- **Research tool defaults guidance**: Added to setup.md reference
  - Don't override timeout parameters - use foundry-mcp.toml defaults
  - Don't specify providers - use configured consensus providers
  - Minimal call pattern: `action="consensus" prompt="..." strategy="synthesize"`

## [1.6.4] - 2026-01-04

### Removed

- **Tutorial skill**: Removed `/tutorial` command and `foundry-tutorial` skill
  - Deleted `commands/tutorial.md`, `skills/foundry-tutorial/`
  - Deleted `BIKELANE.md` (moved to specs/.bikelane/)
  - Updated setup completion messages to remove tutorial references

## [1.6.3] - 2026-01-04

### Changed

- **Strengthen critical reference links across all skills**: Extended CRITICAL markers to remaining skills
  - `run-tests`: `tool-selection.md` (prompt templates, strategy parameters)
  - `sdd-pr`: `long-running.md` (mandatory timeout rules)
  - `sdd-refactor`: `impact-analysis.md` (required impact report format)
  - `sdd-review`: `lsp-integration.md` (required LSP operation patterns)
  - `research`: `deep-research-workflow.md` (polling strategy, MCP parameters)

## [1.6.2] - 2026-01-04

### Added

- **Model selection for delegated tasks**: New `--model` flag for `/implement` command
  - Specify `haiku`, `sonnet`, or `opus` for subagent tasks
  - Default: `haiku` (fast, cost-effective for most tasks)
  - Configurable via TOML: `[implement] model = "haiku"`
  - CLI flag overrides TOML default

### Changed

- **Strengthen critical reference links**: Changed soft links to mandatory directives for essential workflow references
  - `verification.md`: Now marked CRITICAL (contains skill dispatch rules)
  - `research-workflow.md`: Now marked CRITICAL (contains required command syntax)
  - `parallel-mode.md`: Now marked CRITICAL when using `--parallel` (contains required JSON formats)

## [1.6.1] - 2026-01-04

### Fixed

- **Outdated spec template documentation**: Updated all references from deprecated templates (`simple`, `medium`, `complex`, `security`) to `empty`
  - `skills/sdd-plan/SKILL.md`: Fixed 2 examples
  - `skills/sdd-plan/references/json-spec.md`: Updated template list and removed complex/security language
  - `skills/sdd-plan/references/task-hierarchy.md`: Removed complex/security template references
  - `skills/sdd-plan/references/troubleshooting.md`: Fixed example

## [1.6.0] - 2026-01-03

### Added

- **Research nodes**: New `type: "research"` task type for AI-powered investigation in specs
  - Five research workflows: `chat`, `consensus`, `thinkdeep`, `ideate`, `deep-research`
  - Blocking modes: `none`, `soft` (default), `hard`
  - New reference: `skills/sdd-plan/references/task-hierarchy.md` (Research Nodes section)
  - New reference: `skills/sdd-implement/references/research-workflow.md`

- **TOML-based implement defaults**: Mode flags configurable via `foundry-mcp.toml`
  - `[implement]` section with `auto`, `delegate`, `parallel` defaults
  - `[git]` section accessible for commit cadence decisions
  - Uses `environment action="get-config"` at entry

- **Orthogonal flag model**: `--auto`, `--delegate`, `--parallel` can be combined
  - `--auto` skips prompts (autonomous execution)
  - `--delegate` uses subagent(s) for implementation
  - `--parallel` runs subagents concurrently (implies `--delegate`)
  - Six distinct mode combinations documented

### Changed

- **run-tests skill refactored**: Simplified to focus on systematic debugging
  - Removed LSP pre-flight phase (import errors surface naturally)
  - Uses native test commands (`pytest`, `go test`, `npm test`) instead of MCP test tool
  - Switched from `provider action="execute"` to `research action="chat"` for AI consultation
  - Added `research action="consensus"` option for multi-perspective analysis
  - Renamed "Fixture" failure category to "Setup"

- **sdd-implement**: Updated execution mode documentation
  - Added `research` and `environment` routers to MCP tooling table
  - Cleaner flag interaction matrix in SKILL.md and command

- **disabled_tools**: Re-enabled `environment` tool (removed from default disabled list)
  - Needed for TOML config loading at implement entry

### Documentation

- Expanded reference files across skills with additional examples and patterns
- Updated `skills/foundry-setup/references/setup.md` with setup workflow details
- Enhanced failure category documentation in `skills/run-tests/references/`

## [1.5.2] - 2026-01-03

### Changed

- **Default disabled_tools**: Now disables `error`, `metrics`, `health`, and `environment` by default
  - Reduces context token usage for common workflows
  - Health and environment tools only needed during initial `/setup`
  - Error and metrics tools only needed for dashboard features

- **Setup workflow**: Removed Step 5 (health check validation) from Phase 2.7
  - Health check after TOML creation is unnecessary
  - Allows health tool to be disabled by default

## [1.5.0] - 2026-01-03

### Added

- **Batch operations integration**: `--auto` and `--parallel` execution modes for sdd-implement
  - **Autonomous mode** (`--auto`): Sequential execution without user prompts between tasks
    - Session state persistence across `/clear` boundaries
    - Pause triggers: context >= 85%, 3+ consecutive errors, blocked tasks, task limit
    - New `session-config` task action with commands: `start`, `status`, `pause`, `resume`, `end`
    - New reference: `skills/sdd-implement/references/autonomous-mode.md`
    - New reference: `skills/sdd-implement/references/session-management.md`
  - **Parallel mode** (`--parallel`): Concurrent execution via subagents
    - File-path conflict detection and exclusion
    - Batch operations: `prepare-batch`, `start-batch`, `complete-batch`, `reset-batch`
    - Partial failure handling with isolated task recovery
    - New reference: `skills/sdd-implement/references/parallel-mode.md`
  - Updated `SKILL.md` with Execution Modes section and flow diagrams
  - Updated `task-lifecycle.md` with Batch Task States section
  - Updated `CLAUDE.md` with sequential vs parallel mode task state behavior

- **disabled_tools configuration**: Tool filtering via `foundry-mcp.toml`
  - New `[tools]` section with `disabled_tools` list
  - Allows disabling specific MCP tools without code changes

- **Context configuration**: Configurable context denominator for context-monitor hook
  - New `[context]` section in `foundry-mcp.toml` with `auto_compact` setting
  - `auto_compact = true` (default): Uses 155k denominator (Claude auto-compacts)
  - `auto_compact = false`: Uses 200k denominator (full context window)
  - Updated `hooks/context-monitor` to read setting dynamically
  - Added Phase 2.2 (Context Configuration) to `/setup` workflow

### Changed

- **implement command**: Added Step 0 for session check and mode selection
  - Flag parsing: `--auto`, `--parallel`
  - Session resume prompt when paused session exists

## [1.4.0] - 2026-01-02

### Added

- **Deep research workflow**: Multi-phase iterative research with query decomposition
  - New `deep` workflow in `/research` command for comprehensive investigation
  - Query decomposition into sub-queries with parallel source gathering
  - Iterative refinement with configurable max iterations
  - Background execution support with status tracking
  - New reference: `skills/research/references/deep-research-workflow.md`

### Changed

- **Session management**: Renamed `thread-management.md` to `session-management.md`
  - Broader scope covering all session types (threads, investigations, ideations, deep-research)
  - Updated SKILL.md references accordingly
  - New reference: `skills/research/references/session-management.md`

- **Research skill updates**: Enhanced documentation for all workflows
  - Updated `commands/research.md` with deep-research routing
  - Updated `SKILL.md` with deep workflow in MCP contract
  - Updated `auto-routing.md` with deep-research intent patterns
  - Updated `troubleshooting.md` with deep-research error handling

## [1.3.1] - 2025-12-30

### Added

- **Research configuration in setup**: `/setup` now configures the `[research]` section in `foundry-mcp.toml`
  - New Phase 2.6 in setup workflow for research provider configuration
  - Uses same `[cli]provider:model` format as consultation for consistency
  - Automatically reuses provider priority from consultation section
  - Updated `foundry-mcp.toml` with example `[research]` section

## [1.3.0] - 2025-12-30

### Added

- **Research skill**: AI-powered research with four workflows and persistent threads
  - New `/research` command with auto-routing and explicit workflow selection
  - **chat**: Single-model iterative conversation with thread persistence
  - **consensus**: Multi-model parallel consultation with synthesis strategies
  - **thinkdeep**: Hypothesis-driven systematic investigation
  - **ideate**: Four-phase creative brainstorming (divergent → convergent → selection → elaboration)
  - Auto-routing: Intent-based workflow detection from natural language prompts
  - Thread management: List, resume, and delete conversation threads
  - New skill files:
    - `skills/research/SKILL.md`: Main skill with MCP contract and flow
    - `commands/research.md`: Command entry point with routing logic
  - New reference files:
    - `references/chat-workflow.md`, `references/consensus-workflow.md`
    - `references/thinkdeep-workflow.md`, `references/ideate-workflow.md`
    - `references/thread-management.md`, `references/auto-routing.md`
    - `references/troubleshooting.md`
  - Updated `CLAUDE.md` skill selection table

## [1.2.0] - 2025-12-29

### Added

- **Bikelane fast-capture integration**: Quick intake queue for ideas/tasks during implementation
  - New `/bikelane` command for manual capture (`add`, `list`, `dismiss` modes)
  - `sdd-implement`: Autonomous capture of ideas, bugs, and documentation gaps during implementation
  - New reference file: `skills/sdd-implement/references/bikelane.md`
  - `CLAUDE.md`: Added foundry-mcp feedback capture instructions (errors, missing tools)
  - `foundry-mcp.toml`: Added `bikelane` workspace entry
  - `.gitignore`: Added exception for `specs/.bikelane/` folder

## [1.1.0] - 2025-12-29

### Changed

- **Skill consolidation**: Reduced 10 SDD skills to 5+1 core skills for reduced cognitive overhead
  - `sdd-plan` now includes: AI review, modification, and validation (absorbed `sdd-plan-review`, `sdd-modify`, `sdd-validate`)
  - `sdd-next` renamed to `sdd-implement` and now includes: auto-completion and journaling (absorbed `sdd-update`)
  - `sdd-fidelity-review` renamed to `sdd-review` for consistency
  - Commands simplified: `next-cmd` → `implement`, `setup-cmd` → `setup`, `tutorial-cmd` → `tutorial`
  - Updated `CLAUDE.md` with simplified 6-skill workflow: `sdd-plan → sdd-implement → sdd-review → run-tests → sdd-pr`

### Added

- **New reference files for sdd-plan**:
  - `plan-review-workflow.md`, `plan-review-dimensions.md`, `plan-review-consensus.md`
  - `modification-workflow.md`, `modification-operations.md`
  - `validation-workflow.md`, `validation-fixes.md`, `validation-issues.md`

- **New reference files for sdd-implement**:
  - `task-lifecycle.md`, `progress-tracking.md`, `journaling.md`, `spec-lifecycle.md`

### Removed

- **Retired skills** (functionality consolidated):
  - `sdd-plan-review` → merged into `sdd-plan`
  - `sdd-modify` → merged into `sdd-plan`
  - `sdd-validate` → merged into `sdd-plan`
  - `sdd-update` → merged into `sdd-implement`

- **Archived completed specs**:
  - `incorporate-mcp-capabilities-2025-12-27-001.json`
  - `sdd-plan-fixes-2025-12-24-001.json`

## [1.0.18] - 2025-12-28

### Added

- **phase-update-metadata documentation**: Integrated `phase-update-metadata` MCP action across SDD skills
  - `sdd-modify`: Added to MCP Tooling table and Supported Operations; full operation documentation in `references/operations.md`
  - `sdd-update`: Added authoring router with `phase-update-metadata`; documented "Update Phase Estimates" workflow in `references/workflow-tracking.md`
  - `sdd-plan`: Added to Authoring Router Actions table; documented "Quick Phase Refinements" in `references/phase-authoring.md`

## [1.0.17] - 2025-12-28

### Added

- **MCP capability documentation**: Comprehensive updates to all SDD skills documenting new MCP server capabilities
  - `sdd-modify`: Added `spec:completeness-check`, `authoring:find-replace`, bulk operations, phase-move workflow
  - `sdd-update`: Added `add-dependency`, `remove-dependency`, `add-requirement` task actions
  - `sdd-validate`: Added `spec:history`, validation history tracking, issue mapping enhancements
  - `sdd-plan`: Added phase-move, dependency management, task reorganization patterns
  - `sdd-fidelity-review`: Added `review:diff`, `spec:diff`, incremental review support
  - `sdd-plan-review`: Added `review:synthesize`, `review:diff` for multi-model consultation
  - `sdd-pr`: Added `spec:history`, `spec:diff` for PR context enrichment
  - `sdd-next`: Added `add-dependency`, `add-requirement` for dependency discovery during implementation
  - `run-tests`: Added `add-requirement` for requirement discovery during debugging

- **New reference files**:
  - `skills/sdd-update/references/workflow-dependencies.md`: Dependency management patterns
  - `skills/sdd-validate/references/history-tracking.md`: Validation history tracking

## [1.0.15] - 2025-12-24

### Changed

- **sdd-plan skill improvements**: Enhanced phase authoring workflow and documentation
  - Updated SKILL.md with clearer workflow guidance (+138 lines)
  - Improved `json-spec.md`, `phase-authoring.md`, `task-hierarchy.md`, `troubleshooting.md` references
- **Configuration updates**: Updated `foundry-mcp.toml` settings

### Fixed

- **Test assertion mismatch**: Fixed `test_metadata_defaults_category_not_string` in foundry-mcp to expect `task_category` field name in error message

## [1.0.10] - 2025-12-23

### Changed

- **Phase-first authoring workflow**: New recommended approach in `sdd-plan` skill
  - Use `spec-create` → `phase-add-bulk` → `sdd-modify` → `spec-update-frontmatter`
  - Updated Flow diagram to align with phase-first approach
  - Added "Phase-First Authoring Workflow" section with step-by-step guide

### Removed

- **Retire schema-export references**: Removed deprecated `schema-export` workflow from `sdd-plan` skill
  - Schema-export was non-portable (absolute paths) and required manual JSON editing
  - Replaced by `phase-add-bulk` macro which creates phases with tasks atomically
  - Updated MCP Tooling table to remove non-existent spec actions (`get`, `get-hierarchy`, `schema-export`)
- **Remove foundry-ctl**: Removed mode toggling functionality (`/on-cmd`, `/off-cmd` commands) since foundry-mcp-ctl has been removed from foundry-mcp package
- Simplified MCP server configuration to direct `python -m foundry_mcp.server` command
- Removed `mcp_ctl` permission category and foundry-ctl references from documentation

## [1.0.9] - 2025-12-22

### Fixed

- **Fix MCP action name**: Changed `action="verify"` → `action="verify-env"` in setup preflight checks

### Changed

- **Updated command references**: All documentation now uses renamed commands (`/setup-cmd`, `/next-cmd`, `/tutorial-cmd`)

## [1.0.8] - 2025-12-22

### Changed

- **Fix namespace collisions**: Renamed commands to avoid conflicts with same-named skills
  - `foundry-setup` → `setup-cmd`
  - `foundry-tutorial` → `tutorial-cmd`
  - `sdd-next` → `next-cmd`
- Skills now load correctly without being shadowed by command wrappers

## [1.0.7] - 2025-12-22

### Changed

- **Convert commands to skills**: `foundry-setup` and `foundry-tutorial` are now skills with proper path resolution
- Commands are thin wrappers that invoke the corresponding skills
- Fixes path resolution failure when running from different workspaces

## [1.0.6] - 2025-12-22

### Changed

- **Progressive disclosure for commands**: Simplified `foundry-setup.md` and `foundry-tutorial.md` to flow notation
- Moved detailed command content to `references/` directory for on-demand loading

## [1.0.5] - 2025-12-22

### Changed

- **Modular references**: Split single `reference.md` files into topic-specific files in `references/` directories
- Each skill now has 5-7 focused reference files (e.g., `ai-review.md`, `troubleshooting.md`, `json-spec.md`)
- Enables on-demand loading of specific topics vs loading entire reference
- Added `docs/flow-notation.md` documentation

## [1.0.2] - 2025-12-22

### Changed

- **Progressive disclosure refactor**: Trimmed SKILL.md files to core workflows, moved detailed content to reference.md files for on-demand loading
- Added comprehensive skills guide (`docs/claude_code_best_practices/04-skills.md`)
- Expanded context management documentation

## [0.3.1] - 2025-12-21

### Added

- **New skill**: `sdd-refactor` for spec-driven refactoring with impact analysis
- **LSP-enhanced analysis** in skills:
  - `sdd-plan`: Impact analysis via `findReferences`, dead code detection
  - `sdd-next`: Dependency preview and cross-file impact check before implementation
  - `sdd-fidelity-review`: Structural pre-check verifying expected symbols exist
  - `run-tests`: Phase 0 pre-flight diagnostics catching import errors early

### Removed

- Removed `codebase.json` hook (no longer needed)

## [0.3.0] - 2025-12-15

### Changed

- **Breaking:** Migrated all MCP tool invocations from individual tools to 17-router consolidated architecture
  - Old pattern: `mcp__plugin_foundry_foundry-mcp__spec-validate`
  - New pattern: `mcp__plugin_foundry_foundry-mcp__spec action="validate"`
- Updated all 11 skills to use router+action pattern
- Updated all 12 documentation files in `docs/foundry_mcp_reference/`
- Updated all 4 hook scripts with new tool syntax in error messages
- Updated commands (`sdd-next.md`, `foundry-setup.md`, `foundry-tutorial.md`)
- Simplified permissions from ~120 individual tools to 17 router permissions

### Removed

- **Breaking:** Removed all session tool references (`session-work-mode`, `session-context`, `session-generate-marker`)
- Removed work mode feature from `sdd-next` skill (autonomous vs single mode routing)
- Removed context checking pattern (85% threshold) from workflows

### Fixed

- Hook error messages now display correct MCP tool syntax examples

## [0.2.0] - 2025-12-09

### Added

- Initial plugin release with SDD toolkit skills
- 11 skills: sdd-plan, sdd-plan-review, sdd-modify, sdd-next, sdd-update, sdd-fidelity-review, sdd-validate, sdd-pr, sdd-render, run-tests, doc-query
- Comprehensive documentation in `docs/foundry_mcp_reference/`
- Hook scripts for blocking direct codebase.json access
- Commands for setup and workflow entry points

## [0.1.0] - 2025-12-04

### Added

- Initial project structure
- Basic spec-driven development workflow
