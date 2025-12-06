# 18. Context Management

> Context is a finite attention budget. Effective context management means finding the smallest set of high-signal tokens that maximize the likelihood of your desired outcome.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Every token in Claude's context window competes for attention. Research confirms that as token count increases, Claude's ability to accurately recall and follow instructions decreases—a phenomenon called **context rot**.

### The Challenge

- Claude Code's system prompt already contains ~50 instructions
- Every CLAUDE.md, active skill, and conversation message adds more
- Frontier LLMs reliably follow ~150-200 instructions; beyond that, quality drops uniformly
- More context ≠ better results

### The Goal

Find the **smallest possible set of high-signal tokens** that enable Claude to perform your task effectively.

---

## Context Hierarchy

Understanding when different content loads helps you budget tokens effectively:

| Layer | When Loaded | Token Impact | Budget Guidance |
|-------|-------------|--------------|-----------------|
| **CLAUDE.md** | Every turn | High | <100 lines ideal, <60 better |
| **Skill metadata** | Startup (all skills) | Low | <1024 chars per description |
| **SKILL.md body** | On activation | Medium | <2000 chars |
| **Supporting files** | On-demand | Variable | Load selectively |
| **Agent prompt** | On delegation | Medium-High | Summarize output |
| **Conversation history** | Every turn | Grows over time | Use `/clear` frequently |

### Always-On Content

Content that loads every turn has the highest token cost:

```
Every Turn:
├── Claude Code system prompt (~50 instructions)
├── CLAUDE.md (all levels merged)
├── Skill metadata (names + descriptions)
└── Conversation history
```

---

## Token Budget Principles

### 1. CLAUDE.md Is Prime Real Estate

Your CLAUDE.md file loads on **every single turn**. Guard it carefully:

| Requirement | Details |
|-------------|---------|
| SHOULD | Keep under 100 lines total |
| SHOULD | Aim for 60 lines or fewer |
| MUST NOT | Include task-specific instructions |
| MUST NOT | Duplicate content available elsewhere |
| MUST NOT | Include code snippets (use file:line references) |

### 2. Instruction Count Matters

LLMs bias toward instructions at the peripheries of the prompt. As instruction count increases, following quality decreases uniformly across all instructions.

```
Instructions │ Reliability
─────────────┼────────────
     50      │  Excellent
    100      │  Good
    150      │  Acceptable
    200      │  Degraded
    250+     │  Unreliable
```

Claude Code starts with ~50 instructions. Your CLAUDE.md and skills share the remaining budget.

### 3. Progressive Disclosure Saves Tokens

Don't load everything upfront. Use a tiered approach:

```
Tier 1 (Startup):     name + description only
Tier 2 (Activation):  SKILL.md core content
Tier 3 (On-demand):   reference.md, examples.md, etc.
```

---

## What to Include vs. Exclude

### Include in CLAUDE.md

| Content | Why | Example |
|---------|-----|---------|
| **Critical constraints** | Prevents costly mistakes | "Never push to main directly" |
| **Stack overview** | Enables understanding | "TypeScript + React + PostgreSQL" |
| **Build/test commands** | Frequently needed | "`npm run test` for tests" |
| **Architecture summary** | Guides navigation | "API in /api, UI in /web" |
| **Non-obvious conventions** | Prevents confusion | "Use snake_case for DB columns" |

### Exclude from CLAUDE.md

| Content | Why | Alternative |
|---------|-----|-------------|
| **Code snippets** | Become stale | Use `file:line` references |
| **Complete schemas** | Token bloat | Link to schema file |
| **Style guidelines** | Linters handle this | `.eslintrc`, `.prettierrc` |
| **Task-specific rules** | Not universal | Put in skills |
| **Database contents** | Dynamic | Query when needed |
| **Verbose examples** | Wastes context | Separate examples file |
| **Auto-generated content** | Often low-signal | Write manually |

### Include in SKILL.md

| Content | Why |
|---------|-----|
| **Core workflow** | Guides execution |
| **Trigger conditions** | Enables activation |
| **Output format** | Ensures consistency |
| **Reference cues** | Points to detailed docs |
| **Key constraints** | Prevents errors |

### Exclude from SKILL.md

| Content | Why | Alternative |
|---------|-----|-------------|
| **Complete specifications** | Too long | `reference.md` |
| **Extensive examples** | Token cost | `examples.md` |
| **Implementation details** | Rarely needed | Load on-demand |
| **Version history** | Low priority | End of file or separate |

---

## Progressive Disclosure

### Three-Tier Pattern

```
my-skill/
├── SKILL.md           # Tier 2: Core workflow (~2k chars)
├── reference.md       # Tier 3: Detailed specs (load on-demand)
├── examples.md        # Tier 3: Use cases (load on-demand)
└── scripts/           # Tier 3: Utilities (execute when needed)
```

### Reference Cues

Teach Claude when to load additional files:

```markdown
# SKILL.md

## Reference Cues

- Need severity definitions? Read `reference.md#Severity Matrix`
- Need sample report format? Read `examples.md#Report Template`
- Need validation logic? Run `scripts/validate.sh`
```

### Conditional Loading

Guide Claude to load only what's needed:

```markdown
## When to Load Reference Files

- `reference.md#API Endpoints` - When working with REST endpoints
- `reference.md#Database Schema` - When writing queries
- `examples.md#Error Handling` - When implementing error responses
- Skip reference files for simple tasks
```

---

## CLAUDE.md Best Practices

### The WHY-WHAT-HOW Framework

Structure your CLAUDE.md around three questions:

```markdown
# Project Name

## WHY (Purpose)
Brief description of what this project does and why.

## WHAT (Stack)
- Language: TypeScript
- Framework: Next.js 14
- Database: PostgreSQL with Prisma
- Testing: Jest + Playwright

## HOW (Commands)
- `npm run dev` - Start development server
- `npm run test` - Run unit tests
- `npm run lint` - Check code style
```

### Effective CLAUDE.md Example

```markdown
# E-commerce Platform

## Stack
TypeScript, Next.js 14, PostgreSQL, Prisma, Stripe

## Structure
- `/app` - Next.js app router pages
- `/lib` - Shared utilities and API clients
- `/prisma` - Database schema and migrations

## Commands
- `npm run dev` - Development server (port 3000)
- `npm run test` - Jest tests
- `npm run db:migrate` - Run migrations
- `npm run db:seed` - Seed test data

## Conventions
- Use Prisma for all DB operations
- API routes return `{ data, error }` shape
- Use `zod` for request validation
- Feature branches: `feat/ticket-number-description`

## Constraints
- Never commit .env files
- Always run tests before PR
- Use transactions for multi-table updates
```

This example is ~40 lines and provides essential context without bloat.

### What NOT to Put in CLAUDE.md

```markdown
# Bad: Style guide (use linter)
- Use 2-space indentation
- Semicolons required
- Single quotes for strings

# Bad: Code snippets (become stale)
Example API call:
```typescript
const response = await fetch('/api/users', {
  method: 'POST',
  // ... 20 lines of example code
});
```

# Bad: Complete schema (too long)
Database Tables:
- users: id, email, name, created_at, updated_at, ...
- orders: id, user_id, status, total, ...
[continues for 200 lines]

# Bad: Task-specific rules (put in skills)
When generating reports:
1. Always include headers
2. Format dates as ISO 8601
3. [continues with specific rules]
```

---

## Agent Output Management

### Summarize Before Returning

Agents return results to a parent conversation. Large outputs consume parent context:

```markdown
## Output Format

Return a concise summary:

**Result**: [1-2 sentences]

**Key Findings**:
- [Finding 1]
- [Finding 2]

**Details**: Available in [file] if needed

Do NOT return:
- Full file contents
- Complete logs
- Raw command output
```

### Lead with Summary

Structure agent output for quick parsing:

```markdown
# Good: Summary first
**Status**: 3 tests fixed, 1 requires manual review

Fixed:
- test_auth.py:45 - Missing mock
- test_api.py:102 - Async timing
- test_db.py:78 - Connection cleanup

Needs Review:
- test_integration.py:200 - Flaky, may need redesign

# Bad: Details first
Looking at test_auth.py, I found that line 45 was missing a mock for the
authentication service. The test was attempting to call the real service
which caused a timeout. I fixed this by adding...
[500 words later]
...In summary, I fixed 3 tests.
```

### Optional Expansion

Provide details on request, not by default:

```markdown
**Summary**: Refactored authentication module (4 files changed)

**Changes**:
- auth.ts: Extracted token validation
- middleware.ts: Added rate limiting
- tests/auth.test.ts: Added edge cases
- types.ts: New TokenPayload interface

<details>
<summary>Full diff available</summary>
[Detailed changes here - only loaded if expanded]
</details>
```

---

## Session Hygiene

### Use `/clear` Frequently

Long sessions accumulate irrelevant context:

| Requirement | Details |
|-------------|---------|
| SHOULD | Clear between unrelated tasks |
| SHOULD | Clear after completing major features |
| SHOULD | Clear when Claude seems confused |
| MAY | Clear after ~10-15 substantive exchanges |

### Context Compaction

For extended tasks across context limits:

```markdown
# In CLAUDE.md or skill prompt
Your context window may be compacted as it approaches limits.
When resuming work:
1. Read filesystem state (don't rely on memory)
2. Check git status for recent changes
3. Review any TODO.md or PROGRESS.md files
```

### Structured State Tracking

For long-running tasks, externalize state:

```markdown
## State Management

- Track test results in `test-status.json`
- Track progress in `PROGRESS.md`
- Use git commits as checkpoints
- Start fresh windows by reading state files
```

---

## Anti-Patterns

### 1. Stuffed CLAUDE.md

```markdown
# Bad: 500+ lines of everything
[Complete API documentation]
[Full database schema]
[All coding conventions]
[Example code for every pattern]
[Team member responsibilities]
```

**Fix:** Keep only universally-applicable essentials. Move details to skills and reference files.

### 2. Duplicate Information

```markdown
# Bad: Same info in multiple places
CLAUDE.md: "Run tests with npm test"
skill/testing/SKILL.md: "Run tests with npm test"
README.md: "Run tests with npm test"
```

**Fix:** Single source of truth. Reference, don't duplicate.

### 3. Auto-Generated Content

```markdown
# Bad: Using /init or auto-generation
[Generic boilerplate that doesn't match your project]
[Verbose output from documentation generators]
```

**Fix:** Write CLAUDE.md manually. Every line should earn its place.

### 4. Static Code Snippets

```markdown
# Bad: Code that will become stale
The auth middleware looks like this:
```typescript
export function authMiddleware(req, res, next) {
  // Code from 6 months ago that's been refactored 3 times
}
```

**Fix:** Use `file:line` references: "See auth middleware at `src/middleware/auth.ts:15`"

### 5. Kitchen Sink Skills

```markdown
# Bad: SKILL.md with entire manual
---
name: mega-skill
description: Does everything
---

[2000 lines of documentation, examples, schemas, history...]
```

**Fix:** Keep SKILL.md under 2000 chars. Use progressive disclosure.

---

## Measurement

### Estimating Token Usage

Rough guidelines for token estimation:

| Content | Approximate Tokens |
|---------|-------------------|
| 100 lines of markdown | ~800-1200 tokens |
| 1000 characters | ~250-400 tokens |
| Average CLAUDE.md (60 lines) | ~500-700 tokens |
| Skill description (1024 chars) | ~300 tokens |
| SKILL.md body (2000 chars) | ~500-600 tokens |

### Warning Signs

Your context may be bloated if:

- Claude forgets instructions from earlier in conversation
- Claude asks questions already answered
- Claude ignores specific constraints
- Response quality degrades over session
- Claude summarizes rather than acting

---

## Related Documents

- [16-description-writing.md](./16-description-writing.md) - Writing effective descriptions
- [17-system-prompts.md](./17-system-prompts.md) - System prompt design
- [12-claude-md.md](./12-claude-md.md) - CLAUDE.md configuration and hierarchy
- [04-skills.md](./04-skills.md) - Skill structure and progressive disclosure

---

**Navigation:** [← Previous: System Prompts](./17-system-prompts.md) | [Index](./README.md)
