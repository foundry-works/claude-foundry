# 4. Skills

> Model-invoked capabilities that Claude automatically activates based on task context.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Skills are specialized prompt templates that inject domain-specific instructions into Claude's context. Unlike commands (user-triggered) or agents (delegated), skills are **model-invoked**—Claude autonomously decides when to use them based on task semantics and the skill's description.

### Key Characteristics

| Aspect | Description |
|--------|-------------|
| **Trigger** | Automatic, based on task relevance |
| **Loading** | Progressive disclosure (metadata → SKILL.md → reference files) |
| **Scope** | Domain expertise, workflows, specialized knowledge |
| **Token cost** | On-demand; only loaded when relevant |

---

## Progressive Disclosure Architecture

Skills use a three-tier loading system that minimizes token consumption:

```
Tier 1 (Startup):     name + description from frontmatter (all skills)
Tier 2 (Activation):  SKILL.md body (when skill triggered)
Tier 3 (On-demand):   reference.md, examples.md, etc. (when Claude reads them)
```

### Token Impact by Tier

| Tier | When Loaded | Token Cost | Budget Guidance |
|------|-------------|------------|-----------------|
| **Metadata** | Every session startup | Low (~300 tokens per skill) | Description ≤1024 chars |
| **SKILL.md body** | On skill activation | Medium | <500 lines, <2000 chars ideal |
| **Reference files** | When Claude reads them | Variable | No limit; zero cost until read |

### Why This Matters

Reference files consume **zero context tokens** until Claude actually reads them. This means you can bundle extensive documentation, examples, and schemas without impacting performance—Claude loads only what each task requires.

---

## Directory Structure

### Required Structure

```
skills/
└── my-skill/
    └── SKILL.md           # Required: main instructions
```

### Recommended Structure

```
skills/
└── my-skill/
    ├── SKILL.md           # Core workflow (<500 lines)
    ├── reference.md       # Single file for small skills (<200 lines)
    ├── references/        # OR folder for larger skills
    │   ├── workflow.md
    │   ├── troubleshooting.md
    │   └── examples.md
    └── scripts/
        └── validate.py    # Utilities (executed, not loaded)
```

**Choosing between single file and folder:**

| Use `reference.md` | Use `references/` folder |
|--------------------|--------------------------|
| Reference content <200 lines | Reference content >200 lines |
| Sections are tightly coupled | Sections accessed independently |
| Simple skill with few edge cases | Complex skill with distinct topics |

### Domain-Specific Organization

For skills with multiple domains, organize to enable selective loading:

```
bigquery-skill/
├── SKILL.md               # Overview and navigation
└── references/
    ├── finance.md         # Revenue, billing metrics
    ├── sales.md           # Pipeline, opportunities
    ├── product.md         # Usage analytics
    └── marketing.md       # Campaigns, attribution
```

When a user asks about sales metrics, Claude only loads `references/sales.md`—the other files remain unread, consuming zero tokens.

### Section-Based Organization

For skills with large reference documents containing distinct logical sections, split by section rather than domain:

```
my-skill/
├── SKILL.md               # Core workflow with section links
└── references/
    ├── workflow.md        # Step-by-step procedures
    ├── troubleshooting.md # Error handling and fixes
    ├── configuration.md   # Settings and options
    └── best-practices.md  # Do's and don'ts
```

When Claude needs troubleshooting help, it only loads `references/troubleshooting.md`—the workflow and configuration docs remain unread.

**Why this matters for partial loading:**

Using anchor links like `reference.md#troubleshooting` requires loading the entire `reference.md` file to find the section. With separate files, Claude loads only the specific file it needs.

| Pattern | Link Style | Tokens Loaded |
|---------|-----------|---------------|
| Single file with anchors | `reference.md#troubleshooting` | Entire file (~800 lines) |
| Separate files | `references/troubleshooting.md` | Just that section (~50 lines) |

**Granularity guidelines:**
- Minimum file size: ~25-30 lines (combine smaller sections)
- Add TOC if file exceeds 50 lines
- Group related small sections rather than creating many tiny files

---

## SKILL.md Structure

### Required Frontmatter

```yaml
---
name: processing-pdfs
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

### Frontmatter Requirements

| Field | Constraints | Purpose |
|-------|-------------|---------|
| `name` | ≤64 chars, lowercase, hyphens only | Identifier for invocation |
| `description` | ≤1024 chars, non-empty, third-person | Discovery and activation trigger |

| Requirement | Details |
|-------------|---------|
| MUST | Include both `name` and `description` |
| MUST | Use lowercase letters, numbers, and hyphens only in name |
| MUST | Write description in third person |
| MUST NOT | Use reserved words in `name`: "anthropic", "claude" |
| SHOULD | Avoid reserved words in `description` unless part of an official product or plugin name |
| MUST NOT | Include XML tags in name or description |

### Body Content Guidelines

| Requirement | Details |
|-------------|---------|
| SHOULD | Keep body under 500 lines |
| SHOULD | Keep body under 2000 characters for optimal performance |
| MUST | Include only context Claude doesn't already have |
| MUST | Link to reference files for detailed content |
| SHOULD | Provide quick-start examples inline |
| MUST NOT | Include verbose explanations of common concepts |

---

## What Goes Where

### SKILL.md Content (Tier 2)

Include in SKILL.md:

| Content | Why | Example |
|---------|-----|---------|
| **Core workflow** | Essential for task execution | Step-by-step process |
| **Quick-start example** | Immediate utility | Minimal working code |
| **Reference cues** | Guide to detailed docs | "See reference.md#API" |
| **Critical constraints** | Prevent errors | "Always validate before submit" |
| **Output format** | Ensure consistency | Expected response structure |

### Reference Files (Tier 3)

Move to reference.md/examples.md:

| Content | Why | File |
|---------|-----|------|
| **Complete API specs** | Too detailed for core | `reference.md` |
| **Extensive examples** | Token cost | `examples.md` |
| **Edge cases** | Rarely needed | `reference.md` |
| **Historical patterns** | Context-specific | `reference.md` |
| **Troubleshooting** | On-demand | `troubleshooting.md` |

### Conciseness Example

```markdown
# Good: Concise (~50 tokens)
## Extract PDF text
Use pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

# Bad: Verbose (~150 tokens)
PDF (Portable Document Format) files are a common file format
that contains text, images, and other content. To extract text
from a PDF, you'll need to use a library. There are many
libraries available for PDF processing, but we recommend
pdfplumber because it's easy to use...
```

The concise version assumes Claude knows what PDFs are and how libraries work.

---

## Writing Effective Descriptions

The description field is critical for skill discovery. Claude uses it to decide when to activate your skill from potentially 100+ available skills.

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Write in third person |
| MUST | Include what the skill does |
| MUST | Include when to use it (trigger conditions) |
| SHOULD | Include key terms users might mention |
| MUST NOT | Use first person ("I can help...") |
| MUST NOT | Use second person ("You can use this...") |

### Good Examples

```yaml
# PDF Processing
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# Excel Analysis
description: Analyzes Excel spreadsheets, creates pivot tables, generates charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.

# Git Commit Helper
description: Generates descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

### Bad Examples

```yaml
# Too vague
description: Helps with documents

# First person
description: I can help you process Excel files

# Missing trigger conditions
description: Processes data
```

---

## Naming Conventions

### Use Gerund Form (Recommended)

Use verb + -ing to describe the activity:

| Good | Avoid |
|------|-------|
| `processing-pdfs` | `pdf-processor` |
| `analyzing-spreadsheets` | `spreadsheet-analysis` |
| `managing-databases` | `db-manager` |
| `testing-code` | `test-runner` |

### Naming Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Use lowercase letters, numbers, hyphens only |
| MUST NOT | Use spaces or underscores |
| MUST NOT | Use reserved words in names ("anthropic", "claude") |
| SHOULD | Be descriptive and specific |
| SHOULD NOT | Use vague names ("helper", "utils", "tools") |

---

## Reference File Patterns

### Pattern 1: High-Level Guide with References

```markdown
# SKILL.md

## Quick start
[Minimal working example]

## Advanced features
**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [reference.md](reference.md) for all methods
**Examples**: See [examples.md](examples.md) for common patterns
```

### Pattern 2: Conditional Loading

```markdown
# SKILL.md

## When to Load Reference Files

- `reference.md#API-Endpoints` - When working with REST endpoints
- `reference.md#Database-Schema` - When writing queries
- `examples.md#Error-Handling` - When implementing error responses
- Skip reference files for simple tasks
```

### Pattern 3: Domain-Specific Routing

```markdown
# SKILL.md (BigQuery skill)

## Available Datasets

**Finance**: Revenue, ARR, billing → See [references/finance.md](references/finance.md)
**Sales**: Opportunities, pipeline → See [references/sales.md](references/sales.md)
**Product**: API usage, features → See [references/product.md](references/product.md)
```

### Pattern 4: Section-Based References

```markdown
# SKILL.md

## Detailed Reference

For specific topics, see:
- Workflows → `references/workflow.md`
- Troubleshooting → `references/troubleshooting.md`
- Configuration → `references/configuration.md`
- Best practices → `references/best-practices.md`
```

This pattern enables true partial loading—Claude reads only the specific file needed rather than loading a monolithic reference.md and searching for an anchor.

---

## Anti-Patterns

### 1. Deeply Nested References

Claude may partially read files when they're referenced from other referenced files.

```markdown
# Bad: Too deep
SKILL.md → advanced.md → details.md → actual-info.md

# Good: One level deep
SKILL.md → references/workflow.md
SKILL.md → references/troubleshooting.md
SKILL.md → examples.md
```

| Requirement | Details |
|-------------|---------|
| MUST | Keep references one level deep from SKILL.md |
| MUST NOT | Chain reference files (A → B → C) |

### 2. Missing Table of Contents

For reference files over 100 lines, Claude may preview with partial reads.

```markdown
# Good: TOC enables navigation
## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns

## Authentication and setup
...
```

| Requirement | Details |
|-------------|---------|
| SHOULD | Include TOC for files over 100 lines |

### 3. Verbose Explanations

```markdown
# Bad: Explaining common concepts
PDF (Portable Document Format) is a file format developed by Adobe...

# Good: Assume Claude knows
Use pdfplumber for PDF text extraction.
```

| Requirement | Details |
|-------------|---------|
| MUST | Assume Claude knows common concepts |
| MUST | Only add context Claude doesn't have |

### 4. Kitchen Sink SKILL.md

```markdown
# Bad: Everything in one file
---
name: mega-skill
description: Does everything
---
[2000 lines of documentation, examples, schemas, history...]
```

| Requirement | Details |
|-------------|---------|
| MUST | Keep SKILL.md under 500 lines |
| MUST | Use progressive disclosure for detailed content |

### 5. Windows-Style Paths

```markdown
# Bad
scripts\helper.py
references\guide.md

# Good
scripts/helper.py
references/guide.md
```

| Requirement | Details |
|-------------|---------|
| MUST | Use forward slashes in all file paths |

### 6. Sub-Numbered Steps

Complex sub-numbering (3a, 3b, 3.5) reduces scannability. If a step has multiple sub-steps, it likely belongs in a reference file.

```markdown
# Bad: Sub-numbered steps
### Step 3: Create Plan
### Step 3a: Use Templates
### Step 3b: Use Bulk Macro
### Step 3.5: AI Review
### Step 4: Validate

# Good: Flat steps with reference link
### Step 3: Create Plan
Options: templates, bulk macro, AI review
> See `references/plan-authoring.md` for details
### Step 4: Validate
```

| Requirement | Details |
|-------------|---------|
| SHOULD | Use flat step numbering (1, 2, 3, 4, 5) |
| SHOULD | Move sub-step details to reference files |

---

## Scripts and Utilities

### Execution vs Reading

Make clear whether Claude should execute or read scripts:

```markdown
# Execute (most common)
Run `python scripts/validate.py input.pdf` to check form fields.

# Read as reference (for understanding logic)
See `scripts/validate.py` for the validation algorithm.
```

### Benefits of Utility Scripts

| Benefit | Description |
|---------|-------------|
| **Reliability** | Pre-tested, more reliable than generated code |
| **Token efficiency** | Script content not loaded; only output consumes tokens |
| **Consistency** | Same behavior across uses |
| **Speed** | No code generation required |

### Script Guidelines

| Requirement | Details |
|-------------|---------|
| SHOULD | Place scripts in `scripts/` subdirectory |
| SHOULD | Make scripts executable |
| SHOULD | Handle errors explicitly (don't punt to Claude) |
| MUST | Document required dependencies |
| MUST NOT | Use "magic numbers" without explanation |

---

## Testing Skills

### Test with Multiple Models

Skills effectiveness depends on the underlying model:

| Model | Consideration |
|-------|---------------|
| **Haiku** | Does the skill provide enough guidance? |
| **Sonnet** | Is the skill clear and efficient? |
| **Opus** | Does the skill avoid over-explaining? |

| Requirement | Details |
|-------------|---------|
| SHOULD | Test with all models you plan to use |
| SHOULD | Provide more detail if targeting Haiku |
| SHOULD | Reduce verbosity if targeting Opus |

### Evaluation-Driven Development

1. **Identify gaps**: Run Claude on tasks without the skill. Document failures.
2. **Create evaluations**: Build 3+ scenarios that test these gaps.
3. **Establish baseline**: Measure performance without the skill.
4. **Write minimal instructions**: Just enough to pass evaluations.
5. **Iterate**: Execute evaluations, compare, refine.

### Observe Navigation Patterns

Watch how Claude actually uses your skill:

| Observation | Action |
|-------------|--------|
| Unexpected file reading order | Restructure for intuitive navigation |
| Missed references | Make links more explicit |
| Repeated file reads | Move content to SKILL.md |
| Ignored files | Remove or improve signaling |

---

## Degrees of Freedom

Match instruction specificity to task fragility:

### High Freedom (Text-Based Instructions)

Use when multiple approaches are valid:

```markdown
## Code Review Process
1. Analyze code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability
4. Verify adherence to project conventions
```

### Medium Freedom (Pseudocode/Templates)

Use when a preferred pattern exists:

```markdown
## Generate Report
Use this template and customize as needed:
```python
def generate_report(data, format="markdown"):
    # Process data
    # Generate output in specified format
```

### Low Freedom (Specific Scripts)

Use when operations are fragile:

```markdown
## Database Migration
Run exactly this script:
```bash
python scripts/migrate.py --verify --backup
```
Do not modify the command or add additional flags.
```

---

## Workflows and Feedback Loops

### Checklist Pattern

For complex multi-step tasks:

```markdown
## PDF Form Workflow

Copy this checklist:
- [ ] Step 1: Analyze form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate.py)
- [ ] Step 4: Fill form (run fill_form.py)
- [ ] Step 5: Verify output (run verify.py)
```

### Feedback Loop Pattern

Run validator → fix errors → repeat:

```markdown
## Document Editing

1. Make edits to document.xml
2. **Validate immediately**: `python validate.py`
3. If validation fails:
   - Review error message
   - Fix issues
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild document
```

---

## Quick Reference

### File Size Guidelines

| File | Recommendation |
|------|----------------|
| SKILL.md body | <500 lines, <2000 chars |
| Description | ≤1024 chars |
| Reference files | Add TOC if >100 lines |

### Checklist for Effective Skills

**Core Quality:**
- [ ] Description is specific and includes trigger conditions
- [ ] Description written in third person
- [ ] SKILL.md body under 500 lines
- [ ] Additional details in separate reference files
- [ ] References one level deep (no chaining)
- [ ] Consistent terminology throughout
- [ ] Concise examples (no verbose explanations)

**Structure:**
- [ ] Forward slashes in all paths
- [ ] TOC in reference files >100 lines
- [ ] Scripts in `scripts/` subdirectory
- [ ] Clear execute vs read instructions for scripts

**Testing:**
- [ ] Tested with target models (Haiku/Sonnet/Opus)
- [ ] At least 3 evaluation scenarios
- [ ] Observed actual navigation patterns

---

## Related Documents

- [09-file-structure.md](./09-file-structure.md) - Directory layout and naming
- [10-frontmatter.md](./10-frontmatter.md) - Frontmatter schemas
- [16-description-writing.md](./16-description-writing.md) - Writing effective descriptions
- [18-context-management.md](./18-context-management.md) - Token budgets and progressive disclosure

---

**Navigation:** [← Previous: Agents](./03-agents.md) | [Index](./README.md) | [Next: Hooks →](./05-hooks.md)
