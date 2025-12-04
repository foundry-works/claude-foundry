# 3. Skills

> Model-invoked capabilities that Claude automatically activates based on task context and skill descriptions.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Skills are directory-based capabilities that Claude autonomously discovers and uses. Unlike commands (user-invoked) or agents (task-delegated), skills are:

- **Automatically activated** based on context matching
- **Directory-structured** with a required SKILL.md and optional supporting files
- **Progressive** - Claude reads supporting files only when needed
- **Portable** - Reusable knowledge that any agent can apply

---

## Directory Structure

### Skill Organization

Each skill is a directory containing:

```
skill-name/
├── SKILL.md           # Required: frontmatter + core instructions
├── reference.md       # Optional: detailed technical documentation
├── examples.md        # Optional: concrete use cases
├── scripts/           # Optional: utility scripts
└── templates/         # Optional: reusable templates
```

### Storage Locations

| Scope | Location | Visibility |
|-------|----------|------------|
| Personal | `~/.claude/skills/skill-name/` | Private to user |
| Project | `.claude/skills/skill-name/` | Team-shared via git |
| Plugin | `skills/skill-name/` in plugin | Distributed with plugin |

---

## SKILL.md Format

### Frontmatter Schema

```yaml
---
name: pdf-extractor
description: Extract text and tables from PDF files. Use when working with PDF files, forms, or scanned documents.
allowed-tools: Read, Bash
---
```

### Field Reference

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `name` | string | MUST | Lowercase, numbers, hyphens only. Max 64 chars |
| `description` | string | MUST | What + when to use. Max 1024 chars |
| `allowed-tools` | array | MAY | Tool whitelist without permission prompts |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include `name` with valid characters |
| MUST | Include `description` with clear trigger context |
| SHOULD | Specify `allowed-tools` to restrict capabilities |
| SHOULD | Keep SKILL.md focused; use supporting files for details |

---

## Description Design

The description is critical - it determines when Claude activates the skill.

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Explain what the skill does |
| MUST | Explain when/why to use it |
| SHOULD | Name specific file types, formats, or tools |
| SHOULD | Include trigger contexts |

### Examples

**Bad: Vague description**
```yaml
description: Helps with documents
```

**Good: Specific with trigger context**
```yaml
description: Extract text and tables from PDF files. Use when working with PDF files, forms, or scanned documents.
```

**Good: Clear capability and context**
```yaml
description: Analyze Python code for security vulnerabilities including injection, auth bypass, and data exposure. Use when reviewing Python code for security issues.
```

---

## Skill Activation

### How Claude Selects Skills

1. **Discovery** - Claude detects skills from all locations
2. **Analysis** - Descriptions are matched against current task
3. **Activation** - Matching skills are automatically used
4. **Execution** - Skill content injected into context

### Automatic vs Explicit

| Type | Mechanism |
|------|-----------|
| Automatic | Claude matches description to task context |
| Explicit | User can reference skill by name if needed |

### No Registration Required

Skills are discovered automatically - no manifest declaration needed.

---

## Progressive Disclosure

Claude reads skill files progressively to manage context:

| File | When Read |
|------|-----------|
| SKILL.md | Always (on activation) |
| reference.md | When detailed docs needed |
| examples.md | When examples requested |
| scripts/* | When execution needed |

### Best Practice

| Requirement | Details |
|-------------|---------|
| SHOULD | Keep SKILL.md focused on high-level approach |
| SHOULD | Move detailed specs to reference.md |
| SHOULD | Include examples.md for concrete use cases |
| MUST NOT | Put entire technical guides in SKILL.md |

---

## Plugin Skills

### Directory Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    ├── skill-one/
    │   ├── SKILL.md
    │   └── reference.md
    └── skill-two/
        └── SKILL.md
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Place skills/ at plugin root (not inside .claude-plugin/) |
| MUST | Each skill in its own subdirectory |
| MAY | Use namespace prefix in description for clarity |

### Namespace Convention

Plugin skills often use `plugin-name:skill-name` pattern:
- `sdd-toolkit:doc-query`
- `model-chorus:consensus`

---

## Design Patterns

### Single Responsibility

Each skill should do one thing well:

**Bad: Too broad**
```yaml
name: document-processor
description: Process all types of documents
```

**Good: Focused**
```yaml
name: pdf-extractor
description: Extract text and tables from PDF files
```

```yaml
name: excel-analyzer
description: Analyze Excel spreadsheets and extract data
```

### Conflict Resolution

When multiple similar skills exist, use distinct triggers:

```yaml
# Skill 1
description: Extract sales data from Excel files. Use for sales reports and revenue analysis.

# Skill 2
description: Parse web analytics JSON logs. Use for traffic and conversion metrics.
```

### Context Management

Structure for progressive disclosure:

```
security-scanner/
├── SKILL.md           # Activation rules, high-level workflow
├── reference.md       # Vulnerability types, severity levels
├── examples.md        # Sample scan outputs
└── scripts/
    └── run-scan.sh    # Utility script
```

---

## Examples

### PDF Extraction Skill

```
pdf-extractor/
├── SKILL.md
└── examples.md
```

**SKILL.md:**
```yaml
---
name: pdf-extractor
description: Extract text, tables, and metadata from PDF files. Use when working with PDF documents, forms, or scanned reports.
allowed-tools: Read, Bash
---

# PDF Extraction

Extract content from PDF files using appropriate tools based on PDF type:

1. **Text-based PDFs**: Use pdftotext for direct extraction
2. **Scanned PDFs**: Use OCR tools for image-based content
3. **Forms**: Extract field data and structure

## Workflow

1. Identify PDF type (text vs scanned)
2. Select appropriate extraction method
3. Structure output in requested format
4. Validate extraction quality

## Output Formats

- Plain text
- Markdown with tables
- JSON for structured data
```

### Code Review Skill

```
security-review/
├── SKILL.md
├── reference.md
└── examples.md
```

**SKILL.md:**
```yaml
---
name: security-review
description: Review code for security vulnerabilities including injection, authentication bypass, and data exposure. Use when auditing code for security issues.
allowed-tools: Read, Grep, Glob
---

# Security Code Review

Systematic security analysis of code changes.

## Focus Areas

1. **Injection** - SQL, command, LDAP injection
2. **Authentication** - Bypass, weak credentials
3. **Authorization** - Privilege escalation
4. **Data Exposure** - Sensitive data leaks
5. **Cryptography** - Weak algorithms, key management

## Workflow

1. Identify security-sensitive code paths
2. Check for common vulnerability patterns
3. Review input validation and sanitization
4. Assess authentication and authorization logic
5. Report findings with severity ratings

## Output

- Critical (immediate fix required)
- High (fix before release)
- Medium (fix in next sprint)
- Low (best practice improvement)
```

---

## Anti-Patterns

### Don't: Vague Descriptions

```yaml
# Bad: Claude can't determine when to use
description: Helps with things
```

```yaml
# Good: Clear capability and trigger
description: Extract customer data from Salesforce reports. Use when processing CRM export files.
```

### Don't: Combine Unrelated Capabilities

```yaml
# Bad: Too many responsibilities
name: utility-skill
description: Process documents, analyze images, and manage databases
```

```yaml
# Good: Separate focused skills
name: doc-processor
description: Process text documents and extract structured content
```

### Don't: Conflicting Descriptions

```yaml
# Bad: Two skills with identical triggers
# Skill 1
description: Analyze data files

# Skill 2
description: Analyze data files
```

```yaml
# Good: Distinct triggers
# Skill 1
description: Analyze CSV sales data files for revenue patterns

# Skill 2
description: Analyze JSON log files for error patterns
```

### Don't: Overload SKILL.md

```yaml
# Bad: 5000 characters of documentation in SKILL.md
---
name: complex-skill
description: ...
---
[Entire technical manual here]
```

```yaml
# Good: Focused SKILL.md with supporting files
---
name: complex-skill
description: ...
---
Core workflow and activation rules here.

See reference.md for detailed specifications.
See examples.md for usage examples.
```

---

## Skills vs Commands vs Agents

| Aspect | Skills | Commands | Agents |
|--------|--------|----------|--------|
| Invocation | Automatic (context-based) | Explicit (`/command`) | Delegated (Task tool) |
| Structure | Directory + SKILL.md | Single .md file | Agent .md file |
| Discovery | Automatic | User-initiated | User or Claude request |
| Best for | Reusable knowledge | Quick shortcuts | Complex workflows |
| Context | Progressive disclosure | Single prompt | Isolated execution |

### When to Use Skills

- Claude should discover capability automatically
- Knowledge is reusable across conversations
- Multiple files or scripts needed
- Team needs standardized guidance

### When to Use Commands

- User-triggered shortcuts
- Simple prompt templates
- Explicit control required

### When to Use Agents

- Autonomous task execution
- Isolated context needed
- Long-running workflows

---

## Performance Considerations

| Requirement | Details |
|-------------|---------|
| SHOULD | Keep SKILL.md under 2000 characters |
| SHOULD | Use supporting files for detailed content |
| SHOULD | Cache heavy computation in templates/ |
| MAY | Use scripts/ for reusable utilities |

---

## Related Documents

- [01-commands.md](./01-commands.md) - User-invoked slash commands
- [02-agents.md](./02-agents.md) - Autonomous task handlers
- [06-plugin-manifest.md](./06-plugin-manifest.md) - Plugin configuration
- [08-file-structure.md](./08-file-structure.md) - Directory conventions

---

**Navigation:** [← Previous: Agents](./02-agents.md) | [Index](./README.md) | [Next: Hooks →](./04-hooks.md)
