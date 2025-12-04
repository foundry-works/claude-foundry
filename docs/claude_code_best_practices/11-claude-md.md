# 11. CLAUDE.md

> Project and user instructions that persist across sessions and guide Claude's behavior.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

CLAUDE.md files store instructions and preferences that Claude Code remembers across sessions. They provide:

- **Persistent context** - Guidelines preserved between conversations
- **Project standards** - Coding styles, patterns, and conventions
- **Behavioral guidance** - How Claude should approach tasks
- **Team alignment** - Shared standards via version control

---

## File Hierarchy

### Four-Level System

| Level | Location | Scope | Precedence |
|-------|----------|-------|------------|
| Enterprise | System paths | Organization-wide | Highest |
| Project (shared) | `./CLAUDE.md` | Team via git | High |
| User | `~/.claude/CLAUDE.md` | All projects | Medium |
| Project (local) | `./CLAUDE.local.md` | Personal, single project | Lowest |

### System Paths (Enterprise)

| Platform | Path |
|----------|------|
| macOS | `/Library/Application Support/ClaudeCode/CLAUDE.md` |
| Linux | `/etc/claude-code/CLAUDE.md` |
| Windows | `C:\ProgramData\ClaudeCode\CLAUDE.md` |

### Project Paths

| File | Purpose | Version Control |
|------|---------|-----------------|
| `./CLAUDE.md` | Shared project standards | Yes |
| `./.claude/CLAUDE.md` | Alternative location | Yes |
| `./CLAUDE.local.md` | Personal overrides | No (gitignore) |

### Discovery

Claude Code discovers CLAUDE.md files by:
1. Recursing up from current directory to (but not including) root
2. Loading files from higher hierarchy levels first
3. Discovering nested memories during file reads

---

## Content Format

### Markdown Structure

Standard markdown with hierarchical organization:

```markdown
# Project Standards

## Code Style
- Use 2-space indentation
- Prefer const over let
- Use TypeScript strict mode

## Architecture
- Follow hexagonal architecture pattern
- Keep business logic in /src/domain
- External integrations in /src/adapters

## Testing
- Write unit tests for all business logic
- Use integration tests for adapters
- Minimum 80% coverage for new code
```

### Import Syntax

Reference external files with `@` syntax:

```markdown
# Project Guidelines

See coding standards:
@docs/coding-standards.md

Architecture overview:
@docs/architecture.md
```

| Requirement | Details |
|-------------|---------|
| MUST | Use relative or absolute paths |
| MAY | Import recursively (up to 5 levels deep) |
| SHOULD | Keep imports focused and relevant |

---

## Content Guidelines

### What to Include

| Category | Examples |
|----------|----------|
| **Coding style** | Indentation, naming conventions, formatting |
| **Architecture** | Patterns, directory structure, dependencies |
| **Code review** | Criteria, common issues to flag |
| **Workflow** | Common commands, tools, processes |
| **Domain context** | Terminology, business rules |
| **Preferences** | Frameworks, libraries, approaches |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Be specific and concrete (e.g., "2-space indentation") |
| MUST | Be actionable (Claude can apply the guidance) |
| SHOULD | Use examples when helpful |
| SHOULD | Keep content focused and concise |
| MUST NOT | Include secrets, credentials, or API keys |
| MUST NOT | Include vague statements ("use best practices") |

---

## Examples

### Project CLAUDE.md

```markdown
# Project Standards

## Code Style
- TypeScript with strict mode enabled
- 2-space indentation
- Single quotes for strings
- No semicolons (Prettier handles this)

## Architecture
- React components in /src/components
- Business logic in /src/hooks
- API calls through /src/api

## Testing
- Jest for unit tests
- React Testing Library for component tests
- MSW for API mocking

## Git Conventions
- Conventional commits (feat:, fix:, chore:)
- Branch naming: feature/, bugfix/, hotfix/
- Squash merge to main
```

### User CLAUDE.md

```markdown
# Personal Preferences

## Workflow
- Always explain changes before making them
- Show file paths with line numbers
- Run tests after significant changes

## Communication
- Be concise in explanations
- Use bullet points for lists
- Include code examples when relevant
```

### Enterprise CLAUDE.md

```markdown
# Organization Standards

## Security
- Never commit secrets or credentials
- Use environment variables for configuration
- Follow OWASP security guidelines

## Compliance
- Include license headers in new files
- Document all public APIs
- Maintain changelog entries

## Code Review
- All changes require review
- Security-sensitive changes need security team approval
```

---

## CLAUDE.md vs settings.json

| Aspect | CLAUDE.md | settings.json |
|--------|-----------|---------------|
| **Purpose** | Instructions and guidance | Technical configuration |
| **Format** | Markdown | JSON |
| **Content** | How to work | What tools to use |
| **Examples** | "Use React hooks" | `"Bash(npm run:*)"` |
| **Scope** | Behavioral patterns | Permissions, environment |

### Use CLAUDE.md For

- Coding style preferences
- Architecture guidelines
- Code review criteria
- Workflow conventions
- Domain-specific context

### Use settings.json For

- Tool permissions (allow/deny/ask)
- Environment variables
- MCP server configuration
- Permission modes
- Hook configurations

---

## Inheritance and Merging

### How Files Merge

1. All applicable CLAUDE.md files load into context
2. Higher-hierarchy files establish foundation
3. Lower-hierarchy files add or override
4. Enterprise policies always apply

### Precedence (Conflicts)

When the same topic appears in multiple files:

1. Enterprise (highest) - Cannot be overridden
2. Project local (`./CLAUDE.local.md`)
3. Project shared (`./CLAUDE.md`)
4. User (`~/.claude/CLAUDE.md`) - Lowest

---

## Best Practices

### Organization

| Requirement | Details |
|-------------|---------|
| SHOULD | Use clear headings (H1, H2, H3) |
| SHOULD | Group related items under sections |
| SHOULD | Keep file length reasonable |
| MAY | Import external docs for detailed specs |

### Maintenance

| Requirement | Details |
|-------------|---------|
| SHOULD | Review and update as projects evolve |
| SHOULD | Remove outdated guidance |
| SHOULD | Keep in sync with actual practices |

### Team Collaboration

| Requirement | Details |
|-------------|---------|
| SHOULD | Commit shared CLAUDE.md to version control |
| SHOULD | Review CLAUDE.md changes in PRs |
| SHOULD | Use CLAUDE.local.md for personal preferences |
| MUST NOT | Commit CLAUDE.local.md (add to .gitignore) |

---

## Security Considerations

### Sensitive Information

| Requirement | Details |
|-------------|---------|
| MUST NOT | Include API keys or tokens |
| MUST NOT | Include passwords or credentials |
| MUST NOT | Include personal identifying information |
| SHOULD | Review before committing to git |

### Shared Files

| Requirement | Details |
|-------------|---------|
| SHOULD | Audit content before sharing |
| SHOULD | Ensure guidelines reflect security standards |
| SHOULD | Track changes in version control |

---

## Commands

### /init

Initialize project with CLAUDE.md:
```
/init
```

Creates initial `./CLAUDE.md` with project context.

### /memory

Edit CLAUDE.md files:
```
/memory
```

Opens menu to view and edit memory files.

---

## Anti-Patterns

### Don't: Vague Instructions

```markdown
# Bad
- Use best practices
- Write good code
- Follow standards
```

```markdown
# Good
- Use 2-space indentation
- Prefix private methods with underscore
- Write JSDoc for public functions
```

### Don't: Include Secrets

```markdown
# Bad
API_KEY=sk-abc123...
DATABASE_URL=postgres://user:pass@host/db
```

```markdown
# Good
- API keys stored in .env (not committed)
- Database config via environment variables
```

### Don't: Overload with Content

```markdown
# Bad
[5000 lines of detailed specifications]
```

```markdown
# Good
# Standards

Key principles...

For detailed specs:
@docs/detailed-standards.md
```

### Don't: Duplicate settings.json

```markdown
# Bad
Allow Bash commands: npm run test, npm run build
Deny file access to .env
```

```markdown
# Good
[Use settings.json for permissions]
[Use CLAUDE.md for coding guidelines]
```

---

## File Templates

### Minimal Project

```markdown
# Project: [Name]

## Stack
- [Framework/Language]
- [Key dependencies]

## Conventions
- [Key coding standards]

## Commands
- `npm run dev` - Start development
- `npm test` - Run tests
```

### Comprehensive Project

```markdown
# Project: [Name]

## Overview
[Brief project description]

## Architecture
[Key architectural decisions]

## Code Style
[Formatting, naming, patterns]

## Testing
[Testing approach and requirements]

## Git Workflow
[Branch naming, commit conventions]

## Common Tasks
[Frequently used commands]
```

---

## Related Documents

- [07-permissions.md](./07-permissions.md) - Permission configuration (settings.json)
- [08-file-structure.md](./08-file-structure.md) - Project structure
- [06-plugin-manifest.md](./06-plugin-manifest.md) - Plugin configuration

---

**Navigation:** [← Previous: MCP Consumption](./10-mcp-consumption.md) | [Index](./README.md)
