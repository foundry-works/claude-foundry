# Agent Delegation

## When to Use sdd-planner vs Implement Directly

| Scenario | Recommendation |
|----------|---------------|
| Task has clear, detailed instructions | Implement directly |
| Task requires architectural decisions | Consider sdd-plan first |
| Task is exploratory/investigation | Use Explore subagent |
| Task affects multiple systems | Consider sdd-plan first |
| Task has ambiguous requirements | Ask user via AskUserQuestion |

## Delegation Flow

```
Task Ready
    |
    +-- Clear instructions? --> Implement directly
    |
    +-- Needs architecture? --> Skill(foundry:sdd-plan) for sub-spec
    |
    +-- Needs exploration? --> Use Explore subagent
    |
    +-- Ambiguous? --> AskUserQuestion
```

## NEVER Delegate Back to sdd-implement

The anti-recursion rule is critical. If you find yourself about to call `Skill(sdd-implement)` from within this skill:

1. **STOP** - This indicates a workflow error
2. **Review** - Check if you're trying to surface the next task
3. **Use MCP directly** - Call `task action="prepare"` instead
4. **Continue workflow** - Proceed to Surface Next Recommendation
