# Changelog

All notable changes to claude-foundry will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
