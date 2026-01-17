# 17. System Prompt Design

> System prompts define how Claude behaves once a component is activated. They transform Claude from a general assistant into a specialized tool for your specific workflow.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

While descriptions tell Claude **when** to use a component, system prompts tell Claude **how** to behave once activated. System prompts appear in:

| Component | System Prompt Location | When Injected |
|-----------|------------------------|---------------|
| **Skills** | SKILL.md body (after frontmatter) | On skill activation |
| **Agents** | Agent .md body | On agent delegation |
| **Commands** | Command .md body | On command invocation |
| **CLAUDE.md** | File contents | Every conversation turn |

A well-designed system prompt provides clear guidance without being brittle or overwhelming.

---

## The "Right Altitude" Principle

The most effective system prompts balance two extremes:

| Extreme | Problem | Example |
|---------|---------|---------|
| **Too Specific** | Brittle logic that breaks on edge cases | "If file contains exactly 3 functions, do X; if 4 functions, do Y" |
| **Too Vague** | Assumes shared understanding Claude doesn't have | "Be helpful with code" |

**Right altitude:** Specific enough to guide behavior effectively, flexible enough to provide strong heuristics.

### Finding the Right Altitude

```markdown
# Too vague
Review the code and provide feedback.

# Too specific (brittle)
1. Count lines of code
2. If > 500 lines, split into smaller files
3. If function names contain "get", suggest renaming to "fetch"
4. Check for exactly 4 spaces of indentation

# Right altitude
Review code for maintainability issues:
- Functions over 50 lines that could be split
- Naming inconsistencies within the codebase
- Missing error handling on external calls

Focus on high-impact suggestions. Skip style nitpicks covered by linters.
```

---

## Structural Patterns

### Standard Sections

Organize system prompts with consistent sections:

| Section | Purpose | Required |
|---------|---------|----------|
| **Role** | Persona and expertise scope | SHOULD |
| **Responsibilities** | What Claude must do | MUST |
| **Workflow** | Step-by-step approach | SHOULD |
| **Constraints** | What Claude must NOT do | SHOULD |
| **Output Format** | Expected deliverable structure | SHOULD |
| **Reference Cues** | When to load supporting files | MAY |

### Template

```markdown
# Role

You are a [persona] specializing in [domain]. Your scope is [boundaries].

## Responsibilities

1. [Primary responsibility]
2. [Secondary responsibility]
3. [Tertiary responsibility]

## Workflow

1. **Analyze**: [First step]
2. **Plan**: [Second step]
3. **Execute**: [Third step]
4. **Verify**: [Validation step]

## Constraints

- Do NOT [prohibited action]
- Do NOT [another prohibition]
- If [edge case], then [alternative approach]

## Output Format

- **Summary**: 2-3 sentences
- **Findings**: Table with columns [A, B, C]
- **Recommendations**: Prioritized bullet list

## Reference Cues

- Need [detailed specs]? Read `reference.md#Section`
- Need [examples]? Read `examples.md`
```

---

## Clarity and Specificity

### Use Imperative Mood

State instructions as direct commands:

```markdown
# Weak
You might want to check for security issues.

# Strong
Check for security vulnerabilities in all user input handlers.
```

### Tell Claude What TO Do

Frame instructions positively rather than as prohibitions:

```markdown
# Negative framing (harder to follow)
Don't write long responses.
Don't include unnecessary details.
Don't skip the summary.

# Positive framing (clearer)
Keep responses under 500 words.
Include only details relevant to the user's question.
Always start with a 2-sentence summary.
```

### Explain Why

Context helps Claude generalize rules correctly:

```markdown
# Without why (Claude may misapply)
Never use ellipses in output.

# With why (Claude understands the constraint)
Never use ellipses in output because responses are read aloud by a text-to-speech engine, and ellipses cause awkward pauses.
```

### Be Explicit About Actions

Claude 4.x models follow instructions precisely. If you want Claude to implement changes (not just suggest), say so:

```markdown
# Ambiguous (may just suggest)
Review this function for improvements.

# Explicit (will implement)
Refactor this function to improve readability. Make the changes directly.
```

### Define Edge Cases

Anticipate ambiguous situations:

```markdown
## Edge Cases

- If the file doesn't exist, create it with a minimal template
- If multiple valid approaches exist, present options with trade-offs
- If the task requires information not available, ask before guessing
```

---

## Component-Specific Guidance

### Skills (SKILL.md)

| Requirement | Details |
|-------------|---------|
| SHOULD | Keep SKILL.md under 2000 characters |
| MUST | Treat SKILL.md as a "business card"—high-level only |
| SHOULD | Include reference cues for detailed documentation |
| SHOULD | Define explicit triggers for activation |

**Pattern: Business Card + Reference Cues**

```markdown
---
name: api-migration
description: Migrate REST APIs to GraphQL. Use for endpoint conversion or schema design.
---

# API Migration

Convert REST endpoints to GraphQL schemas and resolvers.

## Triggers

Activate when:
- User mentions REST to GraphQL migration
- Working with `.graphql` or resolver files
- API modernization discussions

## Workflow

1. Analyze existing REST endpoints
2. Design GraphQL schema
3. Generate resolver templates
4. Map REST responses to GraphQL types

## Reference Cues

- Need schema patterns? Read `reference.md#Schema Patterns`
- Need resolver examples? Read `examples.md#Resolvers`
```

### Agents

| Requirement | Details |
|-------------|---------|
| MUST | Define task completion criteria |
| MUST | Specify output format for orchestrator |
| SHOULD | Provide tool usage guidance |
| SHOULD | Acknowledge stateless execution |

**Agent prompts return results to a parent conversation. Structure output accordingly:**

```markdown
---
name: test-investigator
description: Investigate test failures through systematic analysis
tools: Read, Grep, Bash
---

# Test Failure Investigation

You are investigating why tests are failing. You operate as a stateless agent—you won't remember previous runs.

## Responsibilities

1. Identify the failing test(s) from provided output
2. Trace the failure to root cause
3. Propose specific fixes

## Output Format

Return a structured report:

**Summary**: [1-2 sentences on root cause]

**Failing Tests**:
| Test | Error | Root Cause |
|------|-------|------------|

**Recommended Fixes**:
1. [File: change description]
2. [File: change description]

**Confidence**: High/Medium/Low

## Tool Guidance

- Use `Grep` to search for error patterns
- Use `Read` to examine test files and source
- Use `Bash` sparingly for test isolation runs

## Constraints

- Do NOT modify files—report findings only
- If root cause is unclear, state uncertainty explicitly
```

### Commands

| Requirement | Details |
|-------------|---------|
| SHOULD | Document argument handling |
| SHOULD | Include file reference patterns |
| MAY | Use `$ARGUMENTS` for parameterization |

**Command prompts are user-triggered templates:**

```markdown
---
name: fix-issue
description: Analyze and fix a GitHub issue
argument-hint: <issue-number>
---

# Fix GitHub Issue

Analyze GitHub issue #$ARGUMENTS and implement a fix.

## Workflow

1. Fetch issue details using `gh issue view $ARGUMENTS`
2. Understand the reported problem
3. Locate relevant code
4. Implement the fix
5. Write tests for the fix
6. Prepare commit with issue reference

## Constraints

- Do NOT push without user confirmation
- Reference issue number in commit message
```

---

## Subagent Integration Patterns

System prompts can guide Claude to leverage built-in subagents for efficient execution. This is especially valuable for skills and agents that need to explore codebases before taking action.

### Built-in Subagents

| Subagent | Model | Tools | Purpose |
|----------|-------|-------|---------|
| **Explore** | Haiku | Glob, Grep, Read, Bash (read-only) | Fast codebase search |
| **Plan** | Sonnet | Read, Glob, Grep, Bash (read-only) | Research in plan mode |
| **general-purpose** | Sonnet | All tools | Complex multi-step tasks |

### Exploration-First Workflow

For tasks requiring codebase discovery, structure the workflow to use Explore first:

```markdown
## Workflow

1. **Explore**: Use the Explore subagent to find relevant files
   - Thoroughness: medium (balance speed and coverage)
   - Focus: [specific patterns or file types]

2. **Analyze**: Review discovered files in detail
3. **Report**: Summarize findings
```

### When to Recommend Subagents

| Scenario | Recommendation |
|----------|---------------|
| Finding all files matching patterns | Explore (quick) |
| Understanding unfamiliar codebase areas | Explore (very thorough) |
| Complex refactoring research | general-purpose |
| Initial investigation before changes | Explore (medium) |

### Example: Code Review with Subagent

```markdown
---
name: code-reviewer
description: Review code for quality, security, and maintainability
tools: Read, Grep, Glob
---

# Code Review

## Responsibilities
Review code for quality, security, and maintainability.

## Workflow

Before detailed review, use the Explore subagent to:
- Find related test files for the changed code
- Identify similar implementations for consistency check
- Locate documentation that may need updates

Then conduct detailed review of discovered context.

## Output Format
- **Summary**: 2-3 sentences
- **Issues**: Table with Severity | Issue | Location
- **Recommendations**: Prioritized list
```

### Subagent Guidance in Prompts

| Requirement | Details |
|-------------|---------|
| SHOULD | Specify which subagent to use for discovery tasks |
| SHOULD | Include thoroughness level for Explore |
| SHOULD | Explain why subagent delegation benefits the task |
| MAY | Suggest parallel Explore agents for large codebases |

---

## Claude 4.x Considerations

### Extended Thinking

For complex tasks, prompt Claude to think before acting:

```markdown
## Approach

Before implementing:
1. Analyze the problem scope
2. Consider 2-3 possible approaches
3. Select the approach with best trade-offs
4. Then proceed with implementation
```

### Thinking Triggers

Claude 4.x responds to thinking cues:

| Cue | Effect |
|-----|--------|
| "think" | Standard reflection |
| "think step by step" | Structured reasoning |
| "think hard" | Deeper analysis |
| "think harder" | Extended deliberation |

```markdown
# For complex architectural decisions
Before proposing a solution, think hard about:
- Existing patterns in this codebase
- Trade-offs between approaches
- Long-term maintenance implications
```

### Reflection After Tool Use

Prompt Claude to evaluate results:

```markdown
After receiving tool results, reflect on:
- Did the results answer the question?
- Is additional investigation needed?
- Are there any unexpected findings to address?
```

### Format Steering

Guide Claude's output style:

```markdown
# For technical documentation
Write in clear, technical prose. Use:
- Short paragraphs (3-4 sentences max)
- Code blocks for all code references
- Tables for comparisons
- Bullet lists for requirements

# For conversational responses
Keep responses concise and direct. Avoid:
- Excessive hedging language
- Unnecessary preambles
- Self-referential statements
```

---

## Anti-Patterns

### 1. Undersized Prompts

```markdown
# Bad: No guidance
Analyze the code.

# Good: Clear responsibilities and output
Analyze the code for security vulnerabilities.

Focus on:
- SQL injection
- Command injection
- Authentication bypass

Output: Table with Severity | Issue | Location | Recommendation
```

### 2. Overstuffed Prompts

```markdown
# Bad: Entire manual pasted in (thousands of tokens)
[500 lines of detailed specifications, examples, edge cases, history...]

# Good: Core guidance with reference cues
Key workflow and constraints here.

For detailed severity definitions, see `reference.md#Severity Matrix`.
For example reports, see `examples.md`.
```

### 3. Missing Constraints

```markdown
# Bad: No boundaries
Help with database operations.

# Good: Clear boundaries
Execute read-only database queries for analysis.

Constraints:
- Do NOT run UPDATE, DELETE, or DROP statements
- Do NOT access production databases
- Limit query results to 1000 rows
```

### 4. Unclear Output Expectations

```markdown
# Bad: What format? What content?
Analyze and report findings.

# Good: Explicit format
Report findings as:

**Summary**: [2-3 sentences]

**Issues Found**:
| Severity | Issue | File:Line | Fix |
|----------|-------|-----------|-----|

**Recommended Actions**: [Prioritized list]
```

### 5. Mixed Perspectives

```markdown
# Bad: Confusing I/you/Claude mixing
I will analyze the code. You should look for bugs. Claude then reports...

# Good: Consistent perspective
Analyze the code for bugs. Report findings with severity ratings.
```

### 6. No Decision Framework

```markdown
# Bad: How to choose?
Handle errors appropriately.

# Good: Decision matrix
Handle errors based on severity:

| Error Type | Action |
|------------|--------|
| Validation | Return user-friendly message |
| Auth | Log and redirect to login |
| System | Log full stack trace, return generic message |
| Network | Retry 3 times, then fail gracefully |
```

---

## Prompt Refinement

### Iterative Improvement

System prompts should be treated like code—refined based on actual behavior:

| Requirement | Details |
|-------------|---------|
| SHOULD | Test prompts with representative tasks |
| SHOULD | Note where Claude misinterprets instructions |
| SHOULD | Add clarification for failure cases |
| MAY | Use Anthropic's prompt improver for optimization |

### Emphasis Keywords

When critical instructions are being ignored:

```markdown
# Standard
Always include test cases.

# Emphasized (use sparingly)
IMPORTANT: Always include test cases.

# Strong emphasis
YOU MUST include test cases for every function.
```

> **Warning:** Overusing emphasis reduces its effectiveness. Reserve for truly critical instructions.

---

## Related Documents

- [16-description-writing.md](./16-description-writing.md) - Writing effective descriptions
- [18-context-management.md](./18-context-management.md) - Token budgets and context optimization
- [04-skills.md](./04-skills.md) - Skill structure and SKILL.md format
- [03-agents.md](./03-agents.md) - Agent configuration and delegation
- [12-claude-md.md](./12-claude-md.md) - Project-wide instructions

---

**Navigation:** [← Previous: Description Writing](./16-description-writing.md) | [Index](./README.md) | [Next: Context Management →](./18-context-management.md)
