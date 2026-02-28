# Built-in Subagent Patterns

Claude Code provides built-in subagents for efficient exploration and implementation without bloating main context.

## Available Subagents

| Subagent | Best For |
|----------|----------|
| **Explore** | File discovery, pattern search, codebase questions |
| **general-purpose** | Complex multi-step research, code analysis, task implementation |
| **Plan** | Architecture design, implementation planning |

## Model Selection Guidance

Choose the smallest model appropriate for the task:

| Task Complexity | Recommended Model | Examples |
|-----------------|-------------------|----------|
| Simple / mechanical | haiku | Single-file edits, config changes, boilerplate |
| Moderate / multi-file | sonnet | Feature implementation, refactoring, test writing |
| Complex / architectural | opus | Cross-cutting changes, intricate logic, novel patterns |

## Explore Agent Thoroughness Levels

| Level | Use Case | Example |
|-------|----------|---------|
| `quick` | Known location, simple search | Find a specific file |
| `medium` | Moderate exploration | Find related implementations |
| `very thorough` | Comprehensive analysis | Understand entire subsystem |

## Pre-Implementation Exploration

Before implementing a task, gather context efficiently:

```
Use the Explore agent (medium thoroughness) to find:
- Existing implementations of similar patterns
- Test files for the target module
- Related documentation that may need updates
- Import/export patterns in the target directory
```

## Pattern: Find Related Code

```
Use the Explore agent (quick thoroughness) to find:
- All files importing {module-name}
- Test coverage for {function-name}
- Configuration files affecting {feature}
```

## Pattern: Understand Architecture

```
Use the Explore agent (very thorough) to understand:
- How {subsystem} is structured
- Data flow through {component}
- Integration points with {external-system}
```

## When to Delegate Implementation

Use your judgment on whether to delegate tasks to subagents. Consider:

- **Context pressure**: When context usage is high (60%+), delegation preserves main context for orchestration
- **Task isolation**: Tasks touching a single file with clear boundaries are ideal delegation candidates
- **Fresh context**: Complex tasks sometimes benefit from a subagent's clean context window
- **Long sessions**: After many tasks inline, delegation prevents context degradation

## When NOT to Use Subagents

- Single file already in context
- Task specifies exact file paths and changes are trivial
- Near context limit (80%+) — finish current task, then pause
- Simple, isolated changes that don't benefit from fresh context
