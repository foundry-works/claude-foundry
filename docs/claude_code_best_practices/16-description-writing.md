# 16. Writing Effective Descriptions

> Descriptions determine when Claude activates skills, delegates to agents, and surfaces commands. Well-crafted descriptions are the key to reliable component discovery.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Every Claude Code component (skills, agents, commands) has a `description` field that serves as its discovery mechanism. The description tells Claude **when** to use a component—not just what it does.

| Component | Description Purpose | Discovery Mechanism |
|-----------|--------------------|--------------------|
| **Skills** | Automatic context-based activation | Claude matches task to description |
| **Agents** | Delegation decisions | Claude decides when to spawn |
| **Commands** | User discovery via `/help` | User browses or searches |

At startup, Claude pre-loads all component names and descriptions into its system prompt. A poor description means Claude won't find the component when needed—or will activate it inappropriately.

---

## Description Anatomy

Every effective description answers three questions:

### 1. What (Capability)

What does this component do? State the core capability clearly.

```yaml
# Good: Clear capability
description: Extract text and tables from PDF files.

# Bad: Vague capability
description: Helps with documents.
```

### 2. When (Trigger Context)

When should Claude use this? Include specific triggers:
- File types (`.pdf`, `.csv`, `.json`)
- Task types (debugging, refactoring, migration)
- Domains (authentication, testing, deployment)
- Keywords users might say

```yaml
# Good: Clear trigger context
description: Extract text and tables from PDF files. Use when working with PDF documents, forms, or scanned reports.

# Bad: No trigger context
description: Extract text and tables from PDF files.
```

### 3. Scope (Boundaries)

What does this component NOT do? For overlapping capabilities, clarify boundaries.

```yaml
# Good: Clear scope boundaries
description: Analyze Python code for security vulnerabilities. Use for security audits, not general code review.

# Okay if no overlap exists
description: Generate database migration scripts from schema changes.
```

---

## Requirements by Component

### Skills

| Requirement | Details |
|-------------|---------|
| MUST | Write in **third person** ("Analyzes...", "Extracts...") |
| MUST | Stay under **1024 characters** |
| MUST | Include trigger context (when to activate) |
| SHOULD | Name specific file types, formats, or domains |
| SHOULD | Use action verbs at the start |
| MUST NOT | Use first person ("I analyze...") |
| MUST NOT | Use second person ("You can use this to...") |

> **Why third person?** Skill descriptions are injected into Claude's system prompt. Mixed perspectives (I/you/Claude) confuse the model during discovery.

### Agents

| Requirement | Details |
|-------------|---------|
| SHOULD | Describe task complexity that warrants delegation |
| SHOULD | Include workflow indicators (multi-step, research, analysis) |
| MAY | Use second or third person |
| SHOULD | Clarify what tasks are too simple for this agent |

### Commands

| Requirement | Details |
|-------------|---------|
| SHOULD | Focus on user intent and actions |
| SHOULD | Describe expected arguments if any |
| MAY | Use second person ("Use this to...") |
| SHOULD | Be scannable in `/help` output |

---

## Description Patterns

### Skill Descriptions

Focus on file types, formats, and automatic triggers:

```yaml
# Pattern: [Action] [target]. Use when [trigger conditions].

# PDF extraction
description: Extract text and tables from PDF files. Use when working with PDF documents, forms, or scanned reports.

# Security analysis
description: Analyze Python code for security vulnerabilities including injection, authentication bypass, and data exposure. Use when reviewing code for security issues.

# Data transformation
description: Convert CSV files to JSON format with schema inference. Use for data migration or API payload preparation.
```

### Agent Descriptions

Focus on task complexity and workflow type:

```yaml
# Pattern: [Task type] when [complexity indicator]. [Scope clarification].

# Code exploration
description: Explore codebases to answer questions about architecture, patterns, and implementation. Use for research tasks requiring multiple file reads.

# Test debugging
description: Investigate test failures through systematic analysis of logs, stack traces, and test isolation. Use when tests fail unexpectedly.

# Planning
description: Design implementation approaches for complex features. Use when the task requires architectural decisions or affects multiple components.
```

### Command Descriptions

Focus on user actions and workflow shortcuts:

```yaml
# Pattern: [User action] to [outcome]. [Argument hint if needed].

# Git workflow
description: Create a pull request with AI-generated description. Analyzes commits and generates summary.

# Documentation
description: Generate API documentation from source code. Provide file paths as arguments.

# Testing
description: Run tests and fix failures automatically. Optionally specify test file path.
```

---

## Examples: Before and After

| Before (Poor) | After (Better) | Issue Fixed |
|---------------|----------------|-------------|
| `Helps with code` | `Refactor Python code for improved readability and performance. Use when cleaning up technical debt or optimizing hot paths.` | Added specificity and triggers |
| `Database tool` | `Execute PostgreSQL queries against the development database. Use for schema inspection, data analysis, or migration validation.` | Named specific technology and use cases |
| `I analyze security issues` | `Analyze code for OWASP Top 10 vulnerabilities. Use when auditing web application code.` | Fixed POV, added trigger |
| `Process documents` | `Extract structured data from Excel spreadsheets. Use when working with .xlsx files or CSV exports.` | Narrowed scope, added file types |
| `Use this to help with testing` | `Generate pytest test cases for Python functions. Use when creating unit tests or expanding test coverage.` | Removed vague language, added specifics |

---

## Anti-Patterns

### 1. Vague Descriptions

```yaml
# Bad: What "things"? When?
description: Helps with things

# Bad: What kind of data? What operations?
description: Data processing tool

# Bad: No actionable trigger
description: Useful for coding tasks
```

**Fix:** Add specific capability + trigger context.

### 2. Missing Trigger Context

```yaml
# Bad: Capability clear, but when to use it?
description: Convert Markdown to HTML

# Good: Added when
description: Convert Markdown to HTML. Use when generating static site content or documentation builds.
```

### 3. Conflicting Descriptions

When two components have overlapping descriptions, Claude may pick the wrong one:

```yaml
# Skill 1 - Bad: ambiguous
description: Analyze data files

# Skill 2 - Bad: identical overlap
description: Analyze data files
```

```yaml
# Skill 1 - Good: distinct triggers
description: Analyze CSV sales data files for revenue patterns and trends

# Skill 2 - Good: different domain
description: Analyze JSON log files for error patterns and system health
```

### 4. Wrong Point-of-View

```yaml
# Bad: First person (skill)
description: I analyze code for bugs

# Bad: Second person (skill)
description: You can use this to analyze code

# Bad: Mixed perspectives
description: Claude analyzes code and I report findings

# Good: Third person (skill)
description: Analyzes code for bugs and reports findings with severity ratings
```

### 5. Too Long

Descriptions over 1024 characters (skills) or excessively long waste startup tokens:

```yaml
# Bad: Entire manual in description
description: This comprehensive tool performs end-to-end analysis of your codebase including but not limited to security vulnerabilities, performance issues, code style violations, dependency problems, and architectural concerns. It generates detailed reports with severity ratings, remediation suggestions, and links to relevant documentation. Use it whenever you need a thorough code review covering all aspects of code quality...

# Good: Concise with reference
description: Analyze codebases for security, performance, and style issues. Generates detailed reports with severity ratings. Use for comprehensive code audits.
```

### 6. No Differentiation

```yaml
# Bad: Generic descriptions that all sound similar
# Skill 1
description: Process Excel files

# Skill 2
description: Handle Excel spreadsheets

# Skill 3
description: Work with Excel data
```

```yaml
# Good: Each has unique purpose
# Skill 1
description: Extract raw data from Excel files into JSON format

# Skill 2
description: Generate Excel reports with charts from data sources

# Skill 3
description: Validate Excel files against schema definitions
```

---

## Description Checklist

Use this checklist before finalizing any description:

| Check | Skills | Agents | Commands |
|-------|--------|--------|----------|
| States what it does? | MUST | MUST | MUST |
| Includes when to use it? | MUST | SHOULD | SHOULD |
| Uses correct POV (third person for skills)? | MUST | - | - |
| Under length limit (1024 chars for skills)? | MUST | - | - |
| Includes specific triggers (file types, domains)? | SHOULD | SHOULD | MAY |
| Avoids overlap with similar components? | SHOULD | SHOULD | SHOULD |
| Uses action verb at start? | SHOULD | MAY | MAY |
| Scannable in list context? | SHOULD | SHOULD | MUST |

---

## Testing Descriptions

### Activation Testing

Verify your descriptions work by testing with various phrasings:

1. **Direct match:** "Extract data from this PDF"
2. **Indirect match:** "I have a scanned document I need to process"
3. **Negative test:** Confirm it doesn't activate for unrelated tasks

### Conflict Testing

If you have multiple similar components:

1. Test each with ambiguous prompts
2. Verify the correct one activates
3. Adjust trigger keywords to differentiate

### Team Testing

| Requirement | Details |
|-------------|---------|
| SHOULD | Have teammates test with natural phrasings |
| SHOULD | Document which phrasings work and which don't |
| MAY | Iterate on descriptions based on activation failures |

---

## Related Documents

- [04-skills.md](./04-skills.md) - Skill structure and SKILL.md format
- [03-agents.md](./03-agents.md) - Agent configuration and delegation
- [02-commands.md](./02-commands.md) - Command templates and arguments
- [17-system-prompts.md](./17-system-prompts.md) - Writing effective system prompts
- [18-context-management.md](./18-context-management.md) - Token budgets and context optimization

---

**Navigation:** [← Previous: Debugging](./15-debugging.md) | [Index](./README.md) | [Next: System Prompts →](./17-system-prompts.md)
