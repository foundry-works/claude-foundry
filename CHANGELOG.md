# Changelog

All notable changes to claude-foundry will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.8] - 2025-12-22

### Changed

- **Fix namespace collisions**: Renamed commands to avoid conflicts with same-named skills
  - `foundry-setup` → `setup-cmd`
  - `foundry-tutorial` → `tutorial-cmd`
  - `sdd-next` → `next-cmd`
  - `sdd-off` → `off-cmd`
  - `sdd-on` → `on-cmd`
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
- Added `mcp_ctl` permission category to setup-permissions script

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

- **Mode toggling**: `/sdd-on` and `/sdd-off` commands to switch between full (17 tools) and minimal (1 tool) modes for context token savings
- **New skill**: `sdd-refactor` for spec-driven refactoring with impact analysis
- **LSP-enhanced analysis** in skills:
  - `sdd-plan`: Impact analysis via `findReferences`, dead code detection
  - `sdd-next`: Dependency preview and cross-file impact check before implementation
  - `sdd-fidelity-review`: Structural pre-check verifying expected symbols exist
  - `run-tests`: Phase 0 pre-flight diagnostics catching import errors early
- Updated MCP configuration docs with `foundry-ctl` wrapper pattern
- Added `foundry-ctl` permissions to settings example

### Changed

- README now documents recommended MCP config for mode toggling
- `foundry-setup` command includes mode toggling configuration

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
