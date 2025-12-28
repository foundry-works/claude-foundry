# Changelog

All notable changes to claude-foundry will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
