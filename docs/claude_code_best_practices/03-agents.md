# 3. Subagents

> Autonomous task handlers that execute complex workflows with isolated context and specialized capabilities.

This document uses RFC 2119 terminology. See [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) for definitions of MUST, SHOULD, and MAY.

---

## Overview

Subagents are specialized agents that Claude invokes to handle complex, multi-step tasks. They operate with:

- **Isolated context** - Separate context window prevents pollution of main conversation
- **Focused capabilities** - Restricted tools improve security and performance
- **Stateless execution** - Each invocation is independent (resumable via agentId)
- **Automatic or explicit invocation** - Claude delegates based on task match or user request

---

## File Location and Naming

### Location

| Scope | Location | Precedence | Visibility |
|-------|----------|------------|------------|
| Project | `.claude/agents/` | Highest | Team-shared via git |
| User | `~/.claude/agents/` | Lower | Private to user |
| Plugin | Declared in `plugin.json` | Via namespace | Distributed with plugin |

### Naming Conventions

| Requirement | Details |
|-------------|---------|
| MUST | Use lowercase letters and hyphens only |
| MUST | Use `.md` extension |
| MUST NOT | Use underscores, spaces, or uppercase |

**Examples:**
```
.claude/agents/
├── code-reviewer.md      ✓ correct
├── test-runner.md        ✓ correct
├── sql-expert.md         ✓ correct
├── TestRunner.md         ✗ wrong (uppercase)
└── test_runner.md        ✗ wrong (underscore)
```

---

## Frontmatter Schema

Agents use YAML frontmatter followed by a system prompt:

```yaml
---
name: code-reviewer
description: Reviews code for security vulnerabilities and performance issues
tools: Read, Grep, Glob
model: sonnet
---

You are an expert code reviewer specializing in security and performance...
```

### Field Reference

| Field | Type | Required | Purpose |
|-------|------|----------|---------|
| `name` | string | MUST | Unique identifier (lowercase, hyphens) |
| `description` | string | MUST | When this agent should be invoked |
| `tools` | string | SHOULD | Comma-separated tool whitelist |
| `model` | string | MAY | Model alias: `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | string | MAY | Permission mode: `default`, `acceptEdits`, `bypassPermissions`, `plan` |
| `skills` | string | MAY | Pre-loaded skill names |

### Model Configuration

| Value | Behavior |
|-------|----------|
| `sonnet` | Use Claude Sonnet |
| `opus` | Use Claude Opus |
| `haiku` | Use Claude Haiku (fast, lightweight) |
| `inherit` | Use same model as parent conversation |

> **Note:** If omitted, the model defaults to the configured subagent model in settings, which can be customized.

### Permission Mode

| Value | Behavior |
|-------|----------|
| `default` | Normal permission handling |
| `acceptEdits` | Auto-approve file edits |
| `bypassPermissions` | Skip all permission prompts |
| `plan` | Read-only research mode |

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Include clear `description` for automatic invocation |
| SHOULD | Restrict `tools` to minimum required |
| SHOULD | Write detailed system prompt with examples |
| MAY | Override `model` for performance-critical agents |

---

## System Prompt Design

The markdown body after frontmatter is the agent's system prompt:

```markdown
---
name: test-runner
description: Run tests and fix failures
tools: Bash, Read, Write
---

You are a test automation expert. Your responsibilities:

1. Run appropriate tests when code changes are detected
2. Analyze test failures and identify root causes
3. Implement fixes that preserve original test intent
4. Verify fixes by re-running tests

When fixing tests:
- Maintain test expectations and semantics
- Add comments explaining fixes
- Report detailed results back to orchestrator

Do not:
- Change test requirements or intent
- Skip or comment out failing tests
- Make unrelated changes
```

### Requirements

| Requirement | Details |
|-------------|---------|
| MUST | Define clear responsibilities |
| MUST | Specify output format expectations |
| SHOULD | Include examples of expected behavior |
| SHOULD | Define explicit constraints ("Do not...") |
| MAY | Include edge case handling |

---

## Built-in Agent Types

Claude Code includes specialized built-in agents:

### Explore Agent

| Aspect | Details |
|--------|---------|
| Purpose | Fast codebase exploration |
| Mode | Read-only (strict) |
| Tools | `ls`, `git status/log/diff`, `find`, `cat`, `head`, `tail` |
| Thoroughness | Quick, Medium, Thorough |

**Invocation:**
```
Use the Explore agent to find all API endpoints in the codebase
```

### Plan Agent

| Aspect | Details |
|--------|---------|
| Purpose | Research and planning |
| Mode | Read + analysis |
| Context | Separate from main conversation |
| Restriction | Cannot spawn other subagents |

### General-Purpose Agent

| Aspect | Details |
|--------|---------|
| Purpose | Complex multi-step tasks |
| Mode | Full capability |
| Tools | All available tools |
| Use cases | Multi-step operations, code modifications |

---

## Agent Management

### Interactive Management

Use the `/agents` command (recommended) for interactive agent management:

| Command | Action |
|---------|--------|
| `/agents` | List all available agents |
| `/agents create` | Create new agent interactively |
| `/agents edit <name>` | Edit existing agent |
| `/agents delete <name>` | Remove an agent |

Changes take effect on next session start.

### CLI Configuration

Agents can also be defined via command line:

```bash
# Load agents from a specific directory
claude --agents /path/to/agents/

# Combine with other options
claude --agents ./custom-agents/ --verbose
```

### Direct File Management

Alternatively, manage agent files directly:
- Create/edit `.md` files in `.claude/agents/` or `~/.claude/agents/`
- Use any text editor
- Validate frontmatter YAML syntax
- Restart Claude Code to load changes

---

## Agent Invocation

### Automatic Invocation

Claude automatically delegates based on:
- Task description matching agent `description`
- Tool requirements matching agent capabilities
- Context isolation benefits

### Explicit Invocation

Users can request specific agents:
```
Use the code-reviewer agent to check my changes
Have the sql-expert review this query
```

### Task Tool Parameters

The Task tool invokes agents with these parameters:

| Parameter | Required | Purpose |
|-----------|----------|---------|
| `prompt` | MUST | Detailed task description |
| `subagent_type` | MUST | Target agent identifier |
| `model` | MAY | Override model selection |
| `resume` | MAY | Continue previous execution |

### Stateless Execution

| Requirement | Details |
|-------------|---------|
| MUST | Provide complete task description in prompt |
| MUST | Specify expected output format |
| MUST NOT | Expect agent to receive follow-up messages |
| MAY | Use `resume` to continue previous work |

---

## Tool Access Control

### Default Behavior

Agents inherit all available tools unless `tools` field is specified.

### Restricting Tools

```yaml
---
name: read-only-analyzer
description: Analyze code without making changes
tools: Read, Grep, Glob
---
```

### Requirements

| Requirement | Details |
|-------------|---------|
| SHOULD | Restrict to minimum required tools |
| SHOULD | Use read-only tools for analysis agents |
| MUST NOT | Grant write access to review-only agents |

---

## Plugin Agents

### Registration

Plugin agents are declared in `plugin.json`:

```json
{
  "name": "my-plugin",
  "agents": [
    "./agents/specialist.md",
    "./agents/researcher.md"
  ]
}
```

### Namespace Convention

Plugin components are namespaced:

| Pattern | Example |
|---------|---------|
| `plugin-name:agent-name` | `sdd-toolkit:sdd-plan` |

### Directory Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── agents/
    ├── specialist.md
    └── researcher.md
```

---

## Design Patterns

### Orchestration Pattern

Main agent delegates to focused subagents:

```
Orchestrator → code-reviewer (security)
            → code-reviewer (performance)
            → test-runner
            → documentation-checker
```

### Concurrent Execution Pattern

Launch independent agents simultaneously:

```
# Multiple agents in parallel
Task: code-reviewer → security analysis
Task: test-runner → test coverage
Task: style-checker → formatting
```

### Expert Pattern

Create domain-specific agents:

```
sql-expert.md      → Database queries
security-expert.md → Vulnerability analysis
perf-expert.md     → Performance optimization
```

---

## Performance Considerations

### Context Isolation

| Benefit | Details |
|---------|---------|
| Prevents pollution | Agent context separate from main conversation |
| Focused results | Only relevant information returned |
| Automatic compaction | Built-in context management |

### Best Practices

| Requirement | Details |
|-------------|---------|
| SHOULD | Return summaries, not raw output |
| SHOULD | Parallelize independent tasks |
| MUST NOT | Dump verbose logs to orchestrator |

### Token Discipline

- Keep system prompts under control—factor reusable doctrine into skills or CLAUDE.md, and link to specs instead of pasting them inline.
- Scope tool usage carefully (e.g., `rg` over whole repo vs. target directories) so transcripts stay readable when relayed back to the main agent.
- When returning results, lead with a compact summary plus optional “expand” sections so the orchestrator can choose whether to pull more detail.

---

## Examples

### Starter Template

```yaml
---
name: <agent-name>
description: <When to delegate to this agent>
tools: Read, Grep, Glob
model: sonnet
skills: security-review, performance-analysis
---

# Role
You are <persona>. Own this task end-to-end inside your isolated context.

## Responsibilities
1. <Gather context>
2. <Perform analysis>
3. <Summarize results>

## Workflow
- **Prep:** Run `<command>` if recent changes are unclear.
- **Analysis:** Use `skills/security-review` for security checks. Load `docs/architecture.md` only if the feature touches integrations.
- **Decision Matrix:**
  | Severity | Criteria |
  |----------|----------|
  | Critical | ... |
  | High | ... |

## Output Format
1. **Summary** – 2 sentences.
2. **Findings** – table with Severity / File / Detail.
3. **Next Steps** – ordered list of actions for the orchestrator.

## Constraints
- Do **not** modify files outside `src/`.
- Keep logs concise; link to files instead of pasting long diffs.
- If you need additional help, stop and explain what’s blocking you.
```

This scaffold shows how to combine responsibilities, progressive disclosure cues, and explicit output requirements in one agent prompt.

### Code Review Agent

```yaml
---
name: code-reviewer
description: Reviews code changes for security, performance, and best practices
tools: Read, Grep, Glob
model: sonnet
---

You are an expert code reviewer. For each review:

1. **Security**: Check for vulnerabilities (injection, XSS, auth issues)
2. **Performance**: Identify bottlenecks and inefficiencies
3. **Best Practices**: Verify coding standards compliance

Output format:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (nice to have)

Include file paths and line numbers for each finding.
```

### Test Runner Agent

```yaml
---
name: test-runner
description: Run tests, analyze failures, and implement fixes
tools: Bash, Read, Write
model: sonnet
---

You are a test automation expert.

Workflow:
1. Run the test suite: `npm test` or `pytest`
2. Parse failures and identify root causes
3. Implement minimal fixes that preserve test intent
4. Re-run to verify fixes

Constraints:
- Never change test expectations
- Never skip failing tests
- Report all changes made
```

### Documentation Agent

```yaml
---
name: doc-checker
description: Verify documentation accuracy and completeness
tools: Read, Grep, Glob
model: haiku
---

You verify that documentation matches implementation.

Check:
- API docs match actual signatures
- README reflects current features
- Code comments are accurate

Report:
- Outdated documentation
- Missing documentation
- Inconsistencies
```

---

## Anti-Patterns

### Don't: Vague Descriptions

```yaml
# Bad: unclear when to invoke
---
name: helper
description: Helps with things
---
```

```yaml
# Good: specific invocation criteria
---
name: sql-optimizer
description: Analyzes SQL queries for performance issues and suggests index improvements
---
```

### Don't: Overly Broad Tool Access

```yaml
# Bad: unnecessary write access
---
name: analyzer
description: Analyze code
tools: Read, Write, Bash, Grep
---
```

```yaml
# Good: read-only for analysis
---
name: analyzer
description: Analyze code
tools: Read, Grep, Glob
---
```

### Don't: Undersized Prompts

```yaml
# Bad: no guidance
---
name: reviewer
description: Review code
---
Review the code.
```

```yaml
# Good: detailed instructions
---
name: reviewer
description: Review code
---
You are an expert code reviewer. Evaluate:
1. Security vulnerabilities
2. Performance issues
3. Code quality

Output structured findings with severity levels.
```

### Don't: Agent Nesting

```markdown
# Bad: subagents cannot spawn subagents
When complex, delegate to another agent...
```

```markdown
# Good: flat design
Handle all aspects of this task directly.
Return results to orchestrator.
```

---

## Agents vs Commands vs Skills

| Use Case | Best Choice |
|----------|-------------|
| Complex workflows with isolated context | **Agent** |
| User-triggered shortcuts | **Command** |
| Reusable expertise across agents | **Skill** |
| Independent task execution | **Agent** |
| Quick templates with arguments | **Command** |
| Portable knowledge | **Skill** |

---

## Related Documents

- [02-commands.md](./02-commands.md) - User-invoked slash commands
- [04-skills.md](./04-skills.md) - Reusable expertise
- [07-plugin-manifest.md](./07-plugin-manifest.md) - Plugin configuration
- [08-permissions.md](./08-permissions.md) - Permission model
- [16-description-writing.md](./16-description-writing.md) - Writing effective agent descriptions
- [17-system-prompts.md](./17-system-prompts.md) - System prompt design patterns

---

**Navigation:** [← Previous: Commands](./02-commands.md) | [Index](./README.md) | [Next: Skills →](./04-skills.md)
